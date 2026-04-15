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
