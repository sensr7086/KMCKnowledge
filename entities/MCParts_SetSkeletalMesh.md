---
title: "UMCParts_SetSkeletalMesh"
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

# UMCParts_SetSkeletalMesh

## 한 줄 정의
Parts Skeletal Mesh 설정 노드 — `UMCParts_SetMesh` 자손. `USkeletalMesh*` + Dynamic Material 캐시. SetSkinnedMesh 와 사실상 동일 구조 (🟡 — 의도된 분리?).

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(BlueprintType, Blueprintable)` `class MCPLAYMODULE_API UMCParts_SetSkeletalMesh : public UMCParts_SetMesh`

## 핵심 구조
- **UPROPERTY**:
  - `EditAnywhere BlueprintReadWrite USkeletalMesh* pSkeletalMesh`.
  - `Transient TArray<UMaterialInstanceDynamic*> DynamicMaterial`.
- 생성자 + `GetDescription` + `GetOrCreateDynamicMaterial` + `#if WITH_EDITOR GetProperty/GetNodeMaterial`.

## 따르는 패턴
- [[entities/MCParts_SetStaticMesh]] 와 동등 패턴 — Mesh type 만 다름.

## ⚠ 함정
- **SetSkinnedMesh 와 멤버 시그니처 동일** — 구분 의도 불명확. Loader 가 NodeType / SetSkinned vs SetSkeletal 어떻게 분기하는지 .cpp 확인 필요 (🟡).
- 베이스 함정 적용.

## 연관 entity
- [[entities/MCParts_SetMesh]] — 직접 베이스.
- [[entities/MCParts_SetSkinnedMesh]] — 사실상 동일 구조.
- [[entities/MCSoftSkeletalMeshComponent]] — 런타임 적용 대상 가능성.
