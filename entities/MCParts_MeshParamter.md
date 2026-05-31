---
title: "UMCParts_MeshParamter"
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

# UMCParts_MeshParamter

## 한 줄 정의
Parts 메시 파라미터 노드 — `UMCParts_Node` 자손. `FPartsNodeMeshParam` 보관. **명칭 오타** (Paramter = Parameter).

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(BlueprintType, Blueprintable)` `class MCPLAYMODULE_API UMCParts_MeshParamter : public UMCParts_Node`

## 핵심 구조
- **UPROPERTY**: `EditAnywhere BlueprintReadOnly FPartsNodeMeshParam Paramter` (sic).
- 생성자 + `virtual FString GetDescription() const override`.

## 따르는 패턴
- 베이스의 GUID 토폴로지 그대로. Mesh 노드 (SetMesh 등) 의 부착 파라미터 후보 (🟡 추론).

## ⚠ 함정
- **명칭 오타**: `Paramter` (의도 = Parameter). BP 노출 변경 시 redirector.
- 베이스 함정 적용.

## 연관 entity
- [[entities/MCParts_Node]] — 직접 베이스.
- FPartsNodeMeshParam (MCCoreStruct — 미인덱싱).
