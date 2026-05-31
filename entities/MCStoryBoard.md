---
title: "UMCStoryBoard"
kind: entity
category: Asset
base_class: UDataAsset
module: MCPLAYMODULE
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCPlayModule/MCStory/MCStoryBoard.h
  - KMCProject/MCPlayModule/MCStory/MCStoryBoard.cpp
vault_refs: []
last_ingested: 2026-05-29
---

# UMCStoryBoard

## 한 줄 정의
Story 의 캐릭터 바인딩 데이터 — UDataAsset 베이스, 각 Story 자산이 참조. 현재 헤더는 `ActorCharacter` 단일 멤버만 — 향후 확장 placeholder 성격.

## 소속 / 상속
- 카테고리: Asset (UDataAsset)
- `UCLASS(abstract, Blueprintable)` `class MCPLAYMODULE_API UMCStoryBoard : public UDataAsset`
- Abstract — 자손 BP 클래스 정의 후 인스턴스화 가정.

## 핵심 구조
- **UPROPERTY**:
  - `EditAnywhere BlueprintReadWrite AActor* ActorCharacter` — Story 가 동작할 대상 캐릭터.
- **현재 본문은 placeholder 수준** — 멤버 1개, 메서드 0개. 자손 BP 가 데이터 확장 가정 (🟡 추론).

## 따르는 패턴
- UDataAsset Abstract 베이스 + Blueprintable 자손 패턴 — 디자이너가 BP 자손에 데이터 추가 (🟢 — UCLASS specifier).
- `ActorCharacter` 가 raw `AActor*` (TObjectPtr 아님) — UE 5.0+ 권장 미적용 (🟡).

## ⚠ 함정
- **헤더 정보 극소** — 설계 의도·확장 정책 명시 없음. 자손 BP 의 멤버 정의가 사실상 자료 모델.
- **abstract** specifier — C++ 직접 인스턴스화 차단. BP 자손 의무.
- `AActor*` Raw 포인터 — 자산이 액터를 직접 참조 시 라이프사이클 mismatch 가능. 자산 로드 시점에 액터가 존재하지 않으면 nullptr.

## 연관 entity
- [[entities/MCStoryAsset]] — `StoryBoardClass` (Soft) + `StoryBoard` (Hard) 로 본 자산 참조.
- [[entities/MCStory_Node]] — `TWeakObjectPtr<UMCStoryBoard> StoryBoard` 멤버로 노드가 본 자산에 접근.
