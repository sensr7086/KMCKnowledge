---
title: "UMCComboTransformTrack"
kind: entity
category: GraphNode
base_class: UMCComboTrack
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/MCCombo/Tracks/MCComboTransformTrack.h
  - KMCProject/MCPlayModule/MCCombo/Tracks/MCComboTransformTrack.cpp
vault_refs:
  - sources/ue-levelsequence-tracks
last_ingested: 2026-05-29
---

# UMCComboTransformTrack

## 한 줄 정의
Transform Track — UMCComboTrack 자손. UMCComboTransformSection 만 호스트. **Section 1개만 허용** (Phase 5p+7 단일 Section 중복 차단). PostLoad 가 RowIndex normalize (lane 분리 의미 없음).

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(BlueprintType)` `class MCPLAYMODULE_API UMCComboTransformTrack : public UMCComboTrack`

## 핵심 구조
- `virtual TSubclassOf<UMCComboSection> SupportsSectionClass() const override { return UMCComboTransformSection::StaticClass(); }`.
- **override AddSection (Phase 5p+7 단일 Section)**: `virtual UMCComboSection* AddSection(FFrameNumber InStart, FFrameNumber InEnd, TSubclassOf<UMCComboSection> OverrideClass = nullptr) override` — 이미 Section 있으면 기존 반환 + warning log.
- **override PostLoad (Phase 5p+9)**: `virtual void PostLoad() override` — Legacy 자산 마이그레이션 + lane normalize. 모든 Section RowIndex=0 강제 → GetLaneCount()=1 → 단일 lane.

## 따르는 패턴
- UMovieScene3DTransformTrack + FMovieSceneFloatChannel × 9 패턴 미러 (KMCProject lite 는 TArray<FMCComboFloatKey> × 9 자체) → [[sources/ue-levelsequence-tracks]] §5.
- Phase 3+ Track 중복 차단 옵션 C 패턴 (Section level — Track 안 Section 1개 보장).
- Phase 5p+9 — TransformTrack 은 keyframe 시퀀스 시스템 → lane 분리 의미 없음 → RowIndex normalize.

## ⚠ 함정
- **단일 Section 강제**: 외부 호출 시도 시 기존 반환 (신규 X) + warning log. 호출자 인지 필요.
- **PostLoad lane normalize**: Section 개수 변경 없음 — RowIndex 만 normalize. 사용자 의도 (vertical drag RowIndex 1+ 설정) 가 자동 0 으로 reset 됨에 주의.
- 베이스 함정 적용.

## 연관 entity
- [[entities/MCComboTrack]] — 직접 베이스.
- [[entities/MCComboTransformSection]] — 단일 호스트 대상 Section.
