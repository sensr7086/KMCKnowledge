---
title: "GraphNode 대분류"
kind: catalog
category: GraphNode
last_ingested: 2026-05-29
---

# GraphNode 대분류

> KMCProject 의 *런타임* 그래프 콘텐츠 노드 — `UMCStory_*` / `UMCParts_*` / `UMCCombo*` 자손. **UEdGraphNode 가 *아닌* 콘텐츠 측** (편집기 노드는 미래의 Editor 카테고리). **카테고리 도입**: 2026-05-29 (배치 4 Story 직후).

## Story 시스템 (12)

### 그래프
- 🟡 [[entities/MCStory_Graph]]

### 베이스 (Abstract)
- 🟢 [[entities/MCStory_Node]] · 🟡 [[entities/MCStory_DecortorBase]] (sic)

### Story 노드 구현체
- 🟡 [[entities/MCStory_Entry]] · 🟡 [[entities/MCStory_Condition]] · 🟢 [[entities/MCStory_DecoratorCondition]] · 🟡 [[entities/MCStory_Sequence]] · 🟡 [[entities/MCStory_Action]] · 🟡 [[entities/MCStory_Comment]]

### Story Action 자손 (C++)
- 🟡 [[entities/MCStory_ActionAddParts]] · 🟡 [[entities/MCStory_ActionVisibleParts]] · 🟡 [[entities/MCStory_ActionRunStroy]] (sic)

## Parts 시스템 (20)

### 그래프
- 🟡 [[entities/MCParts_Graph]]

### 베이스 (Abstract)
- 🟢 [[entities/MCParts_Node]] · 🟡 [[entities/MCParts_SetMesh]] · 🟡 [[entities/MCParts_Parameter]]

### Mesh 구현체 (3)
- 🟡 [[entities/MCParts_SetStaticMesh]] · 🟡 [[entities/MCParts_SetSkeletalMesh]] · 🟡 [[entities/MCParts_SetSkinnedMesh]]

### 부가 구현체 (8)
- 🟡 [[entities/MCParts_AddVirtualSocket]] · 🟡 [[entities/MCParts_AddNiagara]] · 🟡 [[entities/MCParts_AddSubStaticMesh]] · 🟡 [[entities/MCParts_AddSubSkeletalMesh]] · 🟡 [[entities/MCParts_AddDecal]] · 🟡 [[entities/MCParts_SetOverlayMaterial]] · 🟡 [[entities/MCParts_SkinMeshDeformer]] · 🟡 [[entities/MCParts_Constraint]] · 🟡 [[entities/MCParts_AddSplineCurve]]

### Parameter 구현체 (3)
- 🟡 [[entities/MCParts_MeshParamter]] (sic) · 🟡 [[entities/MCParts_MaterialParameter]] · 🟡 [[entities/MCParts_TransformParameter]]

### Comment
- 🟡 [[entities/MCParts_Comment]]

## Combo 시스템 (13)

### Binding (Phase 4a — SkeletalMesh container)
- 🟢 [[entities/MCComboBinding]] — UMCComboAsset 의 SkeletalMesh Binding. Phase 4 5단계 계층의 중간 단계.

### 베이스 (Abstract)
- 🟢 [[entities/MCComboTrack]] — Track 추상 베이스. UMovieSceneTrack 6 PURE_VIRTUAL 단순화. Phase 6g Binding scope + Phase 7a evaluation dispatcher + Phase 4f Lane + Phase 4 F5 Solo + Phase 3+ §A sort cache.
- 🟢 [[entities/MCComboSection]] — Section 추상 베이스. UMovieSceneSection 패턴. Phase 5e/5f/5o blend + Phase 4f Lane allocation + Phase 6/6a/6a-2/6b/6b-2/6c-2 추상화 + Phase 7a/8/8.2 evaluation/visitor/channel iterator.

### Track 변형 (5)
- 🟢 [[entities/MCComboAudioTrack]] (Asset scope) · 🟢 [[entities/MCComboInputTrack]] (Asset scope) · 🟢 [[entities/MCComboMontageTrack]] (default SkeletalMesh scope) · 🟢 [[entities/MCComboNotifyTrack]] (default scope) · 🟢 [[entities/MCComboTransformTrack]] (단일 Section 강제)

### Section 변형 (5)
- 🟢 [[entities/MCComboAudioSection]] — Soft USoundBase + Volume/Pitch.
- 🟢 [[entities/MCComboInputSection]] — Soft UInputAction + RequiredTriggerEvent + ToleranceFrames.
- 🟢 [[entities/MCComboMontageSection]] — Soft UAnimMontage + SlotName + Phase 4g/5o 통합.
- 🟢 [[entities/MCComboNotifySection]] — FGameplayTag + FireMode.
- 🟢 [[entities/MCComboTransformSection]] — 9 channel keyframe + Constant/Linear/Cubic 보간.

## 페어 카탈로그
- [[catalogs/asset]] — 자산이 본 노드 컬렉션을 보관.
- [[catalogs/subsystem]] — 노드 트리의 실행 호스트.
- [[catalogs/component]] — 런타임 walk 실행자 ([[entities/MCPartsLoaderComponent]]).
- [[catalogs/interface]] — Visitor 패턴 페어 ([[entities/MCComboPreviewVisitor]]).

## 공통 함정
- **GUID 토폴로지** (Story / Parts): `MyGuid` / `ParentGuid` / `ChildGuid[]` 영구. `TMap<FGuid, UNode*> NodeMap` 과 `ChildNode[]` weak ptr 은 런타임 reconstructed.
- **Parts 의 ParameterGuid 별도 컬렉션** — Story 에는 없는 추가 토폴로지.
- **Combo 의 5단계 계층**: AssetRoot → Binding → Track → Section → SubProperty. 각 단계의 책임 분리.
- **`UCLASS(Abstract, BlueprintType, Blueprintable)`** + EditInlineNew/DefaultToInstanced (Combo): 베이스는 BP 자손 설계 가정 + 인스턴스 인라인 보유.
- **WITH_EDITOR hook 차이**:
  - Story 노드: `SetBreakPoint / GetBreakPoint`.
  - Parts 노드: `GetProperty() / GetNodeMaterial()`.
  - Combo Track/Section: `PostEditChangeProperty` 의 sort cache invalidate (Phase 3+ §A 4 trigger).
- **GetWorld override**: Story / Parts 베이스가 UObject 의 GetWorld override — 호출 시 nullptr 가드 의무.
- **TArray<U*_Node*> raw 포인터** — Story / Parts 양쪽 TObjectPtr 미적용. Combo 는 TObjectPtr 사용 (Phase 4 신규).
- **명칭 오타 누적**: Story (`DecortorBase` / `ComplateDecorator` / `ActionRunStroy` / `SetNodeOnwer`) + Parts (`Paramter` / `Parmater` / `Niaraga` / `Constranit`) — Combo 는 오타 없음 (헤더 작성 시기 차이).
- **시간 jump 함정 (Combo Phase 7a)**: PrevFrame ↔ CurrentFrame 간격이 Section duration 보다 크면 통과 Section 미발화. Sequencer 표준은 동시 호출.
- **함정 11 ⭐ (Combo)**: 자손 PostLoad 베이스 Super 선행 호출 의무 — _DEPRECATED 마이그레이션 위험.
- **함정 27 ⭐ (Combo)**: Section 의 Outer Cast 실패 null check (외부 spawning context 보호).
- **함정 30 (Combo)**: mutable 멤버 UPROPERTY 미부착 (Sort cache 등).
- **권고 #4 (Combo Phase 8.2)**: Channel iterator UFUNCTION 미부착 — TArrayView BP 노출 불가. BP 자손 channel-bearing Section 구현 불가, C++ 만.
- **헤더 주석 — confidence 분포**:
  - Story: 14 entity 중 12 🟡 / 2 🟢 — 주석 부족.
  - Parts: 21 entity 중 18 🟡 / 3 🟢 — 주석 부족.
  - Combo: 14 entity 모두 🟢 — 풍부한 vault refs + Phase 단계별 명시 설계 이력.
