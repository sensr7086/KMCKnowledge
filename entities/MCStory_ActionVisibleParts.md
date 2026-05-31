---
title: "UMCStory_ActionVisibleParts"
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

# UMCStory_ActionVisibleParts

## 한 줄 정의
Parts 가시성 토글 액션 노드 — `UMCStory_Action` 자손. Owner 캐릭터의 named Parts 를 Visible 비트로 토글.

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(BlueprintType, Blueprintable)` `class MCPLAYMODULE_API UMCStory_ActionVisibleParts : public UMCStory_Action`

## 핵심 구조
- **UPROPERTY**:
  - `EditAnywhere BlueprintReadWrite FName PartsName = NAME_None`.
  - `EditAnywhere BlueprintReadWrite bool Visible = false`.
- 생성자: `UMCStory_ActionVisibleParts(const FObjectInitializer&)`.
- 오버라이드: `Start / Update / End`.

## 따르는 패턴
- Start 에서 [[entities/MCCharacter]] 의 `VisiblePartsObject(PartsName, Visible)` 호출 추론 (🟡 추론 — .cpp 미확인).
- 즉시 완료 액션 (동기) — Update 단계 없이 Start 직후 OnFinish 가정 (🟡).

## ⚠ 함정
- 베이스 함정 적용.

## 연관 entity
- [[entities/MCStory_Action]] — 직접 베이스.
- [[entities/MCCharacter]] — VisiblePartsObject API.
- [[entities/MCPartsLoaderComponent]] — VisibleParts(name, bVisible).
