---
title: "UMCParts_Graph"
kind: entity
category: GraphNode
base_class: UObject
module: MCPLAYMODULE
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCPlayModule/MCParts/MCPartsAsset.h
vault_refs: []
last_ingested: 2026-05-29
---

# UMCParts_Graph

## 한 줄 정의
Parts 노드 컬렉션 보관 UObject — `UMCPartsAsset::Graph` 의 type. `TArray<UMCParts_Node*> Nodes` 만 노출.

## 소속 / 상속
- 카테고리: GraphNode (그래프 관리자)
- `UCLASS()` `class MCPLAYMODULE_API UMCParts_Graph : public UObject`

## 핵심 구조
- **UPROPERTY**: `TArray<UMCParts_Node*> Nodes` — Raw 포인터.
- 메서드 0개.

## 따르는 패턴
- [[entities/MCStory_Graph]] 와 동일 패턴 — flat array + 노드별 GUID 토폴로지 (MyGuid/Parent/Child/Parameter).

## ⚠ 함정
- `Nodes` 가 raw `UMCParts_Node*` TArray — TObjectPtr 미적용.

## 연관 entity
- [[entities/MCPartsAsset]] — Outer.
- [[entities/MCParts_Node]] — 보관 대상.
