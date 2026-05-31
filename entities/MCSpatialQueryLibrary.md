---
title: "UMCSpatialQueryLibrary"
kind: entity
category: BlueprintLibrary
base_class: UBlueprintFunctionLibrary
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/BlueprintLib/MCSpatialQueryLibrary.h
  - KMCProject/MCPlayModule/BlueprintLib/MCSpatialQueryLibrary.cpp
vault_refs:
  - sources/ue-spatialpartition-toctree2
  - synthesis/mc-actor-spawn-subsystem-implementation
  - sources/ue-ref-09-globaliteratorpolicy
  - sources/ue-coreuobject-interface
  - sources/ue-ref-07-profilingscopeRule
  - concepts/MC-Asset-Validation-Policy
  - sources/ue-significance-skill
last_ingested: 2026-05-29
---

# UMCSpatialQueryLibrary

## 한 줄 정의
공간 쿼리 정적 라이브러리 — [[entities/MCActorSpawnSubsystem]]::GetActorsInRadius 결과를 클래스/태그/Self/Interface/Cone/LOS 필터로 가공. 5 정적 BP API + `FMCSpatialQueryFilter` USTRUCT.

## 소속 / 상속
- 카테고리: BlueprintLibrary
- `UCLASS()` `class MCPLAYMODULE_API UMCSpatialQueryLibrary : public UBlueprintFunctionLibrary`
- 함께 정의: `USTRUCT(BlueprintType) FMCSpatialQueryFilter` — ClassFilter / RequiredTag / IgnoredActor / bRequireFilterableInterface / bRequirePawn.

## 핵심 구조
- **FMCSpatialQueryFilter UPROPERTY (EditAnywhere BlueprintReadWrite Category="MC|SpatialQuery|Filter")**:
  - `TSubclassOf<AActor> ClassFilter = nullptr` — nullptr 면 무시.
  - `FGameplayTag RequiredTag` — 빈 값이면 무시.
  - `TObjectPtr<AActor> IgnoredActor = nullptr` — nullptr 면 무시 (자기 제외 시 호출자 전달).
  - `bool bRequireFilterableInterface = false` — true 면 [[entities/MCSpatialQueryFilterable]] 구현 + `CanBeSpatialQueryResult` true 인 액터만.
  - `bool bRequirePawn = false` — APawn 자손만.
- **5 BP API (모두 static, meta=(WorldContext, AutoCreateRefTerm="Filter"))**:
  1. `GetActorsInRadius(WorldContext, Center, Radius, Filter, OutActors) → bool` — 반경 R 내 액터.
  2. `GetActorsInCone(WorldContext, Origin, Direction, Radius, ConeHalfAngleDegrees, Filter, OutActors) → bool` — 원뿔 안 (NPC perception 시야).
  3. `GetActorsInLineOfSight(WorldContext, Observer, Radius, VisibilityChannel, bIgnoreTargetItself, Filter, OutActors) → bool` — Observer 시점 트레이스 통과.
  4. `GetActorsInConeWithVisibility(WorldContext, Observer, Direction, Radius, ConeHalfAngleDegrees, VisibilityChannel, Filter, OutActors) → bool` — NPC AI perception 표준 (Cone + LOS).
  5. `GetClosestActorInConeWithVisibility(...) → AActor*` — §4 결과 첫 액터 (거리 정렬, 없으면 nullptr).
- **private 헬퍼**:
  - `FetchCandidatesAndFilter(...)` — Subsystem 1차 후보 + Filter 적용.
  - `IsInCone(Origin, NormalizedDirection, CosHalfAngle, TargetLocation)` — `dot >= cos(HalfAngle)`.
  - `HasLineOfSight(World, FromLocation, TargetActor, Channel, bIgnoreTargetItself, IgnoreObserver)` — LineTraceSingle.
  - `GetSubsystemValidated(WorldContextObject)` — Validation 매크로 적용.

## 따르는 패턴
- Octree 박스 prefilter + 정밀 거리 (Subsystem 측에서 적용) → [[sources/ue-spatialpartition-toctree2]] §2.5
- 게임 중 GetActorsInRadius 사용 사례 (본 라이브러리가 §4.2 표준 wrapper) → [[synthesis/mc-actor-spawn-subsystem-implementation]] §4.2
- TActorIterator 금지 + Subsystem 등록 패턴 대안 (본 라이브러리가 retrieve 표준 진입점) → [[sources/ue-ref-09-globaliteratorpolicy]] §4
- UINTERFACE Blueprintable + BlueprintNativeEvent 함정 (Filterable interface) → [[sources/ue-coreuobject-interface]] §5
- 모든 진입점 첫 줄 TRACE_CPUPROFILER_EVENT_SCOPE 의무 → [[sources/ue-ref-07-profilingscopeRule]]
- MC_LOGRET_IF_* 매크로 (Soft fail) → [[concepts/MC-Asset-Validation-Policy]]
- 후속 score 결합 옵션 (본 라이브러리는 retrieve 만, score 결합은 자손) → [[sources/ue-significance-skill]] §4
- 동기 동작 — Delegate 없음 (사용자 결정).
- Subsystem 미설치 World (Editor preview / Commandlet) → OutActors 비우고 false 반환 (Soft fail).

## ⚠ 함정
- **Direction 정규화 의무** (Cone): 외부 전달 Direction 이 비정규화 시 내부에서 Normalize. 0 벡터 입력 시 동작 미정 — 가드 의무 (🟡 — .cpp 확인 필요).
- **`bRequireFilterableInterface = true` + 인터페이스 미구현 액터**: 결과에서 제외. 의도 명확화 (🟢 — 헤더 주석 명시).
- **VisibilityChannel** 보통 `ECC_Visibility` — 다른 채널 시 의도된 trace 채널 확인 필요.
- **Observer 가 null** 일 때 — GetActorsInLineOfSight / WithVisibility 분기 가드 필요 (🟡).
- **Subsystem 미설치 — Soft fail 결과**: false 반환 + 빈 OutActors. 호출자가 false 결과 처리 분기 필요.
- 베이스 함정 적용.

## 연관 entity
- [[entities/MCActorSpawnSubsystem]] — 1차 Octree 후보 fetch.
- [[entities/MCSpatialQueryFilterable]] — bRequireFilterableInterface 체크 대상.
- FMCSpatialQueryFilter (본 entity 안 USTRUCT 정의).
