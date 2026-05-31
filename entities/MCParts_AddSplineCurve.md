---
title: "UMCParts_AddSplineCurve"
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

# UMCParts_AddSplineCurve

## 한 줄 정의
Spline Curve 추가 노드 — `FMCSplineCurves` 보관 + 프리뷰 토글 + 이동 속도. 에디터에서 사용자가 spline 편집 가정.

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(BlueprintType, Blueprintable)` `class MCPLAYMODULE_API UMCParts_AddSplineCurve : public UMCParts_Node`

## 핵심 구조
- **UPROPERTY (EditAnywhere BlueprintReadWrite)**:
  - `FMCSplineCurves SplineCurve`.
  - `bool ShowPreview = false`.
  - `float MoveSpeed = 200.0f`.
- **UPROPERTY (Editor only meta 없음)**: `int32 Selected = 0` — UPROPERTY 마커만, BlueprintReadWrite 아님.
- 생성자 + `GetDescription`.

## 따르는 패턴
- 베이스의 `ENode_Spline` 매핑.

## ⚠ 함정
- 베이스 함정 적용.
- `Selected` UPROPERTY 가 EditAnywhere / BlueprintReadWrite 없이 직렬화만 — 의도 불명확 (🟡).

## 연관 entity
- [[entities/MCParts_Node]] — 직접 베이스.
- FMCSplineCurves (MCCoreStruct — 미인덱싱).
