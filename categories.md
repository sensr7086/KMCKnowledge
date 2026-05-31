# KMCKnowledge Category Registry

> 본 저장소의 entity 분류 카테고리 정의. 새 entity 가 기존 카테고리에 안 맞으면 본 문서를 갱신하고 catalog 파일을 신설하는 절차로 확장.
> **단일 진실원 (SSOT)**: ingest 워크플로우의 대분류 판정, catalog 파일 구조, index.md 의 카테고리 섹션 모두 본 문서를 기준으로 한다.
>
> **횡단 정책과의 관계 (직교)**: 카테고리는 "이 클래스가 *무엇인가*"(분류)를 가른다. 코드 작성의 공통 규약인 **횡단 정책**(profiling/global-iterator/component-6/asset-loading/asset-opt 등)은 [[ue-cross-cutting-policies/index]] 가 SSOT이며 **카테고리와 직교**한다(어느 카테고리든 적용 가능). ingest 는 카테고리 판정(본 문서) **후** 정책 적용 판정([[ue-cross-cutting-policies/index]] §3 매트릭스)을 별도 수행한다. 상세: CLAUDE.md §2.7.

---

## 활성 카테고리 (catalog 파일 존재)

| 카테고리 | catalog | 베이스/판정 기준 | 도입 | 도입 사유 |
|---|---|---|---|---|
| Actor | [[catalogs/actor]] | `A...` 액터 (ACharacter / AActor / APawn / ACameraActor / APhysicsVolume 등 자손) + Actor 보조 UObject (Camera Mode 등) | 2026-05-29 (초기) | STRUCTURE.md §5 초기 정의 |
| Component | [[catalogs/component]] | `U...Component` (UActorComponent / U*MeshComponent / UCharacterMovementComponent 자손) + `UAnimInstance` 자손 | 2026-05-29 (초기) | STRUCTURE.md §5 초기 정의 |
| DataTable | [[catalogs/datatable]] | `FTableRowBase` 자손 + `UDataTable` 로드·관리 역할 UObject (UMCTableManager / UMCTableRegistry 등) | 2026-05-29 (초기) | STRUCTURE.md §5 초기 정의 |
| Subsystem | [[catalogs/subsystem]] | `USubsystem` 자손 (UGameInstanceSubsystem / UWorldSubsystem / UTickableWorldSubsystem / UEditorSubsystem / UEngineSubsystem / ULocalPlayerSubsystem) | 2026-05-29 (배치 3 직후) | 5개 Subsystem 이 Actor(2)/Component(1)/DataTable(2) 강제 배치되어 분류 한계 노출. MCStorySubSystem 의 🟡 "분류 한계" 명시. |
| Interface | [[catalogs/interface]] | `I...Interface` (UINTERFACE + paired native interface, 또는 plain C++ interface) | 2026-05-29 (배치 3 직후) | 4개 인터페이스가 "주 사용 대분류" 로 강제 귀속. IMCComboPreviewVisitor 의 "임시 분류" 명시. |
| AssetUserData | [[catalogs/assetuserdata]] | `UAssetUserData` 자손 (Mesh / Skeleton / 기타 자산에 첨부되는 메타) | 2026-05-29 (배치 3 직후) | 3개 UAssetUserData 가 DataTable 강제. UE 표준상 "에셋 메타" 와 "primary 자산" 는 별개 개념. |
| Asset | [[catalogs/asset]] | UObject 자산 (UMCStoryAsset / UMCPartsAsset 등) + UDataAsset / UPrimaryDataAsset (DataTable 자손 제외) | 2026-05-29 (배치 4 Story 직후) | Story 그래프 배치 진입 — UMCStoryAsset(UObject) / UMCStoryBoard(UDataAsset) 호스팅 필요. MCStorySubSystem 의 분류 한계 본 신설로 자연 해소. |
| GraphNode | [[catalogs/graphnode]] | 프로젝트 고유 런타임 그래프 콘텐츠 노드: `UMCStory_*` / `UMCParts_*` / `UMCCombo*` 자손 (UEdGraphNode 가 *아닌* 콘텐츠 측) | 2026-05-29 (배치 4 Story 직후) | Story 그래프 배치 진입 — 노드 12개. UEdGraphNode 와 분리(런타임 콘텐츠 vs 편집기 노드). |
| BlueprintLibrary | [[catalogs/blueprintlibrary]] | `UBlueprintFunctionLibrary` 자손 — 정적 함수 모음 BP 노출 | 2026-05-29 (배치 7 소묶음 직후) | 3개 라이브러리 (UMCActor/SpatialQuery/Water) — 정적 함수 위주 컬렉션은 Component / Actor 와 분리. |
| Widget | [[catalogs/widget]] | `UUserWidget` / UMG 위젯 자손 (런타임 UI). MCEditorModule 의 Slate `S...Widget` 은 제외 (mcwiki 단일 진실원) | 2026-05-29 (배치 8 MCLoot 직후) | MCLoot 시스템의 UMCLootToastWidget(UUserWidget) 등장 — UI 위젯은 Component/Asset 과 별개 라이프사이클(CreateWidget, BindWidget, BP 시각화 위임). 사용자 승인 신설. |

---

## 예약된 카테고리 (catalog 파일 없음, 트리거 시 자동 생성)

(현재 예약 카테고리 없음.)

## 범위 밖 — 본 저장소 인덱싱 대상이 *아님*

| 영역 | 사유 | 대안 |
|---|---|---|
| **MCEditorModule** (UE 편집기 확장) | 본 영역은 UE 표준 패턴(F\*AssetAction / F\*EditorApplication / S\*Widget / UFactory / UEdGraphNode / UEdGraphSchema 등) 의 직접 적용. KMCProject 고유 지식이라기보다 UE Editor 프레임워크 지식 — **mcwiki** 카탈로그가 단일 진실원. | 생성기 워크플로우가 MCEditorModule 관련 작업 필요 시 mcwiki `[[sources/ue-editor-*]]` / `[[sources/ue-slate-*]]` / `[[sources/ue-editor-unrealed-*]]` 직접 참조. 본 저장소 *경유 안 함*. (2026-05-29 사용자 결정.) |

---

## 폐기된 카테고리 (deprecated)

(없음)

> 폐기 시: 본 표에 항목 추가 + 사유 + 대체 카테고리 명시. catalog 파일은 보존 (히스토리). 영향받는 entity 의 frontmatter 재분류.

---

## 신설 절차 (ingest 가 따르는 규약)

1. **신설 후보 인지** — ingest 가 entity 의 분류를 결정할 때 활성 카테고리 표의 판정 기준에 매칭. 매칭 실패 시 후보 보고:
   - "이 entity 는 기존 N 카테고리 어느 것에도 정확히 맞지 않습니다. 가까운 카테고리: X. 신설 제안: Y."
2. **사용자 확인** — (a) 기존 카테고리로 강제 배치 (분류 한계 본문 명시) / (b) 새 카테고리 신설 / (c) 분류 보류 (미인덱싱 큐) — 사용자 선택.
3. **신설 승인 시**:
   - 본 표(활성 카테고리)에 행 추가 — 카테고리명, 베이스/판정 기준, 도입일(YYYY-MM-DD), 도입 사유.
   - `catalogs/<lowercase-name>.md` 골격 생성 (표준 frontmatter + 베이스/관리자/구현체 섹션).
   - `.claude/skills/ingest-kmc/references/classification.md` 의 판정 표 동기 갱신.
   - 영향받는 기존 entity 의 frontmatter `category` 재분류 + 본 catalog 에 등록.
   - `index.md` 의 카테고리 섹션 추가.
4. **명명 규약**:
   - 카테고리명: PascalCase (`Subsystem`, `AssetUserData`, `BlueprintLibrary`).
   - catalog 파일명: lowercase, separator 없음 (`subsystem.md`, `assetuserdata.md`, `blueprintlibrary.md`).
   - frontmatter `category` 값: PascalCase 일치 (`category: Subsystem`).
   - wiki 링크: `[[catalogs/<lowercase>]]`, `[[entities/<EntityName>]]`.

---

## 자동 분류 규칙 (ingest 가 매 entity 에 적용하는 순서)

1. **인터페이스 우선**: 헤더에 `UINTERFACE(...)` + `class I...Interface` 페어 또는 plain `class I...` 명명 → **Interface**.
2. **상속 베이스 매칭** (top-down):
   - `: public USubsystem` 자손 (UGameInstance/World/Tickable/Editor/Engine/LocalPlayerSubsystem) → **Subsystem**.
   - `: public UAssetUserData` → **AssetUserData**.
   - `: public AActor` 또는 자손 (ACharacter, APawn, ACameraActor, APhysicsVolume 등) → **Actor**.
   - `: public UActorComponent` 또는 자손 → **Component**.
   - `: public UAnimInstance` → **Component** (animation runtime).
   - `: public FTableRowBase` 자손 → **DataTable**.
   - `: public UBlueprintFunctionLibrary` → **BlueprintLibrary** (예약 — 트리거 시 신설).
   - `: public UUserWidget` (또는 UMG 위젯 자손) → **Widget** (런타임 UI). MCEditorModule 의 Slate `S...Widget` 은 제외.
   - `: public UDataAsset / UPrimaryDataAsset` (DataTable 의 *Registry 류 관리자가 아니면*) → **Asset** (예약 — 트리거 시 신설).
   - `: public UEdGraphNode / UEdGraphSchema / UFactory` 또는 MCEditorModule 의 F*/S* 클래스 → **Editor** (예약 — 정책 결정 후 신설).
3. **역할 기준 판정** (베이스가 범용 UObject 일 때):
   - `UDataTable` 컬렉션 로드·관리 → **DataTable** (예: UMCTableManager).
   - `UMCStory_*` / `UMCParts_*` / `UMCCombo*` 그래프 노드 → **GraphNode** (예약 — 트리거 시 신설).
   - Actor 보조 UObject (예: Camera Mode UMCCameraBase) → **Actor**.
4. **매칭 실패** → 신설 절차 1번 (사용자 보고) 발화.

---

## 변경 이력

- **2026-05-29 (배치 8 MCLoot 직후)** — Widget 카테고리 정식 신설 (사용자 승인). UMCLootToastWidget(UUserWidget) 1개. 판정: `UUserWidget`/UMG 위젯 자손(런타임 UI), MCEditorModule Slate `S...Widget` 은 제외. 활성 카테고리 9 → 10. MCLoot 7개 entity (LootableComponent/InventoryComponent → Component, LootSubsystem → Subsystem, LootRoller → BlueprintLibrary, LootTableAsset → Asset, LootToastWidget → Widget, MCData_LootEntry → DataTable) 적재. EMCTableRegistry 에 LootEntry Kind 추가 반영.
- **2026-05-29 (배치 7 직후 — MCEditorModule 범위 결정)** — Editor 카테고리 예약 해제. MCEditorModule 은 본 저장소 인덱싱 대상이 *아님* (사용자 결정 — mcwiki 단일 진실원). 예약 카테고리 1 → 0. ingest 워크플로우 완주.
- **2026-05-29 (배치 7 소묶음 직후)** — BlueprintLibrary 카테고리 정식 신설 (예약 → 활성). UMCActorBlueprintLibrary / UMCSpatialQueryLibrary / UMCWaterBlueprintLibrary 3개. 활성 카테고리 8 → 9, 예약 2 → 1.
- **2026-05-29 (배치 4 Story 직후)** — Asset / GraphNode 카테고리 정식 신설 (예약 → 활성). UMCStoryAsset/Board → Asset, UMCStory_* 14 → GraphNode. MCStorySubSystem 의 분류 한계 본 신설로 해소. 활성 카테고리 6 → 8, 예약 4 → 2.
- **2026-05-29 (배치 3 직후)** — 카테고리 시스템을 3축 고정에서 확장 가능 N축 으로 전환. Subsystem / Interface / AssetUserData 3개 카테고리 정식 신설. 12개 entity 재분류. 예약 카테고리 4개(Asset / GraphNode / BlueprintLibrary / Editor) 등록.
- **2026-05-29 (초기 구축)** — STRUCTURE.md §5 의 3축 (Actor / Component / DataTable) 정의.
