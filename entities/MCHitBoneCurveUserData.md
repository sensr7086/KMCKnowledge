---
title: "UMCHitBoneCurveUserData"
kind: entity
category: AssetUserData
base_class: UAssetUserData
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/Asset/MCHitBoneCurveUserData.h
  - KMCProject/MCPlayModule/Asset/MCHitBoneCurveUserData.cpp
vault_refs:
  - sources/ue-assetclasses-assetuserdata
  - sources/ue-assetclasses-mesh
  - concepts/MC-Asset-Validation-Policy
  - concepts/Component-Policies-6
  - sources/mc-soft-skeletalmesh-ragdoll
  - sources/ue-coreuobject-uobject
last_ingested: 2026-05-29
---

# UMCHitBoneCurveUserData

## 한 줄 정의
캐릭터 페르소나에 부착되는 본별 히트 커브 데이터 — USkeletalMesh 의 UAssetUserData. UCurveVector 어셋 1개로 3축(Pitch/Yaw/Roll degrees) 통합. 본별 *회전만* (Translation 은 Ragdoll/PhysAnim 분담).

## 소속 / 상속
- 대분류: **DataTable** (에셋 류). SkeletalMesh + HitReaction Component 페어 — [[entities/MCSoftSkeletalMeshComponent]] §OnHitReceived 안 SampleAdditiveTransform 통합 대상 (Phase 3).
- `UCLASS(BlueprintType, DefaultToInstanced, EditInlineNew, CollapseCategories, meta=(DisplayName="MC Hit Bone Curve User Data"))`
- `class MCPLAYMODULE_API UMCHitBoneCurveUserData : public UAssetUserData`
- 함께 정의:
  - `USTRUCT(BlueprintType) FMCHitBoneAdditiveCurve` — 단일 본의 시간 기반 Additive Rotation 커브.

## 핵심 구조
- **UPROPERTY**:
  - `TArray<FMCHitBoneAdditiveCurve> BoneCurves` — `meta=(TitleProperty="BoneName")` 디자이너 list view 친화.
  - `float GlobalScale=1.0f` (0~2) — 모든 본 일괄 적용 (Difficulty / 캐릭터 type 별).
- **FMCHitBoneAdditiveCurve 필드**:
  - `FName BoneName` — Skeleton BoneName 일치 (Phase 4 검증 의무).
  - `float Duration=0.5f` (0.01~10, UIMin 0.1~UIMax 2.0).
  - `float IntensityScale=1.0f` (0~2).
  - `float DirectionInfluence=0.0f` (0~1) — 0=world / 1=HitDirection 기준 (본이 *맞은 방향* 으로 밀림).
  - `TObjectPtr<UCurveVector> RotationCurves=nullptr` — 채널 매핑: X=Pitch / Y=Yaw / Z=Roll(degrees). nullable — 미설정 시 Identity 회전 (Soft fail). 본 간 공유 가능 ("HeadHitYawSpin" prefab 등).
  - `FTransform SampleAtTime(float TimeSec, const FVector& OptionalHitDirection=FVector::ZeroVector) const` — TimeSec 만료 또는 nullptr = Identity.
  - `bool IsStructValid() const` — BoneName 비어있지 않고 Duration>0.
- **BP API**:
  - `UFUNCTION(BlueprintPure) bool GetCurveForBone(FName, FMCHitBoneAdditiveCurve& OutCurve) const`
  - `UFUNCTION(BlueprintPure) FTransform SampleAdditiveTransform(FName, float TimeSec, FVector HitDirection=Zero, float ExternalStrength=1.0f) const` — 외부 hit strength × IntensityScale × GlobalScale.
  - `UFUNCTION(BlueprintPure) void GetAllBoneNames(TArray<FName>& OutBoneNames) const`
  - `UFUNCTION(BlueprintPure) bool HasCurveForBone(FName) const`
  - `UFUNCTION(BlueprintPure) bool HasValidBoneCurves() const` — **이름 충돌 회피**: UE 의 `UObject::IsDataValid(TArray<FText>&)` / `IsDataValid(FDataValidationContext&)` 와 name hiding 회피.
- **UObject 라이프사이클**:
  - `virtual void PostInitProperties() override` — `BoneCurves.Reserve(256)` (TArray reallocation 방어, dangling pointer fix defense in depth).
  - `virtual void PostLoad() override` — 디스크 로드 시 동일 capacity 보장.
- **`#if WITH_EDITOR` (Phase 1 + Phase 4 검증)**:
  - `virtual void PostEditChangeOwner(const FPropertyChangedEvent&) override` — BoneName 유효성 + CachedSkeletonBoneNames 갱신.
  - `virtual EDataValidationResult IsDataValid(FDataValidationContext&) const override` — Persona/Content Browser "Validate Data" 통합. BoneName 빈값/Duration<=0 → AddError. Skeleton 미존재 본 → AddWarning. 빈 BoneCurves → Info.
  - `int32 GetInvalidEntryCount(USkeleton*) const` — invalid entry 수 (BoneName 빈값/Duration<=0/Skeleton 미존재).
  - `int32 CleanupInvalidEntries(USkeleton*)` — Modify() + RemoveAll predicate, undo 가능. Reimport 후 stale entry cleanup.
  - `bool IsBoneValidInSkeleton(FName, USkeleton*) const` — SMCHitBoneCurveEditor list row invalid marker 헬퍼.
  - `#if WITH_EDITORONLY_DATA UPROPERTY(Transient) TSet<FName> CachedSkeletonBoneNames` — Phase 4 검증용.

## 따르는 패턴
- UAssetUserData 표준 (DefaultToInstanced + EditInlineNew + abstract 자손) → [[sources/ue-assetclasses-assetuserdata]] §1
- 자손 작성 패턴 (UFootstepData 모범) → [[sources/ue-assetclasses-assetuserdata]] §4
- USkeletalMesh AddAssetUserData / GetAssetUserData<T>() → [[sources/ue-assetclasses-mesh]]
- MC_LOGRET_* 매크로 / silent return 금지 → [[concepts/MC-Asset-Validation-Policy]]
- GC 방어 (UPROPERTY + TObjectPtr) — RotationCurves 의 UCurveVector 보관 → [[concepts/Component-Policies-6]] §3
- OnHitReceived 안 SampleAdditiveTransform 통합 (Phase 3) → [[sources/mc-soft-skeletalmesh-ragdoll]] §6
- UObject::IsDataValid(FDataValidationContext&) 5.x 표준 (Object.h:1101) → [[sources/ue-coreuobject-uobject]]
- 5.6+ 시그니처 함정 #5 → [[sources/ue-assetclasses-assetuserdata]] §9 함정 #5
- 재설계 기록 (2026-05-13 → 2026-05-14): FRuntimeFloatCurve 6개 → UCurveVector 1개. Translation 제거 (Ragdoll/PhysAnim 분담). UCurveVector 어셋 = 외부 .uasset 참조 → 본 간 공유 + SCurveEditor dangling 자체 해소.

## ⚠ 함정
- **🚨 dangling pointer fix (2026-05-14)**: IStructureDetailsView 안 SCurveEditor 가 `FRichCurve*` (BoneCurves[Index] 안 멤버) 캐시 → BoneCurves Add/Remove 시 TArray reallocation → stale → SCurveEditor::OnPaint crash. **해결 2중**: (1) SMCHitBoneCurveEditor::UpdateEntryDetailsView 매번 IStructureDetailsView 전체 재생성 (구조적 fix), (2) `Reserve(256)` (일반 케이스 reallocation 0 보장, defense in depth).
- **이름 충돌 회피 — `HasValidBoneCurves`**: UE 5.x 의 `UObject::IsDataValid(TArray<FText>&)` / `IsDataValid(FDataValidationContext&)` 와 name hiding (C4264) 충돌 회피.
- **`IsDataValid()` 무인자 BP 헬퍼**는 본 override 와 name hiding 충돌 → `HasValidBoneCurves` 로 명명.
- **권장 본 5~10개** (head / spine_03 / hand_l / hand_r / pelvis 등). 일치 안 하는 BoneName 은 런타임 무시 + `LogMCAsset Warning`.
- **외부 강도 곱셈 순서**: 외부 hit strength × IntensityScale × GlobalScale. 음수 시 의도와 반대 회전.

## 연관 entity
- [[entities/MCSoftSkeletalMeshComponent]] — Phase 3 통합 대상 (OnHitReceived 안 SampleAdditiveTransform).
- UCurveVector (UE 표준 — Content Browser "Curve > CurveVector" 어셋).
- SMCHitBoneCurveEditor (MCEditorModule — 미인덱싱, 본 자산의 커스텀 에디터).
- USkeleton (UE 표준 — BoneName 검증 대상).
