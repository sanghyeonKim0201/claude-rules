# Claude Rules

Claude Code가 따라야 할 공유 코딩 규칙 모음. 모든 규칙을 **하나의 저장소**에서 관리하며, 프로젝트에 필요한 규칙만 **sparse-checkout**으로 선택해 가져간다.

## 규칙 구성

| 폴더 | 설명 | 포함 파일 |
|---|---|---|
| [`common/`](./common) | 언어/프레임워크 무관 공통 규칙 | Git 컨벤션, 설계 원칙, 네이밍, 순수 함수 분리 |
| [`nextjs/`](./nextjs) | Next.js / React / TypeScript 규칙 | 네이밍, 설계 원칙, 데이터 페칭, 관심사 분리, FSD-Next.js 통합 |
| [`fsd/`](./fsd) | FSD 아키텍처 전용 규칙 | 레이어 구조, Public API, Import, 관심사 분리 |
| [`ui/`](./ui) | UI/UX 설계 원칙 (프레임워크 무관) | 컴포넌트 API, 접근성, UI 상태, 디자인 토큰, 컴포지션 |

## 조합 원칙

- `common`은 거의 모든 프로젝트에 기본 적용.
- `fsd`는 아키텍처 규칙이므로 언어/프레임워크와 독립적으로 조합 가능.
- `nextjs`는 Next.js/React/TypeScript 프로젝트에서만 사용.
- `ui`는 디자인 시스템·UI 컴포넌트 작업이 있는 프로젝트에서 사용 (프레임워크 무관).

## 프로젝트 유형별 조합

| 프로젝트 유형 | 폴더 조합 |
|---|---|
| Next.js + FSD | `common` + `nextjs` + `fsd` |
| Next.js (FSD 미사용) | `common` + `nextjs` |
| Next.js 디자인 시스템 | `common` + `nextjs` + `ui` |
| Spring Boot + FSD | `common` + `fsd` + (향후 spring) |
| Flutter + FSD | `common` + `fsd` + (향후 flutter) |
| 비 Next.js / 비 FSD | `common` + (해당 프레임워크 규칙) |

## 설정 가이드

### 1. 프로젝트에 규칙 추가 (필요한 폴더만)

```bash
cd <프로젝트 루트>

# 1) 서브모듈 추가 (전체 monorepo를 논리적으로 연결)
git submodule add https://github.com/sanghyeonKim0201/claude-rules.git .claude/rules

# 2) sparse-checkout으로 필요한 폴더만 실제 체크아웃
cd .claude/rules
git sparse-checkout init --cone
git sparse-checkout set common nextjs        # 예: Next.js 프로젝트
cd -

# 3) 커밋
git add .gitmodules .claude/rules
git commit -m "chore: Claude 규칙 서브모듈 추가 (common/nextjs)"
```

결과:
```
.claude/rules/
├── common/
└── nextjs/
```

`fsd`, `ui`, 기타 폴더는 원격에 존재하지만 **이 프로젝트엔 받지 않는다.**

### 2. 이미 서브모듈이 있는 프로젝트를 clone할 때

```bash
# 전체 서브모듈 파일 포함 클론
git clone --recurse-submodules <프로젝트-repo-url>

# sparse-checkout 설정이 자동 적용되지 않으면 수동으로:
cd .claude/rules
git sparse-checkout init --cone
git sparse-checkout set common nextjs        # 이 프로젝트에 맞는 조합으로
```

### 3. 규칙 조합 변경

```bash
cd .claude/rules
git sparse-checkout set common nextjs ui     # ui 추가
# 또는
git sparse-checkout set common fsd           # 완전 재지정
```

`set`은 매번 **전체 조합**을 지정한다. 기존에 없던 폴더는 추가로 받고, 기존에 있던 폴더 중 빠진 건 제거됨.

### 4. 규칙 업데이트

```bash
git submodule update --remote .claude/rules
git add .claude/rules
git commit -m "chore: Claude 규칙 업데이트"
```

sparse-checkout 설정은 유지된다.

### 5. 정상 확인

```bash
ls .claude/rules/
```

지정한 폴더들만 보이면 정상.

## 주의 사항

- 프로젝트 내 `.claude/rules/` 파일을 **직접 수정하지 않는다**. 규칙 변경은 이 저장소에서 수정 후 서브모듈을 업데이트한다.
- 서브모듈 관련 에러 발생 시 `git submodule update --init`을 실행한다.
- CI/CD 파이프라인에서 클론할 때 `--recurse-submodules` 옵션을 추가해야 서브모듈이 함께 받아진다.
- sparse-checkout `set` 명령을 잊고 push하면 팀원은 **전체 폴더**를 받을 수 있다. 설정을 README에 명시하거나 Makefile·스크립트로 자동화하는 것을 권장한다.
