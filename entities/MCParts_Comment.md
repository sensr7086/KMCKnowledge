---
title: "UMCParts_Comment"
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

# UMCParts_Comment

## 한 줄 정의
Parts 그래프 편집기 코멘트 노드 — `UMCParts_Node` 자손. Story 의 [[entities/MCStory_Comment]] 와 동등 패턴. Title/Description/FontSize/Color/Size/Depth 시각화 메타.

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS()` `class MCPLAYMODULE_API UMCParts_Comment : public UMCParts_Node`

## 핵심 구조
- **UPROPERTY (EditAnywhere BlueprintReadWrite — 디자이너 입력)**:
  - `FString Title = TEXT("Comment...")`, `FString Description = TEXT("Description...")`.
  - `int32 FontSize = 10`, `FLinearColor CommentColor = FLinearColor::White`.
- **UPROPERTY (BlueprintReadWrite — 런타임 상태)**:
  - `int32 CommentDepth = 0`, `FVector2D CommentSize = FVector2D(400, 100)`.
- 생성자 + `virtual FString GetDescription() const override` + inline Getter (Title/FontSize/CommentColor).

## 따르는 패턴
- 베이스의 `EMCPartsNodeEnum::ENode_Comment` 매핑.
- 런타임 walk 시 본 노드 skip 가정 (🟡 추론).

## ⚠ 함정
- 베이스 [[entities/MCParts_Node]] 의 함정 적용.

## 연관 entity
- [[entities/MCParts_Node]] — 직접 베이스.
- [[entities/MCStory_Comment]] — 동등 패턴.
