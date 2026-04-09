# Claude Rules

Claude Code가 따라야 할 공유 코딩 규칙 모음입니다. git 서브모듈로 각 프로젝트에 연결하여 사용합니다.

## 규칙 파일 목록

| 파일 | 설명 |
| --- | --- |
| `data-fetching.md` | 데이터 페칭 및 상태 관리 패턴 |
| `design-principles.md` | 디자인 원칙 |
| `fsd-architecture.md` | FSD 아키텍처 상세 규칙 |
| `naming.md` | 네이밍 컨벤션 |
| `separation-of-concerns.md` | 관심사 분리 원칙 |

## 프로젝트에 적용하기

### 새 프로젝트에 서브모듈 추가

```bash
git submodule add https://github.com/sanghyeonKim0201/claude-rules.git .claude/rules
git commit -m "chore: Claude 규칙 서브모듈 추가"
```

### 클론 후 설정

```bash
# 방법 1: 클론 시 함께 받기
git clone --recurse-submodules <repo-url>

# 방법 2: 이미 클론한 경우
git submodule init
git submodule update
```

### 최신 규칙으로 업데이트

```bash
git submodule update --remote .claude/rules
git add .claude/rules
git commit -m "chore: Claude 규칙 서브모듈 업데이트"
```

## 주의 사항

- 규칙 수정은 이 저장소에서 직접 수행합니다. 프로젝트 내에서 파일을 직접 수정하지 마세요.
- 서브모듈 관련 에러 발생 시 `git submodule update --init`을 실행해보세요.
