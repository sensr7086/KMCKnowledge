---
title: "AMCPooledActor"
kind: entity
category: Actor
base_class: AActor
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/Subsystem/MCPooledActor.h
  - KMCProject/MCPlayModule/Subsystem/MCPooledActor.cpp
vault_refs:
  - synthesis/actor-pool-reset-pattern
  - concepts/Profiling-Scope-Rule
  - concepts/MC-Asset-Validation-Policy
  - sources/ue-significance-skill
  - synthesis/character-many-npc-5-fold-optimization
last_ingested: 2026-05-29
---

# AMCPooledActor

## 한 줄 정의
풀링 표준 베이스 액터 — `IMCPoolableInterface` 구현 + 기본 Tick/Hidden/Collision 토글 + Significance 통합.

## 소속 / 상속
- 대분류: Actor
- `UCLASS(Abstract, BlueprintType, Blueprintable)` `class MCPLAYMODULE_API AMCPooledActor : public AActor, public IMCPoolableInterface`
- 다단 상속 동반: IMCPoolableInterface
- 라이프사이클: 풀 첫 SpawnActor 시 비활성 상태로 시작 → Subsystem 토글로 활성/비활성 전환.

## 핵심 구조
- **생성자 디폴트**:
  - Tick OFF (자손이 켜기 전까지)
  - `SetActorHiddenInGame(true)`
  - `SetActorEnableCollision(false)`
  - `bCanBeDamaged = false` (자손 결정)
- **IMCPoolableInterface 구현**:
  - `virtual void OnPoolActivate_Implementation(const FTransform& SpawnTransform) override`
  - `virtual void OnPoolDeactivate_Implementation() override`
  - `virtual bool IsPoolActive_Implementation() const override { return bIsPoolActive != 0; }`
  - `virtual FName GetSignificanceTag_Implementation() const override`
  - `virtual void OnSignificanceChanged_Implementation(float NewScore, float OldScore) override`
- **AActor Interface**:
  - `virtual void BeginPlay() override`
  - `virtual void EndPlay(const EEndPlayReason::Type EndPlayReason) override`
- **BlueprintImplementableEvent (BP 노드)**:
  - `K2_OnPoolReset(const FTransform& SpawnTransform)` — `Event On Pool Reset`. 의무: weak ptr 무효화, 누적 lifetime/counter reset, VFX/Audio Activate.
  - `K2_OnPoolCleanup()` — `Event On Pool Cleanup`. 의무: 비동기 핸들 Cancel, 외부 콜백 unbind, VFX/Audio Deactivate.
  - `K2_OnSignificanceChanged(float NewScore, float OldScore)` — `Event On Significance Changed (BP)`. 예: score<0.1 → Tick OFF/VFX Deactivate, 0.5+ → Tick ON/full LOD.
- **UPROPERTY**:
  - `EditAnywhere BlueprintReadWrite FName SignificanceTag = NAME_None` — 빈 값이면 Subsystem 의 `DefaultSignificanceTag` 사용.
  - `VisibleInstanceOnly BlueprintReadOnly Transient uint8 bIsPoolActive : 1` — 풀 활성 상태 비트필드.
- **BlueprintPure**: `bool IsActive() const { return bIsPoolActive != 0; }`.

## 따르는 패턴
- AMCPooledActor 표준 골격 (Tick OFF / Hidden / NoCollision 으로 시작) → [[synthesis/actor-pool-reset-pattern]] §3
- 라이프사이클 매트릭스 → [[synthesis/actor-pool-reset-pattern]] §2
- 콜백 첫 줄 TRACE_CPUPROFILER_EVENT_SCOPE → [[concepts/Profiling-Scope-Rule]]
- MC_LOGRET_IF_NULL / silent return 금지 → [[concepts/MC-Asset-Validation-Policy]]
- Significance Tag → [[sources/ue-significance-skill]] §2 Tag
- Significance 누적 적용 (Tick/VFX/LOD 분기) → [[synthesis/character-many-npc-5-fold-optimization]] §3

## ⚠ 함정
- **`IsValid(Actor)` 만으로 풀 상태 식별 불가** — 명시 플래그 의무. `bIsPoolActive` UPROPERTY 가 이 목적. → [[synthesis/actor-pool-reset-pattern]] §6 함정 #1.
- Constructor 안 디폴트는 자손이 꺼두지 않으면 첫 SpawnActor 가 비활성 상태 유지. K2_OnPoolReset 단계에서 자손이 명시 활성화 책임.
- 부분 ragdoll/VFX 진행 중 OnPoolDeactivate 시 외부 시스템 콜백 unbind 누락하면 stale callback 발생 가능.

## 연관 entity
- [[entities/MCPoolableInterface]] — 구현 인터페이스 (Pool hook).
- [[entities/MCActorSpawnSubsystem]] — 풀 호스트.
