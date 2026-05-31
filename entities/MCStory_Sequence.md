---
title: "UMCStory_Sequence"
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

# UMCStory_Sequence

## 한 줄 정의
Composite 노드 — `UMCStory_Node` 의 추상 자손. 베이스의 `IsCompositeNode()` 가 본 type 식별. 자손이 자식들의 CompositeNode 로 자기 등록.

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(Abstract, BlueprintType, Blueprintable)` `class MCPLAYMODULE_API UMCStory_Sequence : public UMCStory_Node`

## 핵심 구조
- **UPROPERTY**: `BlueprintReadOnly int32 Selection = 0` — 현재 자식 인덱스 (🟡 추론).
- 생성자: `UMCStory_Sequence(const FObjectInitializer&)`.
- 오버라이드: `virtual FString GetDescription() const override`, `virtual void MoveToNextNode() override`.

## 따르는 패턴
- 베이스의 `IsCompositeNode()` — `NodeType == EMCStoryNodeEnum::ENode_Sequence` 면 true. 본 type 식별의 진실원.
- **CompositeNode 등록**: Sequence 가 자식을 walk 하면서 `SetCompositeNode(this)` 호출 의무 (🟡 추론 — .cpp 미확인).
- Selection 으로 자식 순환 → 자식 종료 시 다음 자식으로 전이.

## ⚠ 함정
- **헤더 정보 극소**: Selection 의 갱신 시점·MoveToNextNode 의 분기 알고리즘 명시 없음 — .cpp 측 검증 필요.
- 베이스 함정 적용 ([[entities/MCStory_Node]]).

## 연관 entity
- [[entities/MCStory_Node]] — 직접 베이스.
- [[entities/MCStory_Action]] / [[entities/MCStory_Condition]] — 자식 후보.
