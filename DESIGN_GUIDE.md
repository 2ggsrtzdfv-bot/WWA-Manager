# WWA Manager Design Guide

Last updated: 2026-08-11
Status: Approved baseline

이 문서는 WWA Manager의 승인된 시각·인터랙션 규칙을 정의한다. WWA는 관리 대시보드나 팬덤 테마 앱이 아니라 영화와 LEGO 자료를 편집하는 뮤지엄 아카이브 제작 도구다.

## 1. Design Direction

핵심 방향:

- 영화 중심 에디토리얼 북
- 뮤지엄 아카이브의 정돈된 정보 위계
- Apple식 절제와 iPhone 친화적 조작
- 밝은 작업 공간과 어두운 감상 공간의 분리
- 기능 수보다 일관된 디자인 품질 우선

사용하지 않는 방향:

- 금색·갈색·양피지 기반 Harry Potter 테마
- 둥근 카드가 반복되는 관리자 대시보드
- 과도한 그림자, 그라데이션, 광택
- 마법 지팡이·별·반짝임·두루마리 같은 장식 아이콘
- 화려한 캡슐형 상태 배지
- 판매, 시세, 수집률을 강조하는 컬렉션 앱 표현

## 2. Surface System

WWA는 두 가지 화면 체계를 사용한다.

| Surface | 적용 화면 | 표현 |
|---|---|---|
| `Cinematic Surface` | Home, Movie, Chapter | 어두운 배경, 공식 영화 이미지, 큰 에디토리얼 제목 |
| `Archive Surface` | LEGO Record, Assets, 편집, Settings | Winter White, 얇은 선, 정돈된 목록, 이미지 중심 |

두 Surface는 같은 브랜드 체계에 속하지만 배경과 정보 밀도가 다르다. Archive 작업 화면을 어두운 팬덤 테마로 바꾸거나 Cinematic 화면을 일반 대시보드로 만들지 않는다.

## 3. Color Palette

아래 다섯 색을 WWA의 기본 팔레트로 고정한다.

| Token | Hex | 역할 |
|---|---|---|
| `Hogwarts Blue Hour` | `#1A2A44` | 제목, 주요 버튼, 선택 상태 |
| `Patronus Blue` | `#7FB7E6` | 체크, 포커스, 현재 위치 |
| `Moon Silver` | `#C7CCD6` | 구분선, 비활성 요소 |
| `Castle Stone` | `#6F7682` | 설명, 메타정보 |
| `Winter White` | `#F7F9FC` | Archive Surface 배경 |

- 기존 `Winter White #F8F8F5`는 폐기한다.
- 색은 장식이 아니라 정보 위계, 선택, 포커스에만 사용한다.
- 금색, 양피지색, 갈색 계열을 브랜드 강조색으로 사용하지 않는다.
- 상태마다 별도의 강한 색상을 늘리지 않는다.

## 4. Typography

| Typeface | 역할 |
|---|---|
| `Bluu Next` | WWA 브랜드명, 대표 화면 제목 |
| `Gloock` | Movie Chapter, 에디토리얼 제목 |
| `Source Sans 3` | 데이터, 버튼, 상태값, 한글 설명 |

권장 크기:

| 역할 | 크기 |
|---|---|
| 대표 제목 | `36–40px` |
| 화면 제목 | `28–32px` |
| 섹션 제목 | `18–20px` |
| 목록 제목 | `15–16px` |
| 본문·한글 설명 | `13–14px` |
| 메타정보 | `10–11px` |
| 입력값 | 최소 `16px` |

입력값은 iPhone Safari의 자동 확대를 막기 위해 실제 렌더링 크기 `16px` 이상을 사용한다.

## 5. Layout and Spacing

- iPhone 기준 설계 폭: `393px`
- iPhone 최대 검증 폭: `430px`
- iPad와 PC에서는 정보 폭을 무리하게 늘리지 않는다.
- 기본 좌우 여백: `20px`
- 간격 체계: `8 / 12 / 16 / 24 / 32px`
- 버튼과 목록 행 터치 영역: 최소 `44px`
- 기본 모서리 반경: `3px`
- 기본 구분선: `1px`
- 기본 그림자: 없음
- 팝업 또는 이미지 확대 레이어에만 매우 약한 그림자를 허용한다.

카드보다 전체 폭 목록과 섹션 구분선을 우선한다. 하나의 화면에 비슷한 카드가 반복되면 정보 구조를 다시 검토한다.

## 6. Navigation

- 하단 내비게이션은 최상위 탐색 화면에만 표시한다.
- Add/Edit 등 집중 편집 화면에서는 하단 내비게이션을 숨긴다.
- 상세 화면은 상단 Back과 명확한 화면 제목을 사용한다.
- 목록 행 전체를 누르면 상세 화면을 연다.
- 행 오른쪽에는 텍스트 `Open` 버튼 대신 `›`를 표시한다.
- 하단 내비게이션 명칭과 순서는 `Home / Records / Assets / Pages`로 고정한다.
- `Collection`은 LEGO Record 기능 구현 시 `Records`로 변경하며 별도 컬렉션 메뉴를 만들지 않는다.

## 7. Lists, Cards, and States

- 정보 탐색은 전체 폭 목록 행을 기본으로 한다.
- 이미지 탐색은 정돈된 썸네일 목록 또는 2열 그리드를 사용할 수 있다.
- 상태는 작은 텍스트와 얇은 선·점으로 표현한다.
- 그라데이션, 그림자, 굵은 외곽선, 여러 강조색을 결합한 캡슐형 배지는 사용하지 않는다.
- `Review`, `Verified` 같은 상태값은 영문 원문을 유지한다.
- 상태와 Lifecycle을 하나의 배지에 합치지 않는다.
- 최대 3개의 연결 항목을 먼저 표시하고 나머지는 `+4 more`처럼 요약할 수 있다.
- Location은 `Hogwarts › Great Hall`처럼 전체 경로로 표시한다.

## 8. Icons

- 얇은 단색 기능형 선 아이콘을 사용한다.
- 아이콘은 이동, 뒤로가기, 검색, 편집, 업로드, 체크 등 명확한 기능만 표현한다.
- 마법 지팡이, 반짝임, 별, 두루마리, 마법광 등 장식 목적 아이콘은 사용하지 않는다.
- 아이콘만으로 의미가 불명확하면 짧은 텍스트 레이블 또는 접근성 이름을 제공한다.
- 선택 상태는 Patronus Blue 체크를 사용한다.

## 9. Images

| 자료 | 표시 비율·방식 |
|---|---|
| 영화 포스터 | `2:3` |
| 영화 스틸 | `16:9` |
| Asset 목록 | `16:10` 기본 |
| LEGO 공식 제품 이미지 | `contain`, 제품이 잘리지 않게 표시 |
| Asset Detail | 원본 비율 유지 |

- 목록에서는 목적에 맞는 크롭을 허용하지만 상세 화면에서 원본 전체를 확인할 수 있어야 한다.
- Asset 원본에는 페이지별 크롭값을 저장하지 않는다.
- 사용자 원본 WWA 로고만 사용하며 AI로 재생성하지 않는다.
- AI 생성 LEGO 이미지와 캡처 이미지를 공식 자료 대용으로 사용하지 않는다.

## 10. Asset Screens

`Assets Library + Asset Detail`은 다음 구성을 따른다.

- Assets Library: 검색·필터, 정돈된 썸네일, 핵심 출처·제작 상태
- Asset Type 필터: `Film / LEGO / Blueprint / My Photo / WWA`
- Asset Detail: 큰 원본 미리보기, Source, Archive Relations, Evidence Usage
- Edit Asset: 영문 필드명과 상태값, 한글 설명과 오류 안내
- Related Record: 연결된 LEGO Record 표시
- Direct Links: 이미지가 직접 보여주는 Location·Subject만 표시
- Evidence Usage: 어느 Story Connection의 근거인지 복수 표시
- Asset 행 전체 터치 + 오른쪽 `›`
- 편집 중 하단 내비게이션 숨김

## 11. Story Connection Screens

- `Context Mapping`은 LEGO Record Detail 안의 섹션으로 배치한다.
- Film–Location–Subject 관계를 얇은 목록 행으로 표시한다.
- 각 행 오른쪽에는 `Review` 또는 `Verified`와 `›`를 표시한다.
- 행 전체를 누르면 전체 화면 Connection Detail을 연다.
- Evidence 선택은 체크 방식의 복수 선택이다.
- Narrative Scope 5종은 작은 칩 묶음이 아니라 전체 폭 선택 목록을 사용한다.
- 연결 오류는 팝업이 아니라 해당 필드 아래 한글 문장으로 표시한다.

## 12. Forms and Feedback

- 필드명·상태값은 영어, 설명·오류·확인 문구는 한글로 표시한다.
- 필수값 누락과 관계 오류는 필드 바로 아래에 표시한다.
- 저장되지 않은 변경사항이 있을 때 화면을 나가면 `Discard Changes` 확인창을 표시한다.
- 확인창은 다음 경우에만 사용한다.
  - Discard Changes
  - Replace Image
  - Archive
- 일반 저장 완료, 필터 변경, 상세 열기에는 확인창을 사용하지 않는다.
- 성공·오류 피드백은 짧고 조용한 방식으로 제공하며 화면을 가리지 않는다.

## 13. Motion and Accessibility

- 기본 전환 시간: 약 `180ms`
- 페이드와 짧은 슬라이드만 사용한다.
- 튕김, 회전, 과도한 패럴랙스는 사용하지 않는다.
- `prefers-reduced-motion` 환경에서는 불필요한 전환을 제거한다.
- 키보드 포커스와 스크린리더 이름을 제공한다.
- 텍스트와 배경은 읽을 수 있는 대비를 유지한다.
- 터치 대상은 최소 `44px`을 지킨다.

## 14. Design Review Checklist

구현 승인 전 다음을 확인한다.

- WWA가 컬렉션 앱보다 영화 기반 뮤지엄 아카이브로 보이는가
- Cinematic Surface와 Archive Surface의 역할이 명확한가
- Winter White가 `#F7F9FC`로 적용되었는가
- 승인된 다섯 색 밖의 장식색이 늘어나지 않았는가
- 상태가 화려한 캡슐 배지로 변하지 않았는가
- 장식 아이콘이 들어가지 않았는가
- 카드, 라운드, 그림자가 과도하지 않은가
- iPhone 393px과 430px에서 깨지지 않는가
- 편집 화면의 입력값이 16px 이상이고 하단 내비게이션이 숨겨지는가
- 이미지 상세에서 원본 전체를 확인할 수 있는가
- 영문 필드명·상태값과 한글 설명 규칙이 지켜지는가
