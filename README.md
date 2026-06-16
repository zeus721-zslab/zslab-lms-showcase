# zslab LMS

학점은행제 · 자격증 과정을 하나의 플랫폼에서 운영할 수 있는 통합 LMS (Learning Management System)

> AI 협업 기반으로 설계·구현·검증한 개인 풀스택 프로젝트

![status](https://img.shields.io/badge/E2E-88_PASS-success) ![docker](https://img.shields.io/badge/Docker-Ready-blue) ![license](https://img.shields.io/badge/license-MIT-green)
- [🌐 라이브 데모 바로가기](https://zslab-lms.duckdns.org) 
- [🛠 관리자 패널 바로가기](https://zslab-lms.duckdns.org/lms-manage)

### 테스트 계정

로그인 페이지 하단의 **데모 로그인** 버튼으로 원클릭 진입 가능합니다.

---

## 개발 현황

MVP 완료 (2026-05-15) · 교수자·튜터 패널 추가 (2026-05-24) · 디자인 리뉴얼 (2026-06-12) · 운영자 UX 리뉴얼 (2026-06-16) · 시험 도메인 재설계 (2026-06-16)

---

## 프로젝트 개요

설계부터 배포·테스트까지 진행한 풀스택 LMS 프로젝트입니다. CRUD 중심 포트폴리오 대신 **실제 운영 가능한 서비스 구조**를 목표로 만들었습니다.

- **규모**: API 약 153개 엔드포인트 · 도메인 모델 30+ · 자동 검증 250+ 케이스 · E2E 28 시나리오
- **범위**: 학습자 · 튜터 · 교수자 · 관리자 4단계 역할 전 영역 구현
- **운영**: Docker 기반 실서버 라이브 운영 · GitHub Actions CI/CD

### 핵심 목표
- 학점은행제 + 자격증 교육 통합 운영
- 역할 기반 LMS 설계
- AI 기능을 포함한 현대적 학습 경험 구현
- Docker 기반 운영 환경 구성
- 자동화 테스트 및 안정성 확보

---

## 주요 기능

### 1. 4단계 역할 기반 시스템 (Multi-Role LMS)

- 학습자 (Student)
- 튜터 (Tutor)
- 교수자 (Professor)
- 관리자 (Admin)

권한 분리 · 접근 제어 · 관리 화면 분리

### 2. AI 학습 도우미

Provider 추상화 구조로 4종 지원:

- Claude · OpenAI · Gemini · Mock

특징:
- SSE 실시간 스트리밍
- 차시 컨텍스트 자동 주입
- `.env` 한 줄로 Provider 전환

### 3. 강좌 추천 시스템

3종 알고리즘 구현:

- Content-Based (콘텐츠 기반)
- Collaborative Filtering (협업 필터링)
- Popularity (인기 기반)

Redis 캐싱 · 비로그인 폴백 지원

### 4. 학습 진도 추적

학습 이탈 대응:
- 30초 주기 Heartbeat
- 페이지 종료 시 sendBeacon 보장
- 95% 시청 시 자동 수료

### 5. 관리자 패널

| 기능 | 설명 |
|------|------|
| 학습자 360 뷰 | 1명 종합 조회 — 수강·시험·과제·결제·자격증·문의 6 탭 |
| 강좌 운영 허브 | 1개 강좌 종합 — 정보·차시·문항·수강생·시험·과제·스태프·통계 7 탭 |
| 글로벌 검색 | Ctrl+K — 학습자·강좌·시험·주문 통합 검색 |
| 주문 상세 | 결제·환불·학습자·강좌 통합 + 환불 다이얼로그 |

### 6. 평가 관리

| 기능 | 설명 |
|------|------|
| 문제은행 | 문항 마스터 관리·난이도·CSV 일괄·일괄 액션·정답률 |
| 시험지 구성 | 고정/랜덤 출제 모드·DnD·문제은행 차용·미리보기 |
| 4배수 풀 검증 | 정부 평가인정 권장 기준 시각화 (🔴🟡🟢) |
| 시험지 복제 | fixed/random 분기·메타 복사·다른 강좌 복제 |
| 응시 스냅샷 | 매 응시마다 본문 스냅샷·legacy fallback |

### 7. 자동 검증

| 영역 | 도구 | 카운트 |
|------|------|--------|
| Backend | PHPUnit | 32 파일 |
| Frontend | Vitest | 3 파일 · 20 케이스 |
| E2E | Playwright | 28 spec |

테스트 데이터는 beforeAll / afterAll 기반으로 생성·정리되며 데모 시더에 의존하지 않습니다.

---

## 아키텍처

```text
인터넷
   │
Gateway (Nginx)  ← SSL 종단·외부 진입점
   │
Caddy Router    ← 내부 라우팅
   │
├── Frontend (Next.js 15)
├── API (Laravel 12 + FrankenPHP)
├── Realtime (Socket.io · 진도 heartbeat)
└── Chat Service (Socket.io · 1:1 문의)

백그라운드
├── Queue Worker (Laravel)
└── Scheduler (Laravel)

공유 인프라
├── MariaDB
├── Redis
└── Elasticsearch (nori 형태소)
```

---

## 기술 스택

### Frontend
- Next.js 15 (App Router)
- TypeScript
- Tailwind CSS
- shadcn/ui
- Framer Motion

### Backend
- Laravel 12
- PHP 8.3
- FrankenPHP (Worker Mode)

### 인프라
- Docker · Docker Compose
- Nginx (Gateway)
- Caddy (내부 라우터)
- Redis
- MariaDB
- Elasticsearch

### Realtime · AI
- Socket.io
- SSE (Server-Sent Events)
- Claude · OpenAI · Gemini

### 테스트
- PHPUnit
- Vitest
- Playwright

---

## 주요 기술적 의사결정

### AI 응답에 WebSocket이 아닌 SSE를 선택한 이유

AI 응답은 서버 → 클라이언트 단방향 스트리밍이므로 양방향 프로토콜인 WebSocket보다 SSE가 적합. 채팅·실시간 진도 등 양방향 통신은 Socket.io로 분리.

### Redis 역할 분리

단일 Redis 인스턴스를 다음 4종 용도로 분리하여 사용:

- Session Store
- Recommendation Cache (TTL 600초)
- Queue Backend
- AI Session History

### 추천 시스템에 단일 알고리즘이 아닌 3종 병렬을 선택한 이유

추천 데이터 부족 문제 대응:

- 비로그인 사용자 → Popularity
- 신규 사용자 (수강 이력 적음) → Content-Based
- 기존 사용자 → Collaborative Filtering

각 화면에서 알고리즘별 섹션을 병렬 노출하여 콜드 스타트 문제 회피.

### Provider 추상화 + 환경변수 전환

AI Provider를 인터페이스로 추상화하여 실제 API 키 없이도 Mock으로 전체 흐름 시연 가능. 운영 전환 시 `.env` 한 줄(`AI_PROVIDER=claude`)로 실연결 가능한 구조.

---

## 로컬 실행

```bash
# 1. 외부 네트워크 생성 (최초 1회)
docker network create infra_net

# 2. zslab-infra 인프라 사전 기동 (MariaDB · Redis · Elasticsearch)
#    https://github.com/zeus721-zslab/zslab-infra 참조

# 3. 저장소 클론
git clone <repo-url> zslab-lms
cd zslab-lms

# 4. 환경변수 설정
cp .env.example .env

# 5. 스택 기동
bash scripts/up.sh
```

---

## 로드맵

### ✅ 완료
- [x] **운영자 UX 리뉴얼** — admin-ops-renewal 9 Stage (학습자 360·강좌 허브·검색·주문·복제·일괄)
- [x] **시험 도메인 재설계** — exam-redesign 7 Stage (문제은행·랜덤·스냅샷·4배수·복제)

### 📌 추후 개선 예정
- [ ] **학습자 응시 UI 보강** — max_attempts·랜덤 안내·재응시·결과 화면 (시험 신구조 학습자 측 반영)
- [ ] **강좌 추천 시스템 고도화** — 협업·콘텐츠 기반 추천 (포트폴리오 트랙)
- [ ] 실제 PG 결제 연동 (StubPaymentGateway → 실 클래스 교체)
- [ ] 영상 직접 업로드 + HLS 자동 트랜스코딩
- [ ] 이메일·SMS·푸시 알림 체계

---

## 개발 방식

이 프로젝트는 **AI 협업 개발 (AI Assisted Development)** 방식으로 진행되었습니다.

AI를 생산성 도구로 활용했으며, 다음 의사결정은 직접 수행했습니다:

- 요구사항 정의
- 아키텍처 설계
- 구현 검증
- 테스트 전략
- 운영 구조 결정 및 최종 품질 책임

---

## 라이선스

MIT License — Copyright (c) 2026 zslab
