---
title: "UMCParts_Constraint"
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

# UMCParts_Constraint

## 한 줄 정의
Physics Constraint 추가 노드 — `UPhysicsConstraintTemplate` 인스턴스 보관 + 소켓·Transform. Loader 가 [[entities/MCPartsLoaderComponent]]::AddPhysicsConstraint 로 처리.

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(BlueprintType, Blueprintable)` `class MCPLAYMODULE_API UMCParts_Constraint : public UMCParts_Node`

## 핵심 구조
- **UPROPERTY**:
  - `EditAnywhere Category=ConstraintComponent meta=(ShowOnlyInnerProperties) UPhysicsConstraintTemplate* ConstraintInstance` — Constraint template.
  - `EditAnywhere BlueprintReadWrite FTransform ConstraintTransform`.
  - `EditAnywhere BlueprintReadWrite FName Socket = NAME_None`.
- 생성자 + `virtual void InitNode() override` (베이스에서 noop, 본 노드가 유일하게 override).

## 따르는 패턴
- 베이스의 `ENode_Constraint` 매핑.
- **InitNode override** — Constraint instance 초기화. 다른 노드는 모두 InitNode default no-op.
- Loader 가 `FMCRuntimeReservedConstranit` 컬렉션에 등록 → MakeConstranit (sic — Constraint 의 오타) 단계에서 실제 생성.

## ⚠ 함정
- **명칭 오타**: `Constranit` (Constraint) — Loader 의 `MakeConstranit` / `RuntimeConstranit` / `FMCRuntimeReservedConstranit` 에서 누적.
- **meta=(ShowOnlyInnerProperties)**: ConstraintInstance 가 inner property 만 디테일 패널에 노출 — 표준 UE Constraint 디테일 패턴.
- 베이스 함정 적용.

## 연관 entity
- [[entities/MCParts_Node]] — 직접 베이스.
- [[entities/MCPartsLoaderComponent]] — AddPhysicsConstraint / MakeConstranit.
- FMCRuntimeReservedConstranit (MCCoreStruct — 미인덱싱).
