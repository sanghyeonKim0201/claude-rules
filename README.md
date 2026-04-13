# Claude Rules

Claude Code가 따라야 할 공유 코딩 규칙 모음. 규칙은 역할별로 분리되어 있으며, 프로젝트에 필요한 조합만 서브모듈로 연결하여 사용한다.

## 규칙 저장소

| 저장소 | 설명 | 포함 규칙 |
|---|---|---|
| [claude-rules-common](https://github.com/sanghyeonKim0201/claude-rules-common) | 언어/프레임워크 무관 공통 규칙 | Git 컨벤션, 설계 원칙, 네이밍 원칙, 순수 함수 분리 |
| [claude-rules-nextjs](https://github.com/sanghyeonKim0201/claude-rules-nextjs) | Next.js/React/TypeScript 규칙 | 네이밍, 설계 원칙, 데이터 페칭, 관심사 분리, FSD-Next.js 통합 규칙 |
| [claude-rules-fsd](https://github.com/sanghyeonKim0201/claude-rules-fsd) | FSD 아키텍처 전용 규칙 | FSD 레이어 구조, Public API, Import, 관심사 분리 |

## 조합 원칙

- `common`은 거의 모든 프로젝트에 기본으로 적용한다.
- `fsd`는 아키텍처 규칙이므로 **특정 언어/프레임워크와 독립적으로** 조합할 수 있어야 한다.
- `nextjs`는 Next.js/React/TypeScript 프로젝트에서만 사용한다.

## 프로젝트 유형별 조합

| 프로젝트 유형 | 서브모듈 조합 |
|---|---|
| Next.js + FSD | common + nextjs + fsd |
| Next.js (FSD 미사용) | common + nextjs |
| Spring Boot + FSD | common + fsd + (향후 spring 규칙) |
| Flutter + FSD | common + fsd + (향후 flutter 규칙) |
| 비 Next.js / 비 FSD | common + (해당 프레임워크 규칙) |

## 설정 가이드

### 1. 새 프로젝트에 서브모듈 추가

일반적인 Next.js + FSD 프로젝트 기준:

```bash
cd <프로젝트 루트>

# 서브모듈 추가
git submodule add https://github.com/sanghyeonKim0201/claude-rules-common.git .claude/rules/common
git submodule add https://github.com/sanghyeonKim0201/claude-rules-nextjs.git .claude/rules/nextjs
git submodule add https://github.com/sanghyeonKim0201/claude-rules-fsd.git .claude/rules/fsd

# 커밋
git add .gitmodules .claude/rules
git commit -m "chore: Claude 규칙 서브모듈 추가 (common/nextjs/fsd)"
```

### 2. 클론 후 초기 설정

서브모듈이 등록된 프로젝트를 클론할 때, 규칙 저장소 URL을 별도로 입력할 필요는 없다. 프로젝트의 `.gitmodules`에 서브모듈 정보가 이미 등록되어 있으므로, **프로젝트 URL만 클론하면 서브모듈이 함께 받아진다.**

```bash
# 방법 1: 클론 시 함께 받기
git clone --recurse-submodules <프로젝트-repo-url>

# 방법 2: 이미 클론한 경우 (.claude/rules/ 하위 폴더가 비어 있을 때)
git submodule init
git submodule update
```

### 3. 정상 확인

```bash
ls .claude/rules/
```

아래처럼 폴더가 보이면 정상이다.

```
common/    — common.md
nextjs/    — naming.md, design-principles.md, data-fetching.md, separation-of-concerns.md, fsd-integration.md
fsd/       — fsd-architecture.md, separation-of-concerns.md
```

### 4. 규칙 업데이트

모든 서브모듈을 최신으로 받을 때:

```bash
git submodule update --remote .claude/rules/common
git submodule update --remote .claude/rules/nextjs
git submodule update --remote .claude/rules/fsd

git add .claude/rules
git commit -m "chore: Claude 규칙 서브모듈 업데이트"
```

## 주의 사항

- 프로젝트 내 `.claude/rules/` 파일을 **직접 수정하지 않는다.** 규칙 변경은 해당 규칙 저장소에서 수정 후 서브모듈을 업데이트한다.
- 서브모듈 관련 에러 발생 시 `git submodule update --init`을 실행해본다.
- CI/CD 파이프라인에서 클론할 때 `--recurse-submodules` 옵션을 추가해야 서브모듈이 함께 받아진다.
