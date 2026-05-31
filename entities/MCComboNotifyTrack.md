---
title: "UMCComboNotifyTrack"
kind: entity
category: GraphNode
base_class: UMCComboTrack
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/MCCombo/Tracks/MCComboNotifyTrack.h
vault_refs:
  - sources/ue-levelsequence-tracks
last_ingested: 2026-05-29
---

# UMCComboNotifyTrack

## 한 줄 정의
Notify Track — UMCComboTrack 자손. UMCComboNotifySection 만 호스트. Event Track 페어 + GAS GameplayTag.

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(BlueprintType)` `class MCPLAYMODULE_API UMCComboNotifyTrack : public UMCComboTrack`

## 핵심 구조
- `virtual TSubclassOf<UMCComboSection> SupportsSectionClass() const override { return UMCComboNotifySection::StaticClass(); }`.
- `GetBindingScope` override 없음 → default SkeletalMesh scope (Binding->Tracks).

## 따르는 패턴
- Event Track + GAS GameplayTag 페어 → [[sources/ue-levelsequence-tracks]] §9.

## ⚠ 함정
- 베이스 함정 적용.

## 연관 entity
- [[entities/MCComboTrack]] — 직접 베이스.
- [[entities/MCComboNotifySection]] — SupportsSectionClass 반환 type.
