---
title: "UMCParts_AddSubSkeletalMesh"
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

# UMCParts_AddSubSkeletalMesh

## 한 줄 정의
서브 Skeletal Mesh 추가 노드 — VirtualSocket 위치에 부착. [[entities/MCParts_AddSubStaticMesh]] 와 동등 패턴 (Static vs Skeletal 차이).

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(BlueprintType, Blueprintable)` `class MCPLAYMODULE_API UMCParts_AddSubSkeletalMesh : public UMCParts_Node`

## 핵심 구조
- **UPROPERTY**:
  - `EditAnywhere BlueprintReadWrite USkeletalMesh* SkeletalMesh`.
  - `BlueprintReadOnly FGuid ParamterNode` (sic).
  - `Transient TArray<UMaterialInstanceDynamic*> DynamicMaterial`.
- 생성자 + `GetDescription` + `GetOrCreateDynamicMaterial` + `#if WITH_EDITOR GetProperty/GetNodeMaterial`.

## ⚠ 함정
- 베이스 함정 + 명칭 오타.

## 연관 entity
- [[entities/MCParts_Node]] — 직접 베이스.
- [[entities/MCParts_AddVirtualSocket]] — 부착점.
- [[entities/MCParts_AddSubStaticMesh]] — Static 변형.
- [[entities/MCPartsLoaderComponent]] — AddSubSkeletalMesh 핸들러.
