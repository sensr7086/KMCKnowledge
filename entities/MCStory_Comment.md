---
title: "UMCStory_Comment"
kind: entity
category: GraphNode
base_class: UMCStory_Node
module: MCPLAYMODULE
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCPlayModule/MCStory/MCStoryAsset.h
  - KMCProject/MCPlayModule/MCStory/MCStoryAsset.cpp
vault_refs: []
last_ingested: 2026-05-29
---

# UMCStory_Comment

## 한 줄 정의
편집기용 코멘트 노드 — `UMCStory_Node` 자손. Title / Description / FontSize / Color / Size / Depth 의 시각화 메타. 런타임 동작 없음 (🟡 추론).

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS()` `class MCPLAYMODULE_API UMCStory_Comment : public UMCStory_Node`
- **Abstract 아님** — 직접 인스턴스화 (BP / 에디터 측).

## 핵심 구조
- **UPROPERTY (EditAnywhere BlueprintReadWrite — 디자이너 입력)**:
  - `FString Title = TEXT("Comment...")`.
  - `FString Description = TEXT("Description...")`.
  - `int32 FontSize = 10`.
  - `FLinearColor CommentColor = FLinearColor::White`.
- **UPROPERTY (BlueprintReadWrite — 런타임 상태)**:
  - `int32 CommentDepth = 0` — Z-order 또는 nesting depth.
  - `FVector2D CommentSize = FVector2D(400, 100)` — 박스 크기.
- 생성자: `UMCStory_Comment(const FObjectInitializer&)`.
- 오버라이드: `virtual FString GetDescription() const override`.
- 인라인 Getter: `GetTitle / GetFontSize / GetCommentColor`.

## 따르는 패턴
- 베이스의 `EMCStoryNodeEnum::ENode_Comment` 매핑.
- 런타임 그래프 walk 시 본 노드는 skip 가정 (🟡 추론 — Comment 노드는 보통 런타임 영향 없음).

## ⚠ 함정
- **런타임 walk skip 정책**: 명시 코드 없음. UMCStoryAsset::RecursiveChildNode 가 Comment 노드를 어떻게 처리하는지 .cpp 확인 필요 — 만약 walk 에 포함되면 OnMCStoryBegin/Tick/End 가 호출됨 (🟡).
- 베이스 [[entities/MCStory_Node]] 의 함정 적용.

## 연관 entity
- [[entities/MCStory_Node]] — 직접 베이스.
