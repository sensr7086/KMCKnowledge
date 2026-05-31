---
title: "UMCComboTrack"
kind: entity
category: GraphNode
base_class: UObject
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/MCCombo/MCComboTrack.h
  - KMCProject/MCPlayModule/MCCombo/MCComboTrack.cpp
vault_refs:
  - sources/ue-levelsequence-moviescene
  - synthesis/mc-combo-editor-levelsequence-lite
  - synthesis/ue-track-area-section-paint-anatomy
last_ingested: 2026-05-29
---

# UMCComboTrack

## 한 줄 정의
Combo Track 추상 베이스 — UMovieSceneTrack 의 6 PURE_VIRTUAL 단순화. Section 컨테이너 + display 메타 + Phase 7a evaluation dispatcher + Phase 4f Lane allocation + Phase 4 F5 Solo + Phase 3+ §A sort cache.

## 소속 / 상속
- 카테고리: GraphNode (5단계 계층 — AssetRoot → Binding → **Track** → Section → SubProperty)
- `UCLASS(Abstract, EditInlineNew, DefaultToInstanced, BlueprintType, Blueprintable)` `class MCPLAYMODULE_API UMCComboTrack : public UObject`
- 함께 정의: `UENUM(BlueprintType) enum class EMCComboTrackBindingScope : uint8 { SkeletalMesh, Asset }` (Phase 6g).

## 핵심 구조
- **Display UPROPERTY**:
  - `FText TrackName`, `FLinearColor TrackColor = (0.15, 0.15, 0.25, 1.0)`.
  - `UPROPERTY() uint8 bIsExpanded : 1` (Phase 5p+8) — Outliner 와 양방향 sync. TransformTrack 자손이 ctor 에서 1.
- **Section 컨테이너**:
  - `EditAnywhere Instanced TArray<TObjectPtr<UMCComboSection>> Sections`.
- **Phase 5i Lane 옵션**: `EditAnywhere bool bAutoLaneOnAdd = false` — false=같은 lane(cross-fade 의도 보존, Sequencer 표준), true=greedy lane 분리.
- **PURE_VIRTUAL 자손 의무**:
  - `virtual TSubclassOf<UMCComboSection> SupportsSectionClass() const PURE_VIRTUAL(...)` — 자손 override (LevelSequence MovieScene §2.4 #2).
- **virtual API**:
  - `virtual EMCComboTrackBindingScope GetBindingScope() const { return SkeletalMesh; }` (Phase 6g) — 자손이 Asset scope 면 override.
  - `virtual UMCComboSection* CreateNewSection(TSubclassOf<UMCComboSection> OverrideClass = nullptr)` (Phase 6f-2 OverrideClass 추가).
  - `virtual UMCComboSection* AddSection(FFrameNumber InStart, FFrameNumber InEnd, TSubclassOf<UMCComboSection> OverrideClass = nullptr)` (Phase 5p+7 virtual 격상 + Phase 6f-2 OverrideClass).
  - **Phase 7a evaluation dispatcher**: `virtual void EvaluateAtFrame(int32 CurrentGlobalFrame, int32 PrevGlobalFrame, float DeltaSeconds, UObject* Context)` — Section transition 검출 + OnSectionBegin/Progress/End BIE 호출.
- **C++ API**:
  - `const TArray<TObjectPtr<UMCComboSection>>& GetAllSections() const`.
  - `void RemoveSection(UMCComboSection*)`.
  - `bool IsEmpty() const`.
- **Phase 3+ §A Sort cache (mutable, non-UPROPERTY)**:
  - `const TArray<int32>& GetSortedSectionIndices() const` — OverlapPriority asc 정렬.
  - `void InvalidateSortedSectionIndices() { bSortCacheDirty = true; }` — 외부 (UMCComboSection::PostEditChangeProperty) 가 호출.
  - private: `mutable TArray<int32> CachedSortedIndices`, `mutable bool bSortCacheDirty = true`, `void RebuildSortCache() const`.
- **Phase 4f Lane (RowIndex) 관리**:
  - `int32 GetMaxRowIndex() const` (MovieSceneTrack.cpp L331 미러).
  - `int32 GetLaneCount() const` — 빈 Track 시 1.
  - `void EnsureCompactRowIndices()` (FixRowIndices 미러).
- **Phase 4 F5 Solo**:
  - `UFUNCTION(BlueprintPure) bool HasAnySoloSection() const`.
  - `UFUNCTION(BlueprintPure) bool IsSectionEffectivelyMuted(const UMCComboSection*) const` — Solo 가 있으면 Solo Section 만 visible.
- **`#if WITH_EDITOR PostEditChangeProperty`** — 함정 29 ⭐ Super 호출 의무.

## 따르는 패턴
- UMovieSceneTrack 6 PURE_VIRTUAL 단순화 → [[sources/ue-levelsequence-moviescene]] §2.4
- SupportsSectionClass / CreateNewSection / GetAllSections 표준 3종.
- Phase 5i — 같은 row 안 누적 (cross-fade 의도 = 디자이너 결정, Sequencer 표준 미러).
- Phase 5p+8 — UObject 측 영속 expansion state (Outliner Item->bIsExpanded 와 양방향 sync, STreeView OnExpansionChanged 패턴).
- Phase 4f — Lane allocation greedy (MovieSceneTrack.cpp L780 InitialPlacement 미러, bAllowMultipleRows=true 분기).
- Phase 4 F5 — Solo 양방향 (UMovieSceneTrackRowDecoration::SetSoloed 패턴).
- Phase 3+ §A — Sort cache (Engine ReferenceSkeleton.h L136 + SClippingVerticalBox.h L100 mutable 캐시 패턴). 4 trigger invalidate.
- Phase 6g — Binding scope (SkeletalMesh-level vs Asset-level Track 분리).
- Phase 7a — Sequencer-lite evaluation dispatcher (transition 검출 + BIE).
- Combo Editor 진행 → [[synthesis/mc-combo-editor-levelsequence-lite]]
- Track Area Section paint anatomy → [[synthesis/ue-track-area-section-paint-anatomy]]

## ⚠ 함정
- **PURE_VIRTUAL SupportsSectionClass()**: 자손 override 의무. 미override 시 abstract instantiation 불가.
- **Sort cache 4 trigger invalidate**: AddSection / RemoveSection / PostEditChangeProperty(Sections) / UMCComboSection::PostEditChangeProperty(OverlapPriority). 누락 시 paint / hit-test stale.
- **함정 30 — mutable 멤버 UPROPERTY 미부착**: GC 영향 없는 단순 C++ 캐시.
- **함정 29 — Super::PostEditChangeProperty 호출 의무** (WITH_EDITOR).
- **Phase 7a transition 매트릭스**: PrevFrame ∉ Section, CurrentFrame ∈ Section → OnSectionBegin. PrevFrame ∈ both → OnSectionProgress. PrevFrame ∈, CurrentFrame ∉ → OnSectionEnd. bIsActive=false / bIsLocked Section skip.
- **시간 jump (스크럽)**: PrevFrame ↔ CurrentFrame 간격이 Section duration 보다 크면 *통과* Section 은 Begin+End 미호출. Sequencer 표준은 동시 호출. 향후 권고.
- **AddSection 의 Phase 5p+7 virtual 격상**: TransformTrack 자손이 단일 Section 제약 등 정책 override 가능.

## 연관 entity
- [[entities/MCComboBinding]] — Outer (SkeletalMesh-scope Track 의 경우).
- [[entities/MCComboAsset]] — Outer (Asset-scope Track 의 경우, AssetLevelTracks).
- [[entities/MCComboSection]] — Sections 보관 대상 베이스.
- 자손 5: [[entities/MCComboAudioTrack]] · [[entities/MCComboInputTrack]] · [[entities/MCComboMontageTrack]] · [[entities/MCComboNotifyTrack]] · [[entities/MCComboTransformTrack]].
