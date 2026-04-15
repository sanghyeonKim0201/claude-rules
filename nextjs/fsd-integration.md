---
paths:
  - "apps/*/app/**"
  - "app/**"
---

# Next.js에서의 FSD 통합 규칙

이 문서는 **FSD 자체 규칙이 아니라**, Next.js App Router에서 FSD를 사용할 때 필요한 프레임워크 종속 규칙을 다룬다.

## App Router 규칙

`app/` 폴더(Next.js App Router)는 **FSD 레이어의 진입점을 import하여 연결하는 어댑터 계층**으로 사용한다.

- `app/` 안에서는 화면 진입, 라우팅 연결, 레이아웃 연결만 담당한다.
- 데이터 페칭, 비즈니스 로직, 상태 관리, 복잡한 조건부 렌더링은 FSD 레이어 내부에서 처리한다.
- 즉 `app/`은 프레임워크 라우팅 규약을 수용하는 얇은 껍데기여야 한다.

```tsx
// ✅ app/company/page.tsx
import { CompanyInfoView } from '@/src/views/companyInfo';

export default function CompanyInfoPage() {
  return <CompanyInfoView />;
}
```

```tsx
// ✅ app/(main)/layout.tsx
import { MainLayout } from '@/src/app/layouts/MainLayout';

export default function Layout({ children }: { children: React.ReactNode }) {
  return <MainLayout>{children}</MainLayout>;
}
```

- `app/` 안에서 직접 query, mutation, domain branching을 작성하지 않는다.
- Next.js 고유 개념(서버 컴포넌트, client boundary, route params 처리)은 여기서 받아들이되, 도메인 규칙은 FSD 레이어로 넘긴다.
