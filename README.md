# 학점은행 평가인정 정합 LMS

솔로 풀스택 개발 · NILE 평가인정 기준 통제 자동화

## 🌐 라이브 데모

| 영역 | URL |
|---|---|
| 학습자 | https://zslab-lms.duckdns.org/ |
| 운영자 | https://zslab-lms.duckdns.org/lms-manage/login |

데모 계정은 학습자 로그인 페이지 하단의 "데모 모드" 버튼으로 진입 가능합니다.

## 📝 본 저장소 안내

본 저장소는 LMS 프로젝트의 시연·문서 저장소입니다. 소스 코드는 비공개 저장소에서 관리됩니다.

## 🎯 프로젝트 개요

- 솔로 풀스택 개발 (1인)
- 기간: 2026-04-27 ~ 진행 중 (약 1.7개월차)
- 도메인: 학점은행제 LMS (NILE 평가인정 정합)

학점은행제 평가인정 기관 운영 시 발생하는 결격 사유(일정 미통제·출석 미관리·이의신청 절차 미비)를 시스템 레벨에서 자동으로 해소하는 LMS입니다. CRUD 중심 포트폴리오 대신 실제 정책 기준(NILE IV.4)을 코드로 구현하는 데 집중했습니다.

## 🛠 기술 스택

| 영역 | 스택 |
|---|---|
| Backend | Laravel 12 · PHP 8.3 · FrankenPHP (Worker Mode) |
| Frontend | Next.js 16 · TypeScript · Tailwind v4 · shadcn/ui |
| DB | MariaDB 10.11 · Redis 7 · Elasticsearch 8 (한국어 nori+ngram) |
| 실시간 | Socket.io |
| 인프라 | Docker Compose · GitHub Actions CI/CD · Nginx |
| 디자인 | Raycast 기반 dark-first · Framer Motion |

## ✨ 핵심 기능

### 1. 학점은행 평가인정 정합 (NILE 결격 사유 해소)

| NILE 기준 | 구현 트랙 | 상태 |
|---|---|---|
| 학기 일정 관리 (IV.4.가) | 학기 phase 4단계 통제 (enrollment / in_progress / appeal / closed) | ✅ |
| 출석 80% 이상 (IV.4.나) | 진도 → 출석 자동 산정 · 2주 인정창 · credit_bank 80% 게이트 | ✅ |
| 이의신청 14일 (IV.4.다) | 학습자 신청 → 운영자 검토 → 답변 비동기 티켓 워크플로우 | ✅ |
| 시험 유형 정합성 | DB enum · PHP backed enum 4층위 잠금 (오타·운영 사고 원천 차단) | ✅ |
| 결과 공개 시점 | 시험 type별 · 학기 phase별 차등 공개 정책 | ✅ |

학점 시험(credit_midterm·credit_final)과 자격증 시험(cert_mock·cert_main)은 정책 격리 — 5개 가드 지점에서 교차 적용 불가.

### 2. 시험 도메인 재설계

- 문제은행 마스터 테이블 · 시험지 빌더 (DnD)
- 스냅샷 기반 응시 — 출제 후 문제 변경에도 응시 이력 보존
- 이중 채점 경로 (스냅샷 우선 · 레거시 폴백)
- 결과 리뷰 화면 — type별 · 학기 phase별 차등 공개
- 4배수 풀 검증 시각화 (정부 평가인정 권장 기준)

### 3. 운영자 도구

- 학습자 360 뷰 (수강·시험·과제·결제·자격증·문의 6탭)
- 강좌 운영 허브 (7탭)
- 글로벌 검색 — Command Palette (Ctrl+K)
- 학점은행 이의신청 관리 (신청·검토·답변 워크플로우)
- 일괄 작업 · 비밀번호 재설정 · 학기/강좌 클론

### 4. 디자인 시스템

- Raycast 기반 dark-first 디자인
- Tailwind v4 @theme inline 토큰
- shadcn/ui 표준화 (14개 컴포넌트)
- Framer Motion (MotionConfig · shared variants)
- Lighthouse 기준점 확보 · 자동 회귀 250+ 케이스

## 🏗 아키텍처

모놀리식 백엔드(Laravel 12 + FrankenPHP)와 SSR 프론트엔드(Next.js standalone 빌드)를 분리하고, Nginx Gateway에서 SSL 종단 처리 후 Caddy 내부 라우터를 통해 서비스로 분배합니다. Elasticsearch는 공유 컨테이너(zslab-infra)에서 관리하며 sync-search.sh로 동기화합니다. GitHub Actions CI/CD → ghcr.io → 운영 서버 자동 배포 파이프라인이 구성되어 있습니다.

자세한 아키텍처 다이어그램과 기술 결정 근거는 `docs/` 하위 문서에서 확인할 수 있습니다.

## 📊 규모 지표

| 항목 | 수치 |
|---|---|
| API 엔드포인트 | 약 153개 |
| 도메인 모델 | 30+ |
| 자동 검증 케이스 | 250+ (PHPUnit 88 · Vitest 37 · E2E Playwright) |
| 기능 커버리지 | 상업용 LMS 기준 65~75% |

## 📚 문서

| 문서 | 내용 |
|---|---|
| [docs/01-architecture.md](docs/01-architecture.md) | 시스템 아키텍처 · 기술 결정 |
| [docs/02-tech-stack.md](docs/02-tech-stack.md) | 스택 선택 근거 |
| [docs/03-credit-bank.md](docs/03-credit-bank.md) | 학점은행 평가인정 트랙 |
| [docs/04-exam-redesign.md](docs/04-exam-redesign.md) | 시험 도메인 재설계 |
| [docs/05-admin-ops.md](docs/05-admin-ops.md) | 운영자 도구 |
| [docs/06-design-system.md](docs/06-design-system.md) | 디자인 시스템 |

## 📸 스크린샷

준비 중 — 학습자·운영자 주요 화면 추가 예정.

---

<!--
README 갱신 가이드:
- 기간 표기는 매월 갱신 권장 (2026-04-27 시작 기준 N개월차 자동 계산)
- 핵심 기능 신규 추가 시 ✨ 섹션에 트랙 단위로 갱신
- docs/screenshots 추가 시 "준비 중" 표기 제거
-->
