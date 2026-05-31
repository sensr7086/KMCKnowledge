---
title: "UMCComboMontageSection"
kind: entity
category: GraphNode
base_class: UMCComboSection
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/MCCombo/Tracks/MCComboMontageTrack.h
  - KMCProject/MCPlayModule/MCCombo/Tracks/MCComboMontageTrack.cpp
vault_refs:
  - sources/ue-ref-11-assetloadingpolicy
  - sources/ue-levelsequence-tracks
  - synthesis/mc-combo-editor-phase-5g-5l-drag-ux-suite
last_ingested: 2026-05-29
---

# UMCComboMontageSection

## 한 줄 정의
Montage Section — UMCComboSection 자손. Soft UAnimMontage 재생. Phase 4g 자동 메타데이터 동기 (DisplayName + SectionRange.End from Montage 길이). Phase 5o EaseIn/Out 에 Montage BlendIn 통합. Phase 6 추상화 override 다수.

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(BlueprintType)` `class MCPLAYMODULE_API UMCComboMontageSection : public UMCComboSection`

## 핵심 구조
- **UPROPERTY (EditAnywhere BlueprintReadOnly Category="Combo|Montage")**:
  - `TSoftObjectPtr<UAnimMontage> Montage` — Soft (vault 11_AssetLoadingPolicy).
  - `FName StartSectionName = NAME_None` — 몽타주 안 named section (빈=처음부터).
  - `FName SlotName = NAME_None` — AnimInstance Slot Name (NAME_None = default Slot).
  - `bool bSkipAnimNotifiers = false` — AnimNotify 발화 skip.
- **Lifecycle**:
  - `virtual void PostLoad() override` — 함정 11 ⭐ 베이스 _DEPRECATED 마이그레이션 + 자손 PlayRate fallback.
  - `#if WITH_EDITOR PostEditChangeProperty` (Phase 4g) — Montage 셋/변경 시 DisplayName/SectionRange.End 자동 + Asset->EnsurePlaybackDurationCoversAllSections 호출.
- **Phase 4g BP API**:
  - `UFUNCTION(BlueprintCallable, CallInEditor) void ApplyMontageMetadata()` — Editor only (LoadSynchronous + MarkPackageDirty).
- **Phase 5o BP API**:
  - `UFUNCTION(BlueprintPure) int32 GetMontageBlendInFrames() / GetMontageBlendOutFrames() const` — UAnimMontage::GetDefaultBlendInTime() 의 frame 변환 (TickResolution 통해).
- **Phase 5o virtual override**:
  - `virtual int32 GetEffectiveEaseInFrames() const override` — `max(manual, auto-overlap, montageBlend)`.
  - `virtual int32 GetEffectiveEaseOutFrames() const override`.
- **Phase 5p #4 진단**:
  - `UFUNCTION(BlueprintCallable, CallInEditor) void LogEffectiveBlendTimes() const` — Detail Panel 안 콘솔 진단.
- **Phase 6/6a/6a-2/6b override**:
  - `virtual int32 GetOutlinerSubRowCount() const override { return 4; }` — SlotName 추가.
  - `virtual void AppendOutlinerSubProperties(...) const override` — Weight / PlayRate / SlotName / OverlapPriority 4 spec.
  - `virtual FString GetSecondaryDisplayString() const override` — SlotName.ToString() (NAME_None 시 빈).
- **Phase 8 Visitor**:
  - `virtual void AcceptPreviewVisitor(IMCComboPreviewVisitor&) override` — `Visitor.VisitMontageSection(this)` 호출.

## 따르는 패턴
- Soft Montage 자산 → [[sources/ue-ref-11-assetloadingpolicy]].
- SkeletalAnimation Section 패턴 → [[sources/ue-levelsequence-tracks]] §5.
- Slot 이름 권위 — UAnimInstance::GetSlotMontageGlobalWeight(L437) / GetSlotMontageLocalWeight(L442) / Blueprint_GetSlotMontageLocalWeight(L605) / FQueuedRootMotionBlend::SlotName(L1619).
- Phase 4g — Montage 메타 동기 (자산명 → DisplayName, 길이 → SectionRange.End, Asset PlaybackDuration grow).
- Phase 5o — Montage BlendIn/Out 통합 (Sequencer LevelSequence SkeletalAnimationSection 미러) → [[synthesis/mc-combo-editor-phase-5g-5l-drag-ux-suite]] §Phase 5o.

## ⚠ 함정
- **함정 11 ⭐ — PostLoad 베이스 Super 선행 호출 의무**.
- **Phase 4g — Editor only**: ApplyMontageMetadata 의 LoadSynchronous + MarkPackageDirty. CallInEditor meta + BlueprintCallable — 런타임 BP 그래프 표시 X (Trap 30 cleanup).
- **GetEffectiveEaseIn/OutFrames virtual override 시 베이스 Super 미호출**: `max(manual, auto, montageBlend)` 직접 계산. 베이스 호출 시 montageBlend 누락.
- **Phase 5o Editor only — Montage->Get() 안전 가정**: 이미 LoadSynchronous 호출 caller 가정. Cooked runtime 비차단.

## 연관 entity
- [[entities/MCComboSection]] — 직접 베이스.
- [[entities/MCComboMontageTrack]] — 호스트.
- [[entities/MCComboBinding]] — Binding scope 페어 (Track 의 호스트).
- [[entities/MCComboPreviewVisitor]] — AcceptPreviewVisitor 의 인자.
