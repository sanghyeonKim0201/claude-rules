---
paths:
  - "apps/*/src/**"
  - "apps/*/app/**"
---

# FSD 아키텍처 상세 규칙

## App Router 규칙

`app/` 폴더(Next.js App Router)는 **오로지 FSD 레이어의 컴포넌트를 import하여 호출하는 용도로만** 사용한다. 데이터 페칭, 비즈니스 로직, 상태 관리, 조건부 렌더링 등 어떤 로직도 `app/` 안에 작성하지 않는다. 모든 로직은 FSD 레이어(`views`, `widgets`, `app/layouts`, `app/providers` 등) 안에서 처리한다.

```tsx
// ✅ app/[locale]/(main)/company/page.tsx — view만 호출
import { CompanyInfoView } from '@/src/views/companyInfo';
export default function CompanyInfoPage() {
  return <CompanyInfoView />;
}

// ✅ app/[locale]/(main)/layout.tsx — layout만 호출
import { MainLayout } from '@/src/app/layouts/MainLayout';
export default function Layout({ children }) {
  return <MainLayout>{children}</MainLayout>;
}

// ❌ app/ 안에서 직접 데이터 페칭, 조건부 렌더링, 비즈니스 로직 작성
```

## 파일 구조 규칙

세그먼트(`model`, `ui`, `api` 등) 내부에서 관련 파일들은 **용도별 폴더**로 묶어 관리한다.

- **UI 세그먼트:** 폴더 + `index.tsx` (컴포넌트 진입점, 폴더명은 PascalCase)
- **그 외 세그먼트:** 명시적 파일명 (`index.ts` 사용 금지, 검색 편의를 위해)

```
features/auth/signIn/
├── index.ts                        # slice public API
├── model/
│   └── schema/
│       └── signInSchema.ts         # ✅ 명시적 파일명 (camelCase)
├── ui/
│   └── SignInForm/
│       └── index.tsx               # ✅ UI는 폴더 + index.tsx
└── api/
    └── postSignUpApi.ts            # ✅ API 파일 네이밍
```

## Public API 규칙

각 slice는 `index.ts`를 통해 외부에 노출한다. 외부에서는 반드시 public API를 통해서만 접근한다.

```ts
// features/auth/signIn/index.ts
export { SignInForm } from './ui/SignInForm';

// ✅ 외부에서 public API를 통해 접근
import { SignInForm } from '@/src/features/auth/signIn';

// ❌ 내부 파일 직접 접근 금지
import { SignInForm } from '@/src/features/auth/signIn/ui/SignInForm';
```

## Import 규칙

- **같은 slice 내부:** 상대 경로
- **다른 layer·slice 참조:** 절대 경로 (`@/src/...`, `@workspace/...`)
- import 블록은 용도별로 **주석 구분**하여 정리한다.

```ts
// libs
import { useTranslations } from 'next-intl';

// entities
import { useReceiptStore } from '@/src/entities/receipt';

// shared
import { Button } from '@workspace/ui/components/button';
import { Input } from '@workspace/ui/components/input';
```

## FSD 레이어 간 공유 규칙

- features 간 공유가 필요한 코드는 **entities 레이어로 내린다.**
- 도메인 특화되지 않은 범용 컴포넌트는 **shared 레이어**에 둔다.

## entities vs features 경계 규칙

- **entities**: **조회** 중심. GET API 함수, queryOptions, 도메인 모델/타입, 읽기용 UI를 담는다.
  - 판단 기준: **"보여주기만 하는가?"** → entities
- **features**: **비즈니스 로직** 중심. POST/PUT/PATCH/DELETE API 함수, useMutation 훅, 사용자 인터랙션 UI를 담는다.
  - 판단 기준: **"사용자가 뭔가를 바꾸는가?"** → features
- features에서 entities의 queryOptions를 import하여 `useSuspenseQuery`로 캐시 데이터를 사용하는 것이 기본 패턴이다.
- **도메인 데이터 기반 Select는 entities의 ui 세그먼트에 둔다.** 직원, 공정 등 도메인 목록을 조회하여 보여주는 Select 컴포넌트는 entities에서 만들고 public API로 export한다. features에서 인라인으로 구현하지 않고, entities의 Select를 import하여 재사용한다.

```tsx
// ✅ entities에 도메인 Select 컴포넌트를 두고 재사용
// entities/employee/ui/EmployeeSelect/index.tsx
export function EmployeeSelect({ value, onValueChange, name, placeholder }: Props) {
  const { data } = useQuery(employeeListQueryOptions({ keyword: '' }, 0, 100));
  return <Select ...>{/* 직원 목록 옵션 */}</Select>;
}

// features에서 import하여 사용
import { EmployeeSelect } from '@/src/entities/employee';
<Form.Field name='manager' label='관리자'>
  {({ value, onChange }) => (
    <EmployeeSelect name='manager' value={value as string} onValueChange={onChange} />
  )}
</Form.Field>

// ❌ features에서 직접 useQuery + Select를 인라인으로 구현
const { data: employeeData } = useQuery(employeeListQueryOptions(...));
<Select>{employeeData?.content.map(...)}</Select>
```

## 리스트 컴포넌트 규칙

- entities·features에는 **리스트 전체를 표현하는 컴포넌트를 두지 않는다.**
- entities·features에는 리스트 안에 표현되는 **개별 아이템 컴포넌트**(Card, Badge, Button 등)만 둔다.
- 리스트 API 호출과 `data.map()`으로 아이템 컴포넌트를 나열하는 것은 **상위 레이어(widgets·views)**의 책임이다.

```tsx
// ✅ entities/features — 개별 아이템만 담당
export function DraftOrderBadge({ draft, onSelect }: Props) { ... }

// ✅ 상위 레이어(widgets/views) — 리스트 조회 + map 조합
const { data } = useSuspenseQuery(draftOrdersQueryOptions());
{data.content.map((draft) => (
  <DraftOrderBadge key={draft.id} draft={draft} onSelect={handleSelect} />
))}

// ❌ entities/features 안에 리스트 컴포넌트를 두지 않는다
export function DraftOrderList() {
  const { data } = useSuspenseQuery(draftOrdersQueryOptions());
  return data.content.map(...);
}
```

## features slice 단일 책임 원칙

- **1 feature slice = 1 사용자 기능(액션)**을 원칙으로 한다.
- 하나의 slice가 여러 독립적인 기능(제출, 임시 저장, 즐겨찾기 등)을 함께 담지 않는다.
- 분리 기준은 **변경 이유가 다른가**이다. 변경 이유가 다르면 별도 slice로 분리한다.

```
// ✅ 기능별로 분리된 features
features/receipt/              — 접수 폼 UI + 제출
features/saveDraftOrder/       — 임시 저장 (mutation + API + UI)
features/toggleFixtureFavorite/ — 즐겨찾기 토글 (mutation + API + UI)

// ❌ 하나의 slice에 여러 기능이 혼재
features/receipt/              — 폼 + 임시 저장 + 즐겨찾기
```

- **create/update를 mode 분기로 한 slice에 넣지 않는다.** 생성과 수정은 변경 이유가 다르므로 별도 slice로 분리한다.

```
// ❌ mode 분기로 한 slice에서 처리
features/fixtureOptionSave/    — mode === 'add' ? postApi : putApi

// ✅ 별도 slice로 분리
features/fixtureOptionAdd/     — 옵션 추가 (POST)
features/fixtureOptionEdit/    — 옵션 수정 (PUT)
```

## features slice UI 우선 export 원칙

- feature slice는 가능한 한 **훅을 직접 export하지 않고, 훅을 사용한 UI 컴포넌트를 export**한다.
- 로직(훅, mutation 등)은 slice 내부에서 소비하고, 외부에는 완성된 UI만 노출한다.
- views에서는 feature가 export한 UI 컴포넌트를 배치·조합하기만 하고, 로직을 직접 다루지 않는다.

```tsx
// ✅ UI 컴포넌트를 export — views는 배치만
import { DraftSaveButton } from '@/src/features/saveDraftOrder';
<DraftSaveButton />

// ❌ 훅을 export — views에서 로직을 다뤄야 함
import { useDraftSave } from '@/src/features/saveDraftOrder';
const { isPending, handleDraftSave } = useDraftSave();
<Button disabled={isPending} onClick={handleDraftSave}>...</Button>
```

## 타입 정의 위치 규칙

- **entities**: 여러 features에서 공유하는 도메인 모델 타입은 `model/types.ts`에 모아두고 `index.ts`로 re-export한다. 소비하는 쪽에서 public API를 통해 깔끔하게 import할 수 있도록 한다.
- **features**: 같은 slice 내에서만 쓰이는 타입은 **정의된 파일에서 그대로 export**한다. 별도 `model/types.ts`로 모을 필요 없이 출처에서 직접 import한다.

```ts
// ✅ entities — 도메인 모델 타입은 model/types.ts에 모아서 public API로 제공
import type { FixtureOptions } from '@/src/entities/receipt';

// ✅ features — slice 내부 타입은 출처에서 직접 import
import type { CreateOrder } from '../../api/postOrderApi';

// ❌ features에서 굳이 model/types.ts로 모을 필요 없음
import type { CreateOrder } from '../../model/types';
```
