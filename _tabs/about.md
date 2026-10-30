---
# the default layout is 'page'
icon: fas fa-info-circle
order: 4
---

## 안녕하세요, 권민찬입니다

**만든 기능이 실제로 동작하는지까지 확인하며, 하나의 문제를 다양한 관점으로 바라보고 해결하려 노력하는 개발자**입니다.

중앙대학교 소프트웨어학부에서 공부하고 있습니다. Spring 기반 API 개발과 AWS 배포, 운영 중 장애 대응을 경험했고, 서비스를 만들고 운영할수록 "내가 만든 이 구조는 안전한가"라는 질문이 커져 클라우드 보안을 공부하고 있습니다.

화이트햇 스쿨 4기에서 EKS 환경의 공급망 보안 체계를 구축하며, 보안 도구를 "설치했다"와 "작동한다"가 다르다는 것을 직접 시험해 확인했습니다. 지금은 HSPACE 나이츠 프론티어 3기에서 운영체제와 네트워크 같은 기초부터 다시 다지고 있습니다.

AI 도구를 적극적으로 쓰되, 결과를 제 말로 설명할 수 있을 때만 코드에 반영합니다.

## 이 블로그는

공부하고 만든 것을 기록하는 곳입니다. 결과뿐 아니라 어디서 막혔고 어떤 질문으로 풀었는지를 함께 남기려고 합니다.

| 카테고리 | 내용 |
| --- | --- |
| 개발 | 백엔드 개발과 프로젝트 기록 |
| CTF/Wargame | 워게임과 CTF 풀이 |
| Bug Bounty | 버그바운티 공부와 기록 |
| 블로그/기술문서 | 기술 분석과 트러블슈팅 |
| 논문/컨퍼런스 | 논문, 보고서, 발표 리뷰 |
| 공모전/자격증 | 공모전, 교육 프로그램, 자격증 |

## 핵심 역량

| 역량 | 근거 |
| --- | --- |
| 외부 연동 장애 격리 | [온담](/posts/ondam-backend-retro/) — GPT·Whisper 호출에 15초 타임아웃 적용, 저장 트랜잭션과 AI 호출 분리 |
| 숨은 장애를 드러내는 검증 | [온담](/posts/ondam-backend-retro/) — 폴백이 가린 API 키 누락 장애 규명<br>[화이트햇 스쿨](/posts/kyverno-ready-but-not-enforcing/) — 정책 점검 스크립트로 이틀간 실제 이상 2건 포착<br>[미리살핌](/posts/mirisalpim-retro/) — 배포 DB의 pgvector 지원을 착수 직후 실측해 아키텍처 확정 |
| 트랜잭션·동시성 설계 | [DrimFlow](/posts/drimflow-execution-engine/) — 실행 버전 스냅샷, 세마포어 기반 동시 실행 제한, 유니크 키 기반 중복 신호 차단 |

## 기술 스택

| 분야 | 주력 | 사용 경험 |
| --- | --- | --- |
| Language | Java 17 | Python |
| Backend | Spring Boot, Spring Data JPA | FastAPI, Django |
| Database | MySQL | PostgreSQL, H2, Vector DB |
| Infra | AWS (EC2, RDS, S3), Nginx | AWS (EKS, ECR, IAM), Docker, GitHub Actions, Railway |
| Security | Gitleaks | Kyverno, Cosign, Falco, Trivy |
| AI | OpenAI API (GPT, Whisper), Claude Code | Cursor |

주력은 구현 과정과 선택 이유까지 설명할 수 있는 기술, 사용 경험은 프로젝트에서 직접 다뤄 본 기술입니다.

## 프로젝트

| 프로젝트 | 기간 | 역할 | 핵심 |
| --- | --- | --- | --- |
| [온담](/posts/ondam-backend-retro/) | 2026.07 ~ 2026.08 | 백엔드 | 기획부터 배포까지, GPT·STT 연동과 장애 대응 |
| [화이트햇 스쿨](/posts/whitehat-4th-project-retro/) | 2026.08 ~ 2026.10 | PM · CI · 권한 | EKS 3계층 공급망 보안 구축과 실증 |
| [미리살핌](/posts/mirisalpim-retro/) | 2026.08 ~ 2026.09 | 백엔드(DB) · 기획 | AI 사기범과 대화하는 금융사기 모의훈련 |
| [DrimFlow](/posts/drimflow-execution-engine/) | 2026.10 ~ 진행 중 | 백엔드(실행 엔진) | Spring 기반 워크플로 실행 엔진 |

자세한 내용은 [Projects](/projects/) 메뉴에 정리했습니다.

## 활동과 교육

- **HSPACE 나이츠 프론티어 3기** (2026.11 ~ 2027.05)
- **화이트햇 스쿨 4기** (2026.08 ~ 2026.10) — 팀 프로젝트 PM
- **2026 금융 AI Challenge** (금융보안원 주최) — 미리살핌 출품
- **바이브코딩 공모전** — 실시간 AI를 활용한 프레젠테이션 도움 서비스 (Python, FastAPI)
- **중앙대학교 소프트웨어학부** (2022.03 ~)

## 연락

- GitHub: [github.com/tronve](https://github.com/tronve)
