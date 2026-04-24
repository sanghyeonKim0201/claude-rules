---
description: superpowers(Claude Code 플러그인) 사용 규칙 — 스킬 카탈로그와 호출 트리거, gstack과의 역할 분담
---

# superpowers 사용 규칙

**모든 답변은 한국어로 작성하세요.**

superpowers는 Claude Code에 "프로세스 규율"을 주입하는 스킬 팩이다. 브레인스토밍·플랜 작성·TDD·체계적 디버깅·코드 리뷰 요청 같은 **개발 흐름의 규율**을 스킬로 강제한다. 이 규칙은 **어떤 상황에서 어떤 스킬이 후보가 되는지**를 정한다.

이 문서는 superpowers를 쓰는 Claude Code 환경에만 적용한다. gstack을 함께 쓰는 경우의 역할 분담은 아래 별도 섹션 참조.

## 핵심 원칙

- **규칙은 플로우가 아니라 신호다.** 순서대로 발동시키지 말고, 상황에 맞는 스킬만 호출한다. 프로젝트 성격·작업 규모에 맞게 유동적으로 판단.
- **스킬 자동 호출을 거스르지 않는다.** superpowers 플러그인은 "1% 규칙"(조금이라도 관련되면 스킬 호출)을 내장. 자동 발동은 그대로 따르고, 이 규칙은 주로 "어떤 스킬을 언제 직접 고를지"를 정한다.
- **작은 변경에는 규율 스킬을 적용하지 않는다.** 답이 이미 명확한 작업에는 `brainstorming`·`writing-plans`·TDD를 강제하지 않는다. 토큰·시간 대비 실익 없음. "제외 조건" 섹션 참조.
- **플랜 파이프라인은 한 번에 하나만.** 같은 작업에 `writing-plans`와 (존재한다면) 외부 플랜 커맨드(`/autoplan` 등)를 둘 다 돌리지 않는다.
- **오버헤드는 조건부로 쓴다.** `using-git-worktrees`는 큰 변경 + 현재 상태 보존 필요일 때만.

## 호출 트리거 (신호 기반)

### 기획 / 계획

| 신호 | 후보 스킬 |
|---|---|
| 기능·컴포넌트·동작 변경을 만들기 전. 요구사항·설계 탐색이 필요 | `brainstorming` |
| 스펙은 있고 멀티 스텝 작업의 계획서가 필요 | `writing-plans` |
| 플랜을 **별도 세션에서** 리뷰 체크포인트와 함께 실행할 예정 | `executing-plans` |
| 플랜의 독립적 태스크들을 **현재 세션에서** 서브에이전트에 병렬 위임 | `subagent-driven-development` |
| 공유 상태·순차 의존성이 없는 **2개 이상 독립 작업**을 병렬화 | `dispatching-parallel-agents` |

### 구현 / 검증

| 신호 | 후보 스킬 |
|---|---|
| 기능·버그픽스 구현 전. 구현보다 테스트를 먼저 작성 | `test-driven-development` |
| 버그·테스트 실패·예상 밖 동작. **수정 제안 전** 근본 원인 파악 | `systematic-debugging` |
| "완료했다"·"고쳤다"·"통과한다"고 주장하기 전 + 커밋·PR 생성 전 | `verification-before-completion` |

### 브랜치 / 작업 공간

| 신호 | 후보 스킬 |
|---|---|
| 대형 리팩터·플랜 실행·실험 브랜치가 현재 워크스페이스와 격리 필요 | `using-git-worktrees` |
| 구현 완료 + 테스트 통과 후 머지·PR·정리 방식 결정 | `finishing-a-development-branch` |

### 리뷰

| 신호 | 후보 스킬 |
|---|---|
| 태스크 완료·주요 기능 구현·머지 직전. 동료 관점 검증 요청 | `requesting-code-review` |
| 코드 리뷰 피드백을 받은 직후. 제안 검증·기술적 의심 해소 | `receiving-code-review` |

### 메타

| 신호 | 후보 스킬 |
|---|---|
| 새 스킬 생성·기존 스킬 편집·스킬 동작 검증 | `writing-skills` |

## 프로젝트 유형별 적용 범위

대부분의 스킬은 프로젝트 유형과 무관하게 적용된다. 예외만 정리하면:

| 프로젝트 유형 | 참고 |
|---|---|
| 웹 / 모바일 / UI 프로덕트 | `verification-before-completion`은 실제 사용자 흐름 확인까지 포함. Visual QA는 superpowers로 못 하므로 gstack(`/qa`·`/design-review`) 또는 수동 확인이 필요 |
| 라이브러리 / SDK / CLI | `verification-before-completion`은 주로 테스트·타입체크·빌드 실행 결과로 충족 |
| 백엔드 / 서비스 | 위와 동일. 배포 시스템 검증은 host에 따라 gstack 또는 수동 |
| 데이터 파이프라인 / ML | TDD가 어려운 경우(모델 학습·대용량 데이터)엔 `verification-before-completion`을 작은 재현 샘플 실행으로 대체 |

## 제외 조건 (무조건 호출 안 함)

아래 중 하나라도 해당하면 규율 스킬을 쓰지 않는다.

- 답이 이미 명확한 작은 변경 (타이포·config·문서 정정·명확한 버그픽스·스타일 정리)
- 1파일·10줄 이하의 변경 (프로젝트별로 조정 가능한 기본값)
- 사용자가 명시적으로 "가볍게 해" / "스킬 없이" / "직접 해" 라고 요청
- 플러그인이 이미 자동 호출한 스킬을 수동으로 중복 호출

## gstack과 함께 쓸 때 역할 분담

superpowers와 gstack을 같이 쓰는 환경(Claude Code + gstack host)에서만 적용한다. 하나만 쓰면 이 섹션은 건너뛴다.

| 영역 | superpowers | gstack | 선택 기준 |
|---|---|---|---|
| Think (제품 가정) | — | `/office-hours` | gstack 전담 (제품 검증 성격) |
| Think (요구사항·설계) | `brainstorming` | — | superpowers 전담 |
| Plan | `writing-plans` | `/autoplan` 등 | 문서 산출물·세션 분리 실행이 목적이면 superpowers. 빠른 멀티-리뷰가 목적이면 gstack |
| Build | `executing-plans`, `TDD`, `subagent-driven-development` | — | superpowers 전담 (gstack엔 구현 규율 없음) |
| Debug | `systematic-debugging` | `/investigate` | **`systematic-debugging` 기본값.** 4-phase 구조화 리포트가 명시적으로 필요할 때만 `/investigate` |
| Review | `requesting-code-review`, `verification-before-completion` | `/review`, `/codex` | 자체 검증·동료 관점은 superpowers. 외부 모델 교차 검증·자동 수정은 gstack |
| Visual QA | — | `/qa`, `/design-review` | gstack 전담 (UI 프로덕트에만 해당). gstack이 없으면 수동 확인 |
| Ship | `finishing-a-development-branch` | `/ship`, `/land-and-deploy`, `/canary` | 머지·배포는 gstack. 브랜치 정리 결정만 superpowers |

### 기획 단계 분기 (`/office-hours` vs `brainstorming`)

두 스킬은 역할이 다르므로 상황에 따라 고른다. 체인이 아니라 **필요한 것만** 쓴다.

| 상황 | 쓰는 것 |
|---|---|
| 새 제품 · 모호한 아이디어 · "이걸 만들어야 하나" 검증 필요 | `/office-hours` 먼저, 이후 필요 시 `brainstorming` |
| 명확한 기능 추가 · 요구사항은 있으나 설계 탐색 필요 | `brainstorming`만 (`/office-hours` 생략) |
| 작은 변경 | 둘 다 생략 |

- `/office-hours` = **"이걸 만들어야 하나?"** (제품 검증)
- `brainstorming` = **"정확히 뭘 만들지?"** (요구사항·설계 탐색)

## 주의사항

- **코어 규칙(common / nextjs / fsd / ui)이 우선.** superpowers는 프로세스 도구이지 코드 컨벤션을 대체하지 않는다. 커밋 메시지·네이밍·아키텍처는 해당 규칙을 따른다.
- **`verification-before-completion`을 생략하고 "완료"를 주장하지 않는다.** 타입 체크·테스트·빌드 실행 결과를 증거로 제시한 뒤에만 성공을 선언한다. UI 변경이면 실제 동작 확인까지.
- **서브에이전트는 공유 상태가 없을 때만 병렬로 보낸다.** 순차 의존성이 있으면 오히려 느려진다.
- **Visual QA는 superpowers로 못한다.** 배포된 화면·인터랙션 검증은 gstack `/qa`·`/design-review`를 쓴다. gstack이 없는 환경이면 수동 확인.
