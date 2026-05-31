---
title: "UMCComboAsset"
kind: entity
category: Asset
base_class: UDataAsset
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/MCCombo/MCComboAsset.h
  - KMCProject/MCPlayModule/MCCombo/MCComboAsset.cpp
vault_refs:
  - sources/ue-levelsequence-moviescene
  - sources/ue-levelsequence-tracks
  - synthesis/mc-combo-editor-levelsequence-lite
last_ingested: 2026-05-29
---

# UMCComboAsset

## 한 줄 정의
Combo 자산 — LevelSequence(ULevelSequence + UMovieScene) 의 데이터 모델만 차용한 시계열 자산. 5단계 계층(AssetRoot → Binding → Track → Section → SubProperty) + Asset 직접 Tracks(global) + Phase 4c 마이그레이션 + Phase 7a 평가 entry.

## 소속 / 상속
- 카테고리: Asset (UDataAsset 자손)
- `UCLASS(BlueprintType)` `class MCPLAYMODULE_API UMCComboAsset : public UDataAsset`
- Sequencer 통합 X — 자체 Slate 트랙 패널 (AnimNotify Track 스타일) 이 시각화.

## 핵심 구조
- **Timeline UPROPERTY (EditAnywhere BlueprintReadOnly Category="Combo|Timeline")**:
  - `FFrameRate TickResolution` — 평가 해상도 (보통 24000 권장). UMovieScene::TickResolution 미러.
  - `FFrameRate DisplayRate` — 표시 fps (24/30/60).
  - `FFrameNumber PlaybackDuration` — 콤보 전체 길이.
- **Phase 4a Bindings (FMovieScenePossessable 미러)**:
  - `EditAnywhere Instanced TArray<TObjectPtr<UMCComboBinding>> Bindings` — SkeletalMesh Binding 컨테이너.
- **Phase 6g Asset-level Tracks (Binding 무관 global)**:
  - `EditAnywhere Instanced TArray<TObjectPtr<UMCComboTrack>> AssetLevelTracks` — Input/Audio 같은 Asset 전역 Track.
- **Phase 4c 마이그레이션 잔여**:
  - `meta=(DeprecatedProperty, DeprecationMessage="Use Bindings → Binding->Tracks instead") TArray<TObjectPtr<UMCComboTrack>> Tracks_DEPRECATED` — PostLoad 가 placeholder Binding 으로 이전.
- **Phase 8.1 auto-suffix 정책**:
  - `EditAnywhere BlueprintReadWrite meta=(DisplayName="Always Suffix First Instance") uint8 bAlwaysSuffixFirst : 1` — Asset-level Track 중복 instance 번호 표기.
- **BP API — Bindings**:
  - `UFUNCTION(BlueprintCallable) UMCComboBinding* AddBinding(TSoftObjectPtr<USkeletalMesh>)` — 중복 차단 (같은 mesh 있으면 기존 반환).
  - `UFUNCTION(BlueprintCallable) void RemoveBinding(UMCComboBinding*)`.
  - `UFUNCTION(BlueprintPure) bool HasBindingOfMesh(TSoftObjectPtr<USkeletalMesh>) const`.
- **BP API — Asset-level Track (Phase 6g/8.1)**:
  - `UFUNCTION(BlueprintCallable) UMCComboTrack* AddAssetLevelTrack(TSubclassOf<UMCComboTrack>)` — 중복 허용 + 자동 numbering.
  - `UFUNCTION(BlueprintCallable) void RemoveAssetLevelTrack(UMCComboTrack*)`.
- **BP API — Phase 7a 평가 entry**:
  - `UFUNCTION(BlueprintCallable) void EvaluateAtFrame(int32 CurrentFrame, int32 PrevFrame, float DeltaSeconds, UObject* Context)` — AssetLevelTracks → Bindings->Tracks 순회 + Track::EvaluateAtFrame dispatch.
  - `UFUNCTION(BlueprintPure) double GetPlaybackSeconds() const`.
- **BP API — Phase 4g**:
  - `UFUNCTION(BlueprintCallable, CallInEditor) void EnsurePlaybackDurationCoversAllSections()` — Section 시간 범위에 맞춰 동적 확장 (one-way grow, 자동 축소 X). MarkPackageDirty WITH_EDITOR.
- **Lifecycle**:
  - `virtual void PostLoad() override` — Phase 4c 자동 마이그레이션 (Tracks_DEPRECATED → placeholder Binding->Tracks).
  - `#if WITH_EDITOR PostEditChangeProperty`.

## 따르는 패턴
- LevelSequence MovieScene 미러 (데이터 모델만, Sequencer 통합 X) → [[sources/ue-levelsequence-moviescene]] §2.3 / [[sources/ue-levelsequence-tracks]] §11
- 5단계 계층 (AssetRoot → Binding → Track → Section → SubProperty) — Phase 4 진입.
- FMovieScenePossessable 미러 (Possessable 패턴) → [[sources/ue-levelsequence-moviescene]] §2.7
- Tracks → Bindings 마이그레이션 (Phase 4c) — Engine MovieScene.h L500-L600 Possessables_DEPRECATED 패턴 + meta=(DeprecatedProperty) serialize 호환.
- Asset-level vs SkeletalMesh-level Track scope (Phase 6g) — `UMCComboTrack::GetBindingScope()` 분기.
- Auto-suffix 정책 (Phase 8.1) — 첫 instance suffix 옵션, 일관된 디자이너 UX.
- Sequencer-lite evaluation (Phase 7a) — Asset → Track → Section 의 transition 검출 + BIE 호출.
- Combo Editor 진행 단계 종합 → [[synthesis/mc-combo-editor-levelsequence-lite]]

## ⚠ 함정
- **마이그레이션 작성 방식 5단계 (헤더 주석 명시)**: (1) "MC Combo Asset" 생성 (UFactory), (2) Outliner Asset(+) → Add Empty Binding, (3) Binding(+) → Track 자손 picker (Binding 안 같은 클래스 1개 중복 차단), (4) 트랙 패널 Section 드래그/드롭, (5) Preview Viewport 스크럽.
- **PostLoad 마이그레이션 1회**: Tracks_DEPRECATED 가 비어있지 않으면 placeholder Binding(SkeletalMesh=nullptr, DisplayName="(Legacy)") 으로 이전 + Rename(REN_DontCreateRedirectors). 이후 저장 시 빈 배열 직렬화.
- **EnsurePlaybackDurationCoversAllSections 가 one-way grow**: 자동 축소 안 함 — 사용자 수동 편집 + Section 작아짐 시 의도 보존. Editor only (MarkPackageDirty).
- **`CallInEditor` meta + BlueprintCallable**: Detail Panel 안 버튼 노출 + 런타임 BP 그래프 표시 X (Trap 30 cleanup, Cycle 5p+3).
- **EvaluateAtFrame 첫 호출 시 `PrevFrame = INDEX_NONE`** — 모든 Section in transition false 처리 의무.
- **시간 jump 함정 (스크럽)**: Sequencer 표준은 통과 Section 도 Begin+End 동시 호출. 현 KMC 구현은 transition 만 검출 → 통과 Section 미발화. 향후 권고.

## 연관 entity
- [[entities/MCComboBinding]] — SkeletalMesh Binding 컨테이너 (Outer=Asset).
- [[entities/MCComboTrack]] — Asset-level + Binding-level 트랙 베이스.
- [[entities/MCComboSection]] — Section 베이스 (트랙 안 시간 구간).
- [[entities/MCComboPreviewVisitor]] — Visitor 패턴 (Phase 8).
- 5 Track 변형: [[entities/MCComboAudioTrack]] · [[entities/MCComboInputTrack]] · [[entities/MCComboMontageTrack]] · [[entities/MCComboNotifyTrack]] · [[entities/MCComboTransformTrack]].
- 5 Section 변형: [[entities/MCComboAudioSection]] · [[entities/MCComboInputSection]] · [[entities/MCComboMontageSection]] · [[entities/MCComboNotifySection]] · [[entities/MCComboTransformSection]].
