---
title: "UMCComboBinding"
kind: entity
category: GraphNode
base_class: UObject
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/MCCombo/MCComboBinding.h
  - KMCProject/MCPlayModule/MCCombo/MCComboBinding.cpp
vault_refs:
  - sources/ue-levelsequence-moviescene
  - sources/ue-ref-11-assetloadingpolicy
  - sources/ue-coreuobject-uobject
last_ingested: 2026-05-29
---

# UMCComboBinding

## 한 줄 정의
SkeletalMesh Binding — Phase 4a 신규 UCLASS. 하나의 SkeletalMesh 자산에 묶인 Tracks 컨테이너. `FMovieScenePossessable` 패턴 미러.

## 소속 / 상속
- 카테고리: GraphNode (5단계 계층의 중간 단계 — AssetRoot → **Binding** → Track → Section → SubProperty)
- `UCLASS(EditInlineNew, DefaultToInstanced, BlueprintType, Blueprintable)` `class MCPLAYMODULE_API UMCComboBinding : public UObject`
- Outer 라이프사이클: UMCComboAsset.

## 핵심 구조
- **UPROPERTY**:
  - `EditAnywhere BlueprintReadOnly Category="Combo|Binding" TSoftObjectPtr<USkeletalMesh> SkeletalMesh` — Soft Reference (vault 11_AssetLoadingPolicy).
  - `EditAnywhere BlueprintReadOnly FName DisplayName = NAME_None` — UI 표시명. 기본값: SkeletalMesh.GetAssetName().
  - `EditAnywhere BlueprintReadOnly FLinearColor BindingColor = (0.35, 0.55, 0.85, 1.0)` — Outliner row 강조 색상.
  - `EditAnywhere Instanced TArray<TObjectPtr<UMCComboTrack>> Tracks` — 본 Binding 의 Track 컨테이너 (5단계 매핑 베이스).
  - `EditAnywhere BlueprintReadOnly uint8 bIsActive : 1` — Eye 토글.
  - `EditAnywhere BlueprintReadOnly uint8 bIsLocked : 1` — Lock 토글.
- **`#if WITH_EDITOR PostEditChangeProperty`** — Phase 4e Outliner refresh hook. Asset Modify() 호출 + OnObjectPropertyChanged 글로벌 delegate broadcast.
- **Track 관리 API**:
  - `UFUNCTION(BlueprintCallable) UMCComboTrack* AddTrack(TSubclassOf<UMCComboTrack>)` — Phase 3+ Track 중복 차단 옵션 C 패턴 (같은 클래스 있으면 기존 반환). Outer=this (Binding).
  - `UFUNCTION(BlueprintCallable) void RemoveTrack(UMCComboTrack*)`.
  - `UFUNCTION(BlueprintPure) bool HasTrackOfClass(TSubclassOf<UMCComboTrack>) const` — Add 메뉴 disable + 중복 차단.

## 따르는 패턴
- FMovieScenePossessable 미러 (UMovieScene::Possessables element) → [[sources/ue-levelsequence-moviescene]] §2.7
- UCLASS 4 meta 표준 (Phase 1 함정 3 회피) → [[sources/ue-coreuobject-uobject]] §2.14 — EditInlineNew + DefaultToInstanced + BlueprintType + Blueprintable.
- TSoftObjectPtr<USkeletalMesh> 패턴 → [[sources/ue-ref-11-assetloadingpolicy]] (lazy load + cooked safe).
- Phase 3+ Track 중복 차단 옵션 C — Binding 안 같은 type Track 1개만 (SkeletalMesh-scoped 의미).
- Outer 라이프사이클: Asset 이 Binding 소유 (Track Outer 는 Binding).

## ⚠ 함정
- **DisplayName 기본값 NAME_None — UI 측 fallback 의무**: 빈 값이면 SkeletalMesh.GetAssetName() 사용.
- **bIsActive / bIsLocked 가 UPROPERTY bitfield**: 단일 비트 직렬화 (UPROPERTY 직렬화 표준).
- **PostEditChangeProperty 가 글로벌 delegate broadcast**: Asset Modify() + FCoreUObjectDelegates::OnObjectPropertyChanged. Outliner / Asset Editor toolkit 의 NotifyTrackChanged 가 listen — 외부 hook 표준 패턴.
- **placeholder Binding (Phase 4c PostLoad)**: 마이그레이션 산물. SkeletalMesh=nullptr + DisplayName="(Legacy)" 시 인지. 새 자산은 명시 mesh 의무.

## 연관 entity
- [[entities/MCComboAsset]] — Outer (Bindings 배열 보관).
- [[entities/MCComboTrack]] — Tracks 보관 대상 베이스.
