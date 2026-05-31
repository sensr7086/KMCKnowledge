---
title: "Component 대분류"
kind: catalog
category: Component
last_ingested: 2026-05-29
---

# Component 대분류

> KMCProject 의 `U...Component` (UActorComponent / U*MeshComponent / UCharacterMovementComponent 자손) + `UAnimInstance` 자손. Subsystem 은 [[catalogs/subsystem]], 인터페이스는 [[catalogs/interface]] 참조.

## 베이스
- 🟡 [[entities/MCActorComponent]] — `UMCActorComponent : UActorComponent`. 모든 MC 컴포넌트의 베이스. Init/Update/End hook + IMCActorInterface 포인터 + MCCOMPONENT_DEF 타입 식별자.

## 구현체 — Animation
- 🟡 [[entities/MCAnimInstance]] — `UAnimInstance` 자손. Move 델리게이트 (FOnMoveMode/Speed/Flag) 구독 + AnimGraph 전이 비트(Run/Walk/Air/Jump).

## 구현체 — Movement
- 🟡 [[entities/MCMoveComponent]] — Move 상태 머신 + 변화 델리게이트.
- 🟡 [[entities/MCCharacterMovementComponent]] — `UCharacterMovementComponent`. Custom Movement(Climbing) + OnLanding.

## 구현체 — Mesh (Soft 패턴)
- 🟢 [[entities/MCSoftSkeletalMeshComponent]] — Soft + Ragdoll + Hit (HitBoneCurve 페어).
- 🟢 [[entities/MCSoftStaticMeshComponent]] — Soft + Materials + AssetUserData(Niagara).

## 구현체 — Mesh (Water 시각화)
- 🟡 [[entities/MCWaterPlaneComponent]] — `UMeshComponent`. 물 수면 자체 SceneProxy.

## 구현체 — Parts
- 🟡 [[entities/MCPartsLoaderComponent]] — UMCPartsAsset 런타임 소비자.

## 구현체 — Physics
- 🟡 [[entities/MCBouyancyComponent]] — 부력 시뮬.

## 구현체 — VFX
- 🟢 [[entities/MCStaticMeshNiagaraSpawnerComponent]] — UAssetUserData → SpawnSystemAttached.

## 구현체 — Loot (MCLoot)
- 🟢 [[entities/MCLootableComponent]] — `UActorComponent`. 루팅 가능 오브젝트 단일 부착 컴포넌트. EMCLootableState 5상태 머신 + Interact + Roll/Grant 흐름 + subsystem 등록. (EMCLootableState 동거 enum)
- 🟢 [[entities/MCInventoryComponent]] — `UActorComponent`. 단순 스택형 인벤토리 (item_id→count 맵, Capacity/MaxStackSize, Add/CanAccept/Clear).

## 페어 카탈로그
- [[catalogs/subsystem]] — Editor preview Subsystem ([[entities/MCNiagaraSocketPreviewSubsystem]]) 등.
- [[catalogs/interface]] — Component 측 관련 인터페이스 (IMCActorInterface 의 AttachedComponent 맵, IMCComboPreviewVisitor).
- [[catalogs/assetuserdata]] — Mesh 컴포넌트가 소비하는 메타 (UMCNiagaraSocketBindings, UMCHitBoneCurveUserData).

## 공통 함정
- **헤더의 한국어 주석 CP949 mojibake**: 일부 컴포넌트의 일부 주석 깨져 보임. CLAUDE.md(KMCProject) 명시 — 임의 재인코딩 금지.
- **MCCOMPONENT_DEF 매크로 의무**: 모든 MC 컴포넌트는 EMCComponentType 식별자 노출.
- **명칭 오타 누적**: `Movemoent`/`Luanch`/`Infomation`/`Virual`/`Constranit`/`Cancle`/`Legde`/`Deteat`/`Overlab`/`DispalcementRto`/`MoveSpeedandle`/`MoveFlagandle`. BP 노출 변경 시 redirector.
- **횡단 정책 — 10 Component 6대 의무** ([[ue-cross-cutting-policies/10_ComponentPolicies]]): Mobility / NewObject(Outer=Owner) / GC(UPROPERTY+TObjectPtr) / GetOwner 캐싱(TWeakObjectPtr) / Tick(빈 메시 회피) / CDO 안전. 본 카테고리 전 entity 공통. 추가로 07 Profiling(Tick 보유 시)·11 AssetLoading(Soft 멤버 시)·12 AssetOpt(Mesh/VFX 자산 시)가 entity별 적용 — 각 entity §6 `횡단 정책 준수` + frontmatter `policy_refs` 참조. 적용 매트릭스 SSOT = [[ue-cross-cutting-policies/index]] §3. (2026-05-31 파일럿 마이그레이션 완료.)
- **FStreamableHandle Pin 의무** + 람다 캡처 = TWeakObjectPtr<this> + IsValid.
- **PhysAnim Constructor 금지** (UnsafeDuringActorConstruction): BeginPlay 이후만.
- **Soft 컴포넌트 + Spawner race**: 권장 — `bAutoSpawnSocketNiagara=true`.
- **Editor preview vs 런타임 분기**: WorldType (Editor/EditorPreview) = Sync, PIE/Cooked = Async + Handle Pin.
- **AnimInstance 델리게이트 unbind**: BeginDestroy / NativeUninitializeAnimation 단계 의무.
