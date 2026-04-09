---
paths:
  - "apps/*/src/**"
---

# 데이터 페칭 및 상태 관리 패턴

## 로딩·에러 처리 원칙

- 로딩 처리와 에러 처리는 **Suspense + ErrorBoundary**를 우선 사용한다.
- 데이터 페칭에는 `useSuspenseQuery`를 기본으로 사용하고, `AsyncBoundary`로 감싼다.
- 수동 `isLoading`/`isError` 분기는 Suspense로 처리할 수 없는 경우에만 사용한다.
- `AsyncBoundary`는 **상위 컴포넌트가 아닌, 해당 컴포넌트 내부**에서 감싼다. Content를 분리하고 export하는 컴포넌트에서 래핑한다.
- Skeleton·Error fallback은 해당 UI 컴포넌트 폴더 안에 `Skeleton/`, `Error/` 폴더(PascalCase)로 관리한다.

```
ui/StepTwoForm/
├── index.tsx               # export 컴포넌트 (AsyncBoundary 래핑)
│                           # Content 컴포넌트 (useSuspenseQuery 등 실제 로직)
├── Skeleton/
│   └── StepTwoFormSkeleton.tsx
├── Error/
│   └── StepTwoFormError.tsx
└── ...
```

```tsx
// ✅ 컴포넌트 내부에서 AsyncBoundary 래핑
function MyComponentContent() {
  const { data } = useSuspenseQuery(...);
  return <div>{data}</div>;
}

export function MyComponent() {
  return (
    <AsyncBoundary
      suspenseFallback={<MyComponentSkeleton />}
      errorFallback={MyComponentError}
    >
      <MyComponentContent />
    </AsyncBoundary>
  );
}
```

## 쿼리 데이터 캐시 활용 원칙

- 부모에서 prefetch 또는 `useSuspenseQuery`로 가져온 데이터를 **props로 자식에게 내려주지 않는다.**
- 자식 컴포넌트가 동일한 데이터를 필요로 하면 **같은 queryOptions로 `useSuspenseQuery`를 직접 호출**하여 캐시에서 가져온다.
- 이미 캐시에 데이터가 있으므로 추가 네트워크 요청 없이 즉시 반환된다.

```tsx
// ❌ props로 데이터를 내려주는 패턴
function Parent() {
  const { data } = useSuspenseQuery(myQueryOptions());
  return <Child data={data} />;
}

// ✅ 자식이 캐시에서 직접 가져오는 패턴
function Parent() {
  useSuspenseQuery(myQueryOptions()); // prefetch 역할
  return <Child />;
}

function Child() {
  const { data } = useSuspenseQuery(myQueryOptions()); // 캐시 히트
  return <div>{data}</div>;
}
```

## queryOptions 작성 패턴

- `queryKey`는 별도의 **상수 함수**로 분리하여 export한다. `invalidateQueries`, `removeQueries` 등에서 독립적으로 사용하기 위함이다.
- queryKey 함수명은 **SCREAMING_SNAKE_CASE**를 따른다.

```ts
// ✅ queryKey를 별도 함수로 분리
export const FIXTURE_OPTIONS_QUERY_KEY = () =>
  ['FIXTURE_OPTIONS'] as const;

export const fixtureOptionsQueryOptions = () => ({
  queryKey: FIXTURE_OPTIONS_QUERY_KEY(),
  queryFn: getFixtureOptionsApi,
});

// 파라미터가 있는 경우
export const PRODUCT_BY_ID_QUERY_KEY = (params: ProductIdentity) =>
  ['PRODUCT_BY_ID', params] as const;

export const productByIdQueryOptions = (params: ProductIdentity) => ({
  queryKey: PRODUCT_BY_ID_QUERY_KEY(params),
  queryFn: () => getProductByIdApi(params),
});
```

## queryOptions 전략 분기 패턴

동일한 도메인 데이터를 **다른 소스**(API, localStorage, 메모리 등)에서 가져와야 하는 경우, `queryKey`는 동일하게 유지하고 `queryFn`만 컨텍스트에 따라 교체한다.

- `queryKey`는 **"무엇을 조회하느냐"** 를 나타내므로, 데이터 소스가 달라도 같은 도메인이면 동일한 키를 사용한다.
- `queryFn`은 **"어디서 가져오느냐"** 를 나타내므로, 컨텍스트에 따라 분기한다.
- 소비하는 컴포넌트는 데이터 소스를 알 필요 없이 동일한 `queryKey`로 캐시를 공유한다.
- 컨텍스트가 전환(로그인/로그아웃 등)되면 해당 `queryKey`를 `invalidateQueries`로 무효화하여 새 `queryFn`으로 다시 조회한다.

```ts
// entities/cart/model/query-options/cartQueryOptions.ts

export const CART_QUERY_KEY = () => ['CART'] as const;

// localStorage 기반 조회
const getGuestCart = async (): Promise<Cart> => {
  const raw = localStorage.getItem('cart');
  return raw ? JSON.parse(raw) : { items: [] };
};

export const cartQueryOptions = (isAuthenticated: boolean) => ({
  queryKey: CART_QUERY_KEY(),
  queryFn: isAuthenticated ? getCartApi : getGuestCart,
});
```

```tsx
// 컴포넌트 — 데이터 소스를 신경 쓰지 않고 동일하게 사용
const { data: cart } = useSuspenseQuery(cartQueryOptions(isAuthenticated));
```

```ts
// 로그인 성공 시 — 컨텍스트 전환으로 캐시 무효화
queryClient.invalidateQueries({ queryKey: CART_QUERY_KEY() });
```

## API 호출은 반드시 TanStack Query를 통해서

- `'use server'`로 정의한 API 함수를 컴포넌트나 훅에서 **직접 호출하지 않는다.**
- 데이터 조회는 `useSuspenseQuery` (또는 `useQuery`), 데이터 변경은 `useMutation`을 통해 호출한다.
- 이를 통해 캐싱, 로딩/에러 상태, 재시도, 낙관적 업데이트 등을 일관되게 관리한다.

```tsx
// ❌ API 함수 직접 호출
const handleSubmit = async () => {
  const data = await postOrderApi(payload);
};

// ✅ useMutation을 통해 호출
const { mutate } = useMutation({ mutationFn: postOrderApi });
const handleSubmit = () => mutate(payload);

// ✅ useSuspenseQuery를 통해 조회
const { data } = useSuspenseQuery(myQueryOptions());
```

## Mock 데이터 전략 (백엔드 미연동 시)

백엔드 연동 전에도 **실제 연동 시와 동일한 데이터 흐름**을 유지한다. API 함수가 유일한 데이터 진입점이므로, 이곳에서만 mock → 실제 호출로 교체하면 queryOptions·컴포넌트는 수정 없이 동작해야 한다.

- mock 데이터는 **API 함수 파일 안에 상수**로 정의한다. 별도 파일로 분리하지 않는다.
- 실제 HTTP 호출 코드를 **주석 + TODO**로 남기고, mock 데이터를 반환하는 함수를 작성한다.
- queryOptions → `useSuspenseQuery` 연결까지 **실제 연동 시와 동일하게 구성**한다.
- 뷰나 훅에서 mock 데이터를 직접 import하지 않는다.

```ts
// entities/customer/api/getCustomerListApi.ts

// TODO: 백엔드 연동 후 실제 API 호출로 교체
// export const getCustomerListApi = async (params) => {
//   const { data } = await http.get<PaginatedData<Customer>>('/v1/customers', { params });
//   return data;
// };

// TODO: 백엔드 연동 후 mock 데이터 및 아래 함수 삭제
const MOCK_CUSTOMERS: Customer[] = [ ... ];

export const getCustomerListApi = async (
  params: CustomerListParams = {},
): Promise<PaginatedData<Customer>> => {
  return { content: MOCK_CUSTOMERS, totalItems: MOCK_CUSTOMERS.length, ... };
};
```

**교체 시:** 주석 해제 → mock 상수와 mock 함수 삭제. queryOptions·컴포넌트는 변경 없음.

## 폼 상태 관리

폼 상태는 **Zustand store + Zod 검증**으로 관리한다. `react-hook-form`은 사용하지 않는다.

- 폼 데이터, UI 상태, 에러를 **하나의 Zustand store**에 통합한다.
- 검증은 **Zod 스키마의 `safeParse()`** 로 수행하고, 에러를 store의 `errors`에 저장한다.
- 에러 표시는 `@workspace/ui/components/field`의 `FieldError` 컴포넌트를 사용한다.
- `setField()` 호출 시 해당 필드의 에러를 자동으로 클리어한다.
- 첫 번째 에러 필드로 포커스 이동은 `focusFirstError()` 유틸을 사용한다.
- Enter 키로 다음 필드 이동은 `focusNextField()` 유틸을 사용한다.

```ts
// 검증 예시
const isValid = validateStepOne(message);
if (!isValid) {
  focusFirstError(useReceiptStore.getState().errors);
  return;
}
```
