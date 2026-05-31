---
title: "UMCParts_SetOverlayMaterial"
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

# UMCParts_SetOverlayMaterial

## 한 줄 정의
Overlay Material 설정 노드 — 메시 위에 overlay 머티리얼 적용. UE 5.1+ SkeletalMesh / StaticMesh 의 Overlay Material 기능 활용.

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(BlueprintType, Blueprintable)` `class MCPLAYMODULE_API UMCParts_SetOverlayMaterial : public UMCParts_Node`

## 핵심 구조
- **UPROPERTY**:
  - `EditAnywhere BlueprintReadWrite UMaterialInterface* OverlayMaterial`.
  - `BlueprintReadOnly FGuid ParamterNode` (sic).
  - `Transient UMaterialInstanceDynamic* DynamicMaterial` — **단일** (TArray 아님).
- 생성자 + `GetDescription` + `UMaterialInstanceDynamic* GetOrCreateDynamicMaterial()` — 단일.
- `#if WITH_EDITOR`:
  - `virtual UObject* GetProperty() override`.
  - `virtual UMaterialInterface* GetNodeMaterial() override { return OverlayMaterial; }` — inline.

## 따르는 패턴
- 베이스의 `ENode_Overlay` 매핑.
- Loader 가 [[entities/MCPartsLoaderComponent]]::SetOverlayMaterial 로 처리 — 부모 mesh 의 Overlay material 슬롯에 적용.

## ⚠ 함정
- **UE 버전 의존**: Overlay Material 은 5.1+. 이전 버전 호환성 없음.
- 베이스 함정 적용.

## 연관 entity
- [[entities/MCParts_Node]] — 직접 베이스.
- [[entities/MCPartsLoaderComponent]] — SetOverlayMaterial 핸들러.
