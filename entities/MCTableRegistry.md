---
title: "UMCTableRegistry"
kind: entity
category: DataTable
base_class: UDataAsset
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/MCGame/MCTableRegistry.h
  - KMCProject/MCPlayModule/MCGame/MCTableRegistry.cpp
vault_refs:
  - sources/ue-assetclasses-data
  - concepts/Asset-Loading-Policy
  - entities/UGameInstance
last_ingested: 2026-05-29
---

# UMCTableRegistry

## 한 줄 정의
게임 전역 테이블 레지스트리 — `EMCTableKind → (TSoftObjectPtr<UDataTable> + ExpectedRowStruct)` 매핑을 UDataAsset 자산 1개로 관리.

## 소속 / 상속
- 대분류: DataTable (역할 기준 — 경로·기대 타입 설정 관리)
- `UCLASS(BlueprintType)` `class MCPLAYMODULE_API UMCTableRegistry : public UDataAsset`
- 함께 정의되는 보조 타입:
  - `UENUM(BlueprintType) enum class EMCTableKind : uint8` — None(Hidden), Item, Skill, Monster, Dialogue, **LootEntry**(2026-05-29 MCLoot 추가, enum 끝 삽입 — 직렬화 값 보존, "루팅엔트리")
  - `USTRUCT(BlueprintType) FMCTableEntry` — `TSoftObjectPtr<UDataTable> TablePath` + `TObjectPtr<UScriptStruct> ExpectedRowStruct`

## 핵심 구조
- **UPROPERTY**:
  - `TMap<EMCTableKind, FMCTableEntry> TablePaths` — Kind → (경로 + 기대 타입). 에디터에서 편집.
- **API**:
  - `const TSoftObjectPtr<UDataTable>& GetTablePath(EMCTableKind) const` — 미등록 Kind 면 `EmptyEntry.TablePath` (안전 reference).
  - `const UScriptStruct* GetExpectedStruct(EMCTableKind) const` — 미등록 또는 ExpectedRowStruct 미설정 시 nullptr.
  - `void GetRegisteredKinds(TArray<EMCTableKind>&) const` — 등록된 Kind 순회용.
- **private**: `static const FMCTableEntry EmptyEntry` — 미등록 Kind 에 대한 안전 fallback.
- **FMCTableEntry::ExpectedRowStruct 의 메타**: `meta=(MetaStruct="/Script/MCPlayModule.MCDataBase")` — 에디터 picker 가 FMCDataBase 자손만 노출. 빈 값 = 검증 스킵.

## 따르는 패턴
- 경로 설정 = DataAsset 자산 1개 (UDeveloperSettings 가 아닌) — 디자이너 접근성·Cooked OK (Editor only 가드 불필요) → [[sources/ue-assetclasses-data]] §UDataAsset
- SoftObjectPath sync TryLoad — Initialize 시점 1회 → [[concepts/Asset-Loading-Policy]] §SoftObjectPath
- UAssetManager Bundle 패턴 (후속 확장 여지) → [[entities/UGameInstance]] §UAssetManager 진입점

## ⚠ 함정
- **MetaStruct long path 강제**: UE 5.5+ UHT 가 `meta=(MetaStruct=...)` 에 long path 강제. short name "MCDataBase" 는 거부됨. 포맷: `/Script/<ModuleName>.<StructName>` (F prefix 제외).
- Kind 추가 시 절차 의무: (1) `EMCTableKind` enum 값 추가, (2) UMCTableRegistry 자산(.uasset) 에서 `TablePaths` 매핑 추가, (3) (선택) 전용 row USTRUCT 정의 `FMCData_NewKind : FMCDataBase`.
- **enum 중간 삽입 금지**: 새 Kind 는 enum **끝**에 삽입 — 기존 직렬화 값 보존(중간 삽입 시 .uasset TablePaths 키 어긋남). LootEntry(2026-05-29) 가 이 규칙 적용 예.
- **LootEntry .uasset 매핑은 사용자 수동**: `EMCTableKind::LootEntry` 값은 코드에 추가됐으나, Registry 자산(.uasset) `TablePaths` 에 `LootEntry → 공용 LootEntry DT 경로 + FMCData_LootEntry` 매핑은 사용자가 에디터에서 수동 등록 필요. (헤더 주석 명시) 미등록 시 [[entities/MCLootRoller]] 가 Warning + 빈 결과.

## 연관 entity
- [[entities/MCTableManager]] — Initialize 단계에서 GetRegisteredKinds·GetTablePath·GetExpectedStruct 호출.
- [[entities/MCGameSubsystem]] — DefaultRegistryPath 로 본 자산을 sync 로드.
- [[entities/MCDataBase]] — FMCTableEntry.ExpectedRowStruct 의 베이스 제약.
- [[entities/MCData_LootEntry]] — `EMCTableKind::LootEntry` Kind 의 row 타입 (MCLoot).
- [[entities/MCLootTableAsset]] / [[entities/MCLootRoller]] — LootEntry Kind 소비자 (CP-1 Option B).
