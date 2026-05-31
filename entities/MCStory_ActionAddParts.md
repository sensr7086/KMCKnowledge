---
title: "UMCStory_ActionAddParts"
kind: entity
category: GraphNode
base_class: UMCStory_Action
module: MCPLAYMODULE
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCPlayModule/MCStory/Action/MCStoryAction.h
  - KMCProject/MCPlayModule/MCStory/Action/MCStoryAction.cpp
vault_refs: []
last_ingested: 2026-05-29
---

# UMCStory_ActionAddParts

## 한 줄 정의
Parts 자산 적용 액션 노드 — `UMCStory_Action` 자손. Soft 참조 `UMCPartsAsset` 을 Owner 캐릭터의 PartsLoader 에 적용.

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(BlueprintType, Blueprintable)` `class MCPLAYMODULE_API UMCStory_ActionAddParts : public UMCStory_Action`

## 핵심 구조
- **UPROPERTY**:
  - `EditAnywhere BlueprintReadWrite TSoftObjectPtr<UMCPartsAsset> PartsAsset` — Soft 참조.
  - `EditAnywhere BlueprintReadWrite FName PartsName = NAME_None` — Parts 식별자.
- 생성자: `UMCStory_ActionAddParts(const FObjectInitializer&)`.
- 오버라이드: `virtual void Start() override`, `virtual void Update(float _tick) override`, `virtual void End() override`.

## 따르는 패턴
- Soft 참조 — 자산 로드 시점 분산 (🟢 — Soft 명시).
- Start 에서 [[entities/MCCharacter]] 의 `AddPartsObject(PartsName, PartsAsset)` 호출 추론 (🟡 — .cpp 미확인).
- Update 에서 로드 완료 폴링 → OnFinish 호출 가정 (🟡 추론).

## ⚠ 함정
- Soft 자산 로드는 *비동기*. Start 직후 동기 적용하면 첫 호출 시 hitching 가능 — 비동기 로드 패턴 [[concepts/Asset-Loading-Policy]] 권장 (🟡 — 실제 구현 확인 필요).
- 베이스 함정 적용 ([[entities/MCStory_Action]] / [[entities/MCStory_Node]]).

## 연관 entity
- [[entities/MCStory_Action]] — 직접 베이스.
- [[entities/MCCharacter]] — Owner 캐릭터, AddPartsObject API.
- [[entities/MCPartsLoaderComponent]] — Parts 적용 컴포넌트.
- [[entities/MCPartsAsset]] — Soft 참조 대상.
