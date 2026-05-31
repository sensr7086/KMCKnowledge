# 대분류 판정 규칙

KMCProject 클래스를 카테고리로 판정한다. **단일 진실원 = 저장소 루트의 `categories.md`** — 본 문서는 ingest 측 수행 순서·예시·역할 기준 판정 보조 설명.

## 판정 순서

1. **자동 분류 규칙 (top-down)** — `categories.md` §자동 분류 규칙. 헤더 첫 매칭 적용:
   - 인터페이스 우선 (`UINTERFACE` + `I*Interface` 페어 또는 plain `class I*`) → **Interface**.
   - 상속 베이스 매칭 (Subsystem → AssetUserData → Actor → Component → DataTable → Widget(`UUserWidget`/UMG) → BlueprintLibrary → Asset/GraphNode → 예약 Editor).
   - 역할 기준 판정 (베이스가 범용 UObject 일 때) — 아래 §역할 기준 참조.
2. **매칭 실패 시** → 사용자 보고 (강제 배치 / 신설 / 보류 3 선택).
3. **신설 결정 시** → `categories.md` §신설 절차 4단계 발화.

## 활성 카테고리 (catalog 파일 존재)

| 상속/특징 | 대분류 | 실제 예시 |
|---|---|---|
| `ACharacter`, `AActor`, `APawn`, `ACameraActor`, `APhysicsVolume` 자손 + Actor 보조 UObject | **Actor** | `AMCCharacter`, `AMCCamera`, `AMCPooledActor`, `AMCWaterVolume`, `UMCCameraBase`(Camera Mode), `UMCCameraPlayable` |
| `UActorComponent` / `U*MeshComponent` / `UCharacterMovementComponent` 자손 + `UAnimInstance` 자손 | **Component** | `UMCActorComponent`, `UMCMoveComponent`, `UMCSoftSkeletalMeshComponent`, `UMCAnimInstance` |
| `USubsystem` 자손 (UGameInstance / UWorld / UTickableWorld / UEditor / UEngine / ULocalPlayerSubsystem) | **Subsystem** | `UMCGameSubsystem`, `UMCActorSpawnSubsystem`, `UMCStorySubSystem`, `UMCNiagaraSocketPreviewSubsystem` |
| `I...Interface` (UINTERFACE + native pair 또는 plain C++) | **Interface** | `IMCActorInterface`, `IMCPoolableInterface`, `IMCSpatialQueryFilterable`, `IMCComboPreviewVisitor` |
| `FTableRowBase` 자손 + UDataTable 로드·관리 역할 UObject | **DataTable** | `FMCDataBase`, `FMCData_Sheet1`, `UMCTableManager`, `UMCTableRegistry` |
| `UAssetUserData` 자손 | **AssetUserData** | `UMCNiagaraSocketBindings`, `UMCHitBoneCurveUserData`, `UMCMeshProxyUserAssetData` |
| `UUserWidget` / UMG 위젯 자손 (런타임 UI; MCEditorModule Slate `S...Widget` 제외) | **Widget** | `UMCLootToastWidget` |
| `UBlueprintFunctionLibrary` 자손 | **BlueprintLibrary** | `UMCActorBlueprintLibrary`, `UMCSpatialQueryLibrary`, `UMCWaterBlueprintLibrary`, `UMCLootRoller` |

## 예약 카테고리 (catalog 파일 없음 — 첫 트리거 시 신설)

| 베이스/판정 | 예약 카테고리 |
|---|---|
| UObject 자산 (UMCStoryAsset 등) + `UDataAsset` / `UPrimaryDataAsset` (DataTable 자손 제외) | **Asset** |
| `UMCStory_*` / `UMCParts_*` / `UMCCombo*` 그래프 노드 자손 (UObject) | **GraphNode** |
| `UBlueprintFunctionLibrary` 자손 | **BlueprintLibrary** |
| MCEditorModule 의 `F*AssetAction` / `F*EditorApplication` / `S*Widget` / `UFactory` / `UEdGraphNode` / `UEdGraphSchema` | **Editor** (정책 결정 필요) |

## 역할 기준 판정 (베이스가 범용 UObject 일 때)

베이스가 명시 매핑에 안 맞지만 역할이 명확하면 그 카테고리로:

- **DataTable**: `UMCTableManager : UObject` — UDataTable 컬렉션 로드·관리 (Table/LoadedTables/RowMap 키워드).
- **Actor 보조**: `UMCCameraBase : UObject (Abstract)` — Camera 모드 객체, CameraSubSystem 이 보관.
- **Asset (예약)**: `UMCStoryAsset : UObject` — 자산 콘텐츠 (Story 그래프) 보관.

본문 "소속/상속" 섹션에 "역할 기준 판정" 명시.

## 다단 상속 / 인터페이스 동반

`AMCCharacter : public ACharacter, public IMCActorInterface` 처럼 인터페이스를 함께 구현하는 경우:
- 대분류는 **클래스 베이스**(ACharacter)로 판정 → **Actor**.
- 구현한 인터페이스(IMCActorInterface)는 "소속/상속" 섹션 + "연관 entity" 에 `[[entities/MCActorInterface]]` 로 링크.
- 인터페이스 entity 자체는 별도 **Interface** 카테고리.

## 신설 시점

매칭 실패 OR 예약 카테고리 첫 트리거:
1. 사용자에게 후보 보고 — "이 entity 는 기존 N 활성 카테고리에 정확히 안 맞습니다. 가장 가까운: X. 예약 카테고리: Y. 신설 제안: Z."
2. 사용자 선택 (강제 배치 / 신설 / 보류).
3. 신설 승인 시 `categories.md` §신설 절차 4단계 (registry / catalog / 본 classification.md / 영향 entity / index) 모두 갱신.
