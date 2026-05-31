---
title: "UMCParts_MaterialParameter"
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

# UMCParts_MaterialParameter

## 한 줄 정의
Material Parameter 노드 — `UMCParts_Parameter` 자손. `FPartsNodeMeshParam Parmater` 보관 (sic — Parameter 의 오타).

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(BlueprintType, Blueprintable)` `class MCPLAYMODULE_API UMCParts_MaterialParameter : public UMCParts_Parameter`

## 핵심 구조
- **UPROPERTY**: `EditAnywhere BlueprintReadWrite FPartsNodeMeshParam Parmater` (sic).
- 생성자 + `GetDescription`.

## 따르는 패턴
- 베이스의 `ENode_MaterialParameter` 매핑.
- 다른 노드 (AddNiagara, SetStatic 등) 의 ParamterNode 가 본 인스턴스 참조 → ParsingParameterValue 가 값 적용 (🟡 추론).

## ⚠ 함정
- **명칭 오타 누적**: 클래스명·멤버 `Paramter` (Parameter). MCEditorModule 의 DetailView 측 (MCMaterialNodeDetails) 와 BP 노출 동시 수정 시 redirector 책임.
- 베이스 함정 적용.

## 연관 entity
- [[entities/MCParts_Parameter]] — 직접 베이스.
- FPartsNodeMeshParam (MCCoreStruct — 미인덱싱).
- [[entities/MCParts_MeshParamter]] — 유사 명칭 다른 노드 (구분 주의).
