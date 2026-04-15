---
paths:
  - "src/**"
  - "apps/**"
  - "packages/**"
  - "modules/**"
---

# FSD 아키텍처 상세 규칙

## 파일 구조 규칙

세그먼트(`model`, `ui`, `api` 등) 내부에서 관련 파일은 **용도별로 명확하게 묶어 관리**한다.

- UI 세그먼트는 컴포넌트 또는 화면 단위로 폴더를 묶어도 된다.
- 그 외 세그먼트는 검색성과 역할이 드러나는 명시적 파일명을 우선한다.
- `index` 파일 사용 여부는 언어와 프레임워크 관례에 따르되, **Public API와 내부 구현의 경계**는 항상 분명해야 한다.

## Public API 규칙

각 slice는 외부에 노출할 진입점을 명시적으로 제공한다.

- 외부에서는 반드시 slice의 public API를 통해 접근한다.
- 내부 구현 파일 경로를 직접 import하지 않는다.
- public API의 구체적 파일명(`index.ts`, `__init__.py`, package export 등)은 언어/프레임워크 관례를 따른다.

## Import 규칙

- **같은 slice 내부:** 로컬 상대 참조 또는 동일 모듈 내부 참조
- **다른 layer·slice 참조:** 프로젝트의 표준 모듈 경로를 사용해 public API를 통해 접근
- import 경로 방식(절대/상대)은 언어와 빌드 시스템 관례에 따르되, **slice 경계 우회는 금지**한다.

## FSD 레이어 간 공유 규칙

- features 간 공유가 필요한 코드는 **entities 레이어로 내린다.**
- 도메인 특화되지 않은 범용 코드는 **shared 레이어**에 둔다.
- 여러 slice에서 공통으로 필요한 조합이 반복되면 **widgets 또는 상위 조합 레이어**로 올린다.

## entities vs features 경계 규칙

- **entities**: 조회 중심. 도메인 모델, 읽기용 데이터 접근, 읽기용 UI를 담는다.
  - 판단 기준: **보여주기만 하는가?** → entities
- **features**: 변경 중심. 사용자 액션, 명령 처리, 쓰기/변경 요청, 인터랙션 UI를 담는다.
  - 판단 기준: **사용자가 무언가를 바꾸는가?** → features
- 도메인 데이터 기반 선택 컴포넌트처럼 **조회 결과를 재사용하는 UI**는 entities에 둔다.
- features는 entities가 제공하는 읽기 모델을 가져와 조합하고, 자신은 변경 행위에 집중한다.

## 리스트 구성 책임 규칙

- entities·features에는 **리스트 전체를 표현하는 상위 조합 컴포넌트**를 두지 않는다.
- entities·features에는 리스트 안에 들어가는 **개별 아이템 컴포넌트**만 둔다.
- 리스트 조회와 순회(map, for, render loop)로 아이템을 나열하는 책임은 **상위 레이어(widgets·views/pages)** 에 둔다.

## features slice 단일 책임 원칙

- **1 feature slice = 1 사용자 기능(액션)** 을 원칙으로 한다.
- 하나의 slice가 여러 독립 기능(제출, 임시 저장, 즐겨찾기 등)을 함께 담지 않는다.
- 분리 기준은 **변경 이유가 같은가**이다. 변경 이유가 다르면 별도 slice로 분리한다.
- create/update를 단순 mode 분기로 한 slice에 넣지 않는다. 생성과 수정은 변경 이유가 다르므로 별도 slice로 분리한다.

## feature 외부 노출 원칙

- feature는 가능한 한 외부에 **완성된 기능 단위**를 노출한다.
- 외부 레이어는 feature 내부의 세부 로직보다, feature가 제공하는 진입점과 조합 가능한 단위를 사용한다.
- 로직/핵심 객체/컴포넌트 중 무엇을 public API로 노출할지는 언어와 프레임워크 관례에 따르되, views/pages가 feature 내부 세부 구현을 직접 다루지 않도록 한다.

## 타입/모델 정의 위치 규칙

- **entities**: 여러 features에서 공유하는 도메인 모델은 entities 내부의 안정된 위치에 두고 public API로 re-export 한다.
- **features**: 해당 slice 내부에서만 쓰이는 타입/DTO/입력 모델은 정의된 위치에서 그대로 사용한다.
- 언어별 구체 문법(TypeScript type, Kotlin data class, Java record, Python dataclass)은 각 언어 규칙에서 정의한다.
