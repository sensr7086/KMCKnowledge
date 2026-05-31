---
title: "UMCStory_Entry"
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

# UMCStory_Entry

## 한 줄 정의
Story 그래프의 진입점 노드 — `UMCStory_Node` 의 추상 자손. `UMCStoryAsset::EntryNode` 의 type, `RunNode()` 시작점.

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(Abstract, BlueprintType, Blueprintable)` `class MCPLAYMODULE_API UMCStory_Entry : public UMCStory_Node`
- Abstract — BP 자손 정의 가정.

## 핵심 구조
- 생성자: `UMCStory_Entry(const FObjectInitializer&)` — NodeType 자동 설정 가정 (🟡 — .cpp 확인 필요).
- `virtual FString GetDescription() const override`.
- 추가 멤버·UPROPERTY 없음 — 베이스 [[entities/MCStory_Node]] 의 전체 인터페이스 승계.

## 따르는 패턴
- 베이스의 `EMCStoryNodeEnum::ENode_Entry` 매핑 — 그래프 워크 시작점 식별.
- BP 자손이 OnMCStoryBegin 에서 진입 로직 정의 (🟡 추론).

## ⚠ 함정
- **헤더 정보 극소** — 베이스 함정 적용.
- 베이스 [[entities/MCStory_Node]] 의 함정 적용 (GUID 토폴로지, ChildNode 런타임 reconstructed, GetWorld override 등).

## 연관 entity
- [[entities/MCStory_Node]] — 직접 베이스.
- [[entities/MCStoryAsset]] — EntryNode 보관자.
