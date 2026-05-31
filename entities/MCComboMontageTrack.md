---
title: "UMCComboMontageTrack"
kind: entity
category: GraphNode
base_class: UMCComboTrack
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/MCCombo/Tracks/MCComboMontageTrack.h
vault_refs:
  - sources/ue-levelsequence-tracks
last_ingested: 2026-05-29
---

# UMCComboMontageTrack

## 한 줄 정의
Montage Track — UMCComboTrack 자손. UMCComboMontageSection 만 호스트. SkeletalAnimation Track 페어. 베이스 default scope (SkeletalMesh — Binding 안).

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(BlueprintType)` `class MCPLAYMODULE_API UMCComboMontageTrack : public UMCComboTrack`

## 핵심 구조
- `virtual TSubclassOf<UMCComboSection> SupportsSectionClass() const override { return UMCComboMontageSection::StaticClass(); }`.
- `GetBindingScope` override 없음 → default SkeletalMesh scope (Binding->Tracks).

## 따르는 패턴
- SkeletalAnimation Track 페어 → [[sources/ue-levelsequence-tracks]] §5.

## ⚠ 함정
- 베이스 함정 적용.

## 연관 entity
- [[entities/MCComboTrack]] — 직접 베이스.
- [[entities/MCComboMontageSection]] — SupportsSectionClass 반환 type.
- [[entities/MCComboBinding]] — 호스트 (SkeletalMesh scope → Binding->Tracks).
