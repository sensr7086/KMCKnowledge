---
title: "UMCStory_Condition"
kind: entity
category: GraphNode
base_class: UMCStory_Node
module: MCPLAYMODULE
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCPlayModule/MCStory/MCStoryAsset.h
  - KMCProject/MCPlayModule/MCStory/MCStoryAsset.cpp
vault_refs: []
last_ingested: 2026-05-29
---

# UMCStory_Condition

## 한 줄 정의
조건 분기 노드 — `UMCStory_Node` 의 추상 자손. `Selection` int 로 BP 로직이 어느 자식으로 이동할지 결정 → `MoveToNextNode` override.

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(Abstract, BlueprintType, Blueprintable)` `class MCPLAYMODULE_API UMCStory_Condition : public UMCStory_Node`

## 핵심 구조
- **UPROPERTY**: `EditAnywhere BlueprintReadWrite int32 Selection = 0` — 선택 인덱스 (자식 ChildNode[Selection] 으로 분기 가정, 🟡).
- 생성자: `UMCStory_Condition(const FObjectInitializer&)`.
- 오버라이드: `virtual FString GetDescription() const override`, `virtual void MoveToNextNode() override` — Selection 값으로 자식 선택.

## 따르는 패턴
- 베이스의 `EMCStoryNodeEnum::ENode_Condition` 매핑.
- BP 측이 OnMCStoryBegin/Tick 에서 `Selection` 값을 설정한 뒤 OnFinish(true) 호출 → MoveToNextNode 가 Selection 인덱스 자식으로 분기 (🟡 추론).

## ⚠ 함정
- **`Selection` 범위 검증**: `ChildNode` 배열 인덱스 초과 시 동작 미정. MoveToNextNode override 가 가드 의무 (🟡 — .cpp 확인 필요).
- 베이스 [[entities/MCStory_Node]] 의 함정 적용.

## 연관 entity
- [[entities/MCStory_Node]] — 직접 베이스.
- [[entities/MCStory_DecoratorCondition]] — 자손 (Decorator 폴링 패턴).
