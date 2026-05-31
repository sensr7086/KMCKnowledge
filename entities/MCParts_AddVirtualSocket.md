---
title: "UMCParts_AddVirtualSocket"
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

# UMCParts_AddVirtualSocket

## 한 줄 정의
가상 소켓 추가 노드 — 부모 Socket 위치에 Transform 오프셋으로 새 소켓 정의. 후속 노드 (AddSubMesh / AddNiagara / AddDecal) 의 부착점.

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(BlueprintType, Blueprintable)` `class MCPLAYMODULE_API UMCParts_AddVirtualSocket : public UMCParts_Node`

## 핵심 구조
- **UPROPERTY (EditAnywhere BlueprintReadWrite)**:
  - `FTransform VirtualSocketTransform` — 부모 socket 기준 오프셋.
  - `FName VirtualSocketName` — 새 socket 이름.
  - `FName ParentSocketName` — 부모 socket 이름.

## 따르는 패턴
- 베이스의 `EMCPartsNodeEnum::ENode_VirtualSocket` 매핑.
- Loader 가 본 노드를 만나면 `FMCRuntimeVirtualSocket` 인스턴스 생성 + `FPartsInfomation::ArrVirualSocket` 맵에 등록. 후속 자손 노드가 이 socket 을 부착점으로 사용.

## ⚠ 함정
- 베이스 함정 적용.
- Socket 이름 중복 시 동작 미정 — 컨플릭트 정책 명시 없음 (🟡).

## 연관 entity
- [[entities/MCParts_Node]] — 직접 베이스.
- [[entities/MCPartsLoaderComponent]] — AddVirtualSocket 처리 (FPartsInfomation::AddVirtualSocket).
- FMCRuntimeVirtualSocket (MCCoreStruct — 미인덱싱).
