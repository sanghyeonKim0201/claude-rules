---
description: 컴포넌트 조합 전략 — headless vs styled, sub-component, 래퍼 vs 훅
---

# 컴포지션 원칙

UI는 **조합으로 설계**한다. 거대한 prop 덩어리 컴포넌트 대신, 작은 조각의 조합으로 다양한 케이스를 커버한다.

## 조합 우선 (Composition over Configuration)

prop으로 분기하는 설계 대신, **자식·sub-component로 조합**하는 설계를 택한다.

```
// ❌ prop 난립 — variant·옵션이 늘어날수록 지옥
<Card
  title="..."
  subtitle="..."
  showHeader
  headerAction={<Button />}
  footer={<Button />}
  footerAlign="right"
/>

// ✅ 조합
<Card>
  <Card.Header>
    <Card.Title>...</Card.Title>
    <Card.Subtitle>...</Card.Subtitle>
    <Card.Action><Button /></Card.Action>
  </Card.Header>
  <Card.Body>...</Card.Body>
  <Card.Footer align="right"><Button /></Card.Footer>
</Card>
```

- 기본 레이아웃은 제공하되, **"이렇게만 써야 한다"고 강요 금지**.
- 필요하면 컴포지션 순서를 바꾸거나 일부만 쓸 수 있어야 한다.

## Sub-component 네이밍

- **점 표기법** (`Card.Header`) 또는 **네임스페이스 prefix** (`CardHeader`) 중 프로젝트 전반에 하나로 통일.
- sub-component는 반드시 **부모와 함께 써야 의미가 있는 것**만. 독립적으로 의미 있으면 별도 컴포넌트로 승격.
- sub-component도 독립적으로 export하여 사용자가 재배치·커스터마이즈 가능하게 한다.

## 플랫폼별 관용 — React는 compound, Flutter는 named slot

같은 "컴포지션" 원칙을 적용하되, **플랫폼의 구문 관용에 맞춘다**. 무리하게 통일하면 오히려 어색해진다.

### React — Compound component

JSX + Context + `React.Children` 덕분에 compound가 자연스럽다. **기본 패턴**.

```tsx
<Dialog>
  <DialogTrigger>열기</DialogTrigger>
  <DialogContent>
    <DialogHeader>
      <DialogTitle>제목</DialogTitle>
    </DialogHeader>
    <DialogBody>본문</DialogBody>
  </DialogContent>
</Dialog>
```

- 각 파트가 형제 위치로 자유 배치
- 상태는 Context로 공유 (`DialogContext`)
- 파트 중 일부만 선택적으로 사용 가능
- 복잡한 UI 구성에서도 읽기 편함

### Flutter — Named slot

Dart 문법상 중첩이 비싸서 compound보다 **생성자 slot**이 관용적.

```dart
ShUiDialog(
  header: ShUiDialogHeader(title: '제목'),
  body: Text('본문'),
  footer: Row(children: [ShUiButton(label: '확인')]),
)
```

- Flutter 표준 패턴 (`Scaffold(appBar:, body:, drawer:)` 등)과 일치
- 생성자 시그니처로 구조가 명확
- 상태 공유 필요 시 `InheritedWidget` + 생성자 slot 조합

### 하면 안 되는 것

- **React에서 named slot으로 prop 폭증**: `<Dialog header={...} footer={...} action={...}>` — compound 쓰자.
- **Flutter에서 JSX 흉내**: `ShUiDialog(child: Column(children: [ShUiDialogHeader(...), ShUiDialogBody(...)]))` — 정보량 대비 문법 비용이 너무 크다. named slot이 낫다.
- **플랫폼 API 강제 통일**: 두 플랫폼 API를 동일하게 맞추려고 하지 않는다. 사용자 경험은 각 플랫폼 관용에 맞출 때 더 좋다.

### 예외 — Flutter에서도 compound를 쓸 때

- **슬롯 배치에 사용자 자유도가 핵심 가치**인 경우 (예: Carousel의 prev/next/indicators 위치)
- **선택적 파트가 많고 named slot으로 담으면 생성자가 비대**해질 때
- 그때도 `Stack` / `Row` / `Column` 안에 자식들을 명시적으로 배치하는 형태가 된다 — 기꺼이 받아들이든지, 아니면 named slot으로 압축하든지 판단.

## Headless vs Styled

| 패턴 | 쓸 때 |
|---|---|
| **Headless** (동작만, 스타일 없음) | 스타일 완전 커스텀이 필요한 경우, 여러 제품에서 UI만 다르게 재사용 |
| **Styled** (동작 + 스타일) | 단일 제품의 디자인 시스템, 일관된 룩앤필 |
| **Hybrid** | 기본 스타일 제공 + 스타일 override 허용 |

- **디자인 시스템은 보통 styled**. 헤드리스는 라이브러리 사용자가 스타일을 처음부터 쓰는 비용이 크다.
- 헤드리스로 가면 **동작·접근성·상태 관리**에 집중, 스타일은 slot·className으로 외부화.

## 래퍼 컴포넌트 vs 훅

같은 기능을 제공하는 두 가지 방식. 트레이드오프:

| 방식 | 장점 | 단점 |
|---|---|---|
| **래퍼 컴포넌트** | 선언적, 쓰기 쉬움, API 명확 | 유연성 제한, API 고정 부담 |
| **훅** | 유연, UI와 분리, 커스텀 쉬움 | 쓰는 쪽에 보일러플레이트 |

### 선택 가이드

- **API가 아직 안정되지 않았다** → 훅 먼저.
- **사용 패턴이 반복 확정됐다** → 래퍼 컴포넌트.
- **둘 다 제공**이 이상적. 래퍼는 훅 위에 얇게 쌓는다.

```
// 훅 (재료)
const { activeId } = useActiveSection({ sectionIds });

// 래퍼 (완제품) — 내부에서 훅 사용
<ScrollSpy sectionIds={[...]}>
  {/* ... */}
</ScrollSpy>
```

- 래퍼 컴포넌트를 먼저 공개하면 **API가 고정**되어 되돌리기 비싸다. 훅부터 시작하는 편이 안전.

## Slot 패턴

- 특정 위치에 사용자가 원하는 엘리먼트를 꽂을 수 있도록 **slot prop**을 노출.
- `startIcon`, `endIcon`, `trailing`, `leading` 같은 명시적 slot prop 또는 sub-component 사용.
- children 하나로 모든 걸 해결하려 하면 구조가 불명확해진다.

## 제어 역전 (Inversion of Control)

- 컴포넌트는 **결정을 내리지 않고 사용자에게 넘긴다.**
- 렌더링 방식을 prop으로 받거나(`renderItem`), 자식으로 받아 사용자가 커스터마이즈 가능하게.

```
// ❌ 내부에서 전부 결정
<List items={items} />

// ✅ 렌더링은 사용자가
<List items={items} renderItem={(item) => <CustomItem {...item} />} />
// 또는
<List>
  {items.map(item => <List.Item key={item.id}>{item.label}</List.Item>)}
</List>
```

## 결합도 낮추기

- sub-component가 부모 context에 **암묵적으로 의존**하는 구조는 피한다.
  - 불가피하면 context 사용 시 **부모 없이 쓰면 명확한 에러**를 던진다.
- 컴포넌트끼리 **직접 import 금지**. 공통 타입·유틸은 별도 모듈로.

## 확장 vs 신규

- 기존 컴포넌트로 **2~3회 반복 변형**이 나오면 그때 prop 추가 또는 변형 컴포넌트 신설.
- 처음 등장한 변형에 대해 미리 prop을 열지 않는다 (AHA).

## 금지

- **전지적 컴포넌트**: prop 20개 이상, 다 처리하려고 하는 컴포넌트.
- **복제된 컴포넌트 군단**: `PrimaryButton`, `SecondaryButton`, `DangerButton` — variant prop으로 통합.
- **내부 구현 노출**: CSS 클래스명·DOM 구조를 prop으로 받는 컴포넌트.
- **강제된 구조**: sub-component 순서·존재를 강제. (예: `Card.Header` 없으면 에러)
- **prop → prop 전달 체인**: 3단계 이상 prop drilling은 context 또는 구조 재검토.
