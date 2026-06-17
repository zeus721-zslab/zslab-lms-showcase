# 기술 스택

## 개요

스택 선택 원칙은 네 가지입니다.

- **성능** — PHP Worker Mode로 요청당 부트스트랩 비용 제거 · Next.js standalone 빌드로 이미지 최소화
- **한국어 검색** — nori 형태소 + ngram 조합으로 자모 분리 검색 지원
- **실시간** — Socket.io 전용 서버 분리로 진도·채팅 부하를 백엔드에서 격리
- **유지보수성** — shadcn/ui 컴포넌트 표준화 · TypeScript + Zod 타입 안전성 · Larastan + PHPStan 정적 분석

---

## 🖥 Backend

| 영역 | 기술 | 버전 | 선택 근거 |
|---|---|---|---|
| 프레임워크 | Laravel | 12 (^13.0) | Eloquent ORM·Sanctum·Queue·Scheduler 통합 생태계 |
| 런타임 | PHP | 8.3 (CI: 8.4) | 타입 안전성 강화·성능 개선 |
| 서버 | FrankenPHP | Worker Mode | 프로세스 상주로 요청당 부트스트랩 비용 제거 |
| 인증 | Laravel Sanctum | 4.3 | SPA Token 인증·API 토큰 관리 |
| 데이터베이스 | MariaDB | 10.11 | MySQL 호환·운영 안정성 |
| 캐시·큐·세션 | Redis | 7 | 4종 용도 분리 (세션·추천 캐시·큐·AI 이력) |
| 전문 검색 | Elasticsearch | 8 | 한국어 nori 형태소 + ngram |
| PDF | barryvdh/laravel-dompdf | 3.1 | 수료증 PDF 발급 |
| QR | simplesoftwareio/simple-qrcode | 4.2 | 자격증 QR 코드 검증 |
| 정적 분석 | Larastan + PHPStan | 3.9 / 2.1 | 컴파일 타임 타입 오류 탐지 |
| 테스트 | PHPUnit | 12.5 | 88 케이스 |

---

## 🌐 Frontend

| 영역 | 기술 | 버전 | 선택 근거 |
|---|---|---|---|
| 프레임워크 | Next.js | 16 (^16.2.6) | App Router SSR · standalone 빌드 |
| 런타임 | React | 19.2.4 | 최신 concurrent 기능 |
| 언어 | TypeScript | 5 | 타입 안전성 |
| 스타일 | Tailwind CSS | 4 | `@theme` inline 토큰으로 디자인 시스템 일원화 |
| 컴포넌트 | shadcn/ui (Radix UI) | — | 14개 접근성 보장 컴포넌트 표준화 |
| 애니메이션 | Framer Motion | 12.40 | `MotionConfig` 공유로 전체 모션 일원화 |
| 폼 | react-hook-form + Zod | 7.74 / 4.3 | 타입 안전 폼 검증 |
| 상태 관리 | Zustand | 5.0 | 경량 전역 상태 |
| 차트 | Recharts | 3.8 | React 컴포넌트 API·Chart.js 대비 SSR 통합 단순 |
| DnD | @dnd-kit | 6.3 (core) | 시험지 문항 드래그 앤 드롭 |
| 실시간 | socket.io-client | 4.8 | 진도 하트비트·채팅 실시간 연결 |
| 영상 재생 | hls.js | 1.6 | HLS 스트리밍 재생 |
| 콘텐츠 저작 | GrapesJS | 0.21 | 강좌 페이지 비주얼 빌더 |
| 토스트 | sonner | 2.0 | 알림 UI |
| 아이콘 | lucide-react | 1.11 | 일관된 아이콘 시스템 |
| 테마 | next-themes | 0.4 | dark-first 테마 전환 |
| 테스트 | Vitest | 4.1 | React 컴포넌트 단위 테스트 37 케이스 |

---

## 🏗 Infra · CI/CD

| 영역 | 기술 | 역할 |
|---|---|---|
| 컨테이너화 | Docker Compose | 7개 서비스 오케스트레이션 |
| 외부 진입 | Nginx | SSL 종단·외부 트래픽 수신 |
| 내부 라우터 | Caddy 2 | 경로 기반 라우팅·SSE flush 제어 |
| CI | GitHub Actions | Backend·Frontend·Realtime 3-job 병렬 / push·PR 트리거 |
| CD | GitHub Actions + SSH | main push → 무중단 롤링 재배포 → 마이그레이션 자동 실행 |
| 테스트 DB | MariaDB + SQLite | CI: Feature 테스트 MariaDB · 단위 테스트 SQLite in-memory 병행 |
| E2E | Playwright | 28 시나리오 |

---

## 🔍 주요 선택 근거

### 1. Next.js 16 + standalone 빌드

Next.js App Router를 채택하여 서버 컴포넌트에서 백엔드 API를 내부 네트워크로 직접 호출합니다. 클라이언트 왕복 없이 초기 HTML을 완성해 전달하므로 Time to First Byte가 줄어듭니다. standalone 빌드는 의존성 트리를 분석하여 필요한 node_modules만 포함해 Docker 이미지 크기를 최소화합니다.

### 2. Tailwind CSS v4 `@theme` inline 토큰

Tailwind CSS v4의 `@theme` 구문으로 색상·간격·폰트 등 디자인 토큰을 CSS 변수로 선언합니다. Raycast 기반 다크 우선 팔레트를 토큰으로 정의하면, 컴포넌트가 토큰을 직접 참조하고 테마 전환 시 CSS 변수 교체 한 번으로 전체에 적용됩니다.

### 3. shadcn/ui 14개 컴포넌트 표준화

shadcn/ui는 컴포넌트 코드를 프로젝트에 직접 복사하는 방식으로, Radix UI의 접근성 보장 프리미티브 위에 Tailwind 스타일을 적용합니다. 외부 컴포넌트 라이브러리 의존 없이 프로젝트 내에서 직접 수정할 수 있습니다. 14개(avatar·badge·button·card·dialog·dropdown-menu·input·label·select·separator·skeleton·switch·tabs·textarea)로 범위를 제한하여 일관성을 유지합니다.

### 4. Elasticsearch + nori 형태소 (한국어 자모 검색)

MariaDB FULLTEXT는 한국어 형태소 분석을 지원하지 않아 부분 음절 검색이 불가능합니다. Elasticsearch의 nori 형태소 분석기와 ngram 토크나이저를 조합하면 자모 분리 검색(예: "ㅎㄱ" → "학점")과 부분 어절 검색이 가능합니다. 운영자 글로벌 검색(Ctrl+K — 학습자·강좌·시험·주문 통합)과 강좌 목록 전문검색에 적용했습니다.

### 5. Recharts (vs Chart.js)

학습자 대시보드 월별 학습시간 차트에 Recharts를 선택했습니다. Recharts는 React 컴포넌트 API로 설계되어 데이터와 상태를 props로 직접 전달합니다. Chart.js는 Canvas 기반으로 React에서 ref와 imperative 초기화가 필요하고, Next.js SSR 환경에서 동적 import 처리가 추가로 필요합니다. 동일한 데이터 흐름 모델을 유지하면서 SSR과의 통합이 더 단순한 Recharts를 채택했습니다.

### 6. Framer Motion `MotionConfig` 공유 패턴

`MotionConfig`를 레이아웃 최상위에서 설정하여 하위 모든 `motion.*` 컴포넌트가 동일한 transition 설정을 공유합니다. 컴포넌트별 개별 transition 설정 없이 단일 지점에서 애니메이션 속도·곡선을 제어하고, 감소 모션(prefers-reduced-motion) 접근성 옵션도 동일하게 적용됩니다.

---

<!--
문서 갱신 가이드:
- 버전 업그레이드 시 표 내 버전 수치 갱신
- 신규 라이브러리 도입 시 해당 표에 행 추가 + 선택 근거 명시
- "주요 선택 근거" 항목 추가 시 선택 배경·대안·채택 이유 3요소 포함
-->
