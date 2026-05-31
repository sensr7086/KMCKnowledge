---
title: "DataTable 대분류"
kind: catalog
category: DataTable
last_ingested: 2026-05-29
---

# DataTable 대분류

> `FTableRowBase` 자손 + `UDataTable` 로드·관리 역할 UObject. AssetUserData(메타 자산)는 [[catalogs/assetuserdata]], Story/Parts 자산 같은 primary 자산은 [[catalogs/asset]] (예약).

## 베이스
- 🟢 [[entities/MCDataBase]] — `FMCDataBase : FTableRowBase`. 모든 DataTable row 의 베이스 USTRUCT. 타입 안전 `IsA<T>`/`CastTo<T>` + `MC_DATA_BODY` 매크로 제공.

## 관리자 (역할 기준 — UDataTable 컬렉션 관리)
- 🟢 [[entities/MCTableManager]] — `UMCTableManager : UObject`. UDataTable 컬렉션을 Kind 단위로 관리. `FindRowAs<T>` 2단 타입검사.
- 🟢 [[entities/MCTableRegistry]] — `UMCTableRegistry : UDataAsset`. Kind → (TSoftObjectPtr<UDataTable> + ExpectedRowStruct) 매핑 자산.

## 구현체 (FTableRowBase 자손)
- 🟢 [[entities/MCData_Sheet1]] — `FMCData_Sheet1 : FMCDataBase`. MCDataTableAuto 가 자동 생성한 시트 'Sheet1' 의 row USTRUCT.
- 🟢 [[entities/MCData_LootEntry]] — `FMCData_LootEntry : FMCDataBase`. 공용 LootEntry 풀 DT row (item_id/weight/min/max/rarity/condition/is_guaranteed). EMCLootRarity·FMCLootResultItem 동거. EMCTableKind::LootEntry Kind 로 로드. (CP-1 Option B 값 보유)

## 페어 카탈로그
- [[catalogs/subsystem]] — DataTable 호스트 Subsystem ([[entities/MCGameSubsystem]] UMCTableManager 소유).
- [[catalogs/assetuserdata]] — AssetUserData 분리 후 별도 카탈로그.

## 공통 함정
- **DataTable RowMap (`TMap<FName, uint8*>`) 재배치**: FindRow 반환 포인터 캐시 시 dangling. 즉시 사용 후 버리기.
- **phantom header**: `Templates/IsDerivedFrom.h` 미존재 — `Templates/UnrealTypeTraits.h` 사용.
- **USTRUCT BP virtual 호출 불가**: BP 노출 함수는 별도 BlueprintFunctionLibrary 로 wrap.
- **BlueprintCallable + `uint8*` 반환 금지**: UHT 거부. 타입 안전 템플릿 사용.
- **MetaStruct long path 강제** (UE 5.5+): `/Script/<ModuleName>.<StructName>` 포맷.
