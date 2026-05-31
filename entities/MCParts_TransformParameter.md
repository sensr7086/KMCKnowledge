---
title: "UMCParts_TransformParameter"
kind: entity
category: GraphNode
base_class: UMCParts_Parameter
module: MCPLAYMODULE
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCPlayModule/MCParts/MCPartsAsset.h
vault_refs: []
last_ingested: 2026-05-29
---

# UMCParts_TransformParameter

## 한 줄 정의
Transform Parameter 노드 — `UMCParts_Parameter` 자손. `FTransform Parmater` 보관 (sic — Parameter 의 오타).

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(BlueprintType, Blueprintable)` `class MCPLAYMODULE_API UMCParts_TransformParameter : public UMCParts_Parameter`

## 핵심 구조
- **UPROPERTY**: `EditAnywhere BlueprintReadWrite FTransform Parmater` (sic).
- 생성자 + `GetDescription`.

## 따르는 패턴
- 베이스의 `ENode_TransformParameter` 매핑.
- VirtualSocket / SubMesh 등 위치 기반 노드의 ParamterNode 참조 후보.

## ⚠ 함정
- 명칭 오타 (`Parmater` = Parameter) — BP 노출 변경 시 redirector.
- 베이스 함정 적용.

## 연관 entity
- [[entities/MCParts_Parameter]] — 직접 베이스.
- [[entities/MCParts_AddVirtualSocket]] / [[entities/MCParts_AddSubStaticMesh]] / [[entities/MCParts_AddSubSkeletalMesh]] — Transform 기반 노드 후보.
