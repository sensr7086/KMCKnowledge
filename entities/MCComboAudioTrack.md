---
title: "UMCComboAudioTrack"
kind: entity
category: GraphNode
base_class: UMCComboTrack
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/MCCombo/Tracks/MCComboAudioTrack.h
vault_refs:
  - sources/ue-levelsequence-tracks
last_ingested: 2026-05-29
---

# UMCComboAudioTrack

## 한 줄 정의
Audio Track — UMCComboTrack 자손. Sequencer Audio Track 패턴 미러. UMCComboAudioSection 만 호스트. **Asset scope** (Binding 무관 global, Phase 6g).

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(BlueprintType)` `class MCPLAYMODULE_API UMCComboAudioTrack : public UMCComboTrack`

## 핵심 구조
- `virtual TSubclassOf<UMCComboSection> SupportsSectionClass() const override { return UMCComboAudioSection::StaticClass(); }`.
- `virtual EMCComboTrackBindingScope GetBindingScope() const override { return EMCComboTrackBindingScope::Asset; }` (Phase 6g — 글로벌 BGM/Cue).

## 따르는 패턴
- Phase 6c 검증 — Track 자동 발견 시스템(Phase 6e) 실측용 신규 골격. `FGraphNodeClassHelper(UMCComboTrack::StaticClass())` 캐시 효과 검증 대상.
- AnimMontage Track 의 SoundCue 버전 — Sequencer Audio Track 패턴 → [[sources/ue-levelsequence-tracks]] §6.

## ⚠ 함정
- 베이스 함정 적용 ([[entities/MCComboTrack]]).

## 연관 entity
- [[entities/MCComboTrack]] — 직접 베이스.
- [[entities/MCComboAudioSection]] — SupportsSectionClass 반환 type.
- [[entities/MCComboAsset]] — Asset scope → AssetLevelTracks 에 보관.
