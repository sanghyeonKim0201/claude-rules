---
paths:
  - "apps/*/src/**"
  - "packages/**"
---

# 관심사 분리 원칙

관심사 분리(Separation of Concerns)는 코드 전반에 걸쳐 가장 중요한 설계 원칙이다. 폴더 구조, 컴포넌트, 훅, 유틸, 아키텍처 설계 모든 수준에서 적용한다.

## 아키텍처·폴더 수준

- **FSD 레이어 간 책임**: `shared`(범용) → `entities`(도메인 모델) → `features`(사용자 인터랙션) → `widgets`(복합 조합) → `views`(페이지) 순으로 의존한다. 상위 레이어가 하위 레이어에 의존하고, **같은 레이어의 slice끼리는 직접 import하지 않는다.**
- **같은 레이어 간 조합이 필요한 경우**: 우선 **상위 레이어(widgets)로 올려서 조합**한다. widgets는 여러 features·entities를 자유롭게 import할 수 있으므로 직접 조합이 가능하다.
- **slot이 필요한 경우**: **entities UI 내부에 features 요소를 삽입**해야 할 때만 slot(ReactNode props)을 사용한다. 예를 들어 entities의 `ProductCard` 안에 features의 즐겨찾기 버튼을 배치해야 하는 경우, 카드 외부에서 조합할 수 없으므로 slot이 유일한 해결책이다. features 간 조합이 필요하면 slot 대신 widgets로 올린다.

```tsx
// ✅ 상위 레이어로 올려서 조합 (slot 불필요 — 나란히 배치 가능한 경우)
// widgets/receiptForm — receipt + toggleFixtureFavorite를 직접 import
import { ReceiptStepOneForm } from '@/src/features/receipt';
import { FixtureFavorites } from '@/src/features/toggleFixtureFavorite';

// ✅ slot 사용 (컴포넌트 내부에 삽입해야 하는 경우)
// entities/product — 카드 내부에 액션 영역을 slot으로 열어둠
export function ProductCard({ product, actionSlot }: Props) {
  return (
    <div>
      <div>{product.name}</div>
      {actionSlot && <div>{actionSlot}</div>}
    </div>
  );
}

// widgets/views — slot에 feature를 주입
<ProductCard product={product} actionSlot={<FavoriteButton id={product.id} />} />
```
- **widgets 레이어 사용 기준**: 여러 features·entities가 조합되는 복합 UI 블록이면서 **여러 페이지에서 재사용**되는 경우 widgets로 분리한다. 한 페이지에서만 사용하는 조합은 **views(페이지 레이어)에서 직접 구현**하는 것이 더 깔끔하다.
- **세그먼트별 역할**: 하나의 세그먼트가 다른 세그먼트의 책임을 침범하지 않는다.
  - `ui/` — UI 컴포넌트, 스타일 등 UI 표현과 직접 관련된 코드
  - `api/` — request 함수, data types, mappers 등 백엔드 통신 및 데이터 로직
  - `model/` — schema, interfaces, store, business logic 등 애플리케이션 도메인 모델
  - `lib/` — 해당 slice에서 여러 모듈이 함께 사용하는 공통 library code (slice 루트에 위치, `model/` 안이 아님)
  - `config/` — configuration files, feature flags 등 환경/기능 설정

## 컴포넌트 수준 (.tsx 파일)

- `.tsx` 파일은 **UI 렌더링에 집중**한다. 비즈니스 로직이 섞이면 파악이 어려워진다.
- 복잡한 로직(여러 상태 + 핸들러, API 호출, 데이터 변환 등)은 **커스텀 훅으로 분리**한다. 컴포넌트에는 훅 호출 + JSX만 남긴다.
- 간단한 로직(한두 줄짜리 핸들러, 단순한 파생 값 등)은 컴포넌트에 인라인해도 무방하다.
- 컴포넌트와 무관한 **순수 함수**(데이터 변환, 필터링, 포맷팅 등)는 컴포넌트 외부 또는 `lib/`에 정의한다.

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
- **검증(validation)은 순수 함수로 분리**한다. mutation 훅 안에 검증 로직을 섞지 않고, `lib/` 또는 `model/`에 별도 함수로 추출한다.
- **데이터 변환(transformation)은 순수 함수로 분리**한다. 폼 데이터 → API 페이로드 변환 등은 훅 외부의 순수 함수로 정의한다.
- **mutation 훅은 mutation 실행에만 집중**한다. "검증 → 변환 → 제출"을 한 훅에서 전부 처리하지 않고, 검증/변환은 분리하여 훅은 제출만 담당한다.

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
