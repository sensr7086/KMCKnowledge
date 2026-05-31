---
title: "FMCDataBase"
kind: entity
category: DataTable
base_class: FTableRowBase
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/MCGame/MCDataBase.h
vault_refs:
  - concepts/UE-FStructProperty-Cast-Type-Safety
  - entities/FStructProperty
  - sources/ue-coreuobject-uobject
  - sources/ue-coreuobject-structutils
  - sources/ue-assetclasses-data
  - concepts/UE-Phantom-Header-Hallucination-Hazard
last_ingested: 2026-05-29
---

# FMCDataBase

## 한 줄 정의
KMCProject 모든 DataTable row 의 베이스 USTRUCT — 타입 안전 다운캐스트(`IsA<T>`/`CastTo<T>`) + 공통 row 헬퍼 제공.

## 소속 / 상속
- 대분류: DataTable
- `struct MCPLAYMODULE_API FMCDataBase : public FTableRowBase`
- `USTRUCT(BlueprintType)` — BP 노출
- 상속 체인: `FMCDataBase → A → B` (자유 다단 상속 허용)
- 라이프사이클: UDataTable row 단위 인스턴스. `virtual ~FMCDataBase() = default` — FTableRowBase 가 이미 virtual dtor 보유(vtable 존재).

## 핵심 구조
- **Layer 1 — 타입 정체성**: `virtual const UScriptStruct* GetScriptStruct() const` — 자손이 override, 자기 StaticStruct() 반환. 매크로 `MC_DATA_BODY(StructType)` 가 override 자동화.
- **Layer 2 — 다형성 검사**: `template<typename T> bool IsA() const` — `UScriptStruct::IsChildOf(T::StaticStruct())` 로 검증. `static_assert(TIsDerivedFrom<T, FMCDataBase>::Value)` 로 컴파일 타임 가드.
- **Layer 3 — 안전 캐스트**: `template<typename T> const T* CastTo() const` / `T* CastTo()` — IsA<T> 통과 후 `static_cast` (reinterpret 아님 → 다중 상속·virtual base 안전).
- **#if WITH_EDITOR**: `FString GetTypeName() const` — 디버그 로깅 헬퍼.
- **매크로 `MC_DATA_BODY(StructType)`**: 자손이 `GENERATED_BODY()` 아래에 한 줄로 추가 → `GetScriptStruct()` override 자동 생성. StructType 토큰 1번만 적음 + override 키워드 자동.

## 따르는 패턴
- USTRUCT 다형성 정공법 = `UScriptStruct::IsChildOf` → [[concepts/UE-FStructProperty-Cast-Type-Safety]]
- USTRUCT 등록된 타입은 plain C++ struct (FExpressionInput 등 `::StaticStruct()` 불가) 와 달리 IsChildOf 직접 사용 가능 → [[entities/FStructProperty]] §Struct->IsChildOf
- FTableRowBase / FInstancedStruct 대안 패턴 → [[sources/ue-coreuobject-structutils]]
- UDataTable row 단위 USTRUCT 컬렉션 → [[sources/ue-assetclasses-data]] §UDataTable
- B 회피 패턴 (FStructOnScope 자손 override) → [[sources/ue-coreuobject-uobject]] §B

## ⚠ 함정
- **DataTable RowMap (`TMap<FName, uint8*>`) 재배치**: 행 add/remove 시 메모리 재배치 → FindRow 반환 포인터 캐시하면 dangling. **반환 즉시 사용 → 버리기**. 재호출 시 다시 FindRow. UE Engine 자체가 `SRowEditor.cpp:44-98` 의 FStructFromDataTable 로 동일 패턴 회피.
- **virtual 메서드 BP 호출 불가**: USTRUCT 의 BP 노출 한계. BP 노출이 필요한 함수는 별도 `UMCDataLibrary` 의 typed UFUNCTION 으로 wrap.
- **phantom header 주의**: `Templates/IsDerivedFrom.h` 는 UE 5.7.4 권위 source 에 **존재한 적 없는 헤더**. `TIsDerivedFrom` 의 권위 위치는 `Templates/UnrealTypeTraits.h` (L37-60 자체 정의) → [[concepts/UE-Phantom-Header-Hallucination-Hazard]]

## 연관 entity
- [[entities/MCData_Sheet1]] — 자손 시트 예시
- [[entities/MCTableManager]] — FindRowAs<T> 의 T 제약 베이스로 사용
- [[entities/MCTableRegistry]] — ExpectedRowStruct 검증 시 베이스로 사용
