# WWA Manager Changelog

이 문서는 WWA Manager의 승인·구현·보류 상태를 기록한다. 최신 명시적 승인이 오래된 시안과 구현보다 우선한다.

## [Unreleased]

### iPhone Release Date Input and Official Check Feedback Implemented — 2026-08-12

- LEGO Record의 `Release Date` 입력을 iPhone에서 하이픈을 입력할 수 있는 전체 키보드로 변경해 승인된 `YYYY / YYYY-MM / YYYY-MM-DD` 부분 날짜를 모두 직접 기록할 수 있도록 수정.
- `Complete Official Check` 성공 즉시 Action을 `Official Check Complete`로 변경하고 버튼 바로 아래에 한글 완료 문구를 표시해 화면 아래의 Verification까지 이동하지 않아도 결과를 확인하도록 개선.
- 저장된 Verified Record를 다시 열 때 공식 주소·Release Date·Pieces·Checked On·Verification을 재확인해 완료 Action과 안내 문구를 복원.
- Set Number·Release Date·Pieces·Official Source를 검증 후 수정하면 `Review`로 되돌리고 `Checked On`과 완료 피드백을 초기화해 이전 검증 상태가 변경값에 남지 않도록 보강.
- 잘못된 공식 주소·날짜·Pieces는 기존 필드 오류와 함께 Official Check Action 바로 아래에도 오류 원인을 표시.

### Official Source and Catalog Attribution Guard Implemented — 2026-08-12

- LEGO Record의 `Verified`를 상태 Select에서 직접 선택하지 못하게 하고 `Complete Official Check`를 통해서만 전환하도록 변경하되, 최종 IndexedDB 저장 계층에서도 공식 출처를 동일하게 재검증하도록 보강.
- Official Source를 HTTPS `lego.com`의 현재 Set Number와 일치하는 공식 상품 페이지 또는 공식 조립 설명서로 제한하고, 일반 사이트·LEGO 검색 결과·다른 세트번호·HTTP·위장 도메인·Rebrickable 주소를 필드 오류로 차단.
- Rebrickable 적용 후 Set Name·Release Date·Pieces·Minifigures를 직접 수정하면 해당 항목을 `catalogImport.appliedFields`와 `Imported · Rebrickable` 표시에서 즉시 제거하고, 적용 항목이 모두 제거되면 Catalog Import 기록 전체를 제거하도록 수정.
- 기존 `catalogImport` 구조와 20개 IndexedDB Store·ZIP format version을 변경하지 않아 이전 Record와 Full ZIP Backup·Restore 호환성을 유지.
- 위험한 공식 출처 6종 차단, 일치하는 LEGO 상품 페이지·조립 설명서 허용, 수동 수정 출처 제거, API Key 비노출, `429` 재시도, 선택 적용, Stable ID, ZIP Backup·Restore와 `393px / 430px` 기존 화면 회귀를 통합 검증.

### Rebrickable Catalog Import Implemented — 2026-08-12

- `Add/Edit LEGO Record`의 Set Number 아래에 영문 단독 `Find Set Information` Action과 기기별 `Rebrickable · Connected` 상태를 추가하고, API Key 미연결 시 집중형 `Connect Rebrickable` 전체 화면으로 이어지도록 구현.
- Rebrickable 개인 API Key를 IndexedDB `settings`에만 저장하고 모든 요청에서 URL 파라미터 대신 `Authorization: key …` 헤더를 사용하며 Record·Asset·Outbox·GitHub 코드·Full ZIP Backup에서 제외.
- 최초 연결은 API Key 정상 확인 뒤에만 저장하고 `Reconnect`는 새 키 검증 성공 뒤 기존 키를 교체하며, `Remove Connection`은 기존 LEGO Record와 Catalog Import 기록을 유지한 채 키만 삭제.
- `76419` 입력을 Rebrickable의 `76419-1`로 조회하고, 세트 기본정보 뒤 약 1초 간격으로 Minifigure 목록을 순차 요청하며 `429` 안내시간 뒤 한 번 자동 재시도하도록 구현.
- 세트 기본정보만 성공하고 Minifigure 조회가 실패하면 기본정보를 보존한 `Retry Minifigures` Action을 제공하고, 실제 미니피겨 없음과 조회 실패를 별도 한글 상태 문구로 구분.
- Add 조회는 `Review Set Information`에서 Set Name·Release Year·Pieces·중복 제거 Minifigure 원문 이름 목록을 검토한 뒤 적용하고, Edit 조회는 `Review Record Updates`에서 변경 항목만 `Current Record / Rebrickable`로 비교해 항목별 선택 적용.
- 기존 값이 있으면 `Keep Current`, 비어 있으면 `Use Rebrickable`을 기본 선택하며, 더 정확한 부분 Release Date·Stable SET ID·Size·가격·구매정보·Classification·Official Source를 자동으로 낮추거나 덮어쓰지 않도록 유지.
- 실제 적용한 값이 있으면 `Record Verification = Review`, `Checked On = null`로 전환하고 최신 적용 기록만 `catalogImport(provider / providerSetNumber / sourceUrl / importedAt / appliedFields)`에 저장해 LEGO Record와 Verified ZIP에 포함.
- Rebrickable 원본 API 응답·이미지·Theme은 저장하지 않고, 적용 필드에는 `Imported · Rebrickable`을 표시하며 `Catalog Import` 전체 폭 목록에서 조회일과 적용 항목을 확인하도록 구현.
- `Official Check`에서 LEGO 공식 상품 페이지 또는 공식 조립 설명서 주소, 부분 Release Date, Pieces를 확인한 뒤 현재 날짜와 `Verified`를 기록하도록 연결하고, 단종 세트의 공식 조립 설명서 검증 예외를 유지.
- 버튼·클릭형 Action은 영문만, 새 섹션·필드는 영문 아래 작은 한글 설명, 오류·진행 안내는 해당 위치의 한글 문구로 구성하고 Add/Edit·Catalog Flow에서 하단 내비게이션을 계속 숨김.
- iPhone `393px / 430px` DOM·IndexedDB 통합 검증에서 최초 연결, Header 인증, `429` 재시도, 중복 Minifigure 이름 제거, 부분 실패 후 단독 재조회, 선택 적용, 공식 검증 전환, Stable ID 유지, API Key의 Record·URL·ZIP 비노출, Full Restore 후 Catalog Import 보존과 API Key 미복원을 확인.
- 기존 Add/Edit Record, Records List, Asset Detail, Home 영화 8개, Assets 필터 6개, IndexedDB 20개 Store와 기존 Full ZIP 검증 회귀 테스트를 통과했으며 Official Source·Catalog Attribution 안전 보강까지 포함해 GitHub `main`과 Pages에 배포.

### Add/Edit LEGO Record Implemented — 2026-08-12

- `LEGO Records` 상단에 44px 이상 `Add Record` 진입점을 추가하고, Add/Edit 집중 편집 화면에서 하단 내비게이션을 숨기도록 구현.
- `Record Identity / Official Information / My Archive / Archive Classification` 순서로 Set Number·Set Name, 부분 Release Date, Pieces, 공식 원문 Minifigure 이름 목록과 자동 Count, 구조화 Size, MSRP, 구매정보, Collection Scope, Verification·Lifecycle을 입력하도록 연결.
- 새 Record는 Local Archive의 `LegoRecord` 번호 블록을 원자적으로 소비해 `SET-000001` 형식의 Stable ID를 발급하고, Edit에서는 기존 `syncId / SET ID / createdAt`을 유지.
- 모든 공백을 제거하고 대소문자를 통일한 Set Number 기준으로 활성 generation 전체를 다시 확인해 중복 생성을 차단하며, Add/Edit Record와 Outbox·번호 소비를 하나의 IndexedDB transaction으로 저장.
- `Verified` 전환에는 Official Source, Checked On, 확인된 Release Date·Pieces를 요구하고, 오류를 해당 필드 아래 한글 문장과 `aria-invalid`로 표시.
- Minifigures는 중복을 제거한 공식 이름 목록으로 저장하고 Count를 자동 계산하며, Size와 MSRP·My Price는 값·단위를 분리하고 통화를 자동 환산하지 않도록 유지.
- Collection Plan의 기반값인 `Reference / Owned / Wanted`만 Record에 저장하고, 수집률·희소성·단종 임박도·구매 순위 화면은 승인된 별도 단계까지 구현하지 않음.
- `Primary Set Family = Unassigned`, `Primary Book Placement = Not placed`를 읽기 전용으로 유지해 미승인 자동 분류와 WWA Page 배치를 추가하지 않음.
- Active에서 Archived로 전환할 때만 확인창을 표시하고, 저장하지 않고 Back·Cancel 시 `Discard Changes` 확인을 적용. Record Detail 구현 전에는 기존 목록 행에서 Edit 화면으로 직접 진입하도록 연결.
- 기존 Migration Record의 숫자형 Minifigure Count, 문자열 Size, 단일값 Price를 다른 필드만 편집할 때 조용히 지우지 않고 원형 보존.
- iPhone `393px / 430px` DOM·IndexedDB 통합 검증에서 Add/Edit, `SET` ID 유지, Set Number 수정·중복 차단, Verified 조건, Archive·Discard 확인, transaction 충돌 롤백, Verified ZIP 내 LEGO Record 포함, 16px 입력, 44px 이상 터치 영역을 확인.
- 기존 Home 영화 8개, Records 검색·5개 필터·대표 Asset, Assets 필터 6개, IndexedDB 20개 Store, Detail 화면 하단 내비게이션 숨김과 `localStorage` 비접촉을 유지.

### LEGO Records List Implemented — 2026-08-11

- 하단 내비게이션을 승인된 `Home / Records / Assets / Pages` 순서로 전환하고 기존 임시 `Collection` 진입점을 첫 실제 `Records` 화면으로 교체.
- iPhone Archive Surface에 `LEGO Records` 제목, `Set Number / Set Name` 검색, `All / Draft / Review / Verified / Archived` 필터와 전체 폭 목록 행을 구현.
- 활성 Archive generation의 `legoRecords` Store만 읽고 공백 제거·대소문자 통일 Set Number 기준으로 중복 표시를 방지하며, 예시 Record를 만들지 않고 실제 저장 데이터만 표시.
- 각 행에 Set Number, Set Name, `Release Year · Pieces`, `Record Verification`, `Record Lifecycle`을 분리 표시하고 오른쪽 `›` 탐색 구조를 적용.
- `relatedLegoRecord` 관계로 연결된 LEGO 공식 제품 이미지, 공식 박스 이미지, 직접 촬영, Blueprint 순으로 대표 Asset을 선택하고 제품 전체가 잘리지 않는 `contain` 방식을 적용.
- 현재 깨끗한 Archive에서는 `00 LEGO Records`와 조용한 빈 상태를 표시하며 Add/Edit, Record Detail, Collection Plan의 작동하지 않는 진입점은 이번 화면에 추가하지 않음.
- iPhone `393px / 430px` DOM·IndexedDB 통합 검증에서 빈 상태, 저장 Record 복원, 검색, 5개 상태 필터, 대표 Asset, 16px 검색 입력, 44px 이상 터치 영역을 확인.
- 기존 Home 영화 8개, Assets 필터 6개, IndexedDB 20개 Store, Movie Detail·Asset Detail 하단 내비게이션 숨김과 `localStorage` 비접촉을 유지.

### Collection Plan Direction Approved — 2026-08-11

- 수집률, 희소성, 미보유 세트 구매 순위는 LEGO Record 본체나 하단 내비게이션에 섞지 않고 향후 `Records` 내부의 별도 `Collection Plan` 기능으로 관리.
- 미보유 세트도 별도 중복 데이터가 아니라 세트번호당 하나의 LEGO Record를 `Wanted` 범위로 관리하고 구매 후 같은 Record를 `Owned`로 전환해 기존 `SET` ID와 조사 이력을 유지.
- `Collection Progress`는 `Owned ÷ (Owned + Wanted)`로 계산하고 `Reference / Archived`는 현재 수집률에서 제외.
- `Purchase Priority`는 현재 단종되지 않은 Wanted 세트 중 공식 `Retiring Soon`, 예상 단종 시점, `High / Medium / Low / Unassessed` 단종 가능성 순으로 자동 정렬하며, 이미 단종된 세트는 `Retired Wanted`로 분리.
- `Rarity`는 단종 가능성과 별도 정보로 유지하고 동일한 단종 위험에서만 보조 정렬 기준으로 사용. 각 단종 판단에는 Availability, Retirement Risk, Expected Retirement, Retirement Source, Checked On을 함께 기록.
- Collection Plan 진입점은 해당 기능의 별도 설계·승인·구현 단계 전까지 화면에 추가하지 않음.

### Detail Navigation and Search Input Fix Implemented — 2026-08-11

- Assets Library 검색 입력값을 iPhone Safari 자동 확대 방지 기준인 실제 `16px`로 통일.
- Movie Detail과 Asset Detail을 최상위 탐색 화면이 아닌 집중 상세 화면으로 처리해 하단 내비게이션을 숨기고, Back으로 목록에 복귀하면 다시 표시하도록 수정.
- Add/Edit Asset과 Archive Data의 기존 하단 내비게이션 숨김 규칙, `Home / Assets / Collection / Pages` 배포 순서와 LEGO Record 구현 시 `Collection → Records` 전환 규칙은 변경하지 않음.

### Reset Test Archive Implemented — 2026-08-11

- 백업·복원 검증으로 생성된 테스트 Local Archive만 폐기할 수 있도록 Manager 내부 `Archive Data`에 `Reset Test Archive`를 추가.
- `RESET`을 정확히 직접 입력하기 전에는 실행 버튼을 비활성화하고, 실행 시 IndexedDB `WWA_Manager`의 Asset·Original·Preview·Version History·번호 예약값을 함께 삭제하도록 구현.
- ZIP 백업 파일, Safari의 다른 웹사이트 데이터, 기존 `localStorage`는 변경하지 않으며 삭제 성공 뒤 새 Archive ID와 새 번호 예약 구간을 만들도록 페이지를 다시 초기화.
- 다른 WWA Manager 탭이 데이터베이스 삭제를 막으면 해당 탭을 닫아야 한다는 상태를 화면에 표시하고, 현재 탭의 데이터베이스 연결은 삭제 전에 닫도록 구현.
- Archive Surface의 `Winter White #F7F9FC`, 3px 모서리, 44px 이상 터치 영역, 16px 확인 입력값과 영문 기능명·한글 안내 규칙을 유지.
- DOM·IndexedDB 통합 검증에서 테스트 Asset·Version·File 제거, 새 Archive ID 생성, `1–25 / next 1` 번호 예약, 오입력 차단, 다른 탭 차단 안내 후 삭제 계속, `localStorage` 비접촉을 확인.

### iPhone Full Restore Image MIME Fix Implemented — 2026-08-11

- iPhone 실기기에서 `WWA_Backup_2026-08-11_0340.zip`을 검증·복원한 뒤 `AST-000001` Record와 `Version 1`은 유지되지만 Original 미리보기가 표시되지 않는 문제를 확인.
- ZIP의 이미지 바이트와 `archive.json`의 `mimeType = image/png` 메타데이터는 정상이며, ZIP에서 추출한 무형식 Blob을 새 generation에 그대로 적재해 iPhone Safari가 이미지를 해석하지 못한 것이 원인으로 확정.
- 복원 적재 시 각 Original·Preview Blob에 저장된 MIME 형식을 다시 부여하고, MIME 메타데이터가 비어 있는 호환 백업은 승인된 이미지 확장자에서 안전하게 형식을 복원하도록 수정.
- 백업 형식과 checksum은 변경하지 않아 기존 `WWA_Backup_2026-08-11_0340.zip`을 새로 만들지 않고 수정 배포본에서 다시 검증·복원할 수 있도록 유지.
- DOM·IndexedDB 통합 검증에서 ZIP 추출 직후 `type = ""` 상태를 재현한 뒤 Original·Preview가 `image/png`로 복원되고, 원본 바이트·`AST-000001`·`Version 1`·다음 안전 번호 구간이 유지되는 것을 확인.
- 원격 `main` 커밋 `dab005d`에 수정 코드를 반영했으며, iPhone에서 기존 백업을 다시 복원해 Original·Preview·Version 이미지가 표시되는 것을 실기기로 확인.

### iPhone Backup Folder Approved — 2026-08-11

- WWA 백업 ZIP의 공식 보관 위치를 `iCloud Drive / WWA Backup` 폴더로 확정.
- iPhone Safari에서 `WWA_Backup_2026-08-11_0321.zip` 생성 후 앱 자체 검증 완료 상태를 확인.
- 해당 백업은 `0 files`인 빈 Archive 백업이므로 ZIP 생성·검증 성공만 확인한 것으로 기록하며, 실제 Asset·Version History가 포함된 복원 검증은 별도로 진행.
- iCloud Drive는 사용자가 백업 생성 때 선택하는 파일 보관 위치이며 앱 자동 동기화 계층으로 사용하지 않음.

### LEGO Record and Navigation Rules Approved — 2026-08-11

- 하단 내비게이션을 `Home / Records / Assets / Pages`로 확정하고, 현재 `Collection`을 LEGO Record 기능 구현 시 `Records`로 변경하되 별도 컬렉션 메뉴는 만들지 않기로 승인.
- LEGO Record 필수값을 `Set Number / Set Name`, 새 Record 기본값을 `Draft / Active / Unassigned / Not placed`로 확정.
- 모든 공백을 제거한 동일 Set Number를 중복으로 판단하며, Set Number 수정 시 중복 검사를 다시 수행하고 기존 `SET` ID는 유지하도록 확정.
- Record의 `Verified` 조건을 공식 출처, `Checked On`, 확인 가능한 핵심 공식 정보로 확정.
- 가격은 금액·통화를 분리하고 자동 환산하지 않으며, Release Date 부분 날짜, 실제 Purchase Date, 이름 목록 기반 Minifigure 수, `Height / Width / Depth + Unit` Size 구조를 승인.
- Location 중복은 전체 경로, Subject 중복은 `Subject Type + 공식 원문 이름`을 기준으로 확정.
- Story Connection 기본 상태를 `Review`, 중복 기준을 `Record + Film + Location + Subject + Narrative Scope` 조합으로 확정하고 기존 Verified Evidence·Checked On 조건을 유지.
- 이 체크포인트는 승인 규칙과 다음 작업 기준만 기록하며 `index.html`과 공개 배포본은 변경하지 않음.

### Local Archive Release Deployed — 2026-08-11

- 로컬 체크포인트의 파일 트리를 GitHub `main` 배포 커밋 `b7e3bc6`과 해시 단위로 대조해 동일함을 확인.
- 공개 GitHub Pages의 Home 8개 영화, 하단 내비게이션 4개, `Local Archive / Full ZIP Backup / Full Restore` 노출을 확인.
- Cloud Sync, Apple 로그인, Sync Now가 공개 화면에 노출되지 않는 것을 확인.
- 다음 개발 브랜치는 배포된 `origin/main`에서 시작하며, 이전 로컬 체크포인트 브랜치는 감사 이력으로 보존.

### Local Archive + Full ZIP Backup Adopted as Official Storage — 2026-08-11

- Apple Developer 유료 가입과 CloudKit 자동 동기화를 사용하지 않기로 한 최신 승인을 반영하고, 공식 저장 방식을 `Local-First + Full ZIP Backup`으로 변경.
- `Archive Data`에서 Cloud Sync·Apple 로그인·Queue·Conflict 안내를 숨기고, 이 기기의 IndexedDB 저장과 Verified ZIP 별도 보관을 설명하는 `Local Archive` 섹션으로 교체.
- 앱 시작 시 CloudKit 초기화와 외부 Script 로드를 실행하지 않으며, 저장 뒤 자동 동기화 예약도 비활성화.
- `settings`, `idReservations`, `assets`를 하나의 IndexedDB readwrite transaction으로 확인해 로컬 `AST` 번호 25개 블록을 예약하고, 남은 번호가 10개 이하일 때 다음 블록을 미리 확보하도록 구현.
- 현재 Asset Stable ID, 기존 예약 상한, ZIP 복원 high-water보다 큰 번호만 발급해 새 설치·기존 Archive·복원 뒤에도 번호를 재사용하지 않도록 유지.
- 기존 CloudKit 구현 코드와 호환 Store는 데이터 손실·schema 변경을 피하기 위해 비활성 상태로 보존하며 사용자 기능이나 공식 운영 방식으로 사용하지 않음.
- DOM·IndexedDB 통합 검증에서 새 Archive의 `AST-000001`부터 `AST-000015`까지 즉시 영구 저장, 남은 10개 시점의 `26–50` 블록 선예약, Verified ZIP 전체 복원 뒤 `AST-000051` 발급, `localStorage` 쓰기 0건을 확인.
- 정적 UI 회귀 검증에서 Home 영화 8개·Asset 필터 6개·하단 내비게이션 4개·고유 DOM ID를 유지하고 CloudKit 설정·외부 Script 주입 0건과 Cloud Sync 섹션 비노출을 확인.
- GitHub 반영과 GitHub Pages 배포는 포함하지 않음.

### Local Format, Offline, Restore, and Responsive Contract Validation Passed — 2026-08-10

- iPhone 기준 `393px / 430px`, iPad 기준 `768px`, PC 보조 기준 `1280px` 검증 매트릭스에서 `width=device-width`, `viewport-fit=cover`, 최대 `430px` 작업 Shell, 편집 입력값 `16px`, 최소 터치 영역 `44px`, Asset Editor·Archive Data의 하단 내비게이션 숨김 계약을 재확인.
- DOM·IndexedDB 통합 환경에서 JPEG 선택 직후 Add Asset Editor가 열리고 Preview가 준비되는지, 필수 Metadata 저장 후 `AST-000001`이 발급되는지, Archive Data 진입 시 Restore가 ZIP 검증 전까지 비활성화되는지 확인.
- JPEG 원본과 HEIC 형식 원본을 각각 저장해 Original의 MIME·확장자·바이트를 유지하고 HEIC 표시용 Preview는 JPEG로 분리되는지 확인. HEIC 디코딩은 WebKit 호환 디코더 경계를 재현한 검증이며 실제 iPhone Photos HEIC 선택·Canvas 변환은 실기기 검증 대상으로 유지.
- 모의 CloudKit Development 환경에서 `AST-000001`부터 `AST-000003`까지 중복 없이 발급하고 File → Version → Asset 순서 Push를 확인한 뒤, 네트워크 차단 상태에서도 세 번째 Asset이 IndexedDB와 Outbox에 남고 기존 Archive 열람이 유지되는지 확인.
- 오프라인 상태에서 생성한 전체 ZIP의 CRC32·SHA-256·Record·파일 참조 검증, 손상 ZIP 무변경 거부, 새 generation 전체 복원, 이전 generation rollback, 재복원·확정과 HEIC Original 형식 유지를 확인.
- 검증 전체에서 `localStorage.setItem / removeItem / clear` 호출 0건을 확인했으며 기존 승인 아키텍처·화면 코드는 수정하지 않음.
- Cloud Browser가 로컬 검증 페이지에 연결되지 않아 실제 브라우저 렌더 캡처는 수행하지 못함. 실제 iPhone·iPad·PC, iPhone Photos HEIC/JPEG, 서로 다른 Apple 계정 간 충돌과 CloudKit CKAsset 왕복은 container identifier·Web API token·실기기가 준비된 뒤 별도 검증해야 함.
- 이번 체크포인트는 로컬 검증 기록만 포함하며 GitHub 반영, GitHub Pages 배포, CloudKit Production schema 변경은 포함하지 않음.

### CloudKit Auto Sync and Conflict Preservation Implemented — 2026-08-10

- `window.WWACloudKitConfig`의 CloudKit container identifier·Web API token·Development/Production 환경값을 명시적 연결 경계로 추가하고, 값이 없거나 Apple 계정에서 로그아웃된 경우 IndexedDB를 그대로 사용하는 `Local Only` 모드로 유지.
- CloudKit private database의 `WWAArchive` custom zone, `WWAArchiveRoot / WWAEntity / WWAFile / WWAIDCounter / WWAIDReservation` Record 계약과 File의 `CKAsset` 전송·다운로드를 구현.
- 앱 시작·네트워크 복귀·로컬 저장·수동 실행을 동기화 진입점으로 연결하고, lease로 중복 실행을 막은 뒤 `pull → validate/apply → dependency-ordered outbox push → root` 순서와 sync token·`recordChangeTag`·지수 백오프 재시도를 적용.
- 서버의 entity별 high-water counter에서 기기별 25개 Stable ID 번호 블록을 원자적으로 예약하고, 남은 번호가 10개 이하일 때 다음 블록을 미리 확보하며 `Awaiting ID Reservation` Draft를 예약 후 자동 승격하도록 구현.
- Asset의 서로 다른 필드 동시 변경은 field clock 기준으로 병합하고, 같은 필드 동시 변경은 양쪽 값을 Conflict에 보존해 `Keep This Device / Use Cloud Value` 선택 전까지 push를 차단하도록 구현.
- 이미지 동시 교체는 양쪽 File·Version을 모두 유지하고 사용자가 현재 Version을 선택하게 하며, 선택 뒤에도 `Source Status = Review`, `Checked On = null` 규칙을 적용.
- Apple 계정 불일치, 지원하지 않는 schema, Stable ID와 Sync ID 불일치, generation 충돌, immutable Record 변경과 CloudKit 물리 삭제를 자동 덮어쓰기·삭제하지 않고 중대한 수동 검토 Conflict로 보존.
- `Archive Data`에 Local/Cloud 모드·Queue·Last Sync·Conflict 수, Apple 로그인/로그아웃, `Sync Now`, 충돌 비교·해결 UI를 추가하되 기존 하단 내비게이션 구조는 변경하지 않음.
- 모의 CloudKit·IndexedDB 통합 검증에서 인증, 1–25 및 26–50 예약, dependency push, Draft 자동 승격, 다른 필드 병합, 같은 필드 충돌, 양쪽 이미지 Version 보존, 일시적 네트워크 실패 재시도, root 무변경 재전송 방지, 계류 Queue·Conflict 0건을 확인하고, 별도 CloudKit JS 호출 계약 검증에서 private database·custom zone·배열형 `saveRecords`·root 1회 저장을 확인.
- 별도 Local Only 회귀 검증에서 CloudKit Script 미주입, Draft 보존, 기존 Home 영화 8개·필터 6개·하단 내비게이션 4개와 `localStorage` 쓰기 0건을 확인.
- 실제 Apple CloudKit Development container의 로그인·schema·CKAsset 왕복은 container identifier와 Web API token이 제공되지 않아 아직 실행하지 않았으며, 이번 체크포인트는 로컬 구현·검증만 포함하고 GitHub 반영과 GitHub Pages 배포는 포함하지 않음.

### Full ZIP Backup and Staged Restore Implemented — 2026-08-10

- Assets의 집중형 `Archive Data` 화면에 `Full ZIP Backup`과 `Full Restore` 진입점을 추가하고, 작업 중 하단 내비게이션을 숨긴 채 검증 상태·복원 범위를 먼저 보여주도록 연결.
- 활성 generation의 Asset·모든 Version·Original·Preview·Draft·관계·Tombstone·Outbox·Conflict·Migration Audit를 `manifest.json / archive.json / checksums.json / originals / previews`로 구성한 표준 저장형 ZIP으로 생성.
- 백업 파일명을 `WWA_Backup_YYYY-MM-DD_HHmm.zip`으로 고정하고, 생성 직후 ZIP 구조·CRC32·SHA-256·schema·Record 및 파일 수·Stable ID·관계와 파일 참조를 다시 검증한 경우에만 다운로드하도록 구현.
- 예약된 25개 번호 블록의 미사용 범위까지 Stable ID high-water에 포함해 백업·복원·rollback 이후에도 발급된 번호가 재사용되지 않도록 보존.
- 손상·누락·중복 경로·지원하지 않는 Store·세대 불일치·참조 단절 백업을 기존 IndexedDB 변경 전에 거부하고, 검증 완료 전 Restore 버튼을 활성화하지 않도록 적용.
- 검증된 전체 Archive를 새 generation에 하나의 IndexedDB transaction으로 적재한 뒤에만 `activeGenerationId`를 전환하고, 이전 generation은 즉시 삭제하지 않은 채 화면 복원 실패 시 되돌릴 수 있도록 구현.
- CloudKit 동기화 진행 상태는 staging 적용 전과 적용 transaction 안에서 다시 확인해 Restore를 차단하고, 복원된 전역 key Record는 기존 generation과 충돌하지 않도록 새 ID와 원본 감사 ID를 함께 보존.
- DOM·IndexedDB 통합 검증에서 자체 검증 ZIP 생성, 손상 ZIP 무변경 거부, 동기화 중 차단, 새 generation 전체 교체, 이전 generation 보존·rollback, Stable ID high-water `25`, reservation 종료, legacy migration 멱등성, 기존 Home 8개 영화·필터 6개·하단 내비게이션 4개와 `localStorage` 비접촉을 확인.
- 이번 체크포인트는 로컬 구현·검증만 포함하며 GitHub 반영과 GitHub Pages 배포는 포함하지 않음.

### Legacy localStorage Read-Only Migration Implemented — 2026-08-10

- 앱 시작 시 현재 키 `wwa-manager-film-contents-v01`과 이전 `v04 / v03 / v02 / v01 / v1` 키를 승인된 우선순위로 읽되 `localStorage`에는 쓰기·수정·삭제하지 않는 read-only migration을 연결.
- 선택한 원본 JSON 바이트의 SHA-256을 `migrationLog`에 기록하고 같은 source checksum은 Preview 재생성이나 Record 중복 없이 `already-migrated`로 종료하는 멱등성 적용.
- legacy LEGO Record를 세트번호 기준으로 한 원본 Record로 묶고, 기존 활성 LEGO Record와 겹치면 사용자 Record를 덮어쓰지 않은 채 기존 Sync ID를 관계 대상으로 유지하고 migration conflict audit에 기록.
- `legoOfficial / boxArt / blueprint / movieStill / myPhoto` Data URL을 승인된 Asset Type으로 매핑하고, 원본 Blob·Preview·Asset Version·Asset·Related LEGO Record Link·Outbox를 번호 예약이 있을 때 하나의 transaction에서 분리 저장.
- legacy Data URL은 `reason = legacy-import`, 품질 경고, source checksum을 유지하며 Draft가 번호 예약을 기다리거나 사용자가 유형을 검토한 뒤 영구 저장돼도 해당 감사정보가 보존되도록 적용.
- 번호 예약이 없으면 승인된 유형 이미지는 `Awaiting ID Reservation` Draft로 보존하고, 유형을 단정할 수 없는 `minifigures`와 미지정 이미지는 자동 분류 없이 `Migration Review · Type Needed` Draft로 Assets의 `All`에서 확인·편집 가능하도록 연결.
- Migration Review 편집은 승인된 6개 Asset Type 중 하나와 출처를 사용자가 직접 선택해야 저장되며, 기존 Add/Edit 필드 검증과 16px 입력·집중 편집 화면·하단 내비게이션 숨김 규칙을 재사용.
- Stable ID·File·Version·Link 충돌을 migration 전체 실패로 처리하고, 강제 충돌 검증에서 새 LEGO Record·Asset·Version·File·Link·Outbox·번호 소비·migration log가 모두 롤백되는지 확인.
- DOM·IndexedDB 통합 검증에서 현재 키 우선 선택, 구형 키 fallback, Draft/영구 분기, `AST-000001` 소비, File 재사용, `legacy-import` 유지, checksum 재실행, 기존 Home 8개 영화·기본 Asset 9개·필터 6개·하단 내비게이션 4개 보존을 확인.
- `localStorage.setItem / removeItem / clear` 호출 0건과 원본 JSON 완전 보존을 확인했으며, 이번 체크포인트는 로컬 구현·검증만 포함하고 전체 ZIP 백업·복원, GitHub 반영, 배포는 포함하지 않음.

### Edit Asset Mobile Validation Passed — 2026-08-10

- iPhone 기준 `393px`과 최대 검증 폭 `430px`에서 실제 저장 Asset의 `Edit Metadata → Replace Image → Version History` 흐름을 DOM·IndexedDB 통합 환경으로 재현.
- 두 폭 모두 Edit 화면의 입력값 `16px`, 버튼·입력 터치 영역 최소 `44px`, 편집 중 하단 내비게이션 숨김, Editor·Action 영역 가로 넘침 없음 확인.
- 읽기 전용 `AST` ID 유지, Metadata 저장 후 Detail 복귀, Image 교체 후 `Current / Preserved` 두 Version 표시와 `Source Status = Review` 전환 확인.
- 인라인 Script 구문, 고유 DOM ID, iPhone viewport meta, `430px` Shell 상한과 편집 화면 폭 계산을 함께 검증.
- 로컬 파일은 Cloud Browser 보안 정책상 직접 열 수 없어 공개 Pages 실제 브라우저 검증은 GitHub 반영 후 배포본에서 수행.

### Edit Metadata, Replace Image, and Version History Implemented — 2026-08-10

- 영구 Asset의 `Asset ID`와 현재 이미지 Version을 유지한 채 Title·Type·Source·검증 상태·Production 상태·Notes·Archive Relations를 편집하는 집중형 `Edit Asset Metadata` 화면을 기존 Add Asset 화면 체계 안에 연결.
- Metadata 저장 시 실제로 변경된 필드의 `fieldClocks`만 갱신하고, Asset과 Outbox를 같은 IndexedDB transaction에 저장하며 이미지가 바뀌지 않으면 새 Asset Version을 만들지 않도록 구현.
- 관계 추가는 결정적 Link ID로 저장하고 관계 제거는 원본 Record를 지우지 않는 Tombstone으로 보존하며, Asset·관계·Outbox 변경을 하나의 transaction에서 처리.
- `Archived` 전환 전 확인창을 표시하고 Original·Version History가 유지됨을 안내하며, Draft는 기존 Original과 입력값을 유지한 채 다시 저장하거나 번호 예약 후 영구 Asset으로 전환할 수 있도록 편집 흐름을 연결.
- `Replace Image`에서 교체 전 확인창을 표시하고 `In Use` Asset이면 현재 WWA Page 사용 상태를 함께 안내하도록 구현.
- 교체 Image의 새 Original·최대 1600px Preview·새 `assetVersions` Record·Asset 현재 Version 포인터·Outbox를 하나의 transaction에 저장하고, 이전 Version과 파일·Stable ID·Archive Relations는 유지하도록 구현.
- Image 교체 직후 `Source Status = Review`, `Checked On = null`을 적용하고 동일한 현재 원본을 다시 선택하면 저장하지 않도록 제한.
- Asset Detail에 `Version History`를 추가해 `Version 1 / Version 2`, `Current / Preserved`, 생성 이유, 원본 파일명, 크기, 용량, 생성 시각을 표시.
- IndexedDB·DOM 통합 검증에서 변경 필드 시계, 관계 Tombstone·추가, Metadata 편집 시 Version 불변, Replace 시 부모 Version 연결, Stable ID·관계 유지, 검증 상태 초기화, Discard Changes, 교체 취소, Version History 표시를 확인.
- Version 생성 충돌을 강제로 발생시켜 교체 중 생성된 File·Version·Outbox와 현재 Version 포인터가 함께 롤백되는지 확인.
- 기존 Home 8개 영화·기본 Asset 9개·승인된 6개 필터 버튼·하단 내비게이션·고유 DOM ID·`localStorage` 비접촉을 확인.
- 이번 체크포인트는 로컬 구현과 검증만 포함하며 GitHub 반영과 GitHub Pages 배포는 포함하지 않음.

### WWA Filter Deployment Verified — 2026-08-10

- 원격 `main`의 `f1e4c9c`가 승인된 `index.html`, `CHANGELOG.md` 두 파일만 변경한 커밋인지 재확인.
- 공개 GitHub Pages가 `200 OK`로 응답하고 배포 HTML의 SHA-256 `e67de52d73d38530262992fb24e3976b7c83536fe9cc264a44b16456dd681114`가 해당 승인본과 일치하는지 확인.
- 배포 HTML에서 `WWA` 필터와 Add Asset Draft 복원 표식을 확인했으며 이 배포 확인 자체에는 코드 변경이나 재배포가 없음.

### WWA Filter Restored and Discard Changes Reverified — 2026-08-10

- 승인된 Asset Type 필터 `Film / LEGO / Blueprint / My Photo / WWA` 중 배포본에서 누락된 `WWA` 버튼을 Assets Library에 복구.
- 기존 `WWA Original → WWA` 분류·렌더링 로직은 변경하지 않고, 누락된 필터 진입점만 추가.
- Add Asset 편집 화면의 Back·Cancel에서 `Discard Changes` 확인 취소 시 입력과 미리보기를 유지하고, 확인 시 미저장 Asset을 제거한 뒤 Library로 복귀하는 흐름을 재검증.
- 이번 체크포인트는 로컬 수정·검증만 포함하며 GitHub 반영과 GitHub Pages 배포는 포함하지 않음.

### Add Asset Safe Integration Verified — 2026-08-10

- 원격 최신 `main` 커밋 `74f46ef`을 다시 확인하고, Add Asset Transaction·UI·IndexedDB 복원 구현이 해당 배포 기준에서 직접 이어지는지 대조.
- 원격 대비 변경 파일을 `index.html`, `CHANGELOG.md` 두 개로 한정하고 `WWA_PROTOCOL.md`, `DESIGN_GUIDE.md`, `STORAGE_SYNC_DESIGN.md`는 변경하지 않음.
- 기존 Home·Assets Library·Asset Detail·Movie Detail·하단 내비게이션 DOM이 원격 배포본과 동일하게 보존되는지 확인.
- 실제 `index.html`을 JSDOM·IndexedDB 표준 모의 환경에서 실행해 20개 Store, 세 식별자 재사용, Draft 복원, `AST-000001`·`AST-000002` 발급, Original 중복 방지, 관계·Outbox 저장을 검증.
- Stable ID 충돌을 강제로 발생시켜 파일·Version·Asset·관계·Outbox·예약 번호가 함께 롤백되는지 확인.
- 손상된 Asset Record 복원 실패 시 기존 Library가 교체되지 않고 유지되며, Blob URL 해제와 `localStorage` 비접촉이 유지되는지 확인.
- 이번 체크포인트는 원격 최신본 대조·로컬 안전 통합·검증만 포함하며 GitHub Push와 GitHub Pages 배포는 포함하지 않음.

### IndexedDB Asset Restore Implemented — 2026-08-10

- 앱 시작 시 `archiveRoots.activeGenerationId`를 다시 확인하고 해당 Generation의 영구 Asset·현재 Version·Original·Preview·Asset Link와 ID 예약 대기 Draft만 읽도록 구현.
- 영구 Asset은 저장된 Preview Blob으로 Assets Library 이미지를 복원하고, Asset Detail은 별도 Original Blob과 저장된 Source·Checked On·Original 파일명·형식·크기·해상도·Notes·Archive Relations를 표시하도록 연결.
- ID 예약이 없는 Draft는 `Draft · ID Reservation Needed`로 구분하고 Draft의 Preview·Original 정보·메타데이터·관계를 새로고침 후 복원하도록 구현.
- 영구 Asset은 Stable ID와 Sync ID를 함께 기준으로 중복 표시를 방지하고, Draft는 Draft ID 기준으로 한 번만 표시하도록 구현.
- 복원용 Blob URL을 반복 복원 뒤 교체하고 실제 페이지 종료 시 해제하되, 브라우저 뒤로가기 캐시가 유지되는 경우에는 즉시 해제하지 않도록 관리.
- 현재 Version·Original·Preview 등 필수 저장 Record가 불완전하면 복원 결과로 기존 Library를 교체하지 않고 기존 화면을 유지하도록 구현.
- 새로고침 상당의 IndexedDB DOM 통합 검증에서 기본 9개 자료 보존, 영구 Asset·Draft 복원, Stable ID·Sync ID 중복 제거, Detail Original·메타데이터·관계 표시, 세 식별자 유지, 반복 복원 Blob URL 해제, 실패 시 기존 목록 유지, `localStorage` 비접촉을 확인.
- 이번 체크포인트는 로컬 복원 구현과 해당 범위 검증만 포함하며 GitHub 반영과 배포는 포함하지 않음.

### Add Asset UI Connected — 2026-08-10

- 이미지 선택 직후 Archive Surface의 집중형 `Add Asset` 입력 화면을 열고, 편집 중 하단 내비게이션을 숨기도록 구현.
- `Image`, `Asset Title`, 승인된 6개 `Asset Type`, `Source`를 필수 입력으로 구성하고 공식 자료의 `Source Link`, `Verified` 상태의 `Checked On` 조건을 필드 아래 한글 오류로 표시하도록 구현.
- `Source Needed`와 `Candidate`를 기본 상태로 적용하고, 새 Asset에서 `In Use`를 직접 선택하지 못하도록 제한.
- `Related LEGO Record`, `Film`, `Location`, `Subject`는 IndexedDB의 실제 활성 Record를 읽어 선택하도록 연결하고, Record가 없으면 관계 입력만 비활성화한 채 관계 없이 저장할 수 있도록 구현.
- 저장 중 Back·Cancel·Save를 잠가 중복 실행을 방지하고, 통합 transaction 완료 뒤에만 `Saved · AST-000001` 또는 `Draft · ID Reservation Needed`를 표시하도록 연결.
- 저장 성공 결과는 현재 세션의 Assets Library에 반영하되, 새로고침 후 IndexedDB 복원은 다음 체크포인트로 분리.
- 저장하지 않고 나갈 때만 `Discard Changes` 확인을 표시하고, 취소하면 이미지와 입력값을 그대로 유지하도록 구현.
- DOM 통합 검증에서 Draft 저장, 영구 ID 1회 소비, Original·Preview·Version·관계·Outbox 생성, 공식 자료·Verified 필드 검증, 중복 제출 방지, Discard Changes, 승인된 6개 Asset Type, 고유 DOM ID와 `localStorage` 비접촉을 확인.
- 이번 체크포인트는 로컬 UI 연결과 해당 범위 검증만 포함하며 새로고침 복원, GitHub 반영과 배포는 포함하지 않음.

### Add Asset Transaction Implemented — 2026-08-10

- 필수값과 승인된 6개 Asset Type을 검증하고, 공식 자료의 `Source Link` 및 `Verified` 상태의 `Checked On` 필수 조건을 적용.
- Original 바이트를 변환하지 않고 SHA-256·파일 메타데이터를 계산하며, 최대 1600px 표시용 Preview를 별도 생성하도록 구현.
- 유효한 기기별 Asset 번호 예약이 있을 때만 `AST-000001` 형식의 Stable ID를 소비하도록 구현.
- 번호 예약이 없으면 Stable ID를 임의 발급하지 않고 Original·Preview·metadata·relations를 `drafts` 한 건으로 보존하도록 구현.
- `files`, `assetVersions`, `assets`, `assetLinks`, `outbox`, 번호 소비를 하나의 IndexedDB transaction으로 묶고 완료 뒤에만 `saved` 결과를 반환하도록 구현.
- 같은 SHA-256·역할의 파일은 활성 generation에서 재사용하고, 관계 중복은 결정적 Link ID로 방지하도록 구현.
- Preview 실패와 transaction 중간 실패 시 파일·Version·Asset·관계·Outbox·번호 소비가 남지 않는 롤백 검증 완료.
- 이번 체크포인트는 저장 API만 구현했으며 Add Asset UI 연결, 새로고침 후 Library 복원, GitHub 반영과 배포는 포함하지 않음.

### IndexedDB Foundation Deployed — 2026-08-09

- 원격 `main`의 승인된 Home·Assets·이미지 업로드 흐름을 유지한 상태로 `WWA_Manager` schema version `1` 기반을 배포.
- 승인된 20개 Object Store와 key path·index 검증을 적용.
- `deviceId`, `archiveId`, `activeGenerationId`를 최초 생성하고 이후 실행에서 재사용하도록 적용.
- 기존 화면 데이터와 `localStorage`를 읽거나 변경하지 않음.
- Add Asset의 Original·Preview·Asset Version·metadata 영구 저장은 다음 구현 단계로 유지.

### Baseline Approved — 2026-08-09

- `WWA_PROTOCOL.md`, `DESIGN_GUIDE.md`, `CHANGELOG.md`의 현재 내용을 WWA Manager 공식 기준으로 확정.
- 이후 설계·구현은 세 문서를 먼저 따르며, 최신 명시적 승인 없이 확정된 아키텍처와 디자인 규칙을 변경하지 않음.
- 저장 방식과 하단 내비게이션 최종 명칭 등 `Pending Decision` 항목은 이번 승인에 포함하지 않음.

### Added — 2026-08-09

- `WWA_PROTOCOL.md`, `DESIGN_GUIDE.md`, `CHANGELOG.md`를 공식 저장소 기준 문서로 추가.
- LEGO Record별 `Story Connections – Context Mapping` 구조 승인.
- Film–Location–Subject 관계, 복수 Evidence, `Review / Verified`, Connection Note, Checked On 규칙 정리.
- `Assets Library + Asset Detail` 통합 화면 방향 승인.
- Asset의 Archive Relations, Direct Links, Evidence Usage 표시 규칙 정리.
- Add Asset의 필수·선택 입력, 기본 상태, `AST-000001` 형식의 고정 ID 발급 규칙 승인.
- Edit Asset과 Replace Image의 버전 보존, 검증 초기화, 관계 유지 규칙 승인.
- Asset Type 6종과 목록 필터 그룹을 최종 확정.
- `STORAGE_SYNC_DESIGN.md`를 저장·동기화 공식 구현 계약으로 승인.

### Changed — 2026-08-09

- `Winter White`를 `#F8F8F5`에서 `#F7F9FC`로 변경하고 이전 값을 폐기.
- Assets와 LEGO Record 작업 화면을 밝은 Archive Surface로 통일.
- 연결 행의 `Open` 텍스트 버튼을 제거하고 행 전체 터치 + 오른쪽 `›` 방식으로 변경.
- Story Connection의 Evidence를 한 장 제한이 아닌 복수 선택으로 확정.
- 이미지 교체 후 `Source Status`를 `Review`로 변경하고 `Checked On`을 초기화하도록 확정.
- `Archive`를 삭제가 아닌 복원 가능한 Lifecycle 상태로 확정.

### Asset Add/Edit Rules Locked — 2026-08-09

- `Asset ID`는 읽기 전용이며 발급 후 변경하거나 재사용하지 않음.
- 이미지 교체 시 기존 `AST` ID와 Story Connection·WWA Page 등 모든 연결 관계 유지.
- 이전 이미지는 `Version History`에 보존.
- 전혀 다른 이미지는 Replace Image가 아닌 새 Asset으로 등록.
- 저장하지 않고 나갈 때 `Discard Changes` 확인.
- `In Use` Asset 교체 시 사용 중인 WWA Page가 있음을 확인창에 표시.

### Asset Types Locked — 2026-08-09

- `Official Film Still`
- `LEGO Official Product Image`
- `LEGO Official Box Image`
- `Official Blueprint`
- `My Photography`
- `WWA Original`
- 목록 필터는 `Film / LEGO / Blueprint / My Photo / WWA`로 고정.
- `Film Poster`, 캡처 이미지, AI 생성물은 Asset Type에서 제외.

### Storage Architecture Locked — 2026-08-09

- WWA 저장 구조를 `Local-First + CloudKit Auto Sync + Full ZIP Backup`으로 확정.
- 각 기기의 주 저장소는 `IndexedDB`로 고정하고 저장 시 네트워크보다 로컬 완료를 우선함.
- Asset 원본, 표시용 Preview, 모든 `Version History`를 보존 대상으로 확정.
- CloudKit을 iPhone·iPad·PC 사이의 승인된 자동 동기화 계층으로 추가.
- 오프라인 변경은 로컬에 유지하고 앱 실행 중 또는 다음 실행 시 동기화하도록 확정.
- 동기화 충돌 시 조용한 덮어쓰기를 금지하고 이미지 양쪽 버전을 보존하도록 확정.
- CloudKit과 독립된 전체 ZIP 백업을 장기 보존 수단으로 유지.
- GitHub Pages와 `index.html` 단일 파일 구조는 그대로 유지.
- 네이티브 앱 전환은 보류하며, 필요 시 같은 CloudKit 데이터를 사용하는 추가 클라이언트로 별도 검토.

### Storage Sync Contract Locked — 2026-08-09

- 사용자용 Stable ID와 내부 Sync ID를 분리.
- Asset 이미지와 Version 파일을 별도 Record로 저장.
- Stable ID를 기기별 25개 번호 블록 예약 방식으로 발급.
- CloudKit private database의 `WWAArchive` custom zone과 공통 `WWAEntity` envelope 사용.
- ZIP 복원은 새 generation에 검증·적재한 뒤 활성 generation을 전환하는 전체 교체 방식으로 제한.
- 기존 압축 Data URL은 `legacy-import`로 보존하고 원본 화질로 표시하지 않음.
- 유형이 불명확한 기존 `minifigures` 이미지는 자동 분류하지 않고 Migration Review Draft로 보존.

### IndexedDB Foundation Implemented — 2026-08-09

- 단일 `index.html` 안에 `WWA_Manager` IndexedDB schema version `1` 기반을 구현.
- 승인된 20개 Object Store와 전체 key path·index를 생성하고, 열린 데이터베이스의 schema가 계약과 일치하는지 검증하도록 구현.
- 최초 실행 시 `deviceId`, `archiveId`, `activeGenerationId`를 하나의 IndexedDB transaction에서 생성하고 이후 실행에서 그대로 재사용하도록 구현.
- 반복 초기화 시 `settings`와 `archiveRoots` Record가 중복 생성되지 않도록 구현.
- 이번 기반 단계는 기존 `localStorage` 데이터를 읽어 이전하거나 변경하지 않으며, 기존 화면 데이터 흐름도 전환하지 않음.
- IndexedDB 초기화 실패를 저장 성공으로 표시하지 않고 오류 상태로 남기도록 구현.
- Add Asset의 Original·Preview·Version·metadata 통합 저장과 목록 복원은 다음 구현 단계로 유지.

### Design Rules Locked — 2026-08-09

- 화려한 상태 배지 사용 제외.
- 지팡이, 반짝임, 별, 두루마리 등 장식 아이콘 사용 제외.
- 얇은 단색 기능형 선 아이콘과 Patronus Blue 체크 사용.
- 기본 모서리 `3px`, 기본 그림자 없음, 최소 터치 영역 `44px`.
- Cinematic Surface와 Archive Surface의 두 화면 체계 적용.
- 화면명·필드명·상태값은 영어, 설명·오류·확인 문구는 한글로 표시.

### Pending Decision

- `Primary Book Placement`, WWA Page 순서, 대표 페이지, 페이지별 크롭·배치는 WWA Page 단계까지 보류.
- Set Family 목록은 별도 승인 전 자동 분류하지 않고 `Unassigned`를 유지.

### Next

1. iPhone용 LEGO Record Detail 설계·승인·구현·검증
2. Records 내부 Collection Plan 별도 설계·승인·구현
3. Story Connections 통합
4. WWA Page 구현
5. iPhone 전체 기능·저장·백업 최종 검증 후 iPad 지원·PC 보조 화면 마무리

## [Repository Checkpoint] — 2026-08-09

### Deployed

- Assets Library, Asset Detail, iPhone 이미지 선택·미리보기 흐름.
- 새 업로드의 임시 `NEW-ASSET` Record.
- Asset Source 상태 예시: `Source Needed / Review / Verified`.
- Asset Production 상태 예시: `Candidate / In Use`.
- `WWA_Manager` IndexedDB schema version `1`과 승인된 20개 Object Store.
- `deviceId · archiveId · activeGenerationId` 최초 생성과 재실행 유지.

### Not Yet Implemented

- 영구 Asset ID 발급.
- 새로고침 후 Asset과 이미지 유지.
- Add/Edit 저장 폼.
- Replace Image 버전 보존.
- Archive Relations와 Evidence Usage 데이터 연결.
- Add Asset Original·Preview·Asset Version·metadata IndexedDB 영구 저장.
- CloudKit 자동 동기화와 충돌 보존.
- 전체 ZIP 백업·복원.

## [0.6] — 2026-08-08

### Released

- 영화 중심 Home archive.
- 승인된 연속형 Home 장면과 Chapter typography.
- GitHub Pages 배포.

## Earlier Approved Foundations

- `index.html` 단일 파일과 GitHub Pages 구조.
- iPhone 우선, iPad 지원, PC 보조.
- Movie를 이야기 탐색 기준으로 사용.
- LEGO Record는 세트번호별 원본 한 개만 생성.
- 한 Record에 여러 영화·장소 연결 허용.
- `Narrative Scope`: `Scene / Location / Subject / Cross-Film / Archive`.
- Set Family와 영화 분류 분리.
- Primary Book Placement는 WWA Page 단계 전까지 `Not placed`.
- WWA Page 전에는 페이지 순서·대표 페이지·최종 크롭을 확정하지 않음.
