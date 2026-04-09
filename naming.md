---
paths:
  - "**/*.ts"
  - "**/*.tsx"
---

# 네이밍 규칙

## 파일·폴더명

| 종류            | 케이스                        | 예시                                             |
|---------------|----------------------------|------------------------------------------------|
| 컴포넌트 (`.tsx`) | **PascalCase** (컴포넌트명과 동일) | `SignInForm.tsx`, `ToothChart.tsx`             |
| 훅 (`.ts`)     | **camelCase** (`use` 접두사)  | `useToothSelection.ts`, `useAuth.ts`           |
| 유틸·헬퍼 (`.ts`) | **camelCase**              | `formatDate.ts`, `getQueryClient.ts`           |
| 상수·타입 (`.ts`) | **camelCase**              | `toothNumbers.ts`, `types.ts`         |
| 설정 파일         | **camelCase**              | `navItems.ts`, `queryOptions.ts`               |
| 스키마 (`.ts`)   | **camelCase**              | `signInSchema.ts`                              |
| 폴더명           | **camelCase**              | `signIn/`, `toothSelection/`, `receptionList/` |
| i18n 메시지 키    | **camelCase**              | `hospitalName`, `patientName`                  |

## 코드 명명 규칙

- **변수·함수명**: **lowerCamelCase**를 사용한다. (단, React 컴포넌트 함수는 PascalCase)
    - 변수명: **명사**로 표기한다. (예: `count`, `patientName`)
    - 함수명: **동사** 또는 **동사구**로 표기한다. (예: `findCamera()`, `getPatientData()`)
- **상수명**: **SCREAMING_SNAKE_CASE**(대문자 언더스코어)를 사용한다. (예: `MAX_RETRY_COUNT`, `API_BASE_URL`)
- **이벤트 핸들러**: **handle** 접두사로 시작한다. (예: `handleClick`, `handleSubmit`, `handleInputChange`)
- **의미 중심 명명**: 축약어 사용을 지양하고, 의미를 알기 쉽게 풀어써서 명명한다.
- **API 타입명**: `Request`/`Response`/`Params`/`Payload` 등 접미사 없이 도메인 중심으로 명명한다. UI·훅에서도 자연스럽게 재사용할 수 있도록 한다.
    - 응답 타입: 도메인 모델명 그대로 (예: `FixtureOptions`, `ReceptionList`)
    - 요청 타입: 동작 + 도메인 모델명 (예: `CreateOrder`, `UpdatePatient`)

## 함수 정의 형식

- React 컴포넌트 (`.tsx`): **function declaration** (`function MyComponent() {}`)
- 그 외 모든 함수: **arrow function** (`const myFunc = () => {}`)

> ✅ 컴포넌트 — function declaration
> `export function SignInForm() { ... }`
>
> ✅ 일반 함수 — arrow function
> `export const getQueryClient = () => { ... }`
> `export const isJwtExpired = (token: string) => { ... }`
>
> ❌ 컴포넌트에 arrow function 사용 금지
> `export const MyComponent = () => { ... }`
>
> ❌ 일반 함수에 function declaration 사용 금지
> `export function myFunction() { ... }`

## API 파일 네이밍 규칙

`api/` 세그먼트의 파일은 백엔드 서버에 직접 요청하는 함수를 담으며, **`{httpMethod}{Purpose}Api`** 패턴을 따른다.

- 파일명과 함수명 모두 동일한 패턴을 사용한다.
- HTTP 메서드를 접두사로, `Api`를 접미사로 붙인다.

| HTTP 메서드 | 파일명 예시                  | 함수명 예시                |
|-----------|--------------------------|------------------------|
| GET       | `getFixtureOptionsApi.ts` | `getFixtureOptionsApi` |
| POST      | `postOrderApi.ts`         | `postOrderApi`         |
| PUT       | `putPatientApi.ts`        | `putPatientApi`        |
| PATCH     | `patchOrderStatusApi.ts`  | `patchOrderStatusApi`  |
| DELETE    | `deleteOrderApi.ts`       | `deleteOrderApi`       |
