---
title: "UMCInventoryComponent"
kind: entity
category: Component
base_class: UActorComponent
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/MCLoot/Component/MCInventoryComponent.h
  - KMCProject/MCPlayModule/MCLoot/Component/MCInventoryComponent.cpp
vault_refs:
  - entities/UActorComponent
  - concepts/Component-Lifecycle
last_ingested: 2026-05-29
---

# UMCInventoryComponent

## 한 줄 정의
로직 전용 단순 스택형 인벤토리 컴포넌트 — `item_id(FName) → 보유 수량(int32)` 맵 모델.

## 소속 / 상속
- 대분류: Component
- `UCLASS(ClassGroup=(MCLoot), meta=(BlueprintSpawnableComponent))` `class MCPLAYMODULE_API UMCInventoryComponent : public UActorComponent`
- 충족 REQ: REQ-21, REQ-22, REQ-23(부분). 설계 출처: `_workspace/02_ue-architect_design.md §2.4`.
- v1 단순 모델: `item_id(FName) → 총 보유 수량(int32)` 맵. "슬롯 수" Capacity = 서로 다른 item_id 종류 수 상한(스택 기반 단순화), 같은 item_id 는 한 스택(최대 MaxStackSize).
- `PrimaryComponentTick.bCanEverTick = false` — 이벤트/호출 기반. (근거: `entities/UActorComponent "로직 전용 베이스"; concepts/Component-Lifecycle "기본 bCanEverTick=false"`)

## 핵심 구조
- **UPROPERTY**:
  - `int32 Capacity = 20` `EditAnywhere, BlueprintReadWrite, meta=(ClampMin="0")` — 서로 다른 item_id 종류 상한. (Q-4)
  - `int32 MaxStackSize = 99` `EditAnywhere, BlueprintReadWrite, meta=(ClampMin="1")` — item_id 당 스택 상한. (Q-4)
  - `TMap<FName, int32> Items` `BlueprintReadOnly` — item_id → 보유 수량.
- **API (UFUNCTION BlueprintCallable, Category="MC|Inventory")**:
  - `bool Add(FName ItemId, int32 Count, int32& OutAccepted, int32& OutOverflow)` — 추가. 하나라도 들어가면 true, 전량 거절(꽉 참)이면 false. (REQ-21, REQ-22)
  - `bool CanAccept(FName ItemId, int32 Count, int32& OutAcceptable) const` — 상태 변경 없는 사전 조회. 최소 1개라도 수용 가능하면 true.
  - `void Clear()` — `Items.Reset()`.
- **수용 로직 (`CanAccept`)**: 기존 item_id 면 `Room = max(0, MaxStackSize - 현재)`, `OutAcceptable = min(Count, Room)`. 신규 item_id 면 `Items.Num() >= Capacity` 일 때 0, 아니면 `min(Count, MaxStackSize)`.
- **추가 로직 (`Add`)**: `OutOverflow` 초기값 = max(0,Count). `CanAccept` 로 Acceptable 산출 → 0이면 false 반환(전량 거절). `Items.FindOrAdd(ItemId) += Acceptable`; `OutAccepted=Acceptable`, `OutOverflow=Count-Acceptable`.

## 따르는 패턴
- 로직 전용 베이스 (Tick 불요) → [[entities/UActorComponent]] / [[concepts/Component-Lifecycle]]
- 스택 병합 모델 — 같은 item_id 는 슬롯 추가 없이 한 스택 누적, 신규 종류만 슬롯(Capacity) 소비. (Q-4 기본값: Capacity=20, MaxStackSize=99)
- 부분 추가 후 잔량 거절(Overflow) — 호출측([[entities/MCLootableComponent]] GrantResults)이 Overflow 를 드롭 훅으로 처리.

## ⚠ 함정
- **전량 거절 시 false + OutOverflow=Count**: 가득 차 하나도 못 넣으면 `Add` 가 false. 호출측은 반환값/Overflow 둘 다 확인해야 잔량 누락 없음. (REQ-21 "꽉 차면 false")
- **Capacity 는 슬롯이 아닌 종류 상한**: 같은 item_id 추가는 Capacity 를 소비하지 않음(스택 병합). UI/디자인이 "슬롯 수"로 오해하지 않도록 주의 — v1 단순화.
- `Count <= 0` 입력은 `Add`/`CanAccept` 모두 false (no-op). 호출측은 0 수량 결과를 미리 거르는 게 안전(LootableComponent::GrantResults 가 `Item.Count<=0` skip).

## 연관 entity
- [[entities/MCLootableComponent]] — `GrantResults` 에서 본 컴포넌트 `Add` 호출 (TargetInventoryOverride 또는 Instigator 탐색).
- [[entities/MCData_LootEntry]] — `FMCLootResultItem.ItemId/Count` 가 지급 입력.
- [[entities/MCActorComponent]] — MC 컴포넌트 베이스 (단, 본 클래스는 `UActorComponent` 직접 상속).
