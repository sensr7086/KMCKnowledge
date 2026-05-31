---
title: "FMCData_LootEntry"
kind: entity
category: DataTable
base_class: FMCDataBase
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/MCLoot/Data/MCLootTypes.h
vault_refs: []
last_ingested: 2026-05-29
---

# FMCData_LootEntry

## 한 줄 정의
공용 LootEntry 풀 DataTable 의 row USTRUCT — 루팅 엔트리 1개의 **값**(item_id/weight/min/max/rarity/condition/is_guaranteed)을 보유. 멤버십은 보유하지 않음(CP-1 Option B).

## 소속 / 상속
- 대분류: DataTable
- `USTRUCT(BlueprintType) struct FMCData_LootEntry : public FMCDataBase`
- `GENERATED_BODY()` + `MC_DATA_BODY(FMCData_LootEntry)` — 프로젝트 표준 DataTable row 매크로. (근거: 헤더 주석 — "모든 DataTable row 는 FMCDataBase 자손", MCGame/MCDataBase.h 컨벤션)
- 헤더 전용 타입 (`.cpp` 불요). 같은 헤더(`MCLootTypes.h`)에 지원 enum/struct 동거.
- 충족 REQ: REQ-10, REQ-11, REQ-12, REQ-13. 설계 출처: `_workspace/02_ue-architect_design.md §2.1`.

## 핵심 구조
- **UPROPERTY (EditAnywhere, BlueprintReadWrite, Category="MC|Loot")**:
  - `FName ItemId` — 지급될 아이템 식별자. (REQ-10)
  - `int32 Weight = 1` `meta=(ClampMin="0")` — 같은 가중치 풀 내 상대 확률. `bIsGuaranteed` 항목에서는 무시. (REQ-10, REQ-12, Q-8)
  - `int32 MinCount = 1` `meta=(ClampMin="0")` — 지급 수량 하한(포함). (REQ-10)
  - `int32 MaxCount = 1` `meta=(ClampMin="0")` — 지급 수량 상한(포함). (REQ-10)
  - `EMCLootRarity Rarity = EMCLootRarity::Common` — 희귀도. (REQ-10, REQ-11)
  - `FName Condition` — 선택적 조건 표현식 태그. v1: 비어있으면 항상 통과, 비어있지 않으면 `UMCLootableComponent::IsConditionMet(Condition)` 위임(기본 true). 문자열 표현식 평가기는 §8 확장점(v1 미구현). (REQ-13, Q-7)
  - `bool bIsGuaranteed = false` — true면 가중치 무시, 조건 통과 시 무조건 1회 지급. (REQ-10, REQ-12)
- **로드 경로 (단일 진입)**: `TableManager->FindRowAs<FMCData_LootEntry>(EMCTableKind::LootEntry, RowName)`. (근거: 헤더 주석 — 타입안전 + 반환 포인터 즉시 사용/버리기, MCGame/MCTableManager.h FindRowAs)

### 동거 지원 타입 (별도 페이지 없음)
- **`EMCLootRarity : uint8`** `UENUM(BlueprintType)` — 아이템 희귀도 4단계: `Common / Uncommon / Rare / Epic`. (REQ-11, PDF §3 "Common / Uncommon / Rare / Epic")
- **`FMCLootResultItem`** `USTRUCT(BlueprintType)` — 굴림 결과 1개 `(ItemId, Count, Rarity)`. (REQ-20)
  - `FName ItemId` (BlueprintReadOnly), `int32 Count = 0` (BlueprintReadOnly), `EMCLootRarity Rarity = Common` (BlueprintReadOnly, "UI 토스트 색상 등 표시용").
  - 생성자: `FMCLootResultItem() = default` + `FMCLootResultItem(FName, int32, EMCLootRarity)`.
  - PDF §5 `results.append(Item(item_id, count))` + §10 `Roll → Item[]`. [[entities/MCLootRoller]] 가 생산, [[entities/MCLootToastWidget]]·[[entities/MCLootableComponent]] 가 소비.

## 따르는 패턴
- 모든 DataTable row 는 `FMCDataBase` 자손 + `MC_DATA_BODY` 매크로 — 베이스 [[entities/MCDataBase]] 의 타입안전 `IsA<T>`/`CastTo<T>` 적용. (헤더 주석 명시)
- **값/멤버십 분리 (CP-1 Option B)**: 이 row 는 엔트리의 **값**만 보유. "어느 row 가 어느 루팅 풀에 속하는가(멤버십)"는 여기에 없고 [[entities/MCLootTableAsset]] 의 `EntryRowNames` 가 큐레이션. (헤더 주석 명시)

## ⚠ 함정
- 베이스 [[entities/MCDataBase]] 의 함정 적용 (FindRow 반환 포인터 캐시 금지 — RowMap 재배치 dangling, 즉시 값 복사). [[entities/MCLootRoller]] 가 실제로 `Entries.Add(*Row)` 로 즉시 값 복사하여 우회.
- **EntryRowNames 이중 동기화 비용 (CP-1)**: 이 공용 DT 에 row 를 추가/이름변경/삭제하면, 이 row 를 참조하는 풀의 [[entities/MCLootTableAsset]] `EntryRowNames` 도 함께 갱신해야 함 — 의도적으로 수용한 트레이드오프. (헤더 주석 명시)
- `is_guaranteed` 항목의 `Weight` 는 무시됨 — 가중치 풀 구성에서 제외. (Q-8)

## 연관 entity
- [[entities/MCDataBase]] — 베이스 USTRUCT (`MC_DATA_BODY` 제공).
- [[entities/MCTableManager]] — `FindRowAs<FMCData_LootEntry>` 단일 로드 경로.
- [[entities/MCTableRegistry]] — `EMCTableKind::LootEntry` 정의 + Registry 자산 매핑.
- [[entities/MCLootTableAsset]] — `EntryRowNames` 로 본 row 의 풀 멤버십 큐레이션.
- [[entities/MCLootRoller]] — 본 row 를 굴려 `FMCLootResultItem` 생산.
- [[entities/MCLootableComponent]] — `IsConditionMet(Condition)` 위임 대상.
