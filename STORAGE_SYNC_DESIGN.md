# WWA Storage and Backup Design

Last updated: 2026-08-12
Status: Approved — Local Archive Operating Contract

이 문서는 승인된 `Local-First + Full ZIP Backup` 아키텍처의 공식 구현 계약이다. Apple Developer 유료 가입과 CloudKit 자동 동기화는 사용하지 않는다. 기존 CloudKit 구현 계약과 호환 Store는 데이터 구조 보존을 위해 이 문서에 남기지만 실행하거나 사용자 화면에 노출하지 않는다. 이 운영 경계가 아래의 과거 CloudKit 세부 계약보다 우선한다.

## 1. Design Goals

- 모든 저장은 네트워크 연결 없이 IndexedDB에서 완료한다.
- Asset 원본, Preview, Version History를 서로 분리해 보존한다.
- `AST-000001` 형식의 ID는 발급 후 변경하거나 재사용하지 않는다.
- Local Archive의 단일 활성 generation과 모든 Version History를 보존한다.
- 오프라인 상태에서도 로컬 열람과 기존 데이터 편집이 가능해야 한다.
- 전체 ZIP은 앱 계정이나 서버에 의존하지 않는 장기 보존본이어야 한다.
- 현재 `localStorage` 데이터는 이전과 백업 검증이 끝날 때까지 삭제하지 않는다.
- GitHub Pages와 단일 `index.html` 구조를 유지한다.

## 2. Identity Model

각 영구 Record는 두 종류의 식별자를 가진다.

| 식별자 | 예시 | 용도 |
|---|---|---|
| Stable ID | `AST-000001` | 사용자에게 표시되는 영구 Archive ID |
| Sync ID | UUID | IndexedDB 기본키·Record 관계·Version 추적 |

- Stable ID는 발급 후 수정하거나 재사용하지 않는다.
- Sync ID도 생성 후 변경하지 않는다.
- Stable ID와 Sync ID의 1:1 관계를 유지한다.
- 이미지 버전, 파일, 관계 레코드는 내부 UUID 또는 결정적 해시 ID를 사용한다.
- 관계 레코드는 같은 연결이 두 기기에서 동시에 생성되어도 중복되지 않도록 연결 양끝과 관계 종류로 결정적 ID를 만든다.

### Sequential Stable ID Allocation

Local Archive는 IndexedDB의 high-water와 기존 Asset·예약 범위를 함께 확인해 작은 번호 블록을 예약한다.

- 기본 예약 블록: 25개
- 남은 번호가 10개 이하가 되면 앱 사용 중 다음 블록을 자동 예약한다.
- 번호 블록 예약은 `settings`, `idReservations`, `assets`를 포함한 단일 IndexedDB readwrite transaction으로 처리한다.
- ZIP 복원에서 가져온 최대 발급 번호와 이전 예약 상한보다 큰 번호만 새로 예약한다.
- 예약 블록 안에서는 오프라인에서도 즉시 Stable ID를 발급할 수 있다.
- 사용되지 않은 예약 번호는 다시 낮추거나 재배정하지 않는다. 번호의 연속성보다 중복·재사용 방지를 우선한다.

## 3. IndexedDB Database

- Database name: `WWA_Manager`
- Initial schema version: `1`
- 모든 영구 Archive Record에는 `archiveId`, `generationId`, `syncId`, `createdAt`, `updatedAt`, `deletedAt`을 공통으로 둔다.
- `deletedAt`은 관계·Record의 논리 삭제를 보존하는 Tombstone이다. `Archived` 상태와 동일하지 않다.
- 모든 날짜는 UTC ISO 8601 문자열로 저장한다.
- Blob 해시에는 SHA-256을 사용한다.

### Object Stores

| Store | Key | 역할 | 주요 Index |
|---|---|---|---|
| `archiveRoots` | `archiveId` | 활성 데이터 세대와 schema version | `activeGenerationId` |
| `settings` | `key` | 기기별 설정·저장 상태 | 없음 |
| `drafts` | `draftId` | 영구 ID 발급 전 Add/Edit 임시 저장 | `entityType`, `updatedAt` |
| `films` | `[generationId, syncId]` | Film 원본 Record | `stableId` |
| `locations` | `[generationId, syncId]` | Location 원본 Record | `stableId`, `parentId` |
| `subjects` | `[generationId, syncId]` | Subject 원본 Record | `stableId`, `type` |
| `legoRecords` | `[generationId, syncId]` | 세트번호별 LEGO Record | `stableId`, `setNumber` |
| `assets` | `[generationId, syncId]` | Asset 메타데이터와 현재 버전 포인터 | `stableId`, `type`, `sourceStatus`, `productionStatus`, `updatedAt` |
| `assetVersions` | `[generationId, versionId]` | 최초 이미지와 모든 교체·충돌 버전 | `assetSyncId`, `createdAt`, `parentVersionId` |
| `files` | `[generationId, fileId]` | 원본·Preview Blob과 파일 메타데이터 | `sha256`, `role`, `createdAt` |
| `assetLinks` | `[generationId, linkId]` | Asset의 Related Record·Direct Film·Location·Subject 연결 | `assetSyncId`, `targetSyncId`, `relationType` |
| `storyConnections` | `[generationId, syncId]` | Film–Location–Subject Context Mapping | `stableId`, `status`, `legoRecordSyncId` |
| `evidenceLinks` | `[generationId, linkId]` | Story Connection과 Evidence Asset의 다대다 연결 | `connectionSyncId`, `assetSyncId` |
| `pages` | `[generationId, syncId]` | 향후 WWA Page Record | `stableId`, `updatedAt` |
| `pagePlacements` | `[generationId, placementId]` | 페이지별 Asset 배치·크롭 관계 | `pageSyncId`, `assetSyncId` |
| `idReservations` | `reservationId` | 기기에 예약된 Stable ID 범위 | `entityType`, `deviceId`, `nextValue` |
| `outbox` | `operationId` | 비활성 CloudKit 구현의 호환 변경 기록 | `state`, `nextAttemptAt`, `entityType`, `entitySyncId` |
| `syncState` | `key` | 비활성 CloudKit 구현의 호환 상태 | 없음 |
| `conflicts` | `conflictId` | 비활성 CloudKit 구현의 호환 충돌 기록 | `status`, `entityType`, `entitySyncId`, `createdAt` |
| `migrationLog` | `migrationId` | 이전 실행·검증·완료 기록 | `state`, `completedAt` |

### Device-only Integration Settings

- Rebrickable API Key는 `settings`의 `rebrickableApiKey` Record에만 저장한다.
- 이 값은 기기·브라우저 프로필 전용이며 Archive Entity가 아니다.
- Full ZIP Backup, Restore staging, LEGO Record, Asset, Outbox, GitHub 코드에 포함하지 않는다.
- Full Restore는 현재 기기의 연결값을 가져오거나 덮어쓰지 않는다.
- `Reset Test Archive` 또는 브라우저 사이트 데이터 삭제 시에는 IndexedDB와 함께 제거된다.
- 실제 적용한 카탈로그 출처 기록은 LEGO Record의 `catalogImport`에 저장하므로 Full ZIP Backup과 Restore 대상이다.
- `catalogImport.appliedFields`는 아직 수동 수정되지 않은 실제 Rebrickable 적용 항목만 유지한다. 적용 뒤 직접 수정된 항목은 제거하고 남은 항목이 없으면 `catalogImport`를 제거한다.
- `Verified` LEGO Record 저장은 UI 상태와 관계없이 저장 계층에서 다시 검증하며, 현재 Set Number와 일치하는 HTTPS `lego.com` 공식 상품 페이지 또는 공식 조립 설명서만 Official Source로 인정한다.

## 4. Asset Records

### `assets`

```text
archiveId
generationId
syncId
stableId
title
type
source
sourceLink
sourceStatus
productionStatus
currentVersionId
notes
checkedOn
createdAt
updatedAt
deletedAt
fieldClocks
cloudChangeTag
```

- `type`은 승인된 6개 Asset Type만 허용한다.
- `sourceStatus`는 `Source Needed / Review / Verified`만 허용한다.
- `productionStatus`는 `Candidate / Selected / In Use / Archived`만 허용한다.
- `fieldClocks`는 필드별 마지막 변경 시각과 device ID를 저장해 서로 다른 필드의 동시 수정은 자동 병합하고, 같은 필드의 충돌은 검토 대상으로 남긴다.
- `Archived`는 `productionStatus` 값이며 `deletedAt`을 만들지 않는다.

### `assetVersions`

```text
archiveId
generationId
versionId
assetSyncId
parentVersionId
originalFileId
previewFileId
reason          initial | replace | conflict | legacy-import
createdAt
createdByDeviceId
deletedAt
cloudChangeTag
```

- Version History의 실제 ID는 UUID다.
- 화면의 `Version 1`, `Version 2` 표시는 Version History 정렬 결과로 계산하며 영구 식별자로 사용하지 않는다.
- 동시 이미지 교체로 두 버전이 생겨도 두 `versionId`를 모두 보존한다.
- 충돌 해결은 `currentVersionId`를 선택하는 새 Asset revision이며, 선택하지 않은 파일을 삭제하지 않는다.

### `files`

```text
archiveId
generationId
fileId
role            original | preview
blob
sha256
mimeType
extension
originalFilename
byteSize
width
height
createdAt
cloudChangeTag
```

- Original은 업로드된 바이트를 변환하지 않고 Blob으로 저장한다.
- Preview는 화면 표시용 파생 파일이며 언제든 원본에서 다시 만들 수 있다.
- 사진 계열 Preview는 JPEG, 투명 배경이 필요한 자료는 PNG를 기본으로 한다.
- 같은 SHA-256 파일은 중복 저장하지 않고 기존 `fileId`를 재사용할 수 있다.
- 원본이 현재 구현의 압축 Data URL에서 이전된 경우 `legacy-import`로 표시하고 원본 화질이라고 오인하지 않게 한다.

## 4.1 Story Connection and Page Link Records

### `storyConnections`

```text
archiveId
generationId
syncId
stableId
legoRecordSyncId
filmMode             Film | Series-wide
filmSyncId
filmSyncIds
locationSyncId
locationSyncIds
locationPath
locationPaths
subjectSyncIds
narrativeScope       Location
status               Review | Verified
connectionNote
checkedOn
createdAt
updatedAt
deletedAt
cloudChangeTag
```

- 현재 첫 구현은 Page 제작에 필요한 `Location` Scope만 입력한다. 복수 Film·Subject·Archive 입력은 별도 승인 범위다.
- Location 원본이 있으면 Sync ID와 전체 경로를 함께 보존하고, 아직 원본이 없으면 전체 경로 문자열을 저장한 `Review` 연결을 허용한다.
- 새 연결과 편집 연결은 저장 계층에서 동일한 중복 키를 검사하며 기존 연결 편집은 `CON` Stable ID를 유지한다.

### `evidenceLinks`

```text
archiveId
generationId
syncId
linkId
connectionSyncId
assetSyncId
createdAt
updatedAt
deletedAt
cloudChangeTag
```

- 현재 Page 연결 UI는 선택 Context에 직접 연결된 `Verified · Official Film Still`만 Evidence로 허용한다.
- Evidence가 없어도 `Review` Story Connection은 저장할 수 있으며 `Verified` 전환 조건은 Protocol의 Evidence·Checked On 규칙을 그대로 따른다.
- Story Connection·Evidence Link·Evidence Asset `In Use` 변경·Page Primary 연결·Outbox는 단일 transaction으로 저장한다.

## 5. Local Save Transactions

### Add Asset

1. 필수값과 공식 자료의 Source Link를 검증한다.
2. 원본 해시와 파일 메타데이터를 계산하고 Preview를 만든다.
3. 예약 블록에서 Stable ID와 새 Sync ID를 발급한다.
4. 하나의 IndexedDB transaction에서 `files`, `assetVersions`, `assets`, `assetLinks`, `outbox`를 함께 저장한다.
5. transaction 완료 뒤에만 `Saved`로 표시하고 화면을 닫는다.
6. 저장 뒤 CloudKit 또는 다른 외부 전송을 자동 실행하지 않는다.

### Edit Metadata

- 변경된 필드의 `fieldClocks`만 갱신한다.
- Asset 메타데이터와 Outbox operation을 같은 transaction에 저장한다.
- 이미지가 바뀌지 않으면 새 Asset Version을 만들지 않는다.

### Replace Image

1. 사용 중인 WWA Page 관계가 있으면 승인된 확인창을 표시한다.
2. 새 Original과 Preview를 먼저 생성·검증한다.
3. 새 `assetVersions` Record를 추가한다.
4. Asset의 `currentVersionId`를 새 버전으로 변경한다.
5. `sourceStatus = Review`, `checkedOn = null`로 변경한다.
6. 기존 Version과 파일은 삭제하지 않는다.
7. 관련 Record·Story Connection·WWA Page 관계는 Asset Sync ID에 연결되어 있으므로 유지한다.

### Archive

- `productionStatus`만 `Archived`로 변경한다.
- Asset, 파일, Version, 관계에 Tombstone을 만들지 않는다.
- 복원은 이전 Production Status를 직접 추정하지 않고 사용자가 새 상태를 선택하게 한다.

## 6. Dormant CloudKit Compatibility Contract

이 절과 7–8절의 CloudKit 세부 내용은 2026-08-10 구현 이력을 보존하기 위한 비활성 호환 계약이다. 현재 앱은 CloudKit을 초기화하거나 Script를 불러오거나 사용자 화면에 노출하지 않는다.

### Container and Database

- Database: authenticated user's CloudKit private database
- Custom zone: `WWAArchive`
- Environment: Development 검증 후 Production schema를 명시적으로 배포
- Web access: CloudKit JS API token을 GitHub Pages origin과 승인된 redirect URL로 제한
- CloudKit 로그아웃 상태에서는 Local-Only로 동작하며 Sync Queue를 지우지 않는다.

### Cloud Record Types

CloudKit은 로컬 검색 데이터베이스가 아니라 동기화 전송 계층으로 사용한다. 메타데이터는 공통 envelope로 저장해 향후 Entity가 늘어나도 CloudKit schema 변경을 최소화한다.

| Record Type | 역할 | 핵심 Field |
|---|---|---|
| `WWAArchiveRoot` | 현재 활성 세대 | `archiveId`, `activeGenerationId`, `schemaVersion` |
| `WWAEntity` | Asset·Version·관계·Record 메타데이터 | `archiveId`, `generationId`, `entityType`, `stableId`, `payloadJSON`, `fieldClocksJSON`, `updatedAt`, `deletedAt`, `deviceId` |
| `WWAFile` | Original·Preview CKAsset | `archiveId`, `generationId`, `sha256`, `role`, `file`, 파일 메타데이터 |
| `WWAIDCounter` | Stable ID 다음 번호 | `archiveId`, `entityType`, `nextValue` |
| `WWAIDReservation` | 번호 범위 발급 감사 기록 | `entityType`, `deviceId`, `startValue`, `endValue`, `reservedAt` |

- CloudKit `recordName`은 Sync ID 또는 파일·관계의 영구 내부 ID를 사용한다.
- 메타데이터 변경은 기존 `recordChangeTag`를 포함한 일반 update로 보낸다.
- `forceUpdate`와 `forceReplace`는 사용하지 않는다.
- 파일을 먼저 업로드하고, Asset Version, Asset 포인터 순으로 반영한다.
- 일부 업로드가 실패해도 로컬 transaction은 되돌리지 않으며 Outbox에서 재시도한다.

## 7. Dormant Sync Cycle

WWA Manager가 실행 중이고 네트워크와 CloudKit 인증이 유효할 때 다음 순서로 동작한다.

1. IndexedDB를 열고 로컬 데이터를 즉시 표시한다.
2. `fetchDatabaseChanges`와 `fetchRecordZoneChanges`를 사용해 마지막 token 이후 변경을 받는다.
3. 원격 변경을 검증한 뒤 로컬 pending change와 비교한다.
4. 충돌이 없는 원격 변경을 IndexedDB transaction으로 반영한다.
5. Outbox를 의존 순서대로 전송한다: File → Version → Entity/Link → current pointer.
6. 모든 update는 `recordChangeTag`로 충돌을 감지한다.
7. 서버 응답의 새 change tag와 zone token을 저장한다.
8. 오류는 재시도 가능 여부를 분류하고 지수 backoff를 적용한다.

동기화 트리거:

- 앱 실행
- CloudKit 로그인 완료
- 네트워크 복귀
- 로컬 저장 직후
- 앱이 열린 동안의 짧은 debounce 후 재시도
- 사용자가 Settings에서 수동 Sync 실행

웹앱이 닫혀 있는 동안의 지속 동기화는 보장하지 않는다.

## 8. Dormant Cloud Conflict Rules

| 충돌 | 처리 |
|---|---|
| 서로 다른 메타데이터 필드 수정 | `fieldClocks` 기준 자동 병합 |
| 같은 필드에 다른 값 | 두 값을 `conflicts`에 보존하고 사용자 검토 |
| 양쪽 기기의 이미지 교체 | 두 Asset Version과 파일을 모두 보존하고 `currentVersionId` 선택 대기 |
| 이미지 교체와 기존 Verified 수정 | 교체에 따른 `Review`와 `checkedOn = null`을 유지하고 재검증 |
| 관계 추가와 다른 관계 추가 | 관계 Record를 각각 유지 |
| 같은 관계 추가 | 결정적 Link ID로 한 건만 유지 |
| 삭제와 수정 | Tombstone과 수정본을 모두 보존하고 삭제 여부 검토 |
| Stable ID가 같은 다른 Sync ID | 자동 병합 금지, Critical ID Conflict로 표시 |

- 충돌 중에도 원본과 모든 Version을 열람할 수 있다.
- 충돌 해결 전에는 관련 Record를 `Synced`로 표시하지 않는다.
- 충돌 해결은 새 revision으로 기록하고 어느 쪽 원본도 물리적으로 삭제하지 않는다.

## 9. Tombstones and Retention

- Asset의 `Archived`는 Tombstone이 아니다.
- 잘못 만든 Story Connection 또는 관계를 삭제할 때만 `deletedAt`을 사용한다.
- Tombstone은 모든 활성 기기가 변경을 받은 것이 확인되기 전 자동 정리하지 않는다.
- Asset Original, Version History, Evidence에 사용된 파일은 자동 삭제하지 않는다.
- 저장 공간 정리 기능은 별도 승인 전 구현하지 않는다.

## 10. Full ZIP Backup

파일명:

`WWA_Backup_YYYY-MM-DD_HHmm.zip`

구성:

```text
manifest.json
archive.json
checksums.json
originals/<fileId>.<ext>
previews/<fileId>.<ext>
```

`manifest.json` 필수 항목:

- app name and backup format version
- schema version
- archive ID and generation ID
- created at
- entity and file counts
- maximum issued Stable ID per entity type

`checksums.json`에는 모든 JSON·Original·Preview 파일의 SHA-256을 기록한다.

### Restore

- 전체 교체 복원만 제공한다.
- ZIP 구조, schema version, 모든 해시, Stable ID 중복, 관계 참조, 파일 누락을 먼저 검증한다.
- 복원 데이터는 새 `generationId`의 staging namespace에 적재한다.
- 검증이 끝나면 `activeGenerationId`만 전환한다.
- 이전 generation은 복원 직후 삭제하지 않아 실패 시 되돌릴 수 있게 한다.
- 복원 적용 중에는 다른 Archive Data 작업을 시작하지 않는다.
- Stable ID counter는 복원 데이터의 최댓값보다 낮아지지 않으며 이전에 발급된 ID를 재사용하지 않는다.

## 11. Legacy Migration

현재 `localStorage` 기반 Record와 압축 Data URL 이미지는 다음 순서로 이전한다.

1. 기존 JSON을 읽고 migration source checksum을 기록한다.
2. LEGO Record를 `legoRecords`로 복사한다.
3. Record 안의 각 이미지 Data URL을 Blob으로 변환한다.
4. 이미지를 독립 Asset, Version, File, Asset Link로 분리한다.
5. 매핑 가능한 기존 항목은 승인된 Asset Type으로 변환한다.

| Legacy key | Asset Type |
|---|---|
| `legoOfficial` | `LEGO Official Product Image` |
| `boxArt` | `LEGO Official Box Image` |
| `blueprint` | `WWA Blueprint` |
| `movieStill` | `Official Film Still` |
| `myPhoto` | `My Photography` |

- 기존 `minifigures` 이미지는 출처와 유형을 안전하게 단정할 수 없으므로 자동 분류하지 않는다. Migration Review Draft로 보존하고 사용자가 승인된 6종 중 하나를 선택한 뒤 영구 Asset으로 저장한다.
- 이전 IndexedDB 또는 Full ZIP의 `Official Blueprint` 값은 복원 검증 뒤 `WWA Blueprint`로 정규화한다. 기존 Stable ID·Sync ID·Version·관계는 변경하지 않고 Type 값만 승계한다.
- `WWA Blueprint`는 공식 자료 유형 집합에 포함하지 않으므로 Source Link 필수 검증을 적용하지 않는다.
- Data URL 이미지는 이미 압축된 파생본이므로 `reason = legacy-import`와 품질 경고를 기록한다.
- Migration은 같은 source checksum에 대해 한 번만 실행되는 idempotent 작업으로 만든다.
- IndexedDB Record 수, Blob 해시, 관계 참조 검증이 끝나도 기존 `localStorage`는 즉시 삭제하지 않는다.
- 첫 전체 ZIP 백업 생성과 복원 테스트까지 완료된 뒤에만 legacy data 정리를 별도 승인받는다.

## 12. Failure Safety

- IndexedDB transaction 실패 시 어느 일부도 `Saved`로 표시하지 않는다.
- Preview 생성 실패는 Original 영구 저장 전에 사용자에게 알린다.
- 저장 공간 부족 시 새 파일 쓰기를 중단하고 기존 데이터를 유지한다.
- 지원하지 않는 ZIP schema version은 기존 Local Archive를 바꾸지 않고 거부한다.
- 다른 기기의 Archive는 검증된 전체 ZIP 복원으로만 가져오며 자동 병합하지 않는다.

## 13. Implementation Order

1. IndexedDB wrapper, schema migration, device/archive identity
2. Add Asset transaction과 새로고침 유지
3. Edit Metadata, Replace Image, Version History
4. Legacy `localStorage` 읽기 전용 migration
5. ZIP 생성·검증·staged restore
6. IndexedDB local Stable ID block reservation
7. CloudKit 초기화·사용자 화면 비활성화
8. iPhone 393px·430px, iPad, PC, HEIC/JPEG, offline, restore 검증
9. GitHub Pages 체크포인트 release
10. LEGO Record·Story Connections 통합

## 14. Approved Decisions

2026-08-11 다음 여섯 항목을 승인했다. 이 결정은 2026-08-09의 CloudKit 사용 승인을 대체한다.

1. 사용자용 Stable ID와 내부 Sync ID를 분리한다.
2. Asset 이미지와 Version 파일을 별도 Record로 저장한다.
3. Stable ID는 IndexedDB high-water를 기준으로 로컬 25개 블록 예약 방식으로 발급한다.
4. CloudKit 자동 동기화와 Apple Developer 유료 가입을 사용하지 않는다.
5. Full ZIP Backup을 장기 보존과 기기 이동의 공식 수단으로 사용한다.
6. ZIP 복원은 새 generation에 적재한 뒤 활성 generation을 전환하는 전체 교체 방식만 사용한다.
