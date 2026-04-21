---
description: gstack(Garry Tan의 Claude Code 스킬 팩) 사용 규칙 — 커맨드 카탈로그와 사용 시기
---

# gstack 사용 규칙

**모든 답변은 한국어로 작성하세요.**

gstack은 Claude Code를 "가상 엔지니어링 팀"으로 만드는 슬래시 커맨드 모음이다. CEO·디자이너·엔지니어링 매니저·리뷰어·QA 리드·릴리즈 엔지니어 역할이 각자 스킬로 존재한다. 이 규칙은 **언제 어떤 커맨드를 쓸지**를 정한다.

## 핵심 원칙

- **gstack은 프로세스다.** 단발성 툴이 아니라 `Think → Plan → Build → Review → Test → Ship → Reflect` 스프린트 흐름에서 이어서 쓴다. 각 단계의 산출물이 다음 단계의 입력이 된다.
- **브라우저 작업은 항상 `/browse`로 한다.** `mcp__claude-in-chrome__*` 도구를 직접 호출하지 않는다.
- **질문 폭주를 피하려면 `/autoplan`을 우선 고려한다.** 개별 `/plan-*-review`를 순차 실행하는 대신 자동 파이프라인이 taste decision만 묻는다.
- **한 번에 한 커맨드만 실행한다.** 커맨드는 상호작용형이고 사이드이펙트가 있다. 파이프라이닝 금지.
- **안전이 필요한 상황에서는 먼저 `/guard`로 감싼다** (prod 작업, 라이브 시스템 디버깅, 마이그레이션 등).

## 스프린트 단계별 커맨드

| 단계 | 기본 커맨드 | 목적 |
|---|---|---|
| Think | `/office-hours` | 제품 가정을 깨뜨리고 진짜 문제 재정의 |
| Plan | `/autoplan` 또는 `/plan-ceo-review` + `/plan-eng-review` | 스코프·아키텍처 락인 |
| Build | (일반 코딩) | 플랜 승인 후 구현 |
| Review | `/review`, `/codex`(2nd opinion) | 코드 리뷰 + 자동 수정 |
| Test | `/qa`, `/qa-only` | 실제 브라우저로 기능 검증 |
| Ship | `/ship` → `/land-and-deploy` → `/canary` | PR 생성·머지·배포·헬스체크 |
| Reflect | `/retro`, `/document-release`, `/learn` | 회고·문서 동기화·학습 기록 |

## 커맨드 카탈로그

### 기획 / 전략

| 커맨드 | 사용 시기 |
|---|---|
| `/office-hours` | 새 아이디어·새 기능을 구상할 때. 코드 쓰기 전 6가지 강제 질문으로 프레이밍 재검토. 산출물은 design doc. |
| `/plan-ceo-review` | 플랜이 있는데 "정말 이게 베스트냐"를 따져야 할 때. 4가지 스코프 모드(확장/선택적 확장/고정/축소). |

### 플랜 리뷰 (구현 전)

| 커맨드 | 대상 | 사용 시기 |
|---|---|---|
| `/autoplan` | 전부 자동 | **기본값.** CEO→디자인→엔지니어링→DX 리뷰를 자동 실행. 질문은 taste decision만 노출. |
| `/plan-eng-review` | 아키텍처·데이터 플로우 | 복잡한 백엔드·상태 머신·성능 이슈 있는 플랜. |
| `/plan-design-review` | UI·UX 플랜 | 사용자 대면 화면이 있는 플랜. |
| `/plan-devex-review` | API·CLI·SDK·문서 | 개발자 대면 인터페이스가 있는 플랜. |

**Plan-Review 3종 선택 기준:**
- End user 대상(UI, 웹앱, 모바일) → `/plan-design-review` → (배포 후) `/design-review`
- 개발자 대상(API, CLI, SDK) → `/plan-devex-review` → (배포 후) `/devex-review`
- 아키텍처·성능·테스트 → `/plan-eng-review` → (배포 후) `/review`
- 전부 해당 → `/autoplan`

### 디자인

| 커맨드 | 사용 시기 |
|---|---|
| `/design-consultation` | 프로젝트에 디자인 시스템이 없을 때. `DESIGN.md` 생성. |
| `/design-shotgun` | "어떻게 생겨야 할지" 감이 없을 때. 4–6개 변형 시안 비교 + 피드백 반복. |
| `/design-html` | 승인된 목업을 실제 프로덕션 HTML/CSS로 변환. |
| `/design-review` | 배포된 화면 시각 QA + 자동 수정. (플랜 단계는 `/plan-design-review`) |

### 리뷰 / 디버깅 / 보안

| 커맨드 | 사용 시기 |
|---|---|
| `/review` | 머지 직전 PR 리뷰. SQL 안전성·LLM 트러스트 경계·조건부 사이드이펙트 체크. 자동 수정 가능한 건 처리. |
| `/codex` | `/review` 이후 **다른 모델(OpenAI Codex)** 의 독립 리뷰. 교차 검증·적대적 챌린지·오픈 상담 3모드. |
| `/investigate` | 원인 불명 버그·"어제까지 됐는데" 이슈. 근본 원인 찾기 전까지 수정 금지(Iron Law). |
| `/cso` | 보안 감사. OWASP Top 10 + STRIDE 위협 모델. daily / comprehensive 모드. |

### QA / 브라우저

| 커맨드 | 사용 시기 |
|---|---|
| `/qa` | 기능 배포 전·후 실제 브라우저로 플로우 테스트 + 버그 발견 시 커밋 단위로 자동 수정·회귀 테스트 생성. |
| `/qa-only` | 수정 없이 버그 리포트만 필요할 때. |
| `/browse` | 임의의 URL을 열거나 스크린샷·상호작용·DOM 확인. **모든 브라우저 작업의 기본값.** |
| `/devex-review` | 배포된 개발자 대면 제품(문서·CLI·API)을 실제로 사용해 보며 TTHW·에러 메시지 평가. |
| `/setup-browser-cookies` | 로그인이 필요한 페이지 QA 전 Chrome/Arc/Brave 쿠키 가져오기. |

### 릴리즈 / 운영

| 커맨드 | 사용 시기 |
|---|---|
| `/ship` | 코드 완성 후. 테스트 실행·커버리지 감사·버전 범프·`CHANGELOG` 갱신·PR 생성까지. |
| `/land-and-deploy` | PR 승인 후 머지·CI·배포·프로덕션 헬스 검증. `/setup-deploy` 선행 필요. |
| `/canary` | 배포 직후 모니터링 루프. 콘솔 에러·성능 리그레션·페이지 실패 감시. |
| `/benchmark` | 페이지 로드·Core Web Vitals·리소스 크기 기준선 설정. PR 전후 비교. |
| `/document-release` | 배포 후 README·ARCHITECTURE·CONTRIBUTING·CLAUDE.md·CHANGELOG 동기화. |
| `/setup-deploy` | `/land-and-deploy` 최초 1회 설정. 배포 플랫폼·헬스체크 엔드포인트 탐지. |

### 안전장치

| 커맨드 | 사용 시기 |
|---|---|
| `/careful` | `rm -rf`, `DROP TABLE`, force-push, `git reset --hard` 등 파괴적 명령 경고. prod 건드릴 때. |
| `/freeze` | 편집 범위를 특정 디렉터리로 제한. 디버깅 중 의도치 않은 광범위 수정 차단. |
| `/guard` | `/careful` + `/freeze` 동시 활성. **prod·라이브 시스템 작업의 기본값.** |
| `/unfreeze` | `/freeze` 해제. |

### 회고 / 메타

| 커맨드 | 사용 시기 |
|---|---|
| `/retro` | 주간 엔지니어링 회고. 커밋 히스토리·시프 퀄리티·트렌드. `retro global`은 모든 프로젝트 교차 분석. |
| `/learn` | 세션 간 학습 내용 관리. 검색·프루닝·익스포트. |
| `/context-save` / `/context-restore` | 작업 도중 세션을 떠나야 할 때. 다른 세션·워크트리에서 이어서. |
| `/gstack-upgrade` | gstack 버전 업데이트. 글로벌 vs vendored 자동 감지. |

### 파워 툴 / 기타

| 커맨드 | 사용 시기 |
|---|---|
| `/pair-agent` | 다른 AI 에이전트(OpenClaw, Hermes, Codex 등)와 브라우저 공유. 각자 자기 탭. |
| `/open-gstack-browser` | 헤디드 Chromium이 필요할 때(봇 차단 사이트, 사이드바 디버깅). |
| `/benchmark-models` | Claude vs GPT vs Gemini를 같은 프롬프트로 비교. 스킬별 최적 모델 결정. |

## 표준 플로우

**신규 기능 (0 → 1):**
```
/office-hours → /autoplan → (승인) → 구현 → /review → /qa → /ship → /land-and-deploy → /canary → /document-release
```

**기존 기능 개선 (UI):**
```
/plan-design-review → 구현 → /design-review → /review → /qa → /ship
```

**버그 수정:**
```
/investigate → 구현 → /review → /qa → /ship
```

**디자인 탐색부터 구현까지:**
```
/design-consultation (최초 1회) → /design-shotgun → /design-html → /qa → /ship
```

**보안 감사:**
```
/cso (daily 또는 comprehensive) → /investigate (발견된 이슈) → 수정 → /review → /ship
```

**릴리즈:**
```
/ship → /land-and-deploy → /canary → /document-release → /retro (주간)
```

## 주의사항

- **기존 프로젝트 규칙(common / nextjs / fsd / ui)이 우선.** gstack은 워크플로우 도구이지 코드 컨벤션을 대체하지 않는다. `/ship`이 만드는 커밋 메시지가 이 레포의 Git 컨벤션(`feat`, `fix`, `docs` 등)을 따르는지 확인한다.
- **버전 범프 규칙은 `common`의 버전 관리 규칙을 따른다.** `/ship`의 기본 동작이 이 규칙과 충돌하면 레포 규칙을 우선한다.
- **`/autoplan`이 없는 플랜에 `/plan-*-review`를 겹쳐 쓰지 않는다.** 중복 질문과 모순된 권고가 나온다. 하나만 고른다.
- **`/qa`와 `/qa-only`는 다르다.** 수정 권한이 없는 브랜치(main, 남의 PR)에서는 `/qa-only`를 쓴다.
- **`/freeze` 활성 상태를 잊지 않는다.** 편집이 차단될 때 `/unfreeze`를 먼저 확인.
- **`/codex`는 `/review` 이후에 쓴다.** 순서를 바꾸면 교차 분석 모드가 활성화되지 않는다.
- **프로덕션에 영향을 주는 커맨드(`/land-and-deploy`, force-push 등) 실행 전에는 사용자 확인을 받는다.** `/guard`가 켜져 있더라도 최종 승인은 사람이 내린다.
- **하네스 설정(`.claude/settings.json`, hooks, custom skill 등)을 변경할 때는 `common`의 외부 도구 사용 전략을 따른다.** 팀 공유 설정은 `settings.json`, 개인/로컬 설정은 `settings.local.json`에 두고 후자는 커밋하지 않는다.
- **gstack 플로우(`/autoplan`, `/plan-*-review`, `/review`, `/qa` 등)는 설계 결정이 필요한 작업에 쓴다.** 답이 이미 명확한 작은 변경(타이포, config 수정, 단순 버그픽스, 문서 정정, 스타일 정리 등)에는 쓰지 않는다 — 토큰 대비 실익이 없다.
