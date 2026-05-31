---
title: "UMCParts_Node"
kind: entity
category: GraphNode
base_class: UObject
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/MCParts/MCPartsAsset.h
  - KMCProject/MCPlayModule/MCParts/MCPartsAsset.cpp
vault_refs: []
last_ingested: 2026-05-29
---

# UMCParts_Node

## 한 줄 정의
Parts 그래프 노드의 추상 베이스 — GUID 토폴로지 (MyGuid/Parent/Child + **ParameterGuid**) + 14종 `EMCPartsNodeEnum` 타입 식별 + WITH_EDITOR property·material 검사 hook.

## 소속 / 상속
- 카테고리: GraphNode (베이스, Abstract)
- `UCLASS(Abstract, config=Game)` `class MCPLAYMODULE_API UMCParts_Node : public UObject`

## 핵심 구조
- **UPROPERTY (BlueprintReadOnly — 토폴로지)**:
  - `FVector2D Position` — 에디터 노드 위치.
  - `FGuid MyGuid`, `FGuid ParentGuid`, `TArray<FGuid> ChildGuid` — 영구 GUID 토폴로지.
  - `TArray<FGuid> ParameterGuid` — **Parameter 노드 참조** (Story 와 다른 점: 별도 컬렉션).
  - `EMCPartsNodeEnum NodeType` — 14종 enum (SetTargetMesh/VirtualSocket/Effect/SubStaticMesh/SubSkeletalMesh/Decal/Overlay/MaterialParameter/TransformParameter/MeshDeformer/Constraint/Comment/Spline).
- **protected**:
  - `TArray<FProperty*> PropertyData` — Reflection.
  - `TArray<TWeakObjectPtr<UMCParts_Node>> ChildNode` — 런타임 reconstructed.
- **virtual API**:
  - `virtual UWorld* GetWorld() const override`.
  - `virtual FString GetDescription() const { return TEXT("None"); }`.
  - `virtual void InitNode() {};` — 자손 InitNode override 가능 (UMCParts_Constraint 만 override).
- **`#if WITH_EDITOR`**:
  - `virtual UObject* GetProperty() { return nullptr; }` — 자손이 자기 prop 반환.
  - `virtual UMaterialInterface* GetNodeMaterial() { return nullptr; }` — 자손이 자기 머티리얼 반환.
  - `bool HasMaterialNode() { return GetNodeMaterial() != nullptr; }`.
- **C++ Setter / Getter**: `SetPosition / SetMyGuid / SetParentGuid / SetNodeType / AddChildGuid / AddParameterNode / AddChildNode / ClearChildNode / ClearChild / ClearParameters` + 대응 Getter.
- **Parameter 헬퍼**: `HasParamNode()`, `IsParameter() const`, `GetParamNode()`, `GetParamNodeAt(int32)`.
- **operator==(FGuid) + IsEqual(FGuid)**.

## 따르는 패턴
- **Story 노드와 유사하지만 차이점 명확**:
  - **ParameterGuid 별도 컬렉션** — Parts 는 노드에 Parameter 노드(MaterialParameter/TransformParameter)를 부착 가능. Story 에는 없음.
  - **InitNode 가상 hook** — Story 의 라이프사이클(Start/Update/End/MoveToNextNode) 과 다름. Parts 는 Loader 가 walk 하면서 InitNode 호출.
  - **WITH_EDITOR property/material 검사 hook** — 에디터 측이 노드별 머티리얼·prop 조회 (Detail customization 페어).
- **GUID 토폴로지** — Story 와 동일 (MyGuid/Parent/Child) + ParameterGuid.
- BP 이벤트 없음 — Parts 는 BP 자손 정의가 적고 C++ 자손 다수.

## ⚠ 함정
- **ChildNode 런타임 reconstructed** — post-load 직후 invalidate. Loader 가 walk 시 명시 재구성.
- **GetWorld override** — 호출 시 nullptr 가드 의무.
- **명칭 오타**: `Paramter` (의도 = Parameter) — 자손 멤버명에서 누적. BP 노출 변경 시 redirector.
- **TArray<FProperty*> PropertyData** — UE Property 시스템 raw 포인터 보관. 모듈 재컴파일 / 핫리로드 시 dangling 가능 (🟡).
- **EMCPartsNodeEnum::ENode_None** — 디폴트 값 가능성. NodeType 미설정 시 enum 0 = None — 분기 누락 시 무동작.

## 연관 entity
- [[entities/MCParts_Graph]] — 컬렉션 호스트.
- [[entities/MCPartsAsset]] — 자산 호스트.
- [[entities/MCPartsLoaderComponent]] — 런타임 walk 실행자.
- 자손 19개 (SetMesh 3 + AddVirtualSocket / Niagara / SubStatic / SubSkeletal / Decal / Overlay / MeshDeformer / Constraint / SplineCurve / Comment / MeshParamter + Parameter 베이스 + Material/Transform Parameter 2).
