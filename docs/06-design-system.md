# 디자인 시스템

## 개요

학습자 영역 전체에 일관된 시각 언어를 적용하기 위해 디자인 시스템을 구축했습니다. 세 가지 목표를 기준으로 설계했습니다.

- **Raycast 톤**: 다크 우선·뉴트럴 그레이·미세 그라데이션·절제된 모션
- **접근성**: WCAG AA 대비 확보 · `prefers-reduced-motion` 자동 대응
- **유지보수성**: 토큰 단일 파일 관리 · 컴포넌트 표준화 · 공용 variants 중앙 정의

관리자 영역(`/lms-manage`)은 별도 운영 UX 트랙에서 처리합니다. 이 문서의 디자인 시스템은 학습자 영역에 적용됩니다.

라이브 확인: [design-tokens 페이지](https://zslab-lms.duckdns.org/design-tokens)

---

## 🎨 디자인 토큰

### Tailwind v4 `@theme inline` 방식

`globals.css` 단일 파일에서 CSS 변수를 선언하고 `@theme inline` 블록으로 Tailwind 유틸리티에 매핑합니다. `tailwind.config.*` 파일 없이 v4 표준 방식으로 토큰을 관리합니다.

```css
/* globals.css — 시맨틱 토큰 예시 */
:root {
  --primary: 221 83% 53%;   /* blue-600 (라이트) */
  --success: 161 94% 30%;
}
.dark {
  --primary: 217 91% 60%;   /* blue-500 (다크) */
  --success: 160 84% 39%;
}
@theme inline {
  --color-primary: hsl(var(--primary));
  --color-success: hsl(var(--success));
}
```

컴포넌트는 `bg-primary`, `text-success`, `shadow-elevation-2` 등 시맨틱 클래스를 직접 참조합니다. 테마 전환 시 `.dark` 클래스 교체 한 번으로 전체에 적용됩니다.

---

### 색상 토큰 (시맨틱 21종)

라이트(`:root`)와 다크(`.dark`) 모드를 분리 선언합니다.

#### 라이트 팔레트 (Slate + Blue)

| 그룹 | 토큰 | 기준 |
|---|---|---|
| 배경·표면 | background / card / popover | white / white / white |
| 전경 | foreground (surface별 포함) | slate-900 |
| primary | primary / primary-foreground | blue-600 / white |
| secondary / muted / accent | 각 surface·foreground | slate-100 / slate-900 |
| 상태 | success / warning | emerald-600 / amber-600 |
| 경계 | border / input / ring | slate-200 / slate-200 / blue-600 |
| 파괴적 | destructive / destructive-foreground | red-600 / white |

#### 다크 팔레트 (Slate 3단계 깊이)

| 표면 | 토큰 | 기준 |
|---|---|---|
| 페이지 | background | slate-900 |
| 카드 | card | slate-800 |
| 팝오버 | popover | slate-700 |
| primary | primary | blue-500 |
| 상태 | success / warning | emerald-500 / amber-500 |

page → card → popover 3단계 계조로 인터페이스 깊이를 표현합니다.

---

### 확장 토큰 (Raycast 고유)

기본 색상 외에 Raycast 톤에 특화된 4개 토큰 그룹을 추가했습니다.

| 그룹 | 토큰 | 설명 |
|---|---|---|
| Motion | `--motion-duration-fast` (120ms) · `--motion-duration-base` (200ms) · `--motion-duration-slow` (320ms) · ease-standard · ease-emphasized | 모드 독립 정의 · 전 컴포넌트 전환 속도 일원화 |
| Elevation | `--elevation-1` / `--elevation-2` / `--elevation-3` | 라이트·다크 rgba 강도 분리 · `shadow-elevation-1/2/3` Tailwind 유틸리티 매핑 |
| Glow | `--glow-primary` / `--glow-success` | 다크 모드 발광 액센트 (라이트 대비 강도 증가) |
| Gradient | `--gradient-surface` / `--gradient-primary` | 배경·주요 요소 미세 그라데이션 |

---

## 🧩 컴포넌트 표준화

shadcn/ui 14개 컴포넌트를 프로젝트에 직접 복사하는 방식으로 관리합니다. Radix UI 접근성 프리미티브 위에 Tailwind 스타일을 적용하며, 외부 컴포넌트 라이브러리 의존 없이 프로젝트 내에서 직접 수정합니다.

모든 컴포넌트의 cva variants 기본값을 Raycast 톤으로 교체해 사용처 코드 변경 없이 일관성을 확보했습니다.

| 컴포넌트 | 주요 변경 |
|---|---|
| button | 기본값 교체 · glow-primary hover |
| card | elevation-2 기본 그림자 |
| input | 기본값 교체 |
| badge | **success · warning variant 신규 추가** |
| dialog | 기본값 교체 |
| select | 기본값 교체 |
| dropdown-menu | 기본값 교체 |
| tabs | 기본값 교체 |
| switch | 기본값 교체 |
| textarea | 기본값 교체 |
| avatar | muted-foreground 명시 |
| label | 기존 최적 — 변경 없음 |
| separator | 기존 최적 — 변경 없음 |
| skeleton | **신설** — `animate-pulse bg-muted` 표준 패턴 |

---

## ✨ 모션 시스템

### MotionConfig 전역 설정

`components/providers.tsx`에서 `MotionConfig reducedMotion="user"`를 최상위에 적용합니다. 하위 모든 `motion.*` 컴포넌트가 OS `prefers-reduced-motion` 설정을 자동으로 따릅니다. 개별 컴포넌트에서 별도 설정이 필요 없습니다.

### 공용 variants (`lib/motion.ts`)

4종 variants를 중앙 파일에 정의해 전 학습자 페이지에서 재사용합니다.

| variant | 동작 | 적용 위치 |
|---|---|---|
| `pageEnter` | opacity 0→1 · y 8px→0 · 200ms | 페이지 진입 전체 |
| `cardContainer` | stagger children · delayChildren 0.05s | 카드 그리드 컨테이너 |
| `cardItem` | opacity 0→1 · y 12px→0 · 200ms | 카드 아이템 개별 |
| `scrollFadeIn` | opacity 0→1 · y 16px→0 · 320ms · viewport once | 스크롤 진입 섹션 |

### `app/template.tsx` 일원화

Next.js `template.tsx` 파일 하나로 학습자 전체 페이지의 진입 애니메이션을 통합했습니다. 관리자(`/lms-manage`) 경로는 분기 제외됩니다. 17개 페이지에 분산된 개별 래퍼를 단일 파일로 교체했습니다.

### 보호 영역 정책

모션을 적용하지 않는 영역을 명시적으로 지정했습니다.

| 영역 | 이유 |
|---|---|
| 영상 플레이어 · 진도바 · HLS 재생 영역 | hls.js 전체화면 전환·진도 기록 정상성 보호 |
| 시험 응시 페이지 | 타이머·자동 제출·beforeunload 이벤트 보호 |
| 과제 제출 페이지 | 텍스트에어리어 포커스 안정성 보호 |
| 헤더·푸터 레이아웃 | App Router 클라이언트 내비게이션 시 재마운트 없음 — 진입 모션 실효 없음 |

---

## 📊 성능·접근성

Lighthouse로 학습자 공개 5개 페이지를 측정하고 기준점을 기록했습니다.

| 카테고리 | 결과 | 비고 |
|---|---|---|
| Performance | 73~79 | 로컬 Docker · 모바일 4G 시뮬레이션 환경 |
| Accessibility | 88~100 | 5개 페이지 중 3개 100점 |
| Best Practices | 100 | 전 페이지 |
| SEO | 100 | 전 페이지 |

| Core Web Vitals | 결과 | 평가 |
|---|---|---|
| TBT (Total Blocking Time) | 4~18ms | ✅ Good 전 페이지 — Framer Motion 메인 스레드 차단 없음 |
| CLS (Cumulative Layout Shift) | 0.003~0.024 | ✅ Good 전 페이지 — Skeleton 로딩 패턴으로 레이아웃 안정화 |
| LCP (Largest Contentful Paint) | 5,100~6,200ms | ⚠️ 로컬 Docker + 모바일 시뮬레이션 환경 특성 (TTFB 주 원인) |

TBT가 전 페이지 완벽(4~18ms)으로 Framer Motion이 메인 스레드를 차단하지 않음을 수치로 확인했습니다.

---

## 🌐 라이브 데모

디자인 토큰·컴포넌트·모션 인터랙션을 다크/라이트 모드 양쪽에서 확인할 수 있습니다.

| 항목 | URL |
|---|---|
| 디자인 시스템 페이지 | https://zslab-lms.duckdns.org/design-tokens |
| 학습자 메인 | https://zslab-lms.duckdns.org/ |

---

<!--
문서 갱신 가이드:
- 토큰 추가 시 해당 그룹 표에 행 추가
- 컴포넌트 추가 시 표에 행 추가
- Lighthouse 재측정 시 수치·날짜 갱신
-->
