---
title: "UMCComboNotifySection"
kind: entity
category: GraphNode
base_class: UMCComboSection
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/MCCombo/Tracks/MCComboNotifyTrack.h
vault_refs: []
last_ingested: 2026-05-29
---

# UMCComboNotifySection

## 한 줄 정의
Notify Section — UMCComboSection 자손. Section 시점에 `FGameplayTag` 발화 (히트박스 / VFX / SFX 트리거).

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(BlueprintType)` `class MCPLAYMODULE_API UMCComboNotifySection : public UMCComboSection`

## 핵심 구조
- **UPROPERTY (EditAnywhere BlueprintReadOnly Category="Combo|Notify")**:
  - `FGameplayTag NotifyTag` — Hitbox.Open / VFX.SpawnSlash / SFX.Whoosh 등.
  - `uint8 FireMode = 0` — Section 안 어디서 발화 (0=Start / 1=End / 2=Both / 3=Continuous).

## 따르는 패턴
- GameplayTag 기반 이벤트 — GAS 페어.
- Section transition 시점에 FireMode 분기.

## ⚠ 함정
- **`FireMode` 가 uint8 (enum 아님)**: 명시 enum 안 쓴 이유 불명확. 0=Start 매직 넘버 (🟡). 향후 EMCComboNotifyFireMode enum 권장.
- 베이스 함정 적용.

## 연관 entity
- [[entities/MCComboSection]] — 직접 베이스.
- [[entities/MCComboNotifyTrack]] — 호스트.
