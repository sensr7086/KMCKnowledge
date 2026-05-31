---
title: "UMCTableManager"
kind: entity
category: DataTable
base_class: UObject
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/MCGame/MCTableManager.h
  - KMCProject/MCPlayModule/MCGame/MCTableManager.cpp
vault_refs:
  - synthesis/subsystem-advanced-patterns
  - concepts/Subsystem-5-Types
  - concepts/UE-FStructProperty-Cast-Type-Safety
  - sources/ue-coreuobject-uobject
  - sources/ue-assetclasses-data
  - entities/UGameInstance
  - concepts/Asset-Loading-Policy
  - concepts/UE-Phantom-Header-Hallucination-Hazard
last_ingested: 2026-05-29
---

# UMCTableManager

## 한 줄 정의
UDataTable 컬렉션을 Kind 단위로 관리하는 전역 관리자. UMCGameSubsystem 이 UPROPERTY 로 소유(패턴 B).

## 소속 / 상속
- 대분류: DataTable
- `UCLASS(Within=MCGameSubsystem, BlueprintType)`
- `class MCPLAYMODULE_API UMCTableManager : public UObject`
- `Within=MCGameSubsystem` → 외부 NewObject 차단 (Outer 강제), 다른 Outer 로 생성 시 ensure
- 라이프사이클: 부모 (UMCGameSubsystem) 의 Initialize 안에서 NewObject + Initialize 호출, Deinitialize 가 역순 해제

## 핵심 구조
- **부모 전용 호출**:
  - `void Initialize(const UMCTableRegistry*)` — Registry 의 모든 Kind 에 대해 sync `TryLoad`. Initialize 안 1회.
  - `void Deinitialize()` — `LoadedTables.Reset()` + `Registry=nullptr` 명시 해제 (UPROPERTY GC 의존 안 함).
- **타입 안전 row lookup (C++ 템플릿, 헤더 inline)**:
  - `template<typename T> const T* FindRowAs(EMCTableKind Kind, FName RowName) const` — 2단 검사.
    - Layer 1 (테이블 단위): `Table->GetRowStruct()->IsChildOf(T::StaticStruct())`
    - Layer 2 (즉시 사용 강제): `reinterpret_cast<const T*>(Table->FindRowUnchecked(RowName))`
  - `template<typename T> static const T* SafeCast(const FMCDataBase* Row)` — 인스턴스 단위 캐스트 (캐시된 row 검증용). FMCDataBase::CastTo<T> 의 static wrapper.
- **BP 친화 API**:
  - `UFUNCTION(BlueprintCallable) UDataTable* GetTable(EMCTableKind Kind) const`
  - `UFUNCTION(BlueprintCallable) void GetRowNames(EMCTableKind, TArray<FName>&) const`
  - `const UMCTableRegistry* GetRegistry() const`
- **#if WITH_EDITOR**:
  - `void ReloadTable(EMCTableKind Kind)` — CSV 재import 후 호출
  - `void ReloadAll()`
- **멤버 (UPROPERTY)**:
  - `TObjectPtr<const UMCTableRegistry> Registry` — Initialize 입력. GC 추적.
  - `TMap<EMCTableKind, TObjectPtr<UDataTable>> LoadedTables` — Strong ref. 게임 진행 동안 메모리 상주.
- **내부**: `bool ValidateExpectedStruct(EMCTableKind, const UDataTable*) const` — Registry 의 ExpectedRowStruct 검증.
- 매크로 의존: `TIsDerivedFrom<T, FMCDataBase>::Value` static_assert (T 가 FMCDataBase 자손 강제).

## 따르는 패턴
- 부모-자식 UObject 소유 (패턴 B) — Subsystem 안에 Subsystem nesting 회피 → [[synthesis/subsystem-advanced-patterns]] §부모-자식 UObject 소유
- UGameInstanceSubsystem 라이프사이클 — Map 전환 살아남음 → [[concepts/Subsystem-5-Types]] / [[entities/UGameInstance]]
- 타입 검증 IsChildOf 정공법 → [[concepts/UE-FStructProperty-Cast-Type-Safety]]
- Sync 로드 1회 (Initialize 시점), 후속 async Bundle 패턴 전환 여지 → [[concepts/Asset-Loading-Policy]] §SoftObjectPath TryLoad sync
- DataTable RowMap dangling 회피 패턴 — UE Engine SRowEditor::FStructFromDataTable → [[sources/ue-coreuobject-uobject]] / [[sources/ue-assetclasses-data]]

## ⚠ 함정
- **FindRow 반환 포인터 캐시 금지**: RowMap (TMap<FName, uint8*>) 재배치 시 dangling. 즉시 사용 후 버리기, 재호출 시 다시 FindRowAs.
- **phantom header**: `Templates/IsDerivedFrom.h` 미존재 (grep 검증 — Engine/Source 0건). `TIsDerivedFrom` 권위 위치는 `Templates/UnrealTypeTraits.h` (L37-60) → [[concepts/UE-Phantom-Header-Hallucination-Hazard]]
- **`BlueprintCallable + uint8* 반환` 금지**: UHT 거부 (raw 포인터는 BP reflection 노출 불가). 2026-05-25 — `FindRowRawUnsafe` 삭제됨. 같은 목적은 `FindRowAs<T>` (C++ 템플릿) 또는 `GetTable(Kind)->FindRowUnchecked(RowName)` 직접 호출.

## 연관 entity
- [[entities/MCGameSubsystem]] — 소유자 (Within=MCGameSubsystem). 외부 진입점.
- [[entities/MCTableRegistry]] — Kind→경로+기대 RowStruct 매핑.
- [[entities/MCDataBase]] — FindRowAs<T>·SafeCast<T> 의 T 제약 베이스.
- [[entities/MCData_Sheet1]] — 시트 자손 예시.
