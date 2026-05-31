---
title: "UMCComboInputSection"
kind: entity
category: GraphNode
base_class: UMCComboSection
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/MCCombo/Tracks/MCComboInputTrack.h
vault_refs:
  - sources/ue-ref-11-assetloadingpolicy
last_ingested: 2026-05-29
---

# UMCComboInputSection

## 한 줄 정의
Input Section — UMCComboSection 자손. Soft UInputAction + RequiredTriggerEvent + ToleranceFrames. 시간 구간 안에 어떤 입력 키가 트리거되어야 하는지 정의.

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(BlueprintType)` `class MCPLAYMODULE_API UMCComboInputSection : public UMCComboSection`

## 핵심 구조
- **UPROPERTY (EditAnywhere BlueprintReadOnly Category="Combo|Input")**:
  - `TSoftObjectPtr<UInputAction> InputAction` — Enhanced Input.
  - `uint8 RequiredTriggerEvent = 1` (default ETriggerEvent::Triggered) — Started(0) / Triggered(1) / Completed(2).
  - `int32 ToleranceFrames = 240 (ClampMin=0)` — 입력 타이밍 관용도. 0.1s @ 2400fps tick.

## 따르는 패턴
- Soft 권장 → [[sources/ue-ref-11-assetloadingpolicy]].
- 매칭 윈도우 — Section [Start, End] + ToleranceFrames 범위.

## ⚠ 함정
- **`RequiredTriggerEvent` 가 uint8 (ETriggerEvent enum 아님)**: 명시 enum 안 쓴 이유 불명확. 1 = Triggered 의 매직 넘버. 다른 값 의미 주석 필요 (🟡).
- 베이스 함정 적용.

## 연관 entity
- [[entities/MCComboSection]] — 직접 베이스.
- [[entities/MCComboInputTrack]] — 호스트.
- UInputAction (Enhanced Input UE 표준).
