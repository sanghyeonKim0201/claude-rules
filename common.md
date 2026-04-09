---
description: 모든 프로젝트에 적용되는 공통 규칙
---

# 공통 규칙

**모든 답변은 한국어로 작성하세요.**

## Git 컨벤션

**브랜치 전략:**

- `live` 브랜치(기본 브랜치)에는 **직접 push 금지**. 반드시 `dev` 브랜치에 push 후 GitHub에서 PR merge를 통해 반영한다.
- `dev` 브랜치에서 개발·테스트 후, `dev → live` PR을 생성하여 merge한다.

커밋 메시지 prefix:

- `feat` — 새로운 기능 추가
- `fix` — 버그 수정
- `docs` — 문서의 수정
- `style` — (코드의 수정 없이) 스타일(style)만 변경 (들여쓰기 같은 포맷이나 세미콜론을 빼먹은 경우)
- `refactor` — 코드를 리팩토링
- `test` — Test 관련한 코드의 추가, 수정
- `chore` — (코드의 수정 없이) 설정을 변경
- `wip` — 작업 중 (Work In Progress)

## 외부 도구 사용 전략

MCP는 AI 대화 중 자동 연동이 필요한 것만 연결하고, 나머지는 CLI로 직접 호출한다.

- **MCP 연결:** Linear, Context7
- **CLI 사용:** `gh` (GitHub), `sentry-cli`, `jenkins CLI`, `doppler CLI`
