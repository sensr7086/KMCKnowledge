---
title: "UMCActorSpawnSubsystem"
kind: entity
category: Subsystem
base_class: UWorldSubsystem
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/Subsystem/MCActorSpawnSubsystem.h
  - KMCProject/MCPlayModule/Subsystem/MCActorSpawnSubsystem.cpp
vault_refs:
  - sources/ue-subsystem-skill
  - concepts/Subsystem-5-Types
  - concepts/Global-Iterator-Avoidance
  - synthesis/spawnactor-hitching-4-step-pattern
  - synthesis/actor-pool-reset-pattern
  - sources/ue-ref-11-assetloadingpolicy
  - concepts/Profiling-Scope-Rule
  - concepts/MC-Asset-Validation-Policy
  - sources/ue-significance-skill
  - sources/ue-spatialpartition-toctree2
  - synthesis/toctree2-worldpartition-pair-pattern
  - synthesis/cooked-first-frame-stability
  - sources/ue-coreuobject-interface
last_ingested: 2026-05-29
---

# UMCActorSpawnSubsystem

## 한 줄 정의
런타임 액터 스폰 관리 — Soft Class 비동기 로드 + SpawnActorDeferred + Pool + USignificanceManager 통합 + TOctree2 공간 인덱스 + 콘솔 디버그 명령 4종.

## 소속 / 상속
- 대분류: Actor (역할 기준 — 액터 라이프사이클·풀·공간 인덱스 관리)
- `UCLASS()` `class MCPLAYMODULE_API UMCActorSpawnSubsystem : public UWorldSubsystem`
- 라이프사이클: 레벨별, World Travel 시 재생성. 싱글플레이 (Authority/Replication 분기 X).
- 함께 정의:
  - `USTRUCT() FMCPooledActorArray` — `TArray<TObjectPtr<AActor>> Actors`. TMap value 용 wrapper (UPROPERTY TMap value 가 TArray<TObjectPtr<>> 직접 못 받음).
  - `DECLARE_DYNAMIC_DELEGATE_OneParam` 3종: `FMCOnActorSpawnedDelegate(AActor*)`, `FMCOnActorSpawnFailedDelegate(FSoftObjectPath)`, `FMCOnActorClassesPreloadedDelegate(int32)`.

## 핵심 구조
- **생성자/소멸자**: `UMCActorSpawnSubsystem()` + `virtual ~UMCActorSpawnSubsystem() override` — Pimpl `TPimplPtr<FOctreeData>` 의 .cpp 안 destruct 필요.
- **USubsystem Interface**:
  - `virtual bool ShouldCreateSubsystem(UObject*) const override`
  - `virtual void Initialize(FSubsystemCollectionBase&) override`
  - `virtual void Deinitialize() override`
- **동작 옵션 UPROPERTY**:
  - `int32 PoolHardLimitPerClass = 32` (0~1024) — 초과 시 Destroy fallback.
  - `int32 LoadPriority = 0` (0~200).
- **Significance UPROPERTY**:
  - `bool bEnableSignificanceManagement = false` — 옵션 (단일 캐릭터 게임 디폴트 off).
  - `FName DefaultSignificanceTag = NAME_None`.
  - `float SignificanceNearDistance = 1000.f`, `SignificanceFarDistance = 10000.f` — score 0~1 선형 보간.
- **Spatial Index UPROPERTY**:
  - `bool bEnableSpatialIndex = true`.
  - `double SpatialIndexExtent = 10000000.0` (100km, LWC 권장).
- **디버그 UPROPERTY**:
  - `bool bDebugDrawEnabled = false` (Transient) — 콘솔 명령 `mc.spawn.octree.draw.toggle` 로 토글.
- **BP API (Spawn / Pool)**:
  - `void RequestSpawnAsync(TSoftClassPtr<AActor>, FTransform, FMCOnActorSpawnedDelegate, FMCOnActorSpawnFailedDelegate)` — 메모리 상주면 즉시 SpawnActor (또는 풀 acquire), 미로드면 FStreamableManager 비동기 → SpawnActorDeferred + FinishSpawning.
  - `void ReleaseToPool(AActor*)` — Destroy 대신 풀 반환. 한도 초과 시 자동 Destroy.
  - `void WarmUpPool(TSoftClassPtr<AActor>, int32 NumInstances)` — 사전 spawn (첫 SpawnActor 히칭 분산).
  - `void ClearAllPools()` — 모든 액터 destroy (World tear-down 외 비권장).
- **BP API (Bundle Preload)**:
  - `void PreloadActorClasses(TArray<TSoftClassPtr<AActor>>, FMCOnActorClassesPreloadedDelegate)` — 여러 SoftClass 한 핸들로 묶음 로드.
  - `int32 GetPreloadedClassCount() const`, `void ReleasePreloadedClasses()`.
- **BP API (쿼리)**:
  - `int32 GetAvailableCount(TSubclassOf<AActor>) const`, `int32 GetActiveCount(TSubclassOf<AActor>) const`.
- **BP API (Spatial Index)**:
  - `void GetActorsInRadius(FVector Center, float Radius, TArray<AActor*>&) const` — O(log N + K).
  - `void NotifyActorMoved(AActor*)` — Octree Remove+Add 페어 (자손 명시 호출 의무, 자동 갱신 비용 회피).
  - `int32 GetSpatialIndexCount() const`.
- **protected 헬퍼**: `HandleClassLoaded(...)`, `AcquireFromPool(TSubclassOf<AActor>, FTransform)`, `SpawnNewActorDeferred(...)`, `ActivateActor(...)`, `DeactivateActor(...)`, `RegisterActorToSignificance(...)`, `UnregisterActorFromSignificance(...)`, `RegisterActorToOctree(...)`, `UnregisterActorFromOctree(...)`.
- **콘솔 명령 (디버그)**: 4종 — `mc.spawn.dump` / `.octree.draw` / `.octree.query <X><Y><Z><R>` / `.significance.dump`. 각 `CmdXxx(const TArray<FString>& Args)`.
- **private 상태**:
  - `UPROPERTY(Transient) TMap<TSubclassOf<AActor>, FMCPooledActorArray> AvailablePoolByClass` — 비활성 풀 (Strong).
  - `UPROPERTY(Transient) TArray<TObjectPtr<AActor>> ActiveActors` — 활성 액터 (Strong).
  - `TArray<TSharedPtr<FStreamableHandle>> ActiveLoadHandles` — 진행 중 핸들 (Pin 보관 의무).
  - `struct FOctreeData;` (전방 선언) + `TPimplPtr<FOctreeData> OctreeData` — .h 가 GenericOctree.h 의존 안 함.
  - `TArray<TUniquePtr<FAutoConsoleCommand>> ConsoleCommands`.
  - `UPROPERTY(Transient) TArray<TSubclassOf<AActor>> PreloadedClassCache` — Strong ref (GC 방어).
  - `TSharedPtr<FStreamableHandle> PreloadHandle`.

## 따르는 패턴
- USubsystem 베이스 3 virtual + 5 종 결정 → [[sources/ue-subsystem-skill]] §3
- UWorldSubsystem 선택 — 레벨별 라이프사이클 → [[concepts/Subsystem-5-Types]]
- TActorIterator 대안 (액터 컬렉션 자체 관리) → [[concepts/Global-Iterator-Avoidance]]
- 4단 회피 (Soft + Deferred + FinishSpawning + Pool) → [[synthesis/spawnactor-hitching-4-step-pattern]] §2
- Pool 라이프사이클 + WorldSubsystem 골격 → [[synthesis/actor-pool-reset-pattern]] §2/§5
- SpawnActorDeferred 4단 표준 → [[sources/ue-ref-11-assetloadingpolicy]] §2.6
- PreLoadAsset 5대 의무 → [[sources/ue-ref-11-assetloadingpolicy]] §2.7
- 사전 spawn 패턴 → [[synthesis/cooked-first-frame-stability]]
- USignificanceManager + FManagedObjectInfo → [[sources/ue-significance-skill]] §2
- TOctree2 박스 prefilter + 정밀 거리 (Spatial Index) → [[sources/ue-spatialpartition-toctree2]] §2.4 / §2.5
- BeginPlay/EndPlay 자동 동기 (TOctree2-WorldPartition 페어) → [[synthesis/toctree2-worldpartition-pair-pattern]] §3.1
- 프로파일링 스코프 → [[concepts/Profiling-Scope-Rule]]
- silent return 금지 / MC_LOGRET_IF_NULL → [[concepts/MC-Asset-Validation-Policy]]
- Pimpl + TUniquePtr C4150 회피 (TPimplPtr<>) → [[sources/ue-coreuobject-interface]] §5 함정 후속

## ⚠ 함정
- **§2.8 함정 #3 (vault) — Handle Pin 의무**: `ActiveLoadHandles` / `PreloadHandle` 미보관 시 GC → 콜백 도착 전 핸들 해제. → [[sources/ue-ref-11-assetloadingpolicy]] §2.8.
- **§3 함정 #5 (vault) — TOctree2 Extent 너무 작으면**: 액터 outside → 검색 누락. `SpatialIndexExtent` 최소 10000.0 ClampMin. LWC 권장 100km (디폴트).
- **§3 함정 #3 (vault) — Element 위치 변경 시 Remove+Add 페어**: 자동 갱신 (매 tick 모든 active actor scan) 은 비용 → 자손 책임 `NotifyActorMoved`. AMCPooledActor 자손이 큰 이동 시 호출, 정적/작은 이동은 호출 불필요.
- **Pimpl + TUniquePtr<IncompleteType> C4150**: `struct FOctreeData;` 전방 선언만으로 `TUniquePtr<FOctreeData>` 멤버 사용 시 destroy 함수 capture 실패 → 클래스 정의 단계에서 인스턴스화 실패. UE 의 `TPimplPtr<>` 는 `MakePimpl<T>()` 시점에 destroy 함수 capture 해 incomplete 안전. → 본 fix 가 vault [[sources/ue-coreuobject-interface]] §5 후속 1차 사례.
- **§7.1 함정 #3 (vault, MCSpatialQueryFilterable 와 공유) — `meta=(CannotImplementInterfaceInBlueprint="false")` 금지** — Blueprintable specifier 단독으로 BP implement 허용.
- **PoolHardLimitPerClass 초과 시 ReleaseToPool 자동 Destroy fallback**: 호출자는 fallback 발생 알 수 없음 — 메모리 추적 시 의도와 다른 destroy 가능.
- **bDebugDrawEnabled 토글은 콘솔 명령으로만** — 디테일 패널 ReadOnly. 다음 콘솔 명령 (draw/query) 실행 시점에 발화.

## 연관 entity
- [[entities/MCPooledActor]] — 풀 통합 베이스 (사용 방식 A).
- [[entities/MCPoolableInterface]] — 풀 hook 인터페이스 (사용 방식 B).
- [[entities/MCSpatialQueryFilterable]] — `GetActorsInRadius` 의 액터 측 필터 (bRequireInterface 옵션).
- UMCSpatialQueryLibrary (미인덱싱 — BPLib 배치, GetActorsInRadius 의 외부 BP wrapper 후보).
