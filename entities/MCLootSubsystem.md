---
title: "UMCLootSubsystem"
kind: entity
category: Subsystem
base_class: UWorldSubsystem
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/MCLoot/Component/MCLootSubsystem.h
  - KMCProject/MCPlayModule/MCLoot/Component/MCLootSubsystem.cpp
vault_refs:
  - concepts/Subsystem-5-Types
  - sources/ue-spatialpartition-toctree2
  - entities/USubsystem
  - synthesis/subsystem-advanced-patterns
  - sources/ue-gameframework-world
last_ingested: 2026-05-29
---

# UMCLootSubsystem

## 한 줄 정의
월드 단위 루팅 후보 풀 관리 + 거리비교 기반 최근접 후보 선정. 게임/PIE 월드에서만 인스턴스화.

## 소속 / 상속
- 대분류: Subsystem (World Subsystem)
- `UCLASS()` `class MCPLAYMODULE_API UMCLootSubsystem : public UWorldSubsystem`
- 충족 REQ: REQ-3, REQ-4(지원). 설계 출처: `_workspace/02_ue-architect_design.md §2.6`.
- 라이프사이클: Map 단위 World-bound. 컴포넌트 BeginPlay/EndPlay 로만 Register/Unregister.
- **Q-1 설계 근거**: USphereComponent 오버랩은 액터에 PrimitiveComponent 강제 → REQ-1 "컴포넌트 1개만 부착" 위반 위험. 대신 [[entities/MCLootableComponent]] 가 자기 등록, subsystem 이 `FVector::DistSquared` 선형 스캔으로 최근접 선정. 소규모 풀이므로 Octree 미사용(대규모 확장 시 TOctree2 도입). (근거: `concepts/Subsystem-5-Types "UWorldSubsystem = Map 단위 World-bound 매니저"; sources/ue-spatialpartition-toctree2 §2.4 Register/Unregister/Query, §함정#1 "Element 는 TWeakObjectPtr + IsValid"`)

## 핵심 구조
- **`virtual bool ShouldCreateSubsystem(UObject* Outer) const override`**: `Super::ShouldCreateSubsystem` 폴백 후, `Cast<UWorld>(Outer)->IsGameWorld()` 로 Game/PIE 한정 (EditorPreview/Inactive 제외). (근거: `synthesis/subsystem-advanced-patterns §3` 조건 분기 후 Super 폴백; `sources/ue-gameframework-world` WorldType)
- **API (UFUNCTION BlueprintCallable, Category="MC|Loot")**:
  - `void RegisterLootable(UMCLootableComponent*)` — `AddUnique`(중복 방지). LootableComponent::BeginPlay 호출. (REQ-3)
  - `void UnregisterLootable(UMCLootableComponent*)` — `RemoveAll` 람다로 대상 + 죽은 약참조 함께 청소. EndPlay 호출.
  - `UMCLootableComponent* FindClosestLootable(const FVector& Location) const` — InteractRadius 내 + IsPromptable(Idle/Locked) 후보 중 최근접 1개, 없으면 nullptr. (REQ-3, REQ-4)
- **`FindClosestLootable` 로직**: 각 후보 `Weak.Get()` → `IsValid` 가드 → `IsPromptable()` 필터 → `RadiusSq = InteractRadius²`, `DistSq = DistSquared(GetInteractLocation(), Location)`; `DistSq <= RadiusSq && DistSq < BestDistSq` 면 갱신.
- **멤버**: `UPROPERTY() TArray<TWeakObjectPtr<UMCLootableComponent>> RegisteredLootables` — 등록 후보 풀.

## 따르는 패턴
- UWorldSubsystem = Map 단위 World-bound 매니저 → [[concepts/Subsystem-5-Types]] / [[entities/USubsystem]]
- ShouldCreateSubsystem 조건 분기 후 Super 폴백 (게임/PIE 한정) → [[synthesis/subsystem-advanced-patterns]] §3
- Register/Unregister/Query 표준 패턴 + 후보 약참조 보관 → [[sources/ue-spatialpartition-toctree2]] §2.4
- 거리비교 채택(콜리전 컴포넌트 회피, REQ-1 충족) — Q-1. 선형 DistSquared 스캔(정밀 거리, §2.5).
- IsGameWorld() = Game/PIE 만 → [[sources/ue-gameframework-world]] WorldType (Game/PIE/Editor/EditorPreview)

## ⚠ 함정
- **죽은 약참조 청소 의무**: `TWeakObjectPtr` 풀은 `IsValid` 가드 필수 — `FindClosestLootable` 에서 invalid skip, `UnregisterLootable` 에서 `!Weak.IsValid()` 항목도 함께 제거. → [[sources/ue-spatialpartition-toctree2]] §함정#1.
- **비게임 월드 인스턴스 회피**: `ShouldCreateSubsystem` 이 false 면 인스턴스 안 만듦 — EditorPreview/Inactive 월드에서 항상 빈 채로 무의미하므로 의도적 제외. → [[entities/USubsystem]] "ShouldCreateSubsystem false → 인스턴스 X".
- **선형 스캔 한계**: 소규모 후보 풀 전제. 대규모 확장 시 TOctree2 도입 필요(현재 미적용). → [[sources/ue-spatialpartition-toctree2]].
- **PIE 다중 인스턴스**: World 마다 별도 subsystem 인스턴스 — 후보 풀도 World 별 분리(공유 아님). (Subsystem 공통 함정)

## 연관 entity
- [[entities/MCLootableComponent]] — Register/Unregister/FindClosest 의 후보 요소 (`GetInteractLocation`/`InteractRadius`/`IsPromptable` 제공).
- [[entities/MCActorSpawnSubsystem]] — 동일 World Subsystem + 공간 인덱스 패턴 선례(이쪽은 TOctree2 사용).
