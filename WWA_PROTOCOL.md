# WWA Manager Protocol

Last updated: 2026-08-21
Status: Approved baseline

이 문서는 WWA Manager의 제품 목적, 승인된 아키텍처, 데이터 원칙, 개발 절차를 고정하는 최상위 기준이다. 기능이나 화면을 설계·구현할 때 이 문서와 `DESIGN_GUIDE.md`, `CHANGELOG.md`를 먼저 확인한다. 저장·백업 구현은 승인된 `STORAGE_SYNC_DESIGN.md`의 데이터 계약도 함께 따른다.

## 1. Product Identity

- 프로젝트명: `WWA Manager`
- 아카이브명: `WWA (Wizarding World Archive)`
- 목적: LEGO Harry Potter 자료를 영화의 이야기와 함께 기록하고 최종 `WWA Book`을 제작하기 위한 개인용 뮤지엄 아카이브 도구
- 최종 결과: WWA Manager 완성 → WWA Book 제작 → 양장본 출력 → 평생 소장
- 판매, 재고, 시세, 희소성, 수집률 중심의 LEGO 컬렉션 앱으로 만들지 않는다.
- `My Collection` 정보는 LEGO Record의 소장·구매 메타데이터이며 메인 탐색축이 아니다.

브랜드 방향은 `Museum Archive + LEGO Collector + Film Book`이며, 핵심 인상은 `Premium / Archive / Museum / Clean / Minimal / Luxury / Timeless`이다.

## 2. Governing Order

서로 다른 기준이 충돌하면 다음 순서로 판단한다.

1. 사용자의 가장 최근 명시적 승인
2. `WWA_PROTOCOL.md`
3. `DESIGN_GUIDE.md`
4. `CHANGELOG.md`의 최신 승인 기록
5. 현재 배포 코드

오래된 시안이나 구현은 최신 승인 사항을 되돌리는 근거가 될 수 없다. 승인 상태가 불분명하면 구현하지 않고 `Pending Decision`으로 남긴다.

## 3. Locked Technical Architecture

- 저장소: `2ggsrtzdfv-bot/WWA-Manager`
- 배포: GitHub Pages
- 앱 소스: `index.html` 단일 파일
- 플랫폼 우선순위: iPhone 우선, iPad 지원, PC 보조
- 데이터 구조: `Local-First + Full ZIP Backup`
- 기술 스택과 배포 방식을 임의로 변경하지 않는다.
- Apple Developer 유료 가입과 CloudKit 자동 동기화는 사용하지 않는다.
- 별도 백엔드, 프레임워크, 빌드 시스템, 다중 페이지 구조는 명시적 승인 없이 추가하지 않는다.
- 네이티브 앱 전환은 현재 범위가 아니다.

## 4. Archive Flow

WWA의 제작 흐름은 다음 순서를 따른다.

`Movie → Location → LEGO Record → Assets → WWA Page → WWA Book`

- `Movie`는 이야기 탐색 기준이다.
- `Location`은 영화 속 공간을 계층 경로로 기록한다.
- `LEGO Record`는 세트번호별 원본 Record 하나만 둔다.
- `Assets`는 출처와 용도가 검증되는 독립 원본 자료다.
- `WWA Page`에서 최종 편집·크롭·배치를 결정한다.
- `WWA Book`은 앱 데이터의 최종 출판 결과다.

영화, 장소, 세트, Asset을 편의를 위해 자동으로 모든 조합에 연결하지 않는다.

## 5. Development Workflow

개발은 다음 순서로만 진행한다.

`Design → Approval → Implementation → GitHub → Testing → Release`

- 한 번에 기능 하나를 설계하고 승인받은 뒤 구현한다.
- 승인 전에는 실제 앱 코드와 배포본을 변경하지 않는다.
- 승인된 아키텍처를 새 기능 때문에 덮어쓰지 않는다.
- 기능 수보다 디자인 품질과 데이터 신뢰성을 우선한다.
- 실제 구현 전 현재 저장소, 원격 `main`, 기존 사용자 수정 여부를 확인한다.
- 사용자 수정과 겹치는 파일은 보존하고, 불명확한 충돌은 임의 병합하지 않는다.
- 구현 후 iPhone을 우선으로 테스트하고, 승인된 체크포인트만 배포한다.

현재 승인 상태와 다음 진행 순서:

- Asset 저장·편집·Version History, Legacy Migration, Full ZIP Backup·Restore, LEGO Record, WWA Pages Phase 1은 구현·검증·배포 완료 상태다.
- WWA Page의 현재 작업 판형과 임시 출력 기준은 A4 Portrait `210 × 297 mm`로 승인한다.
- 최종 출판물은 `일반 양장(비 Layflat)`으로 제작한다.
- Spread의 왼쪽·오른쪽 면은 각각 독립적으로 완결하며, 글자·얼굴·로고·정렬 기준선 등 필수 정보는 중앙 접힘선을 넘지 않는다. 중앙을 잇는 장식 배경은 제본 손실과 좌우 어긋남이 생겨도 의미가 유지되는 경우에만 허용한다.
- 인쇄 내지 PDF의 허용 범위는 `160–224 pages`, 설계 기준은 `192 pages`, 현재 권장 최대치는 `240 pages`로 승인한다. 240 pages를 넘으면 한 권을 비대하게 확장하지 않고 다음 Volume으로 분권한다.
- 192 pages 기준으로 Front Matter 8 pages, 영화 8편 Chapter Opener 16 pages, WWA Record 80 Spreads 160 pages, Index·Credits·Colophon 8 pages를 배정한다.
- 인쇄소·용지가 정해질 때까지 최종 판형, spine, gutter 수치는 확정하지 않는다.

1. 왼쪽 Story Page·오른쪽 LEGO Archive Page의 필수·선택 Slot 승인
2. 첫 두 개 Slot 기반 Template 설계·승인
3. `PAGE-000001`에 필요한 Story Connection Record와 Evidence Link 구현
4. WWA Page Editor·Preview 구현과 iPhone 검증
5. Print Provider Profile·PDF Export·교정쇄 절차 설계

## 6. LEGO Record Rules

- 한 세트번호에는 하나의 LEGO Record만 존재한다.
- `Set Number`의 모든 공백을 제거한 값이 같으면 같은 Record로 판단하고 중복 생성을 차단한다.
- `Set Number` 수정은 허용하되 저장 전에 중복 검사를 다시 수행하며, 기존 `SET` ID는 유지한다.
- 같은 세트를 영화별로 복제하지 않는다.
- 한 Record는 여러 영화·장소·Subject와 연결할 수 있다.
- Film·Location 요약 목록은 `Story Connection`에서 자동 생성한다.
- `Primary Set Family`는 Record당 하나만 사용한다.
- Set Family는 영화 분류와 별개이며 영화·장소를 자동 추론하지 않는다.
- 미확정 Set Family는 `Unassigned`로 둔다.
- Set Family를 새로운 메인 탐색 메뉴로 확장하지 않는다.
- `Primary Book Placement`는 WWA Page 단계 전까지 `Not placed`를 유지한다.
- 새 Record의 기본값은 `Draft / Active / Unassigned / Not placed`다.

기본 LEGO Record 항목:

- Set Number (필수)
- Set Name (필수)
- Release Date
- Pieces
- Minifigures
- Size
- MSRP
- My Price
- Purchase Date
- Purchase Store

필드 저장 규칙:

- `Release Date`는 확인된 범위에 따라 연도, 연·월, 연·월·일의 부분 날짜를 허용한다.
- `Purchase Date`는 실제 구매일을 기록하며 모르면 공란으로 둔다.
- `Minifigures`는 공식 원문 이름 목록으로 저장하고 개수는 목록에서 자동 계산한다.
- `Size`는 `Height / Width / Depth + Unit`으로 분리하며 모르는 값은 공란으로 둔다.
- `MSRP`와 `My Price`는 금액과 통화를 분리 저장하며 자동 환산하지 않는다.

Record 상태는 의미별로 분리한다.

- `Record Verification`: `Draft / Review / Verified`
- `Record Lifecycle`: `Active / Archived`

`Archived`는 검증 실패를 뜻하지 않는다.
`Verified`는 공식 출처, `Checked On`, 확인 가능한 핵심 공식 정보가 모두 있을 때만 허용한다.

### Rebrickable Catalog Import

- Rebrickable은 LEGO Record의 기본 카탈로그 정보를 보조 입력하는 선택형 조회 수단이며 LEGO 공식 검증 출처가 아니다.
- 조회는 `Add/Edit Record`의 `Find Set Information` 버튼을 눌렀을 때만 실행한다.
- 사용자가 `76419`를 입력하면 API에는 `76419-1`로 조회하되 WWA Record에는 사용자가 입력한 `76419`를 저장한다.
- 자동조회 결과는 별도 전체 화면에서 검토한 뒤 적용하며 기존 값을 자동으로 덮어쓰지 않는다.
- Add에서는 Set Name, 확인된 연도, Pieces, 공식 원문 Minifigure 이름 목록을 적용할 수 있다.
- Edit에서는 변경된 항목만 비교하고 기존 값이 있으면 `Keep Current`, 비어 있으면 `Use Rebrickable`을 기본으로 한다.
- 더 정확한 부분 Release Date를 Rebrickable 연도로 낮추지 않으며 Minifigures는 목록 전체를 선택한다.
- 적용된 Record는 `Review`로 전환하고 `Checked On`을 비운다. 기존 `Official Source`는 유지한다.
- Rebrickable 이미지, Theme, API 원문 응답은 Record나 Asset에 저장하지 않는다.
- 실제 적용한 최신 조회 기록만 `catalogImport`로 저장한다: `provider / providerSetNumber / sourceUrl / importedAt / appliedFields`.
- 자동조회 적용 뒤 Set Name·Release Date·Pieces·Minifigures를 직접 수정하면 해당 항목을 `appliedFields`에서 즉시 제거하고 `Imported · Rebrickable` 표시도 제거한다. 적용 항목이 하나도 남지 않으면 `catalogImport` 전체를 제거하며, 출처를 다시 연결하려면 재조회 후 다시 적용한다.
- `catalogImport`는 Record와 Full ZIP Backup에 포함하지만 Rebrickable API Key는 포함하지 않는다.
- API Key는 현재 기기의 IndexedDB `settings`에만 저장하며 GitHub 코드, Record, Asset, ZIP, URL, 오류 문구에 넣지 않는다.
- API 요청은 `Authorization: key …` 헤더를 사용하고 순차 실행한다. `429`는 안내된 대기시간 뒤 한 번만 자동 재시도한다.
- UI에서는 `Verified`를 직접 선택하지 않고 `Complete Official Check`를 통해서만 전환한다. 최종 저장 계층에서도 같은 검증을 다시 수행한다.
- Official Source는 HTTPS `lego.com`의 현재 Set Number와 일치하는 공식 상품 페이지 또는 공식 조립 설명서만 인정한다. LEGO 검색 결과·다른 세트번호·위장 도메인·Rebrickable 주소는 인정하지 않는다.
- LEGO 공식 상품 페이지 또는 공식 조립 설명서 중 하나로 Release Date·Pieces·Official Source를 확인하고 `Checked On`을 기록해야 `Verified`로 완료할 수 있다.
- 상품 페이지가 내려간 단종 세트도 공식 조립 설명서로 검증할 수 있으며, 공식 자료에서 확인되지 않는 Size·한국 MSRP·정확한 날짜는 공란으로 둔다.

## 7. Story Connection Rules

Film과 Location의 실제 관계는 `Story Connection` 단위로 저장한다. Film Links와 Location Links는 Story Connection에서 생성되는 요약 정보다.

각 Story Connection은 다음 정보를 가진다.

- Film 또는 `Series-wide`
- Location 전체 경로
- Subject(필요한 경우)
- Narrative Scope
- `Status`: `Review / Verified`
- Evidence Assets: 복수 선택 가능
- Connection Note
- Checked On

장소가 여러 영화에 등장하면 영화별 Story Connection을 만든다. 영화와 직접 연결되지 않는 장소는 `Series-wide` 관계를 허용한다.

- 새 Story Connection의 기본 상태는 `Review`다.
- `Record + Film + Location + Subject + Narrative Scope` 조합이 같으면 중복 생성을 차단한다.
- Location은 전체 경로가 같을 때만 중복으로 판단한다.
- Subject는 `Subject Type + 공식 원문 이름`이 같을 때 중복으로 판단한다.

### Narrative Scope

| Scope | 필수 연결 | 판정 기준 |
|---|---|---|
| `Scene` | Film 1개 이상 | 특정 장면을 직접 표현 |
| `Location` | Location 1개 이상 | 장소 자체가 중심 |
| `Subject` | Subject 1개 이상 | 인물·생물·물건·탈것 중심 |
| `Cross-Film` | 검증된 Film 2개 이상 | 여러 영화의 요소가 혼합 |
| `Archive` | Connection Note | 특정 영화에 강제로 귀속되지 않는 전시·기록물 |

Scope는 대표 검색·편집 기준이며 모든 특징을 표현하는 태그가 아니다.

### Subject Types

- `Character`
- `Creature`
- `Object`
- `Vehicle`

Subject Links는 모든 Scope에서 선택할 수 있지만 Scope가 `Subject`라면 최소 하나가 필수다.

### Verification

- 공식 영화 스틸, LEGO 공식 자료, 공식 설명을 우선 근거로 사용한다.
- 직접 촬영 사진은 세트의 물리적 디테일 근거로 사용할 수 있다.
- 추정 연결은 `Review`로 둔다.
- 근거 없이 `Verified`로 바꾸지 않는다.
- 잘못된 연결은 삭제할 수 있지만 연결된 Record·Location·Asset 원본은 함께 삭제하지 않는다.
- `Verified Story Connection`은 최소 한 개의 검증된 Evidence와 `Checked On`이 필요하다.

## 8. Asset Rules

Asset은 이미지 원본과 출처·검증·사용 관계를 보존하는 독립 Record다.

허용 원본:

- LEGO 공식 제품 이미지
- LEGO 공식 박스 이미지
- 공식 영화 스틸
- WWA가 공식 자료를 참고해 직접 제작한 Blueprint
- 사용자가 직접 촬영한 굿즈·세트 이미지
- 사용자 제작 WWA 원본 로고·브랜드 자료

금지 원본:

- AI 생성 LEGO 이미지
- AI 재생성 WWA 로고
- 출처가 불분명한 캡처 이미지
- 공식 자료처럼 보이도록 가공한 비공식 이미지

### Asset Types

Asset Type은 다음 6종으로 고정한다.

| Asset Type | 사용 대상 | 목록 필터 |
|---|---|---|
| `Official Film Still` | 공식 영화 장면 이미지 | `Film` |
| `LEGO Official Product Image` | LEGO 공식 제품 이미지 | `LEGO` |
| `LEGO Official Box Image` | LEGO 공식 박스 이미지 | `LEGO` |
| `WWA Blueprint` | WWA가 직접 제작한 도면·구조 시각화 | `Blueprint` |
| `My Photography` | 직접 촬영한 세트·굿즈 사진 | `My Photo` |
| `WWA Original` | 사용자 원본 로고·브랜드 자료 | `WWA` |

- 목록 필터는 `Film / LEGO / Blueprint / My Photo / WWA`로 표시한다.
- `WWA Blueprint`는 LEGO·영화사의 공식 원본이 아니라 WWA 제작 Asset이다. 화면과 데이터에서 `Official`로 표기하지 않는다.
- `WWA Blueprint`의 `Source`에는 제작 근거나 참고 범위를 기록한다. `Source Link`는 선택 사항이며, `Verified`는 제작 근거를 확인했다는 뜻이지 공식 자료라는 뜻이 아니다.
- `WWA Blueprint`를 Story Connection의 공식 Evidence로 자동 연결하지 않는다. 근거로 사용할 때는 출처가 검증된 공식 Asset을 별도로 연결한다.
- `Film Poster`, 캡처 이미지, AI 생성물은 Asset Type에 추가하지 않는다.
- 새 Asset Type은 별도 승인 없이 추가하거나 기존 타입과 합치지 않는다.

Asset 저장 원칙:

- 원본 Asset은 한 번만 저장하고 관계로 재사용한다.
- Asset에는 이미지가 직접 보여주는 Location·Subject만 기록한다.
- Record 전체의 영화·장소 정보를 Asset에 중복 저장하지 않는다.
- 동일 Asset을 여러 Story Connection의 Evidence로 복수 연결할 수 있다.
- 페이지별 크롭과 배치는 Asset 원본이 아니라 WWA Page에서 관리한다.
- `AST` ID는 발급 후 변경하거나 재사용하지 않는다.
- 이미지 교체 시 기존 `AST` ID와 연결 관계를 유지한다.
- 이미지 교체 전 확인창을 표시하고, 이전 원본을 버전으로 보존한다.

### Add Asset

- 필수 입력: `Image`, `Asset Title`, `Asset Type`, `Source`
- 공식 자료는 `Source Link`도 필수다.
- `Source Status` 기본값은 `Source Needed`다.
- `Production Status` 기본값은 `Candidate`다.
- `Related LEGO Record`, `Film`, `Location`, `Subject`, `Notes`는 선택 입력이다.
- Film·Location·Subject는 이미지가 직접 보여주는 관계만 기록한다.
- `Checked On`은 `Verified`로 전환할 때 필수다.
- 첫 저장 시 `AST-000001` 형식의 고정 ID를 자동 발급한다.

### Edit Asset and Replace Image

- `Asset ID`는 읽기 전용이며 수정할 수 없다.
- `Asset ID`를 제외한 메타데이터는 수정할 수 있다.
- 이미지 교체 시 기존 `AST` ID와 모든 연결 관계를 유지한다.
- 교체 전 이미지는 `Version History`에 보존한다.
- 교체 직후 `Source Status`를 `Review`로 변경하고 `Checked On`을 초기화한다.
- 전혀 다른 이미지는 기존 Asset에 교체하지 않고 새 Asset으로 등록한다.
- 저장하지 않고 나갈 때 `Discard Changes` 확인창을 표시한다.
- `Archive`는 삭제가 아니며 복원할 수 있다.
- `In Use` Asset을 교체할 때는 현재 사용 중인 WWA Page가 있음을 확인창에 표시한다.

상태는 의미별로 분리한다.

- `Asset Source`: `Source Needed / Review / Verified`
- `Asset Production`: `Candidate / Selected / In Use / Archived`

`In Use`는 실제 WWA Page에 배치된 뒤에만 사용한다. `Archived`는 삭제나 검증 실패를 뜻하지 않는다.

### Asset Storage and Backup

WWA의 저장 방식은 `Local-First + Full ZIP Backup`으로 고정한다.

세부 데이터 계약은 승인된 `STORAGE_SYNC_DESIGN.md`로 고정하며, 다음 핵심 결정을 임의로 변경하지 않는다.

- 사용자용 Stable ID와 내부 Sync ID를 분리한다.
- Asset 이미지와 Version 파일을 별도 Record로 저장한다.
- Stable ID는 IndexedDB의 high-water를 기준으로 25개 번호 블록을 원자적으로 예약하며, 발급된 번호를 재사용하지 않는다.
- ZIP 복원은 새 generation에 검증·적재한 뒤 활성 generation을 전환하는 전체 교체 방식만 사용한다.

- 주 저장소는 각 기기 브라우저의 `IndexedDB`다.
- 저장 작업은 네트워크 연결 없이 IndexedDB에 완료한다.
- Asset 메타데이터, 원본 이미지, 표시용 Preview, 모든 `Version History`를 로컬 데이터에 포함한다.
- 자동 계정 동기화와 기기 간 자동 병합은 제공하지 않는다.
- 전체 ZIP 백업을 장기 보존과 기기 이동의 공식 수단으로 유지한다.
- ZIP에는 Archive 데이터, Asset 원본, Preview, 모든 이미지 버전과 백업 식별 정보를 포함한다.
- 복원은 파일 검증이 완료되기 전 기존 로컬 데이터를 변경하지 않는다.
- WWA 백업 ZIP의 공식 보관 위치는 `iCloud Drive / WWA Backup` 폴더다. 백업 생성 때 사용자가 해당 폴더를 저장 위치로 선택하며, iCloud Drive는 파일 보관 위치이지 앱 자동 동기화 계층이 아니다.
- `localStorage`는 영구 Asset 원본 저장소로 사용하지 않는다.

## 9. Stable IDs

다음 ID는 발급 후 변경·재사용하지 않는다.

- `SET-...`: LEGO Record
- `LOC-...`: Location
- `SUBJ-...`: Subject
- `CON-...`: Story Connection
- `AST-...`: Asset

세트번호는 Record 중복 검사의 기준이고, Location은 전체 경로를 기준으로 중복 확인한다. Film·Location 연결이 바뀌어도 Set ID와 Asset ID는 유지한다.

## 10. Language Rules

- 화면명, 필드명, 상태값: 영어
- 섹션명·필드명 아래의 작은 설명, 입력 안내, 오류, 확인 문구: 한글
- 버튼과 클릭형 Action: 영문 단독 표기
- 영화명, LEGO 세트명, 장소명: 공식 원문 유지
- 공식 명칭과 코드값을 임의 번역하거나 축약하지 않는다.

## 11. Navigation Guardrail

메인 정보 구조는 영화와 이야기에서 출발해야 한다. `Collection`을 독립적인 메인 탐색축으로 확장하지 않는다.

하단 내비게이션 명칭과 순서는 `Home / Records / Assets / Pages`로 고정한다.

- 현재 배포본의 `Collection`은 LEGO Record 기능 구현 시 `Records`로 변경한다.
- 별도의 컬렉션 메인 메뉴를 만들지 않는다.
- 하단 내비게이션은 최상위 탐색 화면에만 표시한다.
- Add/Edit 등 집중 편집 화면에서는 하단 내비게이션을 숨긴다.

## 12. Deferred Decisions

다음 항목은 WWA Page 설계 단계 전까지 확정하지 않는다.

- Primary Book Placement
- WWA Page 순서
- 페이지별 이미지 크롭과 배치
- 한 세트의 대표 페이지
- 최종 Book 본문
- 같은 Asset을 여러 페이지에서 사용할 때의 개별 크롭값

또한 다음 항목은 별도 승인 전 구현하지 않는다.

- 승인되지 않은 자동 분류·자동 책 배치
