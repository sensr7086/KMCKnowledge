---
title: "UMCStory_DecoratorCondition"
kind: entity
category: GraphNode
base_class: UMCStory_Condition
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/MCStory/MCStoryAsset.h
  - KMCProject/MCPlayModule/MCStory/MCStoryAsset.cpp
vault_refs: []
last_ingested: 2026-05-29
---

# UMCStory_DecoratorCondition

## 한 줄 정의
Decorator 체인 폴링 조건 노드 — `UMCStory_Condition` 자손. `DecoratorClass[]` 의 각 Decorator 를 순차로 시작 + 결과 폴링. 타임아웃 (`CheckDecoratorWaitTime`, -1=무한대) 으로 안전장치.

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(BlueprintType, Blueprintable)` `class MCPLAYMODULE_API UMCStory_DecoratorCondition : public UMCStory_Condition`
- **Abstract 아님** — 직접 인스턴스화 가능.

## 핵심 구조
- **UPROPERTY**:
  - `BlueprintReadOnly TArray<UMCStory_DecortorBase*> DecoratorClass` — 폴링 대상 (Raw 포인터, TObjectPtr 미적용).
  - `EditAnywhere BlueprintReadWrite float CheckDecoratorWaitTime = -1.0f` — 타임아웃 (-1=무한대).
- **런타임 상태** (UPROPERTY 아님):
  - `float SaveDecorateWaitTime = 0.0f` — 잔여 시간.
  - `int32 CheckDecoratorIterator = 0` — 현재 Decorator 인덱스.
  - `bool InfiniteWait = false` — WaitTime==-1 시 true.
- **생성자**: `UMCStory_DecoratorCondition(const FObjectInitializer&)`.
- **오버라이드 (inline 정의)**:
  - `virtual void Start() override` — 베이스 Start + `DecoratorClass[CheckDecoratorIterator]->StartDecorator()` + `SaveDecorateWaitTime=CheckDecoratorWaitTime` + InfiniteWait 판정 (FMath::IsNearlyEqual(-1.0f)).
  - `virtual void Update(float _tick) override` — Tick 마다 ComplateDecorator() 폴링:
    - 완료: ResultDecorator==true 면 다음 Decorator (CheckDecoratorIterator++) 시작. 마지막까지 통과 → `Selection=1` + MoveToNextNode. ResultDecorator==false 면 `Selection=0` + MoveToNextNode.
    - 진행 중: WaitTime 감소 (`SaveDecorateWaitTime -= _tick`). InfiniteWait 면 감소 무시.
    - 타임아웃 (SaveDecorateWaitTime<=0 + !InfiniteWait): `Selection=0` + MoveToNextNode.
  - `virtual void MoveToNextNode() override` — Selection 값으로 자식 선택 (베이스 Condition 패턴).

## 따르는 패턴
- **순차 Decorator 체인 폴링** — 모든 Decorator 가 통과(true)해야 Selection=1, 하나라도 실패 또는 타임아웃이면 Selection=0.
- **타임아웃 정책 — -1 = 무한대**: 디자이너 의도. 명시 시간 (0~N 초) 또는 무한대 폴링.
- BP Decorator 가 BeginDecorator 에서 비동기 조건 검사 후 OnFinish(true/false) 콜백 호출 → 폴링이 ComplateDecorator() 로 결과 수신.

## ⚠ 함정
- **`CheckDecoratorIterator` 범위 가드 누락 가능성**: Start 에서 첫 인덱스 `IsValidIndex(0)` 검사 후 시작하지만 Update 시점에는 직접 가드 — 빈 DecoratorClass 시 Start 가 `IsValidIndex(0)==false` 분기 누락 → 영구 Wait 가능 (🟡).
- **타임아웃 Selection=0 처리 시 의미**: "Decorator 가 시간 내 완료 안 됨 = 실패" → 의도 명확. 디자이너가 -1 (무한대) 의도와 다른 경우 주의.
- 베이스 함정 적용 ([[entities/MCStory_Condition]] / [[entities/MCStory_Node]]).
- **명칭 오타**: DecortorBase 의 자손이라 동일 오타 누적.

## 연관 entity
- [[entities/MCStory_Condition]] — 직접 베이스.
- [[entities/MCStory_DecortorBase]] — DecoratorClass[] 의 베이스 타입.
