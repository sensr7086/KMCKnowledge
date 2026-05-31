---
title: "UMCParts_SkinMeshDeformer"
kind: entity
category: GraphNode
base_class: UMCParts_Node
module: MCPLAYMODULE
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCPlayModule/MCParts/MCPartsAsset.h
vault_refs: []
last_ingested: 2026-05-29
---

# UMCParts_SkinMeshDeformer

## 한 줄 정의
Mesh Deformer 노드 — UE 의 Deformer Graph (UMeshDeformer) 적용. 부모 Skeletal Mesh 의 디포머 슬롯 설정.

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(BlueprintType, Blueprintable)` `class MCPLAYMODULE_API UMCParts_SkinMeshDeformer : public UMCParts_Node`

## 핵심 구조
- **UPROPERTY**: `EditAnywhere BlueprintReadWrite UMeshDeformer* Deformer`.
- 생성자 + `GetDescription`.

## 따르는 패턴
- 베이스의 `ENode_MeshDeformer` 매핑.
- UE 5.1+ Deformer Graph 기능.

## ⚠ 함정
- 베이스 함정 적용.

## 연관 entity
- [[entities/MCParts_Node]] — 직접 베이스.
- [[entities/MCParts_SetSkinnedMesh]] — Skinned mesh 페어 가능성.
