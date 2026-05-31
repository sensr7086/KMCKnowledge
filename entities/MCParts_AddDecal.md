---
title: "UMCParts_AddDecal"
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

# UMCParts_AddDecal

## 한 줄 정의
Decal 추가 노드 — `FPartsNodeDecalParam` 보관. Loader 가 [[entities/MCPartsLoaderComponent]]::AddDecal 로 처리.

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(BlueprintType, Blueprintable)` `class MCPLAYMODULE_API UMCParts_AddDecal : public UMCParts_Node`

## 핵심 구조
- **UPROPERTY**:
  - `EditAnywhere BlueprintReadWrite Category="Decal" FPartsNodeDecalParam DecalParameter`.
  - `BlueprintReadOnly FGuid ParamterNode` (sic).
- 생성자 + `GetDescription` + `UMaterialInstanceDynamic* GetOrCreateDynamicMaterial()` — 단일 (배열 아님).
- `#if WITH_EDITOR`:
  - `virtual UObject* GetProperty() override`.
  - `virtual UMaterialInterface* GetNodeMaterial() override { return DecalParameter.DecalMaterial; }` — inline.

## 따르는 패턴
- 베이스의 `ENode_Decal` 매핑.
- Mesh 노드들과 달리 DynamicMaterial 이 *단일* (TArray 아님) — Decal 은 1 머티리얼.

## ⚠ 함정
- 베이스 함정 + 명칭 오타.

## 연관 entity
- [[entities/MCParts_Node]] — 직접 베이스.
- [[entities/MCPartsLoaderComponent]] — AddDecal.
- FPartsNodeDecalParam (MCCoreStruct — 미인덱싱).
