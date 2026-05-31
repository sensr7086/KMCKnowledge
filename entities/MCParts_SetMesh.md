---
title: "UMCParts_SetMesh"
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

# UMCParts_SetMesh

## 한 줄 정의
Parts 메시 설정 노드의 추상 베이스 — `UMCParts_Node` 자손. Socket 이름 + 별도 ParameterNode 참조. 자손 3종 (SetStaticMesh / SetSkeletalMesh / SetSkinnedMesh).

## 소속 / 상속
- 카테고리: GraphNode (Abstract 베이스)
- `UCLASS(Abstract, BlueprintType, Blueprintable)` `class MCPLAYMODULE_API UMCParts_SetMesh : public UMCParts_Node`

## 핵심 구조
- **UPROPERTY**:
  - `BlueprintReadOnly FGuid ParamterNode` (sic) — 별도 Parameter 노드 참조.
  - `EditAnywhere BlueprintReadOnly FName SocketName` — 부착 소켓 이름.
- 생성자 + `virtual FString GetDescription() const override`.

## 따르는 패턴
- 베이스의 GUID 토폴로지 + 메시 노드 공통 인터페이스. 자손은 실제 메시 자원 (UStaticMesh / USkeletalMesh) 추가.
- ParameterGuid (베이스의 array) 와 별도로 단일 ParamterNode FGuid 보유 — 의도 불명확 (🟡 — 동일 정보 중복?).

## ⚠ 함정
- **`ParamterNode` 와 베이스 `ParameterGuid` 중복 가능성**: 단일 vs 배열 양쪽 모두 보유 — 동기화 책임 불명확 (🟡).
- **명칭 오타**: `Paramter` (Parameter).
- 베이스 함정 적용.

## 연관 entity
- [[entities/MCParts_Node]] — 직접 베이스.
- 자손: [[entities/MCParts_SetStaticMesh]] / [[entities/MCParts_SetSkeletalMesh]] / [[entities/MCParts_SetSkinnedMesh]].
