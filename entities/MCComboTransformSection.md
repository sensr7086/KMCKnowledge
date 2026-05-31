---
title: "UMCComboTransformSection"
kind: entity
category: GraphNode
base_class: UMCComboSection
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/MCCombo/Tracks/MCComboTransformTrack.h
  - KMCProject/MCPlayModule/MCCombo/Tracks/MCComboTransformTrack.cpp
vault_refs:
  - sources/ue-levelsequence-tracks
  - synthesis/mc-combo-editor-phase-8-channel-iterator
last_ingested: 2026-05-29
---

# UMCComboTransformSection

## 한 줄 정의
Transform Section — UMCComboSection 자손. 9 channel (Loc X/Y/Z + Rot Roll/Pitch/Yaw + Scale X/Y/Z) keyframe 시리즈 + 보간(Constant/Linear/Cubic) → FTransform 재조합. UMovieScene3DTransformTrack + 9 FMovieSceneFloatChannel 패턴 미러.

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(BlueprintType)` `class MCPLAYMODULE_API UMCComboTransformSection : public UMCComboSection`

## 핵심 구조
- **9 Channel UPROPERTY (EditAnywhere BlueprintReadWrite)**:
  - `TArray<FMCComboFloatKey> LocationX / Y / Z` (Category="Combo|Transform|Location").
  - `TArray<FMCComboFloatKey> RotationRoll / Pitch / Yaw` (degrees, Category="Combo|Transform|Rotation").
  - `TArray<FMCComboFloatKey> ScaleX / Y / Z` (Category="Combo|Transform|Scale").
- **Phase 5p+8 Group expansion (UObject 영속)**: `UPROPERTY() uint8 bExpandLocation / bExpandRotation / bExpandScale : 1`. Outliner ↔ TrackArea 양방향 sync.
- **Evaluation**:
  - `UFUNCTION(BlueprintCallable) FTransform EvaluateAtFrame(FFrameNumber InLocalFrame) const` — 9 channel 보간 → FRotator(Pitch,Yaw,Roll) → FQuat → FTransform.
  - `UFUNCTION(BlueprintCallable) FTransform EvaluateAtGlobalFrame(FFrameNumber InGlobalFrame) const` — Section StartFrame 자동 변환.
- **Phase 5p+7 + Phase 6b-2 override (베이스 격상 시 OverrideClass 정합)**:
  - `virtual void AddKeyAtGlobalFrame(FFrameNumber InGlobalFrame, EMCComboInterpMode InMode = Linear) override` — 9 channel 모두 현재 보간 값 capture + key 추가 (Sequencer SetKeyAll 미러). Boundary clamp.
  - `virtual void SortAllChannels() override` — 9 channel 시간순 정렬. drag end 호출 의무.
- **Per-channel API (Phase 5p+8 + Phase 6b-2 베이스 override)**:
  - `virtual float GetChannelValueAtLocalFrame(FName ChannelName, FFrameNumber InLocalFrame) const override` — "Location.X" / "Rotation.Roll" / "Scale.Z" 등 매핑. 빈 channel 시 default (Loc/Rot=0, Scale=1).
  - `virtual void SetChannelKeyAtGlobalFrame(FName, FFrameNumber, float, EMCComboInterpMode = Linear) override` — 단일 channel mutate. Boundary clamp + WITH_EDITOR Modify.
- **Phase 5p+6 helpers**:
  - `UFUNCTION(BlueprintCallable) TArray<FFrameNumber> GetUniqueKeyTimes() const` — 9 channel unique time 집합 (Diamond paint).
- **Phase 6/6a/6a-2/6b/6c-2 override**:
  - `virtual int32 GetOutlinerSubRowCount() const override` — 3 group + 펼침 시 +3 channel × 3 groups.
  - `virtual EMCComboRowButtonHints GetEditorRowButtonHints() const override { return KeyframeNav | AddKeyAtScrub; }`.
  - `virtual void AppendOutlinerSubProperties(...) const override` — 3 group + 9 channel = 12 spec (flat tree, ParentIndex).
  - `virtual TArray<FFrameNumber> GetDecorationKeyframes() const override { return GetUniqueKeyTimes(); }`.
  - `virtual void SetSubGroupExpanded(FName, bool) override` — GroupName 매핑.
  - `virtual void AppendSubRowPaintEntries(...) const override` — 3 group + 9 channel + 각 unique key frames.
- **Phase 8 Visitor**: `virtual void AcceptPreviewVisitor(IMCComboPreviewVisitor&) override` — `Visitor.VisitTransformSection(this)` 호출.
- **Phase 8.2 Channel iterator (C++ only)**:
  - `virtual TArrayView<FMCComboFloatKey> GetMutableChannelKeys(FName ChannelName) override` — ChannelName ↔ 9 channel array 매핑. 매칭 안 시 빈 view.
  - `virtual TArray<FName> GetAllChannelNames() const override` — 9 ChannelName 반환.
- **`#if WITH_EDITOR PostEditChangeProperty`** — 9 channel 배열 변경 시 자동 정렬 + sort cache invalidate.
- **private**:
  - `static float EvaluateChannel(const TArray<FMCComboFloatKey>&, FFrameNumber, float DefaultVal)` — Constant/Linear/Cubic 보간 + Boundary clamp.
  - `static void AddKeyToChannel(TArray<FMCComboFloatKey>&, FFrameNumber, float, EMCComboInterpMode)` — 동일 Time overwrite.
  - `static void GatherChannelTimes(const TArray<FMCComboFloatKey>&, TSet<int32>&)`.

## 따르는 패턴
- UMovieScene3DTransformTrack + 9 FMovieSceneFloatChannel 미러 (KMCProject lite = TArray<FMCComboFloatKey> × 9 자체) → [[sources/ue-levelsequence-tracks]] §5.
- Engine 권위 — `FMath::Lerp` / `FMath::CubicInterp` (Hermite, Catmull-Rom tangent) / `FRotator(Pitch,Yaw,Roll)` ctor / `FQuat(FRotator)` Euler→Quat.
- Phase 6/6a/6a-2/6b/6c-2 — Outliner/TrackArea 추상화 override 의 표준 사례.
- Phase 8 Visitor 패턴 → [[synthesis/mc-combo-editor-phase-8-channel-iterator]] §4 (channel iterator canonical case study).

## ⚠ 함정
- **9 channel UPROPERTY 직접 노출**: TArray<FMCComboFloatKey> 9개를 EditAnywhere BlueprintReadWrite — Detail Panel 변경 시 PostEditChangeProperty 의 SortAllChannels 자동 호출 의무.
- **SortAllChannels 누락 시 보간 stale**: SetChannelKeyAtGlobalFrame 의 PostEditChange path + drag end (SMCComboTrackArea::OnMouseButtonUp) path 모두 호출 의무. 누락 시 보간이 비정렬 key 사이에서 잘못.
- **Group 펼침 시 sub-row count 정합**: `GetOutlinerSubRowCount` override 가 group 상태 반영. group 접힘 → child channel sub-row Outliner 트리에서 collapsed → row count 감소.
- **ChannelName 형식 약속**: "Location.X" / "Rotation.Roll" / "Scale.Z" 등. 잘못된 형식 시 default fallback (Loc/Rot=0, Scale=1). 매칭 표준은 SSoT 테이블.
- **Boundary clamp**: AddKeyAtGlobalFrame / SetChannelKeyAtGlobalFrame 모두 Section [Start, End] clamp. 외부 시점 입력 시 clamp 결과로 의도와 다를 수 있음.
- 베이스 함정 적용.

## 연관 entity
- [[entities/MCComboSection]] — 직접 베이스.
- [[entities/MCComboTransformTrack]] — 단일 호스트.
- [[entities/MCComboPreviewVisitor]] — AcceptPreviewVisitor 의 인자.
- `FMCComboFloatKey` (MCComboSection 안 정의된 USTRUCT — generic float keyframe).
