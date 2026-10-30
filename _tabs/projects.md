---
# the default layout is 'page'
icon: fas fa-folder-open
order: 5
---

진행한 프로젝트를 최신순으로 정리했습니다. 프로젝트 이름을 누르면 자세한 회고로 이동합니다.

## [DrimFlow](/posts/drimflow-execution-engine/) <small>진행 중</small>

**드래그 앤 드롭으로 업무 로직을 만드는 워크플로 엔진**

| 항목 | 내용 |
| --- | --- |
| 형태 | 교내 기업 연계 캡스톤 · 2026.10 ~ 진행 중 (15주) |
| 팀 | 풀스택 3명 |
| 역할 | 백엔드 3개 파트 중 실행 파트 — 실행 명령 API(실행·신호·취소)와 실행 엔진 |
| 기술 | Java, Spring Boot 스타터, PostgreSQL·H2, Claude Code |

- 실행 시작 시점의 버전을 스냅샷으로 고정해, 실행 도중 정의가 바뀌어도 결과가 흔들리지 않게 설계
- 세마포어로 동시 실행 수를 제한하고, 유니크 키로 중복 신호를 차단
- 세 명이 동시에 착수할 수 있도록 소유권, 스텁, 계약 테스트 중심의 협업 규칙 수립

## [화이트햇 스쿨 4기 팀 프로젝트](/posts/whitehat-4th-project-retro/)

**클라우드 네이티브 공급망 · EKS 런타임 보안**

| 항목 | 내용 |
| --- | --- |
| 형태 | 화이트햇 스쿨 4기 팀 프로젝트 · 2026.08.03 ~ 2026.10.17 |
| 팀 | 8명 (+ 멘토 1, PL 1) |
| 역할 | PM · CI 파이프라인 권한, IAM 신뢰 조건, 시크릿 탐지 |
| 기술 | AWS EKS·ECR·IAM, GitHub Actions(OIDC), Kyverno, Cosign·Rekor, Falco, Gitleaks, Trivy |
| 링크 | [GitHub](https://github.com/WHS4-learning-crew/whitehat-supply-chain-security) |

- 공급망 공격 사례 8개를 분석해 CI·클러스터·런타임 3계층 방어를 구축
- 사례 기반 공격 시나리오 11개 중 8개 차단·탐지, 3개는 "설치됐지만 작동하지 않은" 원인 규명
- 정책 점검 스크립트로 이틀 사이 실제 이상 2건 포착 ([관련 글](/posts/kyverno-ready-but-not-enforcing/))

## [미리살핌](/posts/mirisalpim-retro/)

**AI 사기범과 직접 대화하며 금융사기 대응을 훈련하는 서비스**

| 항목 | 내용 |
| --- | --- |
| 형태 | 2026 금융 AI Challenge(금융보안원 주최) 출품 팀 프로젝트 · 2026.08 ~ 2026.09 |
| 팀 | 4명 (기획·데이터, AI·프롬프트, 백엔드, 프론트엔드) |
| 역할 | 백엔드 — DB 설계·구축, 배포 DB 검증 · 기획 — 리서치, 경쟁 서비스 조사, 기능명세서, ERD 검토 |
| 기술 | Python, Django, PostgreSQL·pgvector, Docker Compose, Railway |
| 링크 | [GitHub](https://github.com/2026-finance-ai-challenge-team-1234/mirisalpim-web) · [서비스](https://mirisalpim-web.vercel.app) |

- 시나리오, 대화, 판별 단서, RAG 발화 코퍼스를 담는 Django 모델 11개 설계·구현
- 배포 DB의 pgvector 지원을 착수 직후 실측해 팀 아키텍처 확정
- 배포 화면을 끝까지 조작해 안전 설계 원칙과 어긋난 지점을 발견

## [온담](/posts/ondam-backend-retro/)

**떨어져 사는 가족의 건강 변화를 함께 챙기는 서비스**

| 항목 | 내용 |
| --- | --- |
| 형태 | 해커톤 팀 프로젝트, 이후 공모전 출품 · 2026.07 ~ 2026.08 |
| 팀 | 백엔드 2 · 프론트엔드 2 · 기획 1 · 디자인 1 |
| 역할 | 백엔드 — 질문·기록·일지·리포트·편지·검진 도메인, 배포 인프라, AI 연동 |
| 기술 | Java 17, Spring Boot, Spring Data JPA, MySQL 8.4, WebClient, AWS EC2·RDS·S3, Nginx, OpenAI GPT·Whisper |
| 링크 | [GitHub](https://github.com/Dream-of-WildGeese/Wild_Geese_back) |

- API 20여 개 설계·구현, GPT·Whisper 연동
- 외부 AI 호출을 트랜잭션 밖으로 분리하고 15초 타임아웃을 적용해 장애 전파 차단
- 폴백에 가려진 API 키 누락 장애를 로깅으로 규명

## 기타 활동

| 활동 | 내용 | 기술 |
| --- | --- | --- |
| 바이브코딩 공모전 | 실시간 AI를 활용한 프레젠테이션 도움 서비스 | Python, FastAPI |
