---
title: "UMCParts_SetSkinnedMesh"
kind: entity
category: GraphNode
base_class: UMCParts_SetMesh
module: MCPLAYMODULE
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCPlayModule/MCParts/MCPartsAsset.h
  - KMCProject/MCPlayModule/MCParts/MCPartsAsset.cpp
vault_refs: []
last_ingested: 2026-05-29
---

# UMCParts_SetSkinnedMesh

## 한 줄 정의
Parts Skinned Mesh 설정 노드 — `UMCParts_SetMesh` 자손. **헤더 시그니처가 [[entities/MCParts_SetSkeletalMesh]] 와 동일** (`USkeletalMesh* pSkeletalMesh` 보유). 의도된 분리는 .cpp 또는 Loader 측 분기에서 확인 가능.

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(BlueprintType, Blueprintable)` `class MCPLAYMODULE_API UMCParts_SetSkinnedMesh : public UMCParts_SetMesh`

## 핵심 구조
- **UPROPERTY**:
  - `EditAnywhere BlueprintReadWrite USkeletalMesh* pSkeletalMesh` (sic — Skinned 라고 명명되어 있지만 멤버는 SkeletalMesh).
  - `Transient TArray<UMaterialInstanceDynamic*> DynamicMaterial`.
- 생성자 + `GetDescription` + `GetOrCreateDynamicMaterial` + `#if WITH_EDITOR GetProperty/GetNodeMaterial`.

## 따르는 패턴
- 베이스 [[entities/MCParts_SetMesh]] 패턴 승계. NodeType 만 다름 (`ENode_SubSkeletalMesh` vs `ENode_SetTargetMesh` 가정, 🟡 — .cpp 미확인).

## ⚠ 함정
- **클래스 이름과 멤버 이름 불일치**: `SetSkinnedMesh` 인데 멤버는 `pSkeletalMesh` — Skinned mesh 와 Skeletal mesh 의 의미 차이가 코드상 반영 안 됨. 의도된 구분이 어디서 결정되는지 확인 필요 (🟡).
- Skinned mesh 는 보통 [Soft skin deformation](https://docs.unrealengine.com/) 컨셉 — SkeletalMesh 의 한 type. UE 표준상 별도 클래스 아님 (🟡).
- 베이스 함정 적용.

## 연관 entity
- [[entities/MCParts_SetMesh]] — 직접 베이스.
- [[entities/MCParts_SetSkeletalMesh]] — 사실상 동일 구조.
- [[entities/MCParts_SkinMeshDeformer]] — Skinned 변형 페어 가능성.
