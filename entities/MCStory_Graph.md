---
title: "UMCStory_Graph"
kind: entity
category: GraphNode
base_class: UObject
module: MCPLAYMODULE
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCPlayModule/MCStory/MCStoryAsset.h
vault_refs: []
last_ingested: 2026-05-29
---

# UMCStory_Graph

## 한 줄 정의
Story 노드 컬렉션 보관 UObject — `UMCStoryAsset` 이 직접 소유 (`UMCStoryAsset::Graph`). `TArray<UMCStory_Node*> Nodes` 만 노출.

## 소속 / 상속
- 카테고리: GraphNode (그래프 관리자 — 노드 컬렉션 호스트)
- `UCLASS()` `class MCPLAYMODULE_API UMCStory_Graph : public UObject`

## 핵심 구조
- **UPROPERTY**: `TArray<UMCStory_Node*> Nodes` — Raw 포인터 (TObjectPtr 아님).
- 메서드 0개 — 컬렉션 보관 전용.

## 따르는 패턴
- 그래프 = `TArray<UMCStory_Node*>` flat 리스트 + 노드별 GUID 토폴로지 (MyGuid / ParentGuid / ChildGuid).
- UMCStoryAsset 의 `UPROPERTY() UMCStory_Graph* Graph` 가 strong reference, 본 자산이 Outer.

## ⚠ 함정
- **Nodes 가 raw `UMCStory_Node*` TArray** — TObjectPtr 미적용. UE 5.0+ 권장 패턴 미적용.
- 메서드 0개 — 노드 추가/제거/조회는 외부에서 직접 Nodes 배열 조작 가정 (🟡 추론). 외부 mutate 시 GUID 토폴로지 sync 책임이 호출자 측.

## 연관 entity
- [[entities/MCStoryAsset]] — Graph 보유자 (직접 Outer).
- [[entities/MCStory_Node]] — 보관 대상 베이스.
