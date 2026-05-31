---
title: "UMCStory_Action"
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

# UMCStory_Action

## 한 줄 정의
실행 액션 노드 베이스 — `UMCStory_Node` 의 추상 자손. BP 자손 또는 C++ 자손 (AddParts/VisibleParts/RunStroy) 이 실제 게임 동작 정의.

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(Abstract, BlueprintType, Blueprintable)` `class MCPLAYMODULE_API UMCStory_Action : public UMCStory_Node`

## 핵심 구조
- 생성자: `UMCStory_Action(const FObjectInitializer&)`.
- 오버라이드: `virtual FString GetDescription() const override`.
- 추가 멤버 0개 — 베이스 인터페이스 그대로 + C++ 자손 (Action* 3개) 또는 BP 자손이 동작 추가.

## 따르는 패턴
- 베이스의 `EMCStoryNodeEnum::ENode_Action` 매핑.
- BP 자손이 OnMCStoryBegin 에서 동작 시작, OnFinish(true) 로 완료 신호 → MoveToNextNode 발화.
- C++ 자손 3종 (제공된 패턴 — Parts 시스템 연동):
  - [[entities/MCStory_ActionAddParts]] — Soft Parts 자산 적용.
  - [[entities/MCStory_ActionVisibleParts]] — Parts 가시성 토글.
  - [[entities/MCStory_ActionRunStroy]] — 다른 Story 자산 재귀 실행.

## ⚠ 함정
- **헤더 정보 극소** — 베이스 함정만 적용.
- 베이스 [[entities/MCStory_Node]] 의 함정 적용.

## 연관 entity
- [[entities/MCStory_Node]] — 직접 베이스.
- [[entities/MCStory_ActionAddParts]] / [[entities/MCStory_ActionVisibleParts]] / [[entities/MCStory_ActionRunStroy]] — C++ 자손.
