---
title: "UMCSoftStaticMeshComponent"
kind: entity
category: Component
base_class: UStaticMeshComponent
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/Actor/Component/MCSoftStaticMeshComponent.h
  - KMCProject/MCPlayModule/Actor/Component/MCSoftStaticMeshComponent.cpp
vault_refs:
  - concepts/Component-Policies-6
  - concepts/Asset-Loading-Policy
  - concepts/Profiling-Scope-Rule
  - concepts/Soft-Reference-vs-Hard
  - sources/ue-assetclasses-assetuserdata
  - synthesis/mc-soft-asset-component-pattern
policy_refs:
  - component-policies
  - profiling-scope-rule
  - asset-loading-policy
  - asset-optimization-policy
last_ingested: 2026-05-29
---

# UMCSoftStaticMeshComponent

## 한 줄 정의
Soft 참조 전용 StaticMeshComponent — Mesh / Override Materials / AssetUserData(UMCNiagaraSocketBindings) 를 모두 비동기 로드 + Editor 프리뷰까지 통합.

## 소속 / 상속
- 대분류: Component
- `UCLASS(ClassGroup=(MC), meta=(BlueprintSpawnableComponent), Blueprintable, EditInlineNew)` `class MCPLAYMODULE_API UMCSoftStaticMeshComponent : public UStaticMeshComponent`
- 매크로: `MCCOMPONENT_DEF(UMCSoftStaticMeshComponent, EMCComponentType, EMCComponentType::MCSoftStaticMesh)`
- 정책 의무 (헤더 주석 명시): `10_ComponentPolicies` / `11_AssetLoadingPolicy` / `07_ProfilingScopeRule` / `04_OverrideIndex`.

## 핵심 구조
- **UActorComponent 오버라이드**: `BeginPlay()`, `EndPlay(EEndPlayReason::Type)`.
- **`OnRegister() override`** — SCS 인스턴스 / BP archetype preview / Editor placed actor / PIE 전 케이스 호출. Editor World (EWorldType::Editor / EditorPreview) 컨텍스트에서 동기 로드 + SetStaticMesh + Niagara preview. PIE/Cooked 는 BeginPlay 의 RequestLoadAsync 가 처리 (OnRegister 는 noop).
- **`#if WITH_EDITOR PostEditChangeProperty(...)`** — Soft Path 변경 시 동기 로드 + 프리뷰 갱신.
- **UPROPERTY (Soft 참조 데이터)**:
  - `TSoftObjectPtr<UStaticMesh> SoftStaticMesh`
  - `TArray<TSoftObjectPtr<UMaterialInterface>> SoftOverrideMaterials`
- **동작 옵션 UPROPERTY**: `bAutoLoadOnBeginPlay`, `bHiddenUntilLoaded`, `bEnableCollisionOnLoaded`, `int32 LoadPriority` (0~200), `bAutoFillOverrideMaterialsFromMesh` (true 기본 — 빈 배열일 때만 메시 슬롯 mirroring, 명시 override 보존).
- **AssetUserData / Niagara Socket UPROPERTY**: `bAutoSpawnSocketNiagara` (true 기본), `int32 SocketNiagaraLoadPriority` (0~200), `bEditorPreviewSocketNiagara` (true 기본).
- **BP 노출 델리게이트**:
  - `FMCOnSoftMeshLoadedDelegate OnSoftMeshLoaded`
  - `FMCOnSoftMeshLoadFailedDelegate OnSoftMeshLoadFailed`
  - `FMCOnSoftMeshSocketNiagarasSpawnedDelegate OnSocketNiagarasSpawned`
- **BP API**:
  - 로드: `SetSoftStaticMesh(InSoftMesh, bRequestLoadImmediately)`, `RequestLoadAsync()`, `ReleaseLoadedAsset()`, `IsSoftMeshLoaded() const`, `IsLoadInProgress() const`.
  - Socket Niagara: `RequestSocketNiagaraSpawn()` (single-shot, 외부 책임), `DeactivateSpawnedSocketNiagaras()`, `IsSocketNiagaraSpawnInProgress() const`.
- **protected 헬퍼**: `HandleAssetsLoaded()`, `ApplyLoadedAssets()`, `ProcessAssetUserDataAfterMeshLoaded()`, `HandleSocketBindingsLoaded()`. `#if WITH_EDITOR ProcessAssetUserDataAfterMeshLoadedEditorSync()` — Editor 동기 path (LoadAllSynchronous + SpawnAllNow). `AutoFillOverrideMaterialsFromMesh(UStaticMesh*)` — 디자이너 명시 override 보존. `ApplyMeshAndMaterialsSynchronous()` — OnRegister 공용.
- **private 핸들·캐시**:
  - `TSharedPtr<FStreamableHandle> LoadHandle` — Mesh+Materials.
  - `TSharedPtr<FStreamableHandle> SocketBindingsLoadHandle` — Niagara 독립.
  - `UPROPERTY(Transient) TObjectPtr<UMCNiagaraSocketBindings> CachedSocketBindings` — Mesh 로드 시 한 번 캐싱. Transient — 시리얼라이즈 안 됨.
  - `TArray<TWeakObjectPtr<UNiagaraComponent>> SpawnedSocketNiagaras` — Pool AutoRelease 가 lifetime 관리, weak 보관.
  - `TWeakObjectPtr<AActor> CachedOwner`.
  - `uint8 bInitiallyHidden : 1`.

## 따르는 패턴
- Soft + Handle Pin/Release → [[concepts/Asset-Loading-Policy]] §2 단계 5
- Cooked SpawnActor 히칭 §1.1 [2]·[3] (Subobject Hard Reference) 회피
- 6대 Component 의무 → [[concepts/Component-Policies-6]]
- EWorldType::Editor 분기 = Sync 표준 → [[concepts/Asset-Loading-Policy]] §3
- Editor preview path → [[synthesis/mc-soft-asset-component-pattern]] §7.5
- Niagara 확장 (Spawner 컴포넌트 부착 없이 동일 동작) — Soft 컴포넌트가 자기 메시 로드 완료 콜백 안에서 메타 추출 + Niagara spawn → [[synthesis/mc-soft-asset-component-pattern]] §5
- UStaticMesh + UAssetUserData / IInterface_AssetUserData 표준 패턴 → [[sources/ue-assetclasses-assetuserdata]] §4
- 머티리얼은 메시와 함께 Soft 로 묶어 같은 핸들로 로드 → [[concepts/Soft-Reference-vs-Hard]]
- 프로파일링 스코프 → [[concepts/Profiling-Scope-Rule]]

## ⚠ 함정
- **LoadHandle Pin 의무** (11_AssetLoadingPolicy §6) — 멤버 보관 필수.
- **람다 캡처 TWeakObjectPtr<this> + IsValid** — 콜백 도착 전 GC 가능.
- **외부 Spawner 컴포넌트 + Soft 컴포넌트 race**: UMCStaticMeshNiagaraSpawnerComponent::BeginPlay 시점에 Soft Mesh 미로드 → GetStaticMesh()==nullptr → AssetUserData 추출 실패 → Soft fail 후 영구 비활성. 해법: `bAutoSpawnSocketNiagara=true` 로 Soft 컴포넌트가 자체 spawn 처리.
- **`RequestSocketNiagaraSpawn` single-shot**: 메시 로드 안 된 상태에서 호출 시 OnSoftMeshLoaded 후 자동 재호출 안 됨 — 외부 책임.
- **`bAutoSpawnSocketNiagara==false` 면 `bEditorPreviewSocketNiagara` 도 무시** (자동 spawn 비활성 의도 우선).
- **`AutoFillOverrideMaterialsFromMesh`**: 디자이너 명시 override 가 있으면 절대 덮어쓰지 않음 (배열 빈 경우만 mirroring).

## 횡단 정책 준수
> 적용: [[ue-cross-cutting-policies/index]] §3 (Component 행). raw 미마운트 — 추출 본문 근거, 미확인은 ❓.

| 정책 | 적용 | 근거 / 위반·미확인 | 신뢰도 |
|---|---|---|---|
| 11 asset-loading | ✅ | Mesh/Materials/AssetUserData(Niagara) `TSoftObjectPtr` 비동기 + LoadHandle/SocketBindingsLoadHandle Pin + Editor 동기 분기(EWorldType) = 정책 정합. → [[ue-cross-cutting-policies/11_AssetLoadingPolicy]] | 🟢 |
| 10 component | ✅ | TObjectPtr/Transient UPROPERTY GC, CachedOwner TWeak, 빈 메시 Tick 회피 = 6대(헤더 주석 명시). → [[ue-cross-cutting-policies/10_ComponentPolicies]] | 🟢 |
| 07 profiling | ✅ | 헤더 주석 `07_ProfilingScopeRule` 의무 명시. | 🟢 |
| 12 asset-opt | ✅ | StaticMesh — **LOD/Nanite 결정**(12 §2) 대상. 실제 설정 ❓(자산 측). → [[ue-cross-cutting-policies/12_AssetOptimizationPolicy]] | 🟡 |
| 09 global-iterator | ➖ | 미사용. | 🟢 |

## 연관 entity
- [[entities/MCStaticMeshNiagaraSpawnerComponent]] — 동등 동작의 별도 Spawner. Soft 컴포넌트 사용 시 본 페어 대체 가능.
- [[entities/MCNiagaraSocketBindings]] — 본 컴포넌트가 메시 로드 후 추출 + Niagara 비동기 로드 + spawn 하는 메타.
- MCNiagaraSocketBindingHelpers (미인덱싱 — 헬퍼).
