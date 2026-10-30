---
title: "Kyverno 정책은 Ready였는데, 왜 막지 못했나"
date: 2026-10-30 10:10:00 +0900
categories: [블로그/기술문서]
tags: [kubernetes, kyverno, admission-webhook, eks, supply-chain]
---

화이트햇 스쿨 4기 팀 프로젝트에서 AWS EKS 클러스터에 공급망 보안 체계를 구축하고, 실제 사례 기반 공격 시나리오 11개로 검증했다. 그중 가장 많이 배운 것은 "미서명 이미지 배포" 시나리오였다. 정책은 분명히 켜져 있었는데 막지 못했다.

## 배경

프로젝트는 방어를 배포 전(CI), 배포 시점(클러스터), 배포 후(런타임)의 3계층으로 나눴다. 배포 시점 계층에서는 Kyverno로 세 가지 정책을 적용했다.

| 정책 | 역할 |
| --- | --- |
| `verify-image-signature` | Cosign keyless 서명의 발급 주체(GitHub Actions 워크플로 신원)와 OIDC 발급자 확인 |
| `disallow-latest-tag` | `latest` 태그 이미지 금지 |
| `disallow-root-user` | root로 실행되는 컨테이너 금지 |

CI에서 서명하지 않은 이미지는 클러스터에 들어올 수 없어야 한다. 그걸 확인하는 것이 이 시나리오의 목적이었다.

## 증상

정책 상태를 확인하면 Ready였다.

```bash
kubectl get clusterpolicy
```

그런데 서명되지 않은 이미지로 파드를 만들자 거부 없이 그대로 생성됐다. 오류도 경고도 없었다.

## 원인 추적

상태 표시 → 웹훅 등록 → 컨트롤러 로그 순서로 내려가며 확인했다.

Kyverno가 요청을 막으려면 먼저 API 서버가 요청을 Kyverno에게 보내야 한다. 쿠버네티스에서 이 연결을 담당하는 것이 admission webhook이다.

```text
kubectl apply
  → API 서버: 인증 → 인가
  → Mutating Admission (웹훅 호출)
  → 스키마 검증
  → Validating Admission (웹훅 호출)
  → etcd에 저장
```

Kyverno는 정책에 맞춰 `MutatingWebhookConfiguration`과 `ValidatingWebhookConfiguration`을 등록해 이 흐름에 끼어든다. 확인해 보니 이미지 서명 검증을 처리해야 할 웹훅이 등록되어 있지 않았다.

```bash
kubectl get validatingwebhookconfigurations,mutatingwebhookconfigurations | grep kyverno
```

원인은 정책에 설정한 웹훅 타임아웃 값(`timeoutSeconds`) 하나였다. 60초로 설정되어 있었는데, 쿠버네티스는 admission webhook의 타임아웃을 **1~30초**로 제한한다. 범위를 벗어난 값이라 웹훅이 등록되지 못했고, 그러는 동안 정책 객체 자체는 정상적으로 적재되어 Ready로 표시되고 있었다.

웹훅이 없으면 API 서버는 Kyverno에게 물어볼 일 자체가 없다. 정책이 아무리 정확하게 작성되어 있어도 요청은 정책을 거치지 않고 그대로 저장된다.

> Ready는 "정책이 적재되었다"는 뜻이지, "API 서버가 요청을 정책으로 보내고 있다"는 보장이 아니다.
{: .prompt-warning }

타임아웃을 상한 이내로 고치자 웹훅이 등록되었고, 미서명 이미지는 거부됐다.

## 상태를 믿지 않는 점검 스크립트

고치고 나서 든 생각은 "다른 정책도 같은 상태일 수 있지 않나?"였다. 상태 표시를 다시 확인하는 것으로는 답할 수 없는 질문이라, 정책이 **실제로 요청을 거부하는지** 시험하는 점검 스크립트(`check-policy-health.sh`)를 만들었다.

점검 순서는 다음과 같다.

1. 정책 상태 확인
2. 서명 검증 웹훅 등록 여부 확인
3. Kyverno 컨트롤러 오류 로그 확인
4. 서버 dry-run으로 실제 거부 여부 시험

핵심은 4번이다. `kubectl apply --dry-run=server`는 실제로 저장하지 않으면서 admission 단계까지 거치기 때문에, 웹훅이 정말로 요청을 거부하는지 안전하게 시험할 수 있다.

```bash
kubectl apply --dry-run=server -f unsigned-image-pod.yaml
```

시험 입력은 세 가지로 구성했다.

| 입력 | 기대 결과 |
| --- | --- |
| 서명되지 않은 이미지 | 거부 |
| 외부 레지스트리 이미지 | 거부 |
| 서명된 정상 이미지 | 통과 |

거부되어야 하는 입력만 시험하면 "무엇이든 거부하는" 고장은 잡을 수 없다. 그래서 통과해야 하는 대조군을 함께 넣었다.

## 이틀 사이 잡아낸 두 건

이 스크립트로 이틀 사이 실제 이상 두 건을 더 발견했다.

- **서명 주체 고정**: 서명 검증 정책이 작업 브랜치(`feat/ci-pipeline`)의 서명만 허용하고 있어, main에서 빌드한 정상 이미지가 거부됐다. 와일드카드 대신 main의 서명 주체를 정확히 추가해 해결했다.
- **정책 모드 회귀**: 화면상으로는 정책이 모두 Ready였지만, 외부 레지스트리 차단 정책이 Enforce에서 Audit으로 되돌아가 있었다. Audit 모드는 위반을 기록만 하고 막지 않는다. 원본 정책 파일을 Enforce로 맞추고, 이후로는 적용 전에 `kubectl diff`로 변경 내용을 확인하도록 했다.

## 배운 점

"켜져 있다"와 "작동한다"는 다르다. 상태 표시는 도구가 스스로 보고하는 값이고, 그 값이 실제 동작을 보장하지는 않는다. 프로젝트에서 같은 종류의 문제가 반복해서 나왔다.

- Falco 룰 한 줄의 오류로 Falco가 14번 재시작(CrashLoopBackOff)했지만, 사전 검사는 통과했다.
- 휴면 후 사설 IP로 연결하는 시나리오에서, Falco의 기본 `outbound` 매크로가 사설 IP 연결을 제외하고 있어 오류도 경고도 없이 이벤트가 0건이었다.

그래서 보안 장치를 검증할 때는 실패해야 하는 입력과 통과해야 하는 입력을 모두 넣어 **실제 결과**를 확인하는 습관이 중요하다고 정리했다.

## 남은 질문

웹훅 등록에 실패했는데 왜 정책 상태는 Ready로 남아 있었을까? 정책의 상태를 관리하는 부분과 웹훅을 등록하는 부분이 어떻게 나뉘어 있는지 Kyverno의 구조를 더 살펴볼 계획이다.
