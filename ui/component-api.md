---
description: UI 컴포넌트의 공개 API(props, 이벤트, 기본값) 설계 규칙
---

# 컴포넌트 API 설계

컴포넌트의 공개 API는 **한 번 공개하면 되돌리기 비싸다**. 이름·타입·기본값을 정할 때 반드시 아래 원칙을 따른다.

## Prop 네이밍

### Boolean prop

- **`is-` / `has-` / `can-` / `should-`** 접두어를 붙여 상태/능력임을 명확히 한다.
- 긍정형으로 작성한다. 부정형(`isNotDisabled`, `hideIcon`) 금지.

| ✅ | ❌ |
|---|---|
| `isActive`, `isDisabled`, `isLoading` | `active`, `disabled`, `loading` |
| `hasError`, `hasIcon` | `error`, `icon` (boolean 의도일 때) |
| `canDismiss`, `shouldAutoFocus` | `dismissible`, `autoFocus` (의미 모호) |

> 예외: DOM에 그대로 전달되는 HTML 속성(`disabled`, `readonly`, `required`)은 네이티브 이름 유지.

### Variant / size 계열

- **문자열 literal union**으로 정의한다. boolean 여러 개로 쪼개지 않는다.

```
// ✅ 단일 variant prop
variant: "primary" | "secondary" | "ghost"

// ❌ boolean 난립
isPrimary, isSecondary, isGhost
```

### 이벤트 핸들러

- **`on-` 접두어 + 현재형 동사**. (예: `onClick`, `onSelect`, `onOpenChange`)
- 상태 변경 이벤트는 `on{State}Change` 패턴. (예: `onOpenChange`, `onValueChange`)

## Controlled / Uncontrolled

모든 상태 보유 컴포넌트는 **두 모드 모두 지원**한다.

| 모드 | 표현 |
|---|---|
| Uncontrolled | `defaultValue`, `defaultOpen` — 컴포넌트 내부 상태 |
| Controlled | `value` + `onChange`, `open` + `onOpenChange` — 외부 제어 |

- 둘 다 제공하되 동시에 쓰면 controlled가 우선.
- `defaultValue`는 최초 마운트에서만 반영. 런타임 변경은 무시.

```
// ✅ 둘 다 지원
interface Props {
  open?: boolean;              // controlled
  defaultOpen?: boolean;       // uncontrolled
  onOpenChange?: (open: boolean) => void;
}
```

## 기본값 철학

- **가장 안전하고 덜 놀라운 동작**을 기본값으로 한다.
- 파괴적·비가역 동작은 기본값으로 두지 않는다. (예: `destructive: true` 기본값 금지)
- 기본값은 명시적으로 문서화한다 (PropsTable 등).

## asChild / slot 패턴

- 자식 엘리먼트를 감싸지 않고 **자식에게 props를 병합**하는 패턴 지원 여부를 컴포넌트별로 명시한다.
- 라우팅 라이브러리 Link, 커스텀 엘리먼트와의 결합을 위해 대부분의 인터랙티브 컴포넌트는 `asChild` 지원이 권장된다.

```
// ✅ asChild로 Link와 결합
<Button asChild>
  <Link href="/about">About</Link>
</Button>
```

- `asChild` 구현 시 className·data-*·이벤트 핸들러를 **병합**해야 한다(덮어쓰기 금지).

## API 표면 최소화 (YAGNI)

- **2~3회 반복 사용되기 전까지 prop을 추가하지 않는다.**
- "혹시 나중에 필요할지도"로 prop을 열면 되돌리기 어렵다. 필요해지면 그때 추가.
- 실제 사용 사례가 생기기 전까지는 내부에서 커스텀하도록 유도한다.

## 스타일 오버라이드 노출 방식

컴포넌트가 사용자에게 스타일 커스터마이즈를 허용할 때, **기본은 네이티브 CSS
경로(`style` / `className` / `data-*`)** 를 열어두는 것이다. 프로젝트·컴포넌트
전용 CSS 변수는 그것만으로 부족할 때만 추가한다. 변수가 늘어날수록 사용자는
이름을 외워야 하고, 컴포넌트는 사실상의 "API 표면"이 하나 더 생긴다.

디자인 토큰(`--space-*`, `--color-*` 등 프로젝트 공용) 은 **이 원칙의 예외가
아니라 본질**이다. 여기서 말하는 건 "개별 컴포넌트가 자기 이름이 붙은
변수(`--sh-ui-foo-bar`) 를 새로 만드는" 경우다.

### 우선순위

1. **`React.HTMLAttributes<T>` 확장 + `{...rest}` spread** — style / className /
   data-* / aria-* 가 전부 통과. 인터랙티브·컨테이너 컴포넌트의 기본값.
2. **의미 있는 추가 prop** (`variant`, `size`, `tone` 등) — 이름 있는 선택지로
   열어야 하는 축.
3. **컴포넌트 전용 CSS 변수 노출** — 1·2 로 안 되는 아래 "불가피한 경우"에만.

### CSS 변수가 정당한 경우 (예외)

아래 중 **하나 이상 해당**할 때만 도입한다:

- **Pseudo-element 에 동적 값 전달** — `::before` / `::after` 는 인라인 `style`
  로 닿을 수 없다. 부모에 변수를 세팅하고 pseudo 에서 `var(...)` 참조가 **유일한
  방법**이다.
- **테마 / 모드 / `data-state` 전환** — 같은 이름이 컨텍스트에 따라 다른 값을
  가져야 할 때 (다크모드·하이콘트라스트·`data-state="open"` 등). 변수 없이는
  불가능.
- **여러 CSS rule 에서 같은 값 재사용 (DRY)** — 하나의 값이 `:hover`,
  `:focus-visible`, `::after`, 자식 선택자 등 복수 rule 에 등장. 변수로 한 번
  정의해야 유지보수 가능.
- **Cascade 로 다수 후손에 일괄 전파가 상시 요구** — 부모 한 번 지정으로
  중첩 자식 수십 개가 읽어야 할 때. 단 per-level `style` 이나 외부 CSS 로
  보통 대체 가능하므로 **빈도가 실제로 높을 때만** 정당화.
- **런타임 JS 계산 값이 pseudo / 복수 rule 에 반영** — 단일 요소 단일 속성이면
  `style={{ width: x }}` 면 끝. 그 이상 조합이 필요할 때만.

### 변수가 불필요한 경우 (금지)

- **한 컴포넌트의 한 요소에만 적용되는 값을 1:1 로 매핑** — 그냥
  `style={{ gap: ... }}` 로 충분. `--xxx-gap` 변수를 만들지 말 것.
- **사용자가 변수 이름을 외워야 동작하는 상황이 일반 케이스** — 네이티브 CSS
  로 가능하면 네이티브 우선. 변수는 escape hatch 지 primary API 가 아니다.
- **"나중에 편할까 봐" 선제적 노출** — YAGNI. 실제 cascade / 테마 요구가
  반복 관찰된 뒤 추가한다.

### 판단 체크리스트

새 컴포넌트 전용 변수를 만들기 전에 자문:

1. 사용자가 `style={{ ... }}` 로 직접 덮어쓸 수 있는가? → 있으면 변수 불필요
2. pseudo-element 에 값을 전달해야 하는가? → 그러면 변수 **불가피**
3. 같은 값을 여러 CSS rule 에서 반복하는가? → 그러면 변수 정당
4. 다크모드 / 상태에 따라 값이 바뀌는가? → 그러면 변수 정당
5. 위 2~4 중 하나도 해당 없음 → `style`/`className` 경로 열고 변수는 만들지 말 것

### 금지

- **`className` 만 받고 `style` 안 받는** 컴포넌트 — 둘 다 열거나 `HTMLAttributes`
  를 확장한다
- **CSS 변수로만 조정 가능하고 `style` 로는 못 덮어쓰는** 구조 — 내부 element
  에만 적용되는 규칙이나 `!important` 남발 회피
- **변수를 만들어 놓고 실제로 상속/재사용이 없는** 경우 — 삭제하고 `style`
  경로로 통일

## Prop 일관성

- 같은 의미의 prop은 **모든 컴포넌트에서 같은 이름**으로 통일한다.
  - `isDisabled` vs `disabled` 혼용 금지 → 프로젝트 전반에 하나로.
  - `onChange` vs `onValueChange` 혼용 금지 → 값 변경 이벤트는 한 가지 규칙으로.
- 신규 컴포넌트를 만들 때 기존 컴포넌트의 prop 네이밍을 먼저 확인하고 맞춘다.

## 자식 / children

- 자유로운 컨텐츠 영역에는 `children`을 받는다.
- 정해진 슬롯이 여러 개면 `children`보다 **sub-component**(`Card.Header`, `Dialog.Footer`)가 낫다. — `composition.md` 참조.

## 금지

- **boolean prop 5개 이상 난립** → variant로 통합
- **undefined로 기본값 흉내내기** → 명시적 default 지정
- **`...rest` 남용으로 API 숨기기** → DOM에 전달할 것만 명확히 분리
- **prop으로 내부 구현(DOM 구조, CSS 클래스)을 노출** → 구현 바꾸면 깨짐
