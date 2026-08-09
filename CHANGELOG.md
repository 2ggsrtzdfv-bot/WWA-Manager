# WWA Manager Changelog

이 문서는 WWA Manager의 승인·구현·보류 상태를 기록한다. 최신 명시적 승인이 오래된 시안과 구현보다 우선한다.

## [Unreleased]

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

- 하단 내비게이션 최종 명칭과 순서.
  - 과거 승인안: `Contents / Records / Assets / Pages / Settings`
  - 최근 시안안: `Home / Archive / Assets / WWA Page / Settings`
  - 현재 배포본: `Home / Assets / Collection / Pages`
  - 별도 승인 전 명칭이나 순서를 변경하지 않음.

### Next

1. Add Asset 저장·복원 흐름의 iPhone 검증과 승인된 체크포인트 배포
2. Edit Metadata·Replace Image·Version History 구현
3. 기존 `localStorage` 읽기 전용 이전과 전체 ZIP 백업·복원 구현
4. CloudKit 자동 동기화·충돌 보존 구현
5. HEIC/JPEG, 오프라인, 충돌, 393px·430px·iPad 검증
6. LEGO Record·Story Connections 통합

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
