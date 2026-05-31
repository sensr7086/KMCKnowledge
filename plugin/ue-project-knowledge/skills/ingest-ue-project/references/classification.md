# 카테고리 판정 규칙

UE 프로젝트 클래스를 카테고리로 판정한다. **단일 진실원 = 저장소 루트의 `categories.md`** — 본 문서는 ingest 측 수행 순서·예시·역할 기준 판정 보조 설명.

## 판정 순서

1. **자동 분류 규칙 (top-down)** — `categories.md` §자동 분류 규칙. 헤더 첫 매칭 적용:
   - 인터페이스 우선 (`UINTERFACE` + `I*Interface` 페어 또는 plain `class I*`) → **Interface**.
   - 상속 베이스 매칭 (Subsystem → AssetUserData → Actor → Component → DataTable → 예약 Asset/GraphNode/BlueprintLibrary/Editor).
   - 역할 기준 판정 (베이스가 범용 UObject 일 때) — 아래 §역할 기준 참조.
2. **매칭 실패 시** → 사용자 보고 (강제 배치 / 신설 / 보류 3 선택).
3. **신설 결정 시** → `categories.md` §신설 절차 4단계 발화.

## 활성 카테고리 (catalog 파일 존재 — 인스턴스마다 카운트 다름)

| 상속/특징 | 대분류 | 실제 예시 (generic UE) |
|---|---|---|
| `ACharacter`, `AActor`, `APawn`, `ACameraActor`, `APhysicsVolume` 자손 + Actor 보조 UObject | **Actor** | `AMyCharacter`, `AMyCamera`, `AMyPooledActor`, `UMyCameraBase`(Camera Mode 베이스) |
| `UActorComponent` / `U*MeshComponent` / `UCharacterMovementComponent` 자손 + `UAnimInstance` 자손 | **Component** | `UMyActorComponent`, `UMyMoveComponent`, `UMySoftSkeletalMeshComponent`, `UMyAnimInstance` |
| `USubsystem` 자손 (UGameInstance / UWorld / UTickableWorld / UEditor / UEngine / ULocalPlayerSubsystem) | **Subsystem** | `UMyGameSubsystem`, `UMyActorSpawnSubsystem`, `UMyStorySubSystem`, `UMyEditorSubsystem` |
| `I...Interface` (UINTERFACE + native pair 또는 plain C++) | **Interface** | `IMyActorInterface`, `IMyPoolableInterface`, `IMySpatialQueryFilterable` |
| `FTableRowBase` 자손 + UDataTable 로드·관리 역할 UObject | **DataTable** | `FMyDataBase`, `FMyData_Sheet1`, `UMyTableManager`, `UMyTableRegistry` |
| `UAssetUserData` 자손 | **AssetUserData** | `UMyNiagaraSocketBindings`, `UMyHitBoneCurveUserData` |

## 예약 카테고리 (catalog 파일 없음 — 첫 트리거 시 신설)

| 베이스/판정 | 예약 카테고리 |
|---|---|
| UObject 자산 + `UDataAsset` / `UPrimaryDataAsset` (DataTable 자손 제외) | **Asset** |
| 프로젝트 고유 런타임 그래프 콘텐츠 노드 (`UMyStory_*`, `UMyParts_*`, `UMyCombo*` 같은 패턴 — UEdGraphNode 가 *아닌* 콘텐츠 측) | **GraphNode** |
| `UBlueprintFunctionLibrary` 자손 | **BlueprintLibrary** |
| `UEdGraphNode` / `UEdGraphSchema` / `UFactory` / `FAssetTypeActions` / `F*EditorApplication` / `S*Widget` (Editor 모듈) | **Editor** (정책 결정 필요 — 단일 vs 서브카테고리 분리) |

## 역할 기준 판정 (베이스가 범용 UObject 일 때)

베이스가 명시 매핑에 안 맞지만 역할이 명확하면 그 카테고리로:

- **DataTable**: `UMyTableManager : UObject` — UDataTable 컬렉션 로드·관리 (Table/LoadedTables/RowMap 키워드).
- **Actor 보조**: `UMyCameraBase : UObject (Abstract)` — Camera 모드 객체, CameraSubSystem 이 보관.
- **Asset (예약)**: `UMyStoryAsset : UObject` — 자산 콘텐츠 (그래프) 보관.

본문 "소속/상속" 섹션에 "역할 기준 판정" 명시.

## 다단 상속 / 인터페이스 동반

`AMyCharacter : public ACharacter, public IMyActorInterface` 처럼 인터페이스를 함께 구현하는 경우:
- 대분류는 **클래스 베이스**(ACharacter)로 판정 → **Actor**.
- 구현한 인터페이스(IMyActorInterface)는 "소속/상속" 섹션 + "연관 entity" 에 `[[entities/MyActorInterface]]` 로 링크.
- 인터페이스 entity 자체는 별도 **Interface** 카테고리.

## 신설 시점

매칭 실패 OR 예약 카테고리 첫 트리거:
1. 사용자에게 후보 보고 — "이 entity 는 기존 N 활성 카테고리에 정확히 안 맞습니다. 가장 가까운: X. 예약 카테고리: Y. 신설 제안: Z."
2. 사용자 선택 (강제 배치 / 신설 / 보류).
3. 신설 승인 시 `categories.md` §신설 절차 4단계 (registry / catalog / 본 classification.md / 영향 entity / index) 모두 갱신.

## Editor 카테고리 — 사전 정책 결정 필요

UE Editor 모듈은 클래스가 많고 다양한 종류(F*/S*/UFactory/UEdGraphNode/UEdGraphSchema)가 섞여 있어 단일 Editor 카테고리에 담으면 catalog 가 비대해질 수 있다. 신설 시점 *이전에* 다음 두 옵션 중 선택:

- **(A) 단일 Editor 카테고리** — `catalogs/editor.md` 1개. 베이스/관리자/구현체 + 종류별 서브섹션(Action/Application/Factory/GraphNode/Schema/Widget).
- **(B) 서브카테고리 분리** — `catalogs/editor-action.md`, `catalogs/editor-application.md`, `catalogs/editor-factory.md`, `catalogs/editor-graphnode.md`, `catalogs/editor-schema.md`, `catalogs/editor-widget.md` 등 다수. categories.md 의 카테고리 이름도 PascalCase 다중 단어 (`EditorAction`, `EditorWidget`).

권장: 처음에는 (A), Editor 클래스가 30개 넘어가면 (B) 로 마이그레이션.
