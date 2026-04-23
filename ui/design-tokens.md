---
description: 디자인 토큰 계층 구조 및 사용 원칙
---

# 디자인 토큰

디자인 토큰은 **디자인 결정을 이름으로 고정한 값**이다. 색상·여백·폰트·반경·그림자 등 재사용되는 값을 모두 토큰으로 관리한다.

## 토큰 계층

### 1. Primitive Token (원시 토큰)

- **원시값 자체.** 의미·용도 없음. 팔레트에 가깝다.
- 컴포넌트에서 **직접 사용 금지**. 오직 semantic 토큰을 구성하는 재료.

```
--blue-500: #3b82f6;
--gray-100: #f3f4f6;
--spacing-4: 16px;
--radius-md: 8px;
```

### 2. Semantic Token (의미 토큰)

- **의미·용도를 담은 토큰.** primitive를 참조해 "무엇에 쓰는 값인지" 명명.
- **컴포넌트는 semantic 토큰만 사용**한다.

```
--color-primary: var(--blue-500);
--color-background: var(--gray-50);
--color-border: var(--gray-200);
--color-text-muted: var(--gray-500);
--spacing-card-padding: var(--spacing-4);
```

### 3. Component Token (선택)

- 특정 컴포넌트 전용 토큰. 글로벌하게 쓰이지 않는 값.
- semantic으로 충분하면 만들지 않는다.

```
--button-height-md: 40px;
--dialog-max-width: 560px;
```

## 카테고리별 원칙

### Color

- primitive: 스케일 고정 (50~950, 100 단위 또는 10 단위).
- semantic: 역할 기반 명명.
  - `color-background`, `color-foreground`
  - `color-primary`, `color-secondary`, `color-accent`
  - `color-success`, `color-warning`, `color-danger`, `color-info`
  - `color-border`, `color-ring` (포커스링), `color-muted`
- **다크 모드**는 semantic 계층에서만 스위칭. primitive는 그대로 유지.

### Spacing

- **일관된 스케일** 사용 (4px 기반 권장: 4, 8, 12, 16, 20, 24, 32, 40, 48...).
- "임의의 값" 금지. `padding: 13px` 같은 것.
- 컴포넌트 간 간격(gap)도 토큰 사용.

### Typography

- **font-size, line-height, letter-spacing을 한 묶음**으로 semantic 토큰화.
- `text-xs`, `text-sm`, `text-base`, `text-lg`, `text-xl`, `text-2xl` ...
- font-family는 `font-sans`, `font-mono`, `font-serif` 등 용도별.

### Radius

- `radius-sm`, `radius-md`, `radius-lg`, `radius-full`
- 컴포넌트 간 일관성 중요 (카드·버튼·입력이 서로 다른 반경이면 어색함).

### Shadow / Elevation

- `shadow-sm`, `shadow-md`, `shadow-lg`, `shadow-xl`
- 의미 기반 (`shadow-popover`, `shadow-dialog`)도 가능.

### Z-index

- **임의의 숫자 금지** (`z-index: 9999`). 계층 토큰 사용.
- `z-base`, `z-dropdown`, `z-sticky`, `z-fixed`, `z-modal-backdrop`, `z-modal`, `z-popover`, `z-tooltip`.

### Motion

- `duration-fast` (~150ms), `duration-base` (~250ms), `duration-slow` (~400ms).
- `easing-in`, `easing-out`, `easing-in-out`, `easing-spring`.

## 매직값 금지

컴포넌트 코드 안에 **하드코딩된 값 금지**:

```
// ❌
padding: 13px;
color: #3b82f6;
box-shadow: 0 2px 4px rgba(0,0,0,0.1);
z-index: 9999;

// ✅
padding: var(--spacing-3);
color: var(--color-primary);
box-shadow: var(--shadow-md);
z-index: var(--z-modal);
```

## 토큰 네이밍 원칙

- **역할·용도 기반**, 외형 묘사 금지.
  - ❌ `color-red` (빨강이 에러 색이 아닐 수도 있다)
  - ✅ `color-danger`
- **계층적 네이밍**: `{카테고리}-{역할}-{변형}?` 순서.
  - `color-text-muted`, `color-background-hover`, `spacing-inline-sm`
- **약어 지양**. `bg-pri` 대신 `background-primary`.

## 테마/모드 전환

- 테마(light/dark 등)는 **semantic 토큰만 재할당**한다.
- 컴포넌트 코드는 테마를 몰라야 한다. CSS 변수·속성 selector(`[data-theme="dark"]`)로 스위칭.

```
:root {
  --color-background: var(--gray-50);
  --color-foreground: var(--gray-900);
}

[data-theme="dark"] {
  --color-background: var(--gray-950);
  --color-foreground: var(--gray-50);
}
```

## 토큰 추가 기준 (AHA)

- **2~3회 반복 사용된 값만 토큰화**한다. 한 번 쓴 값을 섣불리 토큰으로 올리지 않는다.
- 반대로 3회 이상 반복되는 매직값은 즉시 토큰화.
- **프로젝트 공용 semantic 토큰**(여기서 다루는 주제)과 **개별 컴포넌트 전용
  CSS 변수**(예: `--sh-ui-foo-gap`)는 다른 결정이다. 후자의 도입 기준은
  `component-api.md` 의 "스타일 오버라이드 노출 방식"을 따른다. 요약:
  **pseudo-element·테마 전환·다중 rule 재사용·실제로 필요한 cascade** 중
  하나에 해당할 때만 열고, 단일 요소의 값을 1:1 매핑하는 용도면 `style` 경로를
  쓴다.

## 문서화

- 모든 semantic 토큰은 **용도와 사용 예시**를 문서화.
- 디자이너·개발자가 같은 이름으로 소통할 수 있어야 한다 (Figma 토큰 ↔ 코드 토큰 1:1 매칭 권장).

## 금지

- **컴포넌트에서 primitive 토큰 직접 사용** (`var(--blue-500)`).
- **의미 없는 축약** (`--c-p` 대신 `--color-primary`).
- **하드코딩된 값** 재등장.
- **테마 분기를 컴포넌트 안에서** 처리 (토큰 레벨에서 해결).
- **숫자 접미사로 의미 구분** (`color-1`, `color-2` — 의미 전달 실패).
