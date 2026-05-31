---
title: "UMCComboInputTrack"
kind: entity
category: GraphNode
base_class: UMCComboTrack
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/MCCombo/Tracks/MCComboInputTrack.h
vault_refs:
  - sources/ue-levelsequence-tracks
last_ingested: 2026-05-29
---

# UMCComboInputTrack

## 한 줄 정의
Input Track — UMCComboTrack 자손. Enhanced Input UInputAction 페어. UMCComboInputSection 만 호스트. **Asset scope** (Phase 6g — 사용자 입력 timeline 은 SkeletalMesh 무관).

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(BlueprintType)` `class MCPLAYMODULE_API UMCComboInputTrack : public UMCComboTrack`

## 핵심 구조
- `virtual TSubclassOf<UMCComboSection> SupportsSectionClass() const override { return UMCComboInputSection::StaticClass(); }`.
- `virtual EMCComboTrackBindingScope GetBindingScope() const override { return EMCComboTrackBindingScope::Asset; }` (Phase 6g).

## 따르는 패턴
- Property Track 패턴 + Enhanced Input 페어 → [[sources/ue-levelsequence-tracks]] §2.

## ⚠ 함정
- 베이스 함정 적용.

## 연관 entity
- [[entities/MCComboTrack]] — 직접 베이스.
- [[entities/MCComboInputSection]] — SupportsSectionClass 반환 type.
