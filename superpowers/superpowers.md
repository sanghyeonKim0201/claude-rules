---
description: superpowers(Claude Code 플러그인) 사용 규칙 — 스킬 카탈로그와 gstack과의 역할 분담
---

# superpowers 사용 규칙

**모든 답변은 한국어로 작성하세요.**

superpowers는 Claude Code에 "프로세스 규율"을 주입하는 스킬 팩이다. 브레인스토밍·플랜 작성·TDD·체계적 디버깅·코드 리뷰 요청 같은 **개발 흐름의 규율**을 스킬로 강제한다. 이 규칙은 **언제 어떤 스킬을 쓸지**와 **gstack과 어떻게 역할을 나눌지**를 정한다.

이 문서는 superpowers를 쓰는 Claude Code 환경에만 적용한다. gstack은 여러 AI 코딩 에이전트 host에서 독립적으로 사용할 수 있으며, 아래 역할 분담은 **Claude Code에서 superpowers와 gstack을 함께 사용할 때**의 기준이다.

## 핵심 원칙

- **superpowers는 규율이고 gstack은 역할이다.** superpowers는 Plan/Build/Debug의 **프로세스**를 잡고, gstack은 CEO·디자이너·QA 같은 **가상 팀 역할**을 제공한다. 둘을 스프린트 단계별로 나눠서 쓴다.
- **스킬 자동 호출을 거스르지 않는다.** superpowers 플러그인은 "1% 규칙"(조금이라도 관련되면 스킬 호출)을 내장하고 있다. 플러그인이 강제하는 동작(스킬 자동 발동, Skill 도구 호출 의무)은 이 규칙에서 반복 기술하지 않는다.
- **작은 변경에는 규율 스킬을 적용하지 않는다.** 타이포·config 수정·문서 정정·단순 버그픽스 등 답이 이미 명확한 작업에는 `brainstorming`·`writing-plans`·TDD를 강제하지 않는다. 토큰·시간 대비 실익이 없다.
- **플랜 파이프라인은 한 번에 하나만.** gstack `/autoplan`과 superpowers `writing-plans`를 같은 작업에 **둘 다** 돌리지 않는다. 하나만 선택한다.
- **`using-git-worktrees`는 오버헤드가 있다.** "큰 변경 + 현재 워크스페이스 보존 필요"일 때만 쓴다. 일반 수정엔 불필요.

## gstack ↔ superpowers 역할 분담

| 스프린트 단계 | gstack | superpowers | 우선순위 / 선택 기준 |
|---|---|---|---|
| Think | `/office-hours` | `brainstorming` | 제품 가정 검증은 `/office-hours`, 요구사항·설계 탐색은 `brainstorming`. 역할이 다름 (아래 "기획 단계 분기" 참조) |
| Plan | `/autoplan` | `writing-plans` | 플랜 문서 산출물이 필요하거나 세션을 분리해서 실행할 거면 `writing-plans`. 빠른 리뷰가 목적이면 `/autoplan` |
| Build | (없음) | `executing-plans`, `subagent-driven-development`, `using-git-worktrees`, `test-driven-development` | **superpowers 전담.** gstack엔 구현 규율 도구 없음 |
| Debug | `/investigate` | `systematic-debugging` | **`systematic-debugging` 기본값** (자동 호출). gstack 4-phase 리포트가 명시적으로 필요할 때만 `/investigate` |
| Review | `/review`, `/codex` | `requesting-code-review`, `verification-before-completion` | 자체 검증·동료 관점 요청은 superpowers, 외부 모델 교차 검증·자동 수정은 gstack |
| Visual QA | `/qa`, `/design-review` | (없음) | **gstack 전담.** superpowers엔 브라우저 QA 없음 |
| Ship | `/ship`, `/land-and-deploy`, `/canary` | `finishing-a-development-branch` | 머지·배포는 gstack, 브랜치 정리 결정은 `finishing-a-development-branch` |

## 기획 단계 분기 (`/office-hours` vs `brainstorming`)

두 스킬은 역할이 다르므로 상황에 따라 선택한다.

| 상황 | 사용 흐름 |
|---|---|
| 새 제품 · 모호한 아이디어 · "이걸 만들어야 하나" 검증 필요 | `/office-hours` → `brainstorming` → `writing-plans` |
| 명확한 기능 추가 · 요구사항은 있으나 설계 탐색 필요 | `brainstorming` → `writing-plans` (`/office-hours` 생략) |
| 작은 변경 (타이포 · config · 단순 버그픽스) | 둘 다 생략 |

- `/office-hours` = **"이걸 만들어야 하나?"** (제품 검증, YC식 6가지 강제 질문)
- `brainstorming` = **"정확히 뭘 만들지?"** (요구사항·설계 탐색)

## 스킬 카탈로그

### 기획 / 계획

| 스킬 | 사용 시기 |
|---|---|
| `brainstorming` | 기능·컴포넌트·동작 변경을 만들기 전. 사용자 의도·요구사항·설계를 먼저 정리. **창의적 작업 전 필수.** |
| `writing-plans` | 스펙·요구사항이 있고, 멀티 스텝 작업을 코드 건드리기 전에 계획할 때. 산출물은 플랜 문서. |
| `executing-plans` | `writing-plans`로 만든 플랜을 **별도 세션에서** 리뷰 체크포인트와 함께 실행할 때. |
| `subagent-driven-development` | 플랜의 독립적인 태스크들을 **현재 세션에서** 서브에이전트에게 병렬로 시킬 때. |
| `dispatching-parallel-agents` | 공유 상태·순차 의존성이 없는 **2개 이상의 독립 작업**을 병렬화할 때. |

### 구현 / 검증

| 스킬 | 사용 시기 |
|---|---|
| `test-driven-development` | 기능·버그픽스 구현 전. 구현 코드 쓰기 전에 테스트부터 작성. |
| `systematic-debugging` | 버그·테스트 실패·예상 밖 동작. **수정 제안 전에** 반드시 사용. 근본 원인 파악 전 수정 금지. |
| `verification-before-completion` | "완료했다"·"고쳤다"·"통과한다"고 주장하기 전. 커밋·PR 생성 전. 증거(명령 실행 결과) 없이 성공 주장 금지. |

### 브랜치 / 작업 공간

| 스킬 | 사용 시기 |
|---|---|
| `using-git-worktrees` | 현재 워크스페이스와 **격리**된 작업이 필요할 때 (대형 리팩터·플랜 실행·실험 브랜치). 일반 수정엔 오버헤드. |
| `finishing-a-development-branch` | 구현 완료 + 테스트 통과 후. 머지·PR·정리 방식을 구조화된 옵션으로 결정. |

### 리뷰

| 스킬 | 사용 시기 |
|---|---|
| `requesting-code-review` | 태스크 완료·주요 기능 구현·머지 직전. 요구사항 충족 여부를 동료 관점으로 검증 요청. |
| `receiving-code-review` | 코드 리뷰 피드백을 받았을 때. 제안이 불명확하거나 기술적으로 의심스러우면 맹목 수용 금지, 검증 우선. |

### 메타

| 스킬 | 사용 시기 |
|---|---|
| `writing-skills` | 새 스킬 생성·기존 스킬 편집·스킬이 의도대로 동작하는지 검증. |
| `using-superpowers` | 세션 시작 시 스킬 발견·사용 방식을 확립. 플러그인이 자동 호출. |

## 표준 플로우

**신규 기능 (0 → 1, 모호한 아이디어):**
```
/office-hours → brainstorming → writing-plans
→ using-git-worktrees → executing-plans (+ TDD, subagent-driven-development)
→ verification-before-completion → requesting-code-review
→ /review → /codex → /qa → /ship → /land-and-deploy → /canary
```

**신규 기능 (요구사항 명확):**
```
brainstorming → writing-plans
→ executing-plans (+ TDD)
→ verification-before-completion → /review → /qa → /ship
```

**기존 기능 개선 (UI):**
```
brainstorming → /plan-design-review (또는 writing-plans)
→ 구현 (+ TDD) → /design-review → /review → /qa → /ship
```

**버그 수정:**
```
systematic-debugging → (TDD로 재현 테스트) → 수정
→ verification-before-completion → /review → /qa → /ship
```

**병렬 작업 (독립 태스크 다수):**
```
writing-plans → dispatching-parallel-agents
→ 각 태스크별로 verification-before-completion
→ finishing-a-development-branch → /ship
```

## 주의사항

- **기존 프로젝트 규칙(common / nextjs / fsd / ui)이 우선.** superpowers는 프로세스 도구이지 코드 컨벤션을 대체하지 않는다. 커밋 메시지·네이밍·아키텍처는 해당 규칙을 따른다.
- **플러그인이 자동 호출하는 스킬을 수동으로 중복 호출하지 않는다.** `using-superpowers`처럼 세션 시작 시 자동 발동되는 건 사용자가 다시 `/using-superpowers` 같이 호출할 필요 없음.
- **`writing-plans`와 `/autoplan`은 겹치지 않게 쓴다.** 같은 작업에 둘 다 돌리면 질문 중복·모순된 권고가 발생한다. 하나만 선택.
- **`systematic-debugging`과 `/investigate`는 둘 중 하나만.** 프로젝트에서 기본값을 `systematic-debugging`으로 고정. gstack 특유의 4-phase 리포트가 **명시적으로** 필요할 때만 `/investigate`로 전환.
- **`using-git-worktrees`는 선택적이다.** 작은 수정엔 불필요한 오버헤드. "큰 변경 + 현재 상태 보존 필요" 조건일 때만.
- **`verification-before-completion`을 생략하고 "완료"를 주장하지 않는다.** 타입 체크·테스트·빌드 실행 결과를 증거로 제시한 뒤에만 성공을 선언한다. UI 변경은 실제 브라우저 확인까지.
- **`brainstorming`·`writing-plans`를 작은 변경에 강제하지 않는다.** 타이포·config·단순 버그픽스엔 과한 오버헤드. 판단 기준이 모호하면 "수정 규모가 1파일·10줄 이하"를 제외 기준으로 삼는다.
- **서브에이전트는 공유 상태가 없을 때만 병렬로 보낸다.** `dispatching-parallel-agents`·`subagent-driven-development`는 태스크 간 의존성이 없을 때 효과. 순차 작업에 쓰면 오히려 느려진다.
- **Visual QA는 superpowers로 못한다.** 배포된 화면·인터랙션 검증은 `/qa`·`/design-review` 같은 gstack 커맨드를 쓴다.
