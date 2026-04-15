---
paths:
  - "**/*.ts"
  - "**/*.tsx"
---

# React/TypeScript 설계 원칙

## TypeScript 원칙

- **`any` 타입 사용 금지**: `any` 대신 `unknown` + 타입 가드, 제네릭 등을 사용하여 타입 안전성을 확보한다.

## React 원칙

- **조건부 렌더링 가독성**: 삼항 연산자 중첩 금지. 복잡한 분기는 Early Return이나 변수 추출로 처리한다.

## useEffect·useState 최소화

- `useEffect`는 **최대한 사용하지 않는다.**
- 외부 시스템과의 동기화(DOM 조작, 구독 등) 등 진정한 사이드 이펙트에만 한정하여 사용한다.
- 파생 상태는 렌더링 중 계산(`useMemo`, 변수 선언)으로 처리한다.
- 이벤트에 대한 반응은 이벤트 핸들러에서 직접 처리한다.
- `useState`도 **진짜 클라이언트 로컬 상태가 필요한 경우에만** 사용한다.
- 서버에서 관리하는 데이터는 `useState` 대신 **TanStack Query(`useSuspenseQuery`, `useMutation`)로 관리**한다.
- props나 다른 상태에서 파생 가능한 값은 `useState`로 복제하지 않고 계산으로 처리한다.

## 서버 컴포넌트 우선 원칙

- 컴포넌트는 기본적으로 **서버 컴포넌트(RSC)**로 작성한다.
- `'use client'`는 클라이언트 API(`useState`, `useEffect`, 이벤트 핸들러 등)가 **실제로 필요한 컴포넌트에만** 선언한다.
- 클라이언트 경계는 가능한 **트리의 말단(leaf)**으로 내린다. 부모가 서버 컴포넌트를 유지할 수 있으면 자식만 클라이언트로 분리한다.

## UI 설계

반응형 디자인, 모바일 우선 접근. Tailwind CSS + shadcn/ui 컴포넌트 사용.
테마(다크 모드 등)를 고려하여 설계한다.

- **shadcn/ui 우선 사용:** UI 요소를 구현할 때 shadcn/ui에 해당 컴포넌트가 있으면 **반드시 shadcn/ui 컴포넌트를 사용**한다. 네이티브 HTML 요소(`<select>`, `<input>`, `<dialog>` 등)로 직접 구현하지 않는다.
- **Base UI 기반:** shadcn/ui 컴포넌트의 내부 프리미티브는 Radix UI가 아닌 **Base UI(`@base-ui/react`)**를 사용한다. 따라서 `asChild` prop 대신 `render` prop을 사용하는 등 Base UI의 API를 따른다.
- **`px` 등 고정 단위 사용 금지:** 크기·간격·폰트 등 모든 스타일링에 `px`, `pt` 같은 절대 단위를 사용하지 않는다. Tailwind의 상대 단위 클래스(`text-sm`, `p-2`, `gap-4` 등)를 사용하고, 불가피하게 커스텀 값이 필요할 때도 `rem`, `em`, `%`, `vw/vh` 등 상대 단위를 우선한다.
