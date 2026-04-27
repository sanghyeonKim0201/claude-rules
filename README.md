# AI Rules

AI 코딩 에이전트가 따라야 할 공유 코딩 규칙 모음. Claude Code, Codex, Cursor, Gemini CLI, Windsurf 등 특정 모델·제품에 종속되지 않는 **프로젝트 규칙 저장소**로 사용한다.

모든 규칙은 **하나의 저장소**에서 관리하며, 프로젝트에 필요한 규칙만 **sparse-checkout**으로 선택해 가져간다. 에이전트별 설정 파일은 이 저장소의 규칙을 읽어 오거나 참조하는 얇은 어댑터로 둔다.

## 규칙 구성

| 폴더 | 설명 | 포함 파일 |
|---|---|---|
| [`common/`](./common) | 언어/프레임워크 무관 공통 규칙 | Git 컨벤션, 설계 원칙, 네이밍, 순수 함수 분리 |
| [`nextjs/`](./nextjs) | Next.js / React / TypeScript 규칙 | 네이밍, 설계 원칙, 데이터 페칭, 관심사 분리, FSD-Next.js 통합 |
| [`fsd/`](./fsd) | FSD 아키텍처 전용 규칙 | 레이어 구조, Public API, Import, 관심사 분리 |
| [`ui/`](./ui) | UI/UX 설계 원칙 (프레임워크 무관) | 컴포넌트 API, 접근성, UI 상태, 디자인 토큰, 컴포지션 |
| [`gstack/`](./gstack) | gstack 워크플로우 스킬/커맨드 사용 규칙 (지원 AI 에이전트 공통 선택 규칙) | 커맨드 카탈로그, 호출 트리거(신호 기반), 프로젝트 유형별 적용 범위 |
| [`templates/`](./templates) | 프로젝트 엔트리포인트 템플릿 | `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, `copilot-instructions.md` |

## 조합 원칙

- `common`은 거의 모든 프로젝트에 기본 적용.
- `fsd`는 아키텍처 규칙이므로 언어/프레임워크와 독립적으로 조합 가능.
- `nextjs`는 Next.js/React/TypeScript 프로젝트에서만 사용.
- `ui`는 디자인 시스템·UI 컴포넌트 작업이 있는 프로젝트에서 사용 (프레임워크 무관).
- `gstack`은 로컬 머신에 gstack이 설치돼 있고 현재 사용하는 AI 코딩 에이전트가 gstack host로 지원될 때 추가한다. 코어 규칙이 아니라 도구 실행 환경용 워크플로우 어댑터다.
- 새 에이전트 전용 규칙이 필요하면 기존 코어 폴더를 수정하지 말고 별도 폴더로 분리한다. 예: `codex/`, `cursor/`, `gemini/`.

## 프로젝트 유형별 조합

| 프로젝트 유형 | 폴더 조합 |
|---|---|
| Next.js + FSD | `common` + `nextjs` + `fsd` |
| Next.js (FSD 미사용) | `common` + `nextjs` |
| Next.js 디자인 시스템 | `common` + `nextjs` + `ui` |
| Spring Boot + FSD | `common` + `fsd` + (향후 spring) |
| Flutter + FSD | `common` + `fsd` + (향후 flutter) |
| 비 Next.js / 비 FSD | `common` + (해당 프레임워크 규칙) |
| 특정 에이전트 워크플로우 사용 | (기존 조합) + 해당 에이전트 전용 폴더 |
| gstack 워크플로우 사용 | (기존 조합) + `gstack` |

## 에이전트 중립 원칙

- 코어 규칙(`common`, `nextjs`, `fsd`, `ui`)에는 특정 모델명, 제품명, 전용 명령어를 넣지 않는다.
- 특정 에이전트에서만 동작하는 명령어·훅·플러그인 규칙은 별도 폴더에 둔다.
- 여러 에이전트에서 동작하는 도구라도 host별 설치 경로·명령 호출 방식이 다르면 본문에서 지원 범위를 명시한다.
- 같은 프로젝트에서 여러 AI 도구를 써도 동일한 코어 규칙을 공유한다.
- 에이전트별 설정 파일은 코어 규칙을 복사하거나 요약하지 말고 가능한 한 이 저장소의 파일을 참조한다.
- 저장소 경로는 관례일 뿐이다. `.ai/rules`, `.claude/rules`, `.cursor/rules`, `.codex/rules` 등 팀의 도구 체인에 맞게 정한다.

## 설정 가이드

### 1. 프로젝트에 규칙 추가 (필요한 폴더만)

기본 경로는 `.ai/rules`로 통일한다. Claude Code, Codex, Cursor, Gemini CLI 등 어떤 에이전트를 쓰든 동일한 경로를 사용해 규칙을 한 곳에서 관리한다.

```bash
cd <프로젝트 루트>

# 1) 서브모듈 추가 (전체 monorepo를 논리적으로 연결)
git submodule add <rules-repo-url> .ai/rules

# 2) 템플릿 복사 (sparse-checkout 적용 전, templates가 아직 보일 때 한 번만)
cp .ai/rules/templates/CLAUDE.md CLAUDE.md
cp .ai/rules/templates/AGENTS.md AGENTS.md
cp .ai/rules/templates/GEMINI.md GEMINI.md
mkdir -p .github
cp .ai/rules/templates/copilot-instructions.md .github/copilot-instructions.md

# 3) sparse-checkout으로 실제 사용할 규칙 폴더만 남긴다 (templates는 자동 제외)
cd .ai/rules
git sparse-checkout init --cone
git sparse-checkout set common nextjs        # 예: Next.js 프로젝트
cd -

# 4) 커밋
git add .gitmodules .ai/rules CLAUDE.md AGENTS.md GEMINI.md .github/copilot-instructions.md
git commit -m "chore: AI 규칙 서브모듈 추가 (common/nextjs)"
```

결과:
```
.ai/rules/
├── common/
└── nextjs/
```

`fsd`, `ui`, `templates`, 기타 폴더는 원격에 존재하지만 **이 프로젝트엔 남지 않는다.** `templates`는 최초 setup에서 한 번 복사하고 sparse-checkout으로 자동 제외된다.

기존에 `.claude/rules` 등 도구별 경로를 쓰고 있었다면 `.ai/rules`로 마이그레이션한다. 도구별 경로가 꼭 필요한 경우에는 실제 파일을 복사하지 말고 심볼릭 링크로 `.ai/rules`를 가리키게 한다.

```bash
# 예: Claude Code가 .claude/rules 경로를 별도로 요구하는 경우
ln -s ../.ai/rules .claude/rules
```

### 2. 프로젝트 루트 엔트리포인트 연결

서브모듈만 추가하면 AI 도구가 규칙을 자동으로 읽는다는 보장이 없다. 프로젝트 루트 또는 도구별 설정 경로에 각 도구가 읽는 엔트리포인트 파일을 만들고, 실제 규칙 파일 링크를 명시한다.

아래 목록은 대표 관례이며, 도구 버전에 따라 달라질 수 있으므로 팀에서 쓰는 도구의 공식 문서를 우선한다.

| 도구 | 권장 파일 |
|---|---|
| 여러 AI 에이전트 공통 | `AGENTS.md` |
| Claude Code | `CLAUDE.md`, `CLAUDE.local.md` (본문은 `.ai/rules/*` 링크로 구성) |
| Codex | `AGENTS.md` |
| Gemini CLI | `GEMINI.md` 또는 설정한 context file name |
| GitHub Copilot | `.github/copilot-instructions.md`, `.github/instructions/*.instructions.md`, `AGENTS.md` |
| Cursor | `AGENTS.md`, `.cursor/rules/*.mdc`, `.cursorrules`(legacy) |
| Windsurf | `AGENTS.md`, `.windsurf/rules/*.md` |
| 여러 도구 병행 | `AGENTS.md`를 공통 허브로 두고, 도구 전용 파일은 `AGENTS.md` 또는 `.ai/rules`를 참조 |

Claude Code도 다른 에이전트와 동일하게 규칙 본문은 `.ai/rules`에 두고, `CLAUDE.md`에는 해당 규칙 파일을 가리키는 **심볼릭 링크**(마크다운 링크)만 둔다. 규칙 본문을 `CLAUDE.md`에 복사하지 않는다.

템플릿 복사는 **1번 절차의 step 2**에서 sparse-checkout 적용 전에 한 번만 수행한다. 이후에는 `.ai/rules/templates/`가 작업 트리에 남지 않으므로 같은 명령을 다시 실행할 수 없다. 템플릿을 다시 받고 싶다면 일시적으로 sparse-checkout에 `templates`를 추가했다가 복사 후 다시 빼면 된다.

```bash
# 템플릿이 필요할 때만 일시 추가
cd .ai/rules && git sparse-checkout set common nextjs templates && cd -
cp .ai/rules/templates/<file> .
cd .ai/rules && git sparse-checkout set common nextjs && cd -
```

프로젝트에 실제로 체크아웃한 규칙만 남긴다. 예를 들어 `common + nextjs`만 쓰면 엔트리포인트에는 다음 링크만 남긴다:

```markdown
- [공통 규칙](.ai/rules/common/common.md)
- [Next.js 규칙](.ai/rules/nextjs/naming.md)
- [Next.js 설계 원칙](.ai/rules/nextjs/design-principles.md)
- [Next.js 데이터 페칭](.ai/rules/nextjs/data-fetching.md)
- [Next.js 관심사 분리](.ai/rules/nextjs/separation-of-concerns.md)
```

`AGEMNT.md`가 아니라 `AGENTS.md`를 쓴다. Codex, GitHub Copilot, Cursor, Windsurf 등 여러 에이전트 런타임에서 이 이름을 지원하거나 관례적으로 사용한다.

### 3. 이미 서브모듈이 있는 프로젝트를 clone할 때

```bash
# 전체 서브모듈 파일 포함 클론
git clone --recurse-submodules <프로젝트-repo-url>

# sparse-checkout 설정이 자동 적용되지 않으면 수동으로:
cd .ai/rules
git sparse-checkout init --cone
git sparse-checkout set common nextjs        # 이 프로젝트에 맞는 조합으로
```

### 4. 규칙 조합 변경

```bash
cd .ai/rules
git sparse-checkout set common nextjs ui     # ui 추가
# 또는
git sparse-checkout set common fsd           # 완전 재지정
```

`set`은 매번 **전체 조합**을 지정한다. 기존에 없던 폴더는 추가로 받고, 기존에 있던 폴더 중 빠진 건 제거됨.

### 5. 규칙 업데이트

```bash
git submodule update --remote .ai/rules
git add .ai/rules
git commit -m "chore: AI 규칙 업데이트"
```

sparse-checkout 설정은 유지된다.

### 6. 정상 확인

```bash
ls .ai/rules/
```

지정한 폴더들만 보이면 정상.

## 주의 사항

- 프로젝트 내 규칙 서브모듈 파일을 **직접 수정하지 않는다**. 규칙 변경은 이 저장소에서 수정 후 서브모듈을 업데이트한다.
- 프로젝트 루트의 `CLAUDE.md`, `AGENTS.md`, `GEMINI.md`에는 이 저장소 규칙을 길게 복사하지 말고 링크만 둔다. 규칙 본문을 복사하면 시간이 지나며 서로 달라진다.
- 서브모듈 관련 에러 발생 시 `git submodule update --init`을 실행한다.
- CI/CD 파이프라인에서 클론할 때 `--recurse-submodules` 옵션을 추가해야 서브모듈이 함께 받아진다.
- sparse-checkout `set` 명령을 잊고 push하면 팀원은 **전체 폴더**를 받을 수 있다. 설정을 README에 명시하거나 Makefile·스크립트로 자동화하는 것을 권장한다.
