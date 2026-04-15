---
paths:
  - "apps/*/src/**"
  - "packages/**"
---

# React 관심사 분리 원칙

React 컴포넌트와 훅 수준에서의 관심사 분리 규칙이다.

## 컴포넌트 수준 (.tsx 파일)

- `.tsx` 파일은 **UI 렌더링에 집중**한다. 비즈니스 로직이 섞이면 파악이 어려워진다.
- 복잡한 로직(여러 상태 + 핸들러, API 호출, 데이터 변환 등)은 **커스텀 훅으로 분리**한다. 컴포넌트에는 훅 호출 + JSX만 남긴다.
- 간단한 로직(한두 줄짜리 핸들러, 단순한 파생 값 등)은 컴포넌트에 인라인해도 무방하다.

```tsx
// ❌ .tsx 안에 비즈니스 로직이 뒤섞인 경우
function OrderForm() {
  const [selections, setSelections] = useState({});
  const mutation = useMutation({ ... });
  const handleSubmit = () => { /* 30줄짜리 검증+변환+제출 로직 */ };
  return <form>...</form>;
}

// ✅ 커스텀 훅으로 관심사 분리
function OrderForm() {
  const { selections, handleOptionChange } = useFixtureSelections();
  const { isPending, handleSubmit } = useOrderSubmit(selections);
  return <form>...</form>;
}
```

## 훅 수준

- 하나의 커스텀 훅은 **하나의 관심사**만 담당한다. "선택 상태 관리"와 "주문 제출"은 별도 훅으로 분리한다.
- 여러 곳에서 쓰이지 않더라도, 로직이 복잡하면 가독성을 위해 훅으로 분리한다.
- **mutation 훅은 mutation 실행에만 집중**한다. "검증 → 변환 → 제출"을 한 훅에서 전부 처리하지 않고, 검증/변환은 순수 함수로 분리하여 훅은 제출만 담당한다.

```ts
// ❌ 한 훅에서 검증 + 변환 + 제출을 모두 처리
const useOrderSubmit = () => {
  const handleSubmit = () => {
    const isValid = validate(data);        // 검증
    const payload = transformToPayload(data); // 변환
    mutate(payload);                          // 제출
  };
};

// ✅ 검증·변환은 순수 함수, 훅은 제출만 담당
// lib/validateOrder.ts
export const validateOrder = (data: OrderData): boolean => { ... };

// lib/transformOrder.ts
export const toOrderPayload = (data: OrderData): CreateOrder => { ... };

// model/hooks/useOrderSubmit.ts — 제출만 담당
const useOrderSubmit = () => {
  const { mutate } = useMutation({ mutationFn: postOrderApi });
  const handleSubmit = (payload: CreateOrder) => mutate(payload);
  return { handleSubmit };
};
```
