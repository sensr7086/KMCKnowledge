---
title: "UMCComboSection"
kind: entity
category: GraphNode
base_class: UObject
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/MCCombo/MCComboSection.h
  - KMCProject/MCPlayModule/MCCombo/MCComboSection.cpp
vault_refs:
  - sources/ue-levelsequence-moviescene
  - synthesis/mc-combo-editor-phase-5g-5l-drag-ux-suite
  - synthesis/ue-track-area-section-paint-anatomy
  - synthesis/mc-combo-editor-phase-6-7-inhouse-master
  - synthesis/mc-combo-editor-phase-8-channel-iterator
last_ingested: 2026-05-29
---

# UMCComboSection

## 한 줄 정의
Combo Section 추상 베이스 — UMovieSceneSection 패턴 차용 (Sequencer 통합 X). SectionRange (FMCComboFrameRange USTRUCT 래퍼) + Range/Playback/Overlap/Blend/Layout/State/Easing + Phase 6/7/8 추상화 virtual API 다수.

## 소속 / 상속
- 카테고리: GraphNode (5단계 — AssetRoot → Binding → Track → **Section** → SubProperty)
- `UCLASS(Abstract, EditInlineNew, DefaultToInstanced, BlueprintType, Blueprintable)` `class MCPLAYMODULE_API UMCComboSection : public UObject`
- 함께 정의 (USTRUCT/UENUM 다수):
  - `UENUM EMCComboBlendType { Linear, Cubic, Step }` (Phase 5e).
  - `UENUM EMCComboInterpMode { Constant, Linear, Cubic }` (Phase 6b-2, FRichCurve ERichCurveInterpMode 미러).
  - `USTRUCT FMCComboFloatKey { FFrameNumber Time, float Value, EMCComboInterpMode InterpMode }` (Phase 8.2, FRichCurveKey 미러).
  - `UENUM(Bitflags) EMCComboRowButtonHints { None, KeyframeNav, AddKeyAtScrub }` (Phase 6a).
  - `USTRUCT FMCComboSubPropertySpec { PropertyName, Label, Value, ParentIndex, bDefaultExpanded, bIsGroup }` (Phase 6a-2).
  - `USTRUCT FMCComboSubRowPaintEntry { Label, bIsGroup, KeyframeFrames, ChannelName }` (Phase 6c-2 + Phase 8.2.2 ChannelName).

## 핵심 구조
- **Range/Playback/Overlap/Blend/Layout/State/Easing UPROPERTY** (EditAnywhere BlueprintReadOnly):
  - `FMCComboFrameRange SectionRange` + `_DEPRECATED StartFrame / EndFrame` (Phase 2a 마이그레이션, Cycle 6 종료 시 제거).
  - `FFrameNumber StartFrameOffset` — Slip 대응.
  - `float PlayRate (ClampMin=0.01)` + `uint8 bReverse : 1`.
  - `int32 OverlapPriority` + `float Weight (0~1)` + `int32 RowIndex`.
  - `uint8 bIsActive : 1`, `uint8 bIsLocked : 1`, `uint8 bIsSolo : 1` (Phase 4 F5), `uint8 bIsExpanded : 1` (Phase 5p+8).
  - `int32 EaseInFrames`, `int32 EaseOutFrames`, `EMCComboBlendType BlendType` (Phase 5e).
  - `FText DisplayName`, `FLinearColor SectionColor`.
- **Lifecycle**:
  - `virtual void PostLoad() override` — 자손 override 시 Super::PostLoad() 선행 필수 (_DEPRECATED 마이그레이션).
  - `#if WITH_EDITOR PostEditChangeProperty` — Phase 3+ §A 부모 Track sort cache invalidate (함정 27 ⭐ — Outer Cast 실패 null check).
- **Range API**:
  - `GetStartFrame / GetEndFrame / GetDurationFrames`.
  - `UFUNCTION(BlueprintCallable, CallInEditor) SetDurationFrames(int32)` / `SetDurationSeconds(double)` (Phase 4 Detail Panel 버튼).
  - `void SetRange(FFrameNumber, FFrameNumber, bool bMarkDirty=true)` (Phase 3+ §B bMarkDirty — drag 중 Modify skip).
  - `GetStartSeconds / GetEndSeconds (FFrameRate)`.
- **Blend API (Phase 5e/5f/5o)**:
  - `UFUNCTION(BlueprintPure) float GetEffectiveWeight(int32 GlobalFrameValue) const` — Auto-overlap detection 통합.
  - `static float ApplyBlendCurve(float Alpha, EMCComboBlendType)`.
  - `int32 GetAutoEaseInFrames() / GetAutoEaseOutFrames() const` (Phase 5f) — same row overlap detection.
  - `virtual int32 GetEffectiveEaseInFrames() / GetEffectiveEaseOutFrames() const` (Phase 5o virtual 격상) — MontageSection override 시 Montage BlendIn 통합.
- **Phase 4f Lane allocation**:
  - `bool OverlapsWithOtherOnSameRow(const TArray<TObjectPtr<UMCComboSection>>&) const`.
  - `void AssignAutoRowIndex(const TArray<TObjectPtr<UMCComboSection>>&)`.
- **Phase 6/6a/6a-2/6b/6b-2/6c-2 추상화 virtual** (자손 override):
  - `virtual int32 GetOutlinerSubRowCount() const { return 3; }` — Section 펼침 시 Outliner 안 추가 sub-row 갯수.
  - `virtual EMCComboRowButtonHints GetEditorRowButtonHints() const { return None; }` — row 추가 버튼/UI hint flag.
  - `virtual void AppendOutlinerSubProperties(TArray<FMCComboSubPropertySpec>&) const` — sub-property tree spec.
  - `virtual FString GetSecondaryDisplayString() const { return FString(); }` — Section box secondary 라벨.
  - `virtual TArray<FFrameNumber> GetDecorationKeyframes() const { return {}; }` — keyframe diamond paint.
  - `virtual float GetChannelValueAtLocalFrame(FName, FFrameNumber) const { return 0.0f; }`.
  - `virtual void SetChannelKeyAtGlobalFrame(FName, FFrameNumber, float, EMCComboInterpMode = Linear) {}`.
  - `virtual void AddKeyAtGlobalFrame(FFrameNumber, EMCComboInterpMode = Linear) {}` — SetKeyAll 미러.
  - `virtual void SetSubGroupExpanded(FName GroupName, bool) {}`.
  - `virtual void AppendSubRowPaintEntries(TArray<FMCComboSubRowPaintEntry>&) const {}`.
  - `virtual void AppendSubRowPaintEntriesWithSectionRow(TArray<FMCComboSubRowPaintEntry>&) const` (권고 #2) — index 0 = Section main lane anchor + 펼침 시 sub-row.
- **Phase 8 Visitor**: `virtual void AcceptPreviewVisitor(IMCComboPreviewVisitor&)` — default `VisitGenericSection(this)`.
- **Phase 8.2 Channel iterator (C++ only, BP 노출 X)**:
  - `virtual TArrayView<FMCComboFloatKey> GetMutableChannelKeys(FName ChannelName) { return {}; }`.
  - `virtual TArray<FName> GetAllChannelNames() const { return {}; }`.
  - `virtual void SortAllChannels() {}`.
- **Phase 7a evaluation hooks**:
  - `UFUNCTION(BlueprintPure) bool IsFrameInSection(int32 GlobalFrame) const`.
  - `UFUNCTION(BlueprintPure) float ComputeAlphaAtFrame(int32 GlobalFrame) const`.
  - `UFUNCTION(BlueprintImplementableEvent) void OnSectionBegin(UObject* Context)`.
  - `UFUNCTION(BlueprintImplementableEvent) void OnSectionProgress(UObject* Context, float Alpha, float DeltaTime)`.
  - `UFUNCTION(BlueprintImplementableEvent) void OnSectionEnd(UObject* Context)`.
- **WITH_EDITORONLY_DATA**: `Transient bool bIsSelected = false`.

## 따르는 패턴
- UMovieSceneSection FFrameNumber + SectionRange 표준 → [[sources/ue-levelsequence-moviescene]] §2.5
- FMovieSceneFrameRange USTRUCT 래퍼 의무 (UPROPERTY TRange<FFrameNumber> 직접 부착 불가 — UHT 인식 X) → MovieSceneFrameMigration.h L26-L104.
- DEPRECATED brute force search → Class.cpp L1514 / L1690.
- bIsActive/bIsLocked bitfield packing → MovieSceneSection.h L820/L824.
- Phase 5e Linear/Cubic/Step blend (FMovieSceneEasingSettings IMovieSceneEasingFunction 회피).
- Phase 5f overlap auto-detection — same lane 만 (Sequencer 표준 미러).
- Phase 5o Montage BlendIn/Out 통합 (UAnimMontage GetDefaultBlendInTime).
- Phase 5p+8 — UObject 영속 expansion state.
- Phase 6/6a/6a-2/6b/6b-2/6c-2/8 추상화 → [[synthesis/mc-combo-editor-phase-6-7-inhouse-master]]
- Drag UX suite → [[synthesis/mc-combo-editor-phase-5g-5l-drag-ux-suite]]
- Channel iterator → [[synthesis/mc-combo-editor-phase-8-channel-iterator]]
- Track Area paint anatomy → [[synthesis/ue-track-area-section-paint-anatomy]]
- BlueprintNativeEvent / BlueprintImplementableEvent 미러: UAnimNotifyState Begin/Tick/End.

## ⚠ 함정
- **함정 11 ⭐ — 자손 PostLoad 베이스 Super 선행 호출 의무**: _DEPRECATED 마이그레이션 실패 위험.
- **함정 27 ⭐ — Outer Cast 실패 null check**: Section 의 Outer 가 Track 이라는 보장은 EditInlineNew + AddSection (this 를 Outer 로 전달) 패턴에서만 성립. 외부 spawning context 보호.
- **함정 30 — mutable UPROPERTY 미부착**: Sort cache 등 mutable 멤버는 UPROPERTY 부착 X.
- **Phase 2a _DEPRECATED 제거 시점**: Cycle 6 종료 — `git grep "StartFrame_DEPRECATED"` → 0 hits 확인 후 삭제.
- **Phase 5p+8 — Section bIsExpanded UPROPERTY**: 가드 *밖* 선언. cooked 직렬화 영향.
- **GetEffectiveEaseIn/OutFrames virtual + UFUNCTION(BlueprintPure)**: virtual override 시그니처 호환 (Engine 표준 — UAnimInstance virtual UFUNCTION 다수 사례).
- **권고 #4 — Channel iterator UFUNCTION 의도적 미부착**: TArrayView 가 BP 노출 불가. BP 자손은 단일 channel API (GetChannelValueAtLocalFrame / SetChannelKeyAtGlobalFrame) 만. 새 channel-bearing Section 자손은 C++ 만.
- **SortAllChannels 누락 시 보간 비정렬**: SetChannelKeyAtGlobalFrame 의 PostEditChange path + drag end path 모두 본 메서드 호출 의무.

## 연관 entity
- [[entities/MCComboTrack]] — Sections 보관 부모.
- [[entities/MCComboPreviewVisitor]] — AcceptPreviewVisitor 의 인자 인터페이스.
- 자손 5: [[entities/MCComboAudioSection]] · [[entities/MCComboInputSection]] · [[entities/MCComboMontageSection]] · [[entities/MCComboNotifySection]] · [[entities/MCComboTransformSection]].
- FMCComboFrameRange (MCComboFrameRange.h — Range USTRUCT 래퍼, 본 entity 의 SectionRange type).
