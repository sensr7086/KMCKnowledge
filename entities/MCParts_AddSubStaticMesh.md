---
title: "UMCParts_AddSubStaticMesh"
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

# UMCParts_AddSubStaticMesh

## 한 줄 정의
서브 Static Mesh 추가 노드 — VirtualSocket 위치에 부착. Loader 가 [[entities/MCPartsLoaderComponent]]::AddSubStaticMesh 로 처리.

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(BlueprintType, Blueprintable)` `class MCPLAYMODULE_API UMCParts_AddSubStaticMesh : public UMCParts_Node`

## 핵심 구조
- **UPROPERTY**:
  - `EditAnywhere BlueprintReadWrite UStaticMesh* StaticMesh` — raw 포인터.
  - `BlueprintReadOnly FGuid ParamterNode` (sic).
  - `Transient TArray<UMaterialInstanceDynamic*> DynamicMaterial`.
- 생성자 + `GetDescription` + `GetOrCreateDynamicMaterial` + `#if WITH_EDITOR GetProperty/GetNodeMaterial`.

## ⚠ 함정
- 베이스 함정 + raw 포인터 + 명칭 오타 (`Paramter`).

## 연관 entity
- [[entities/MCParts_Node]] — 직접 베이스.
- [[entities/MCParts_AddVirtualSocket]] — 부착점.
- [[entities/MCPartsLoaderComponent]] — AddSubStaticMesh 핸들러.
