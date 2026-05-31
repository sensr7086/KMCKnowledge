---
title: "UMCParts_AddNiagara"
kind: entity
category: GraphNode
base_class: UMCParts_Node
module: MCPLAYMODULE
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCPlayModule/MCParts/MCPartsAsset.h
  - KMCProject/MCPlayModule/MCParts/MCPartsAsset.cpp
vault_refs: []
last_ingested: 2026-05-29
---

# UMCParts_AddNiagara

## 한 줄 정의
Niagara System 추가 노드 — VirtualSocket 위치에 부착. 파라미터 (`FPartsNodeNiaragaParam`) 보유 + Parameter 노드 참조.

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(BlueprintType, Blueprintable)` `class MCPLAYMODULE_API UMCParts_AddNiagara : public UMCParts_Node`

## 핵심 구조
- **UPROPERTY**:
  - `EditAnywhere BlueprintReadWrite TObjectPtr<UNiagaraSystem> Niagara` — TObjectPtr (5.0+ 패턴, 본 노드만 적용).
  - `EditAnywhere BlueprintReadWrite FPartsNodeNiaragaParam VisibleNiagaraParam` (sic — Niaraga = Niagara).
  - `BlueprintReadOnly FGuid ParamterNode` (sic) — Parameter 노드 참조.
- 생성자 + `void ParsingParameterValue()` — Parameter 노드로부터 값 파싱.
- 오버라이드: `GetDescription`, `#if WITH_EDITOR GetProperty`.

## 따르는 패턴
- 베이스의 `ENode_Effect` 매핑.
- Loader 가 `AddNiagara` 호출 → `FPartsInfomation::AllDynamicMaterial` 캐시.

## ⚠ 함정
- **명칭 오타**: `Niaraga` (Niagara) / `Paramter` (Parameter).
- 다른 자손은 raw `U*` 포인터인데 본 노드만 `TObjectPtr` — 일관성 불일치 (🟡).

## 연관 entity
- [[entities/MCParts_Node]] — 직접 베이스.
- [[entities/MCParts_AddVirtualSocket]] — 부착점 제공.
- FPartsNodeNiaragaParam (MCCoreStruct — 미인덱싱).
