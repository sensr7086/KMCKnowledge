---
title: "UMCSoftSkeletalMeshComponent"
kind: entity
category: Component
base_class: USkeletalMeshComponent
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/Actor/Component/MCSoftSkeletalMeshComponent.h
  - KMCProject/MCPlayModule/Actor/Component/MCSoftSkeletalMeshComponent.cpp
vault_refs:
  - concepts/Component-Policies-6
  - concepts/Asset-Loading-Policy
  - concepts/Profiling-Scope-Rule
  - entities/USkeletalMesh
  - entities/USkeletalMeshComponent
  - sources/ue-assetclasses-mesh
  - sources/ue-assetclasses-physics
  - sources/ue-components-physicscomponents
  - sources/ue-components-primitivecomponent
  - synthesis/mc-character-hit-reaction-pipeline
last_ingested: 2026-05-29
---

# UMCSoftSkeletalMeshComponent

## 한 줄 정의
Soft 참조 전용 SkeletalMeshComponent — Mesh / AnimClass / Override Materials / Ragdoll PhysicsAsset 을 모두 `TSoftObjectPtr` / `TSoftClassPtr` 로 보유하고 비동기 로드 + 통합 Hit Reaction (Ragdoll + Motor + Impulse) 제공.

## 소속 / 상속
- 대분류: Component
- `UCLASS(ClassGroup=(MC), meta=(BlueprintSpawnableComponent), Blueprintable, EditInlineNew)` `class MCPLAYMODULE_API UMCSoftSkeletalMeshComponent : public USkeletalMeshComponent`
- 매크로: `MCCOMPONENT_DEF(UMCSoftSkeletalMeshComponent, EMCComponentType, EMCComponentType::MCSoftSkeletalMesh)`
- 정책 의무 (헤더 주석 명시):
  - `10_ComponentPolicies` 6대 (Mobility / NewObject / GC / GetOwner 캐싱 / Tick / CDO)
  - `11_AssetLoadingPolicy` (Soft + FStreamableManager + Handle Pin/Release)
  - `07_ProfilingScopeRule` (콜백 첫 줄 TRACE_CPUPROFILER_EVENT_SCOPE)
  - `04_OverrideIndex` (BeginPlay = Super FIRST / EndPlay = Super LAST)

## 핵심 구조
- **UActorComponent 오버라이드**: `BeginPlay()`, `EndPlay(EEndPlayReason::Type)`. `#if WITH_EDITOR PostEditChangeProperty(...)` — Soft Path 변경 시 동기 로드 + 프리뷰 갱신.
- **UPROPERTY (Soft 참조 데이터)**:
  - `TSoftObjectPtr<USkeletalMesh> SoftSkeletalMesh`
  - `TSoftClassPtr<UAnimInstance> SoftAnimClass`
  - `TArray<TSoftObjectPtr<UMaterialInterface>> SoftOverrideMaterials`
  - `TSoftObjectPtr<UPhysicsAsset> SoftRagdollPhysicsAsset` — 옵션 override.
- **동작 옵션 UPROPERTY**: `bAutoLoadOnBeginPlay`, `bHiddenUntilLoaded`, `bEnableCollisionOnLoaded`, `bEnableTickOnLoaded`, `int32 LoadPriority` (0~200).
- **Ragdoll UPROPERTY**: `FName DefaultRagdollPivotBone="pelvis"`, `FName RagdollCollisionProfile="Ragdoll"`, `FName RagdollConstraintProfile="Ragdoll"`, `bool bRestoreAnimClassOnDisable`.
- **Hit / PhysAnim UPROPERTY**: `bAutoCreatePhysicalAnimation`, `FName HitProfileName="HitReaction"`, `float DefaultHitMotorStrength=1.0f`, `float DefaultHitImpulseStrength=800.0f`, `float HitUpwardBias=0.25f`, `TObjectPtr<UPhysicalAnimationComponent> PhysicalAnim`.
- **BP 노출 델리게이트**:
  - `FMCOnSoftSkeletalMeshLoadedDelegate OnSoftSkeletalMeshLoaded`
  - `FMCOnSoftSkeletalMeshLoadFailedDelegate OnSoftSkeletalMeshLoadFailed`
  - `FMCOnRagdollStateChangedDelegate OnRagdollStateChanged`
- **BP API (로드)**: `SetSoftSkeletalMesh(InSoftMesh, bRequestLoadImmediately)`, `SetSoftAnimClass(...)`, `RequestLoadAsync()`, `ReleaseLoadedAsset()`, `IsSoftSkeletalMeshLoaded() const`, `IsLoadInProgress() const`.
- **BP API (Ragdoll)**: `EnableFullRagdoll()`, `EnablePartialRagdoll(BoneName, bIncludeSelf=true)`, `DisableRagdoll()`, `SetRagdollConstraintProfile(ProfileName)`, `IsRagdollActive() const`, `SnapMeshToOwnerCapsule()`.
- **BP API (PhysAnim)**: `EnsurePhysicalAnimationComponent()`, `ApplyMotorProfileBelow(BoneName, ProfileName, bIncludeSelf)`, `ApplyMotorSettingsBelow(BoneName, FPhysicalAnimationData, bIncludeSelf)`, `FadeMotorStrength(NewMultiplier)`.
- **BP API (Impulse)**: `ApplyHitImpulse(BoneName, Impulse, bVelChange)`, `ApplyRadialHitImpulse(Origin, Radius, Strength, Falloff, bVelChange)`, `OnHitReceived(BoneName, HitDirection, ImpulseStrength, ProfileNameOverride, bUseFullRagdoll)`, `OnHitFromResult(FHitResult, ImpulseStrength, ProfileNameOverride, bUseFullRagdoll)`.
- **protected 헬퍼**: `HandleAssetsLoaded()`, `ApplyLoadedAssets()`, `HandleRagdollPhysicsAssetLoaded(bFull, BoneIfPartial, bIncludeSelfIfPartial)`, `RequestRagdollActivation(..., OnRagdollReady)`, `ApplyHitMotorAndImpulse(...)`. 상태: `ActivePartialRagdollBone`, `bActivePartialIncludeSelf`.
- **private 핸들·캐시**:
  - `TSharedPtr<FStreamableHandle> LoadHandle` — Mesh+AnimClass+Materials.
  - `TSharedPtr<FStreamableHandle> RagdollLoadHandle` — Ragdoll PhysicsAsset 독립.
  - `TWeakObjectPtr<AActor> CachedOwner`.
  - `TSubclassOf<UAnimInstance> CachedAnimClassBeforeRagdoll` (Transient UPROPERTY).
  - `FName CachedCollisionProfileBeforeRagdoll = NAME_None`.
  - 비트필드: `bInitiallyHidden`, `bRagdollActive`, `bPartialRagdollActive`.

## 따르는 패턴
- Soft + FStreamableManager + Handle Pin/Release → [[concepts/Asset-Loading-Policy]] §2 단계 5 ("Pin 보관 의무")
- Cooked Build SpawnActor 히칭 4단 원인 중 §1.1 [2]·[3] (Subobject Hard Reference 동기 로드) 회피
- 6대 Component 의무 (Mobility / NewObject / GC / GetOwner 캐싱 / Tick / CDO) → [[concepts/Component-Policies-6]]
- 프로파일링 스코프 → [[concepts/Profiling-Scope-Rule]] (콜백 첫 줄 TRACE_CPUPROFILER_EVENT_SCOPE)
- Bundle 사전 로드 외부 시스템(GameMode/Subsystem)이 PreloadPrimaryAssets 묶어 IsSoftSkeletalMeshLoaded()==true 즉시 적용
- Ragdoll 표준 시퀀스 (Mesh→PhysicsAsset→SimulatePhysics) → [[sources/ue-assetclasses-mesh]] §4.2
- 캐릭터 부분 Ragdoll (피격) 결정 트리 → [[sources/ue-components-physicscomponents]] §10
- ApplyPhysicalAnimationProfileBelow + Constraint Profile 동적 전환 → [[sources/ue-components-physicscomponents]] §7.2 / [[sources/ue-assetclasses-physics]] §3
- AddImpulse / AddRadialImpulse → [[sources/ue-components-primitivecomponent]]
- Hit Reaction race fix (Ragdoll 활성 완료 후 motor/impulse 적용) → [[synthesis/mc-character-hit-reaction-pipeline]] §C3
- PhysicsAsset → SetSimulatePhysics 단계 열린 질문 → [[entities/USkeletalMeshComponent]] §3
- 자산 페어 (SkeletalMesh→PhysicsAsset + SetPhysicsAsset) → [[sources/ue-assetclasses-mesh]] §2.4
- AssetUserData → [[entities/USkeletalMesh]] §2.4

## ⚠ 함정
- **LoadHandle Pin 의무**: Pin 안 하면 즉시 GC 후보. 멤버 보관 필수 ([[concepts/Asset-Loading-Policy]] §2 단계 5).
- **람다 캡처는 TWeakObjectPtr<this> + IsValid 검사** — 콜백 도착 전 컴포넌트 GC 가능.
- **SkeletalMesh-only — Tick/AutoActivate**: 생성자에서 `SetComponentTickEnabled(false) + bAutoActivate=false` 로 시작 → 로드 완료 시 활성화. 빈 메시의 TickPose / RefreshBoneTransforms 회피.
- **AnimClass 가 Soft 일 때 순서**: `SetSkeletalMeshAsset` 후 `SetAnimInstanceClass` (순서 중요).
- **함정 #6 (Ragdoll 활성)**: PhysicsAsset 변경 후 `SetSimulatePhysics(true)` 안 호출 = 활성 안 됨 → [[sources/ue-assetclasses-mesh]] §7 함정 #6.
- **함정 #7 (UnsafeDuringActorConstruction)**: Constructor 안 `ApplyPhysicalAnimationProfile` 금지, BeginPlay 이후만 안전 → [[sources/ue-components-physicscomponents]] §11 함정 #7.
- **함정 #8 (motor + simulate 페어)**: `ApplyPhysicalAnimationProfileBelow` 직후 `SetAllBodiesBelowSimulatePhysics` 페어 호출 누락 시 motor 효과 안 나타남 → [[sources/ue-components-physicscomponents]] §11 함정 #8 (헬퍼 안에서 자동 호출).
- **DisableRagdoll 시 본 snap-back 없음** — 자연스러운 복귀를 원하면 `SnapMeshToOwnerCapsule()` 또는 GetUp AnimMontage 별도 트리거.
- **Cloth / Physics Asset 동반 로드**: SkeletalMesh 의 자산이므로 Mesh 로드 시 함께 따라옴 — 별도 Soft 분리 불필요.

## 연관 entity
- [[entities/MCCharacter]] — 주 사용 Owner.
- [[entities/MCActorInterface]].
- [[entities/MCAnimInstance]] — `SoftAnimClass` UPROPERTY 후보 클래스.
- [[entities/MCHitBoneCurveUserData]] — Phase 3 통합 대상 (OnHitReceived 안 SampleAdditiveTransform).
