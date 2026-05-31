---
title: "UMCStory_ActionRunStroy"
kind: entity
category: GraphNode
base_class: UMCStory_Action
module: MCPLAYMODULE
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCPlayModule/MCStory/Action/MCStoryAction.h
  - KMCProject/MCPlayModule/MCStory/Action/MCStoryAction.cpp
vault_refs: []
last_ingested: 2026-05-29
---

# UMCStory_ActionRunStroy

## 한 줄 정의
다른 Story 자산을 재귀 실행하는 액션 노드 — `UMCStory_Action` 자손. Hard 참조 `UMCStoryAsset` 를 보유하고 본 노드 실행 시점에 그 자산 RunNode 트리거 (🟡 추론). **명칭 오타**: ActionRunStroy (의도=ActionRunStory).

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(BlueprintType, Blueprintable)` `class MCPLAYMODULE_API UMCStory_ActionRunStroy : public UMCStory_Action`

## 핵심 구조
- **UPROPERTY**:
  - `EditAnywhere BlueprintReadWrite TObjectPtr<UMCStoryAsset> StoryAsset` — **Hard** 참조 (TObjectPtr).
- 생성자: `UMCStory_ActionRunStroy(const FObjectInitializer&)`.
- 오버라이드: `Start / Update / End`.

## 따르는 패턴
- Hard 자산 참조 — 본 자산 로드 시 함께 따라옴. Cooked 첫 SpawnActor 히칭 4단 [1] 트리거 가능.
- Start 에서 StoryAsset->SetNodeOnwer(Owner) + StoryAsset->RunNode() 호출 추론 (🟡 — .cpp 미확인).

## ⚠ 함정
- **Hard vs Soft 참조 선택**: Story 자산이 다른 Story 를 Hard 로 참조 → 그래프 사이클 가능성 (A→B→A). 무한 재귀 방어 없음 (🟡 추론). Soft 권장 → [[concepts/Asset-Loading-Policy]].
- **명칭 오타**: `ActionRunStroy` (의도 = ActionRunStory). BP 노출 변경 시 redirector.
- 베이스 함정 적용.

## 연관 entity
- [[entities/MCStory_Action]] — 직접 베이스.
- [[entities/MCStoryAsset]] — 재귀 실행 대상.
