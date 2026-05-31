---
title: "UMCStory_Node"
kind: entity
category: GraphNode
base_class: UObject
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/MCStory/MCStoryAsset.h
  - KMCProject/MCPlayModule/MCStory/MCStoryAsset.cpp
vault_refs: []
last_ingested: 2026-05-29
---

# UMCStory_Node

## 한 줄 정의
Story 그래프 노드의 추상 베이스 — GUID 토폴로지 (MyGuid/Parent/Child) + 라이프사이클(Start/Update/End) + BP 이벤트 (OnMCStoryBegin/Tick/End) + Owner/StoryBoard 컨텍스트.

## 소속 / 상속
- 카테고리: GraphNode (베이스, Abstract)
- `UCLASS(Abstract, config=Game)` `class MCPLAYMODULE_API UMCStory_Node : public UObject`
- BP 자손 정의 가정 (Abstract).

## 핵심 구조
- **UPROPERTY (BlueprintReadOnly — 토폴로지·상태)**:
  - `FVector2D Position` — 에디터 노드 위치.
  - `FGuid MyGuid`, `FGuid ParentGuid`, `TArray<FGuid> ChildGuid` — 영구 GUID 토폴로지.
  - `EMCStoryNodeEnum NodeType` — Entry/Condition/Action/Sequence/Comment 5종 enum.
  - `UMCStory_Node* Node` — 자기 참조 (BP 노출용? 🟡 의도 불명확).
  - `FString GeneratedClassPackage` — BP 클래스 패키지 경로.
  - `bool IsBreakPoint = false`.
- **런타임 상태 (UPROPERTY 아님)**:
  - `TArray<TWeakObjectPtr<UMCStory_Node>> ChildNode` — RecursiveChildNode 가 재구성.
  - `TWeakObjectPtr<UMCStory_Node> CompositeNode = nullptr` — Composite(Sequence) 의 부모 참조.
  - `TWeakObjectPtr<UMCStoryBoard> StoryBoard = nullptr`.
  - `TWeakObjectPtr<AActor> Owner = nullptr`.
  - `EMCStoryNodeState NodeState = EMCStoryStoryNone` — None / Start / Wait / End.
- **protected**:
  - `TArray<FProperty*> PropertyData` — Reflection 사용 (🟡).
  - `FOnStoryNodeNext OnNextNode` — 다음 노드 브로드캐스트 델리게이트 (`DECLARE_MULTICAST_DELEGATE_OneParam`).
- **virtual API**:
  - `virtual UWorld* GetWorld() const override` — UObject 의 GetWorld override (실제 World 반환).
  - `virtual FString GetDescription() const { return TEXT("None"); }` — 자손 override.
  - `virtual void Start()` — NodeState 가드 (None → Start), `OnMCStoryBegin()` 호출.
  - `virtual void Update(float _tick)` — 자손 구현.
  - `virtual void End()` — `OnMCStoryEnd()` 호출.
  - `virtual void MoveToNextNode()` — 자손 override (Condition/Sequence 분기 결정).
- **BP API (BlueprintImplementableEvent / Callable / Pure)**:
  - `void OnMCStoryBegin() / OnMCStoryEnd() / OnMCStoryTick(float)` — BP 이벤트.
  - `void OnFinish(bool finish)` — BP 가 종료 신호.
  - `AActor* GetOwner() { return Owner.Get(); }` — BlueprintPure.
  - `UMCStoryBoard* GetStoryBoard()` — BlueprintPure.
- **Setter (C++)**: `SetPosition / SetMyGuid / SetParentGuid / SetNodeType / SetNode / SetGeneratedClassPackage / AddChildGuid / AddChildNode / ClearChildNode / ClearChild / SetCompositeNode / SetStoryBoard / SetOwner`.
- **Getter (C++)**: `GetChildGuids / GetCompositeNode / GetPosition / GetMyGuid / GetParentGuid / GetNodeType / GetNode`.
- **`#if WITH_EDITOR`**:
  - `SetBreakPoint(bool) / GetBreakPoint()` — Editor breakpoint API.
- **operator==(FGuid) + IsEqual(FGuid)**: GUID 비교 헬퍼.

## 따르는 패턴
- **GUID 토폴로지** — MyGuid (자기) / ParentGuid (부모) / ChildGuid[] (자식). Post-load 시점에 weak ptr (ChildNode) 은 RecursiveChildNode 가 reconstructed.
- **라이프사이클 가드 — NodeState**: Start 가 None 상태에서만 진행 (재 Start 방지).
- **BP 이벤트 표준**: `OnMCStoryBegin / OnMCStoryEnd / OnMCStoryTick` — 자손 BP 가 override 로 게임 로직 구현.
- **CompositeNode 패턴**: NodeType==Sequence 인 노드는 자식 노드의 CompositeNode 로 자기 등록.
- **델리게이트 브로드캐스트**: `MoveToNextNode` 가 `OnNextNode.Broadcast(NextNode)` 발화 → UMCStoryAsset::NextNode 가 수신.

## ⚠ 함정
- **`ChildNode` (TArray<TWeakObjectPtr<>>) 는 런타임 only**: `ChildGuid` (UPROPERTY) 가 영구 토폴로지. ChildNode 는 RecursiveChildNode 가 채워야 유효 — post-load 직후 invalidate.
- **`Node` UPROPERTY (자기 참조)** — 의도 불명확. BP 노출용? 자기 자신 가리키는 BP-friendly 참조 의도? (🟡)
- **GetWorld override** — UObject 가 UWorld 를 반환하도록 override. 호출 시 nullptr 가드 의무. 일반 UObject 와 다른 패턴이므로 호출자 인지 필요.
- **WITH_EDITOR 가드의 IsBreakPoint UPROPERTY**: 본 UPROPERTY 는 가드 *밖* (선언만 가드 밖). cooked 빌드도 직렬화. 런타임 사용 무시 의도? (🟡)
- **`SetCompositeNode` 호출 책임**: Sequence 노드가 자손을 walk 하면서 `SetCompositeNode(this)` 호출 의무 (🟡 추론).
- **명칭 오타**: `DecortorBase` (sic Decorator) — 자손 클래스명에서 누적.

## 연관 entity
- [[entities/MCStory_Graph]] — 노드 컬렉션 호스트.
- [[entities/MCStoryAsset]] — 노드 워크 실행자.
- [[entities/MCStoryBoard]] — 캐릭터 컨텍스트.
- 자손: [[entities/MCStory_Entry]] / [[entities/MCStory_Condition]] / [[entities/MCStory_DecoratorCondition]] / [[entities/MCStory_Sequence]] / [[entities/MCStory_Action]] / [[entities/MCStory_Comment]].
