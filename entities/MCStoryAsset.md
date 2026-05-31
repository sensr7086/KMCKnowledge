---
title: "UMCStoryAsset"
kind: entity
category: Asset
base_class: UObject
module: MCPLAYMODULE
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCPlayModule/MCStory/MCStoryAsset.h
  - KMCProject/MCPlayModule/MCStory/MCStoryAsset.cpp
vault_refs: []
last_ingested: 2026-05-29
---

# UMCStoryAsset

## 한 줄 정의
Story 그래프 자산 호스트 UObject — `UMCStory_Graph` (노드 컬렉션) + `UMCStoryBoard` (캐릭터 바인딩) 를 보관, runtime 에서 EntryNode → CurrentNode 워크.

## 소속 / 상속
- 카테고리: Asset
- `UCLASS(BlueprintType)` `class MCPLAYMODULE_API UMCStoryAsset : public UObject`
- 라이프사이클: 자산 로드 시 인스턴스. UMCStorySubSystem 이 `RegisterStory(this)` 로 활성 자산 등록 → Tick 위임.

## 핵심 구조
- **UPROPERTY**:
  - `UMCStory_Graph* Graph = nullptr` — 노드 컬렉션 보관 (UPROPERTY).
  - `TMap<FGuid, UMCStory_Node*> NodeMap` — **런타임에만 사용** (post-load 시점에 RecursiveChildNode 가 채움).
  - `EditAnywhere TSoftClassPtr<UMCStoryBoard> StoryBoardClass` — Soft 클래스 참조.
  - `EditAnywhere TObjectPtr<UMCStoryBoard> StoryBoard` — Hard 자산 참조.
- **protected 런타임 상태**:
  - `TWeakObjectPtr<AActor> _owner`
  - `TWeakObjectPtr<UMCStory_Node> EntryNode`
  - `TWeakObjectPtr<UMCStory_Node> CurrentNode`
- **BP API**:
  - `UFUNCTION(BlueprintCallable) void SetNodeOnwer(AActor*)` (sic — Owner 의 오타).
  - `UFUNCTION(BlueprintCallable) void RunNode()` — EntryNode 부터 walk 시작.
  - `UFUNCTION() void NextNode(UMCStory_Node* _next)` — 노드 전이 콜백.
- **C++ API**:
  - `void ClearStory()`
  - `void Tick(float _deltatime)` — UMCStorySubSystem 의 Tick 이 호출.
  - `FGuid GetCurrentActiveNode() const` — Subsystem 의 `GetRuntimeActiveNode` 가 위임.
- **private**: `void RecursiveChildNode(UMCStory_Node*)` — NodeMap 및 ChildNode weak ptr 채움.

## 따르는 패턴
- 자산 호스트 패턴 — Graph 보관 + 런타임 walk. UMCStorySubSystem 이 Tick 위임 (자산 자체는 tickable 아님).
- **GUID 토폴로지** — 노드의 영구 토폴로지는 `MyGuid` / `ParentGuid` / `ChildGuid[]`. 런타임 `NodeMap` + `ChildNode` weak ptr 은 RecursiveChildNode 가 채움. **Post-load 시점에 즉시 유효하지 않음** — Subsystem 의 RegisterStory + RunNode 이전에 reconstructed 가정 (🟡 추론).
- StoryBoard 의 SoftClass + Hard pointer 동시 보유 — Hard 가 instance, Soft 가 클래스 reference 가능성.

## ⚠ 함정
- **NodeMap / ChildNode 가 런타임 reconstructed**: 자산 로드 직후 invalidate — RecursiveChildNode 호출 전 사용 시 nullptr.
- **명칭 오타**: `SetNodeOnwer` (Owner의 오타). BP 노출 변경 시 redirector.
- **헤더 주석 mojibake (CP949)**: 일부 한국어 주석 UTF-8 으로 깨짐 — 임의 재인코딩 금지.
- **TMap<FGuid, UMCStory_Node*> NodeMap UPROPERTY**: Value 가 raw pointer (TObjectPtr 아님). 5.0+ 권장 패턴 미적용 — Garbage Collection 안전성 검토 필요 (🟡).

## 연관 entity
- [[entities/MCStorySubSystem]] — 본 자산 호스트, RegisterStory/Tick 발화.
- [[entities/MCStoryBoard]] — 캐릭터 바인딩.
- [[entities/MCStory_Graph]] — 노드 컬렉션 보관자.
- [[entities/MCStory_Node]] — 노드 베이스.
- [[entities/MCStory_Entry]] — EntryNode 의 type.
- [[entities/MCStory_ActionRunStroy]] — 다른 자산을 재귀 실행 (StoryAsset → StoryAsset 체인).
