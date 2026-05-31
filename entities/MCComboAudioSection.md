---
title: "UMCComboAudioSection"
kind: entity
category: GraphNode
base_class: UMCComboSection
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/MCCombo/Tracks/MCComboAudioTrack.h
vault_refs:
  - sources/ue-ref-11-assetloadingpolicy
last_ingested: 2026-05-29
---

# UMCComboAudioSection

## 한 줄 정의
Audio Section — UMCComboSection 자손. Soft USoundBase + Volume/Pitch multiplier. Phase 6b `GetSecondaryDisplayString` override 로 Sound 자산명 표시.

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(BlueprintType)` `class MCPLAYMODULE_API UMCComboAudioSection : public UMCComboSection`

## 핵심 구조
- **UPROPERTY (EditAnywhere BlueprintReadOnly)**:
  - `TSoftObjectPtr<USoundBase> Sound` — SoundCue / SoundWave 등.
  - `float VolumeMultiplier = 1.0f (ClampMin=0)`.
  - `float PitchMultiplier = 1.0f (ClampMin=0.01)`.
- **virtual override**:
  - `virtual FString GetSecondaryDisplayString() const override { return Sound.IsNull() ? FString() : Sound.GetAssetName(); }` (Phase 6 추상화 — Sound 자산명 라벨).

## 따르는 패턴
- Soft 권장 → [[sources/ue-ref-11-assetloadingpolicy]].
- Phase 6b GetSecondaryDisplayString virtual override — TrackArea 의 BoxY+BoxH 하단 Yellow 라벨 paint.

## ⚠ 함정
- 베이스 함정 적용 ([[entities/MCComboSection]]).
- Volume/Pitch ClampMin 명시 — 음수 / 0 입력 차단.

## 연관 entity
- [[entities/MCComboSection]] — 직접 베이스.
- [[entities/MCComboAudioTrack]] — 호스트.
