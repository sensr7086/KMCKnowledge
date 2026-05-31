---
title: "UMCLootableComponent"
kind: entity
category: Component
base_class: UActorComponent
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/MCLoot/Component/MCLootableComponent.h
  - KMCProject/MCPlayModule/MCLoot/Component/MCLootableComponent.cpp
vault_refs:
  - concepts/Component-Lifecycle
  - sources/ue-spatialpartition-toctree2
  - entities/UAnimMontage
last_ingested: 2026-05-29
---

# UMCLootableComponent

## 한 줄 정의
루팅 가능 오브젝트에 1개만 부착하는 단일 컴포넌트 — subsystem 감지 등록 + Interact 트리거 + 5상태 머신 + 굴림/지급 흐름.

## 소속 / 상속
- 대분류: Component
- `UCLASS(ClassGroup=(MCLoot), Blueprintable, meta=(BlueprintSpawnableComponent))` `class MCPLAYMODULE_API UMCLootableComponent : public UActorComponent`
- 충족 REQ: REQ-1~4, REQ-6, REQ-7, REQ-26~32. 설계 출처: `_workspace/02_ue-architect_design.md §2.5`.
- REQ-1: 이 컴포넌트 1개만 붙이면 루팅 가능 — 추가 콜리전 컴포넌트 불요(Q-1 거리비교 채택).
- Q-2: 입력 직접 바인딩 안 함. 외부(Character/Controller)가 `Interact(Pawn)` 호출. Enhanced Input 은 호출측 책임.
- `PrimaryComponentTick.bCanEverTick = false` — 감지=subsystem 거리비교, 리젠=타이머 → Tick 불요. (근거: `concepts/Component-Lifecycle "기본 bCanEverTick=false"`)

### 동거 지원 타입: `EMCLootableState` (별도 페이지 없음)
- `UENUM(BlueprintType) enum class EMCLootableState : uint8` (`MCLootTypes.h` 정의) — 5상태 머신 (REQ-26):
  - `Idle` — 초기 상태, 프롬프트 표시 가능 (REQ-27)
  - `Looting` — 상호작용 시작, LootingDuration 동안 애니메이션 (REQ-28)
  - `Looted` — 굴림/지급 완료, 프롬프트 숨김 (REQ-29)
  - `Cooldown` — 리젠 타이머 동작 중, RegenTime 후 Idle 복귀 (REQ-30)
  - `Locked` — 조건 미충족(열쇠 등), 다른 프롬프트 (REQ-31)

## 핵심 구조
- **델리게이트 (DYNAMIC_MULTICAST, BlueprintAssignable)**:
  - `FMCOnLootStateChanged OnStateChanged(EMCLootableState)`
  - `FMCOnLootCompleted OnLootCompleted(const TArray<FMCLootResultItem>&)` — LootUI(ShowResult)가 구독. (REQ-24)
  - `FMCOnLootRejected OnLootRejected(FName Reason)` — 거절 사유. (REQ-7)
  - `FMCOnItemDropped OnItemDropped(FName ItemId, int32 Count)` — 인벤토리 가득 시 드롭 훅(실제 드롭 액터 v1 미생성). (Q-5, REQ-23)
- **설정 UPROPERTY (EditAnywhere)**: `TObjectPtr<UMCLootTableAsset> LootTable`(REQ-2), `float InteractRadius=200`(Q-1), `float RegenTimeSeconds=-1`(-1=일회성, >=0=Cooldown 복귀; Q-3/Q-6), `bool bRerollOnRegen=true`(Q-3), `int32 LootSeed=0`(REQ-15), `float LootingDuration=0.5`(REQ-28), `bool bStartLocked=false`(REQ-31), `TObjectPtr<UMCInventoryComponent> TargetInventoryOverride`(REQ-23).
- **런타임 상태**: `EMCLootableState State = Idle` `VisibleInstanceOnly, BlueprintReadOnly`. (REQ-26)
- **공개 API (UFUNCTION BlueprintCallable)**:
  - `void Interact(APawn* InInstigator)` — 외부 트리거. CanBeLooted 통과 시 Looting 진입. (REQ-6)
  - `bool CanBeLooted(FName& OutRejectReason) const` — Looted/Looting(Busy)/Cooldown/Locked/NoLootTable 거절 분기. (REQ-7)
  - `bool IsConditionMet(FName Condition) const` — `BlueprintNativeEvent, BlueprintCallable`. 기본 `_Implementation` = true; BP override 로 condition 태그별 평가. Roller 가 콜백. (REQ-13, Q-7)
  - `FText GetPromptText() const` `BlueprintPure` — Locked="잠겨있다", Idle="[E] 줍기", 그 외 빈 텍스트. (REQ-4, REQ-29, REQ-31). `NSLOCTEXT("MCLoot", ...)`.
  - `void SetLocked(bool)` — Locked ↔ Idle (진행 중이면 무시). (REQ-31)
- **subsystem 지원 (비-UFUNCTION)**: `FVector GetInteractLocation() const`(소유 액터 위치), `bool IsPromptable() const`(Idle/Locked = inline).
- **라이프사이클**: `BeginPlay`(Super FIRST, bStartLocked→Locked 직접 설정, subsystem RegisterLootable), `EndPlay`(타이머 2개 Clear, UnregisterLootable, Super LAST). (근거: `concepts/Component-Lifecycle`, `ue-spatialpartition-toctree2 §2.6` BeginPlay 등록/EndPlay 해제)
- **private 흐름 함수**: `SetState`(중복 가드 + Broadcast 단일 진입), `OnLootingFinished`(Roll→GrantResults→Looted+OnLootCompleted→FinishLootFlow), `GrantResults`(인벤토리 Add, Overflow→OnItemDropped), `FinishLootFlow`(RegenTimeSeconds<0 이면 Looted 고정, else EnterCooldown), `EnterCooldown`(Cooldown+RegenTimer), `OnRegenComplete`(bRerollOnRegen&&LootSeed!=0 시 시드 흔듦→Idle), `ResolveTargetInventory`(Override 우선, 없으면 Instigator FindComponentByClass).
- **멤버**: `FTimerHandle LootingTimerHandle/RegenTimerHandle`, `TWeakObjectPtr<APawn> CurrentInstigator`.

## 따르는 패턴
- 기본 bCanEverTick=false (감지=subsystem, 리젠=타이머) → [[concepts/Component-Lifecycle]]
- BeginPlay 등록 / EndPlay 해제 (subsystem 후보 풀) → [[sources/ue-spatialpartition-toctree2]] §2.6
- 굴림 위임 — [[entities/MCLootRoller]] `Roll(this, LootTable, LootSeed, this, Results)` (자기 자신이 ConditionSource).
- 지급 위임 — [[entities/MCInventoryComponent]] `Add(...)`, Overflow 는 OnItemDropped 훅.
- 일회성/리젠 분기 (RegenTimeSeconds 부호) — Q-3/Q-6 의도. bRerollOnRegen=false → 시드 고정 → 다음 Interact 직전과 동일 결과 재지급.

## ⚠ 함정
- **EndPlay 타이머 정리 의무**: LootingTimerHandle/RegenTimerHandle 둘 다 Clear + subsystem Unregister. Super FIRST(BeginPlay)/Super LAST(EndPlay) 순서 강제. → [[concepts/Component-Lifecycle]]
- **Looting 몽타주 미구현 (TODO)**: REQ-28 — Looting 진입 시 UAnimMontage 재생 훅은 BP 위임(코드는 시간만 소비). 실제 몽타주 재생은 오브젝트/캐릭터 메시 의존. → [[entities/UAnimMontage]]
- **드롭 액터 v1 미생성**: 인벤토리 가득/없음 시 OnItemDropped broadcast 만 — 실제 월드 드롭 액터는 v1 미구현. (Q-5, REQ-23)
- **bStartLocked 초기 설정은 broadcast 없음**: BeginPlay 에서 `State = Locked` 직접 대입(SetState 미경유) — 구독 전 시점이라 의도적.
- **재굴림 시드 흔들기 조건**: `OnRegenComplete` 는 `bRerollOnRegen && LootSeed != 0` 일 때만 시드 변형. LootSeed==0(비결정)은 매 Roll 이 이미 새 시드라 별도 처리 불요.
- `IsConditionMet` 는 BlueprintNativeEvent — 외부(Roller)는 반드시 `Execute_IsConditionMet` 래퍼로 호출.

## 연관 entity
- [[entities/MCLootSubsystem]] — BeginPlay/EndPlay 에서 Register/Unregister, FindClosestLootable 후보.
- [[entities/MCLootTableAsset]] — `LootTable` 참조.
- [[entities/MCLootRoller]] — `Roll` 호출 + `IsConditionMet` 콜백 대상.
- [[entities/MCInventoryComponent]] — `Add` 지급 대상 (Override 또는 Instigator 탐색).
- [[entities/MCData_LootEntry]] — `FMCLootResultItem` 결과 소비, `EMCLootableState` 동거 enum.
- [[entities/MCLootToastWidget]] — `OnLootCompleted` 구독 측(ShowResult).
