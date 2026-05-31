---
title: "UMCParts_Parameter"
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

# UMCParts_Parameter

## 한 줄 정의
Parameter 노드의 추상 베이스 — `UMCParts_Node` 자손. 다른 노드들이 ParamterNode FGuid 또는 ParameterGuid[] 로 참조하는 파라미터 컬렉션 시스템의 뿌리.

## 소속 / 상속
- 카테고리: GraphNode (Abstract 베이스)
- `UCLASS(Abstract, config=Game)` `class MCPLAYMODULE_API UMCParts_Parameter : public UMCParts_Node`
- 베이스와 동일하게 `config=Game` — 인스턴스 설정 가능.

## 핵심 구조
- 멤버 0개 — 단순 베이스. 자손이 실제 파라미터 값 (Material / Transform) 보관.

## 따르는 패턴
- 베이스의 `EMCPartsNodeEnum::ENode_MaterialParameter` / `ENode_TransformParameter` 가 자손 식별.
- 다른 노드가 ParamterNode (FGuid) 로 본 자손 인스턴스 참조 → ParsingParameterValue 같은 메서드가 값 추출.

## ⚠ 함정
- 베이스 함정 적용.

## 연관 entity
- [[entities/MCParts_Node]] — 직접 베이스.
- 자손: [[entities/MCParts_MaterialParameter]] / [[entities/MCParts_TransformParameter]].
- [[entities/MCParts_AddNiagara]] — ParamterNode 참조 + ParsingParameterValue() 의 페어.
