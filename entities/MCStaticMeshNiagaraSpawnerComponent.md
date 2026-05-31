---
title: "UMCStaticMeshNiagaraSpawnerComponent"
kind: entity
category: Component
base_class: UActorComponent
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/Actor/Component/MCStaticMeshNiagaraSpawnerComponent.h
  - KMCProject/MCPlayModule/Actor/Component/MCStaticMeshNiagaraSpawnerComponent.cpp
vault_refs:
  - concepts/Component-Policies-6
  - concepts/Asset-Loading-Policy
  - concepts/Profiling-Scope-Rule
  - concepts/MC-Asset-Validation-Policy
  - sources/ue-niagara-skill
policy_refs:
  - component-policies
  - profiling-scope-rule
  - asset-loading-policy
  - asset-optimization-policy
last_ingested: 2026-05-29
---

# UMCStaticMeshNiagaraSpawnerComponent

## 한 줄 정의
StaticMesh / SkeletalMesh 의 socket 에 부착된 `UMCNiagaraSocketBindings`(UAssetUserData) 를 읽고 런타임에 `SpawnSystemAttached` 처리. 실제 spawn / load / cleanup 로직은 `MCNiagaraSocketBindingHelpers` 위임 — 본 컴포넌트는 *얇은 래퍼*.

## 소속 / 상속
- 대분류: Component
- `UCLASS(ClassGroup=(MC), meta=(BlueprintSpawnableComponent), Blueprintable)` `class MCPLAYMODULE_API UMCStaticMeshNiagaraSpawnerComponent : public UActorComponent`
- 매크로: `MCCOMPONENT_DEF(UMCStaticMeshNiagaraSpawnerComponent, EMCComponentType, EMCComponentType::MCStaticMeshNiagaraSpawner)`
- 정책 의무 (헤더 주석 명시):
  - 6대 Component 의무 (Mobility / NewObject / GC / GetOwner 캐싱 / Tick / CDO)
  - TSoftObjectPtr 비동기 로드 + Handle Pin
  - TRACE_CPUPROFILER_EVENT_SCOPE
  - MC_LOGRET_IF_NULL / silent return 금지
  - SpawnSystemAttached + ENCPoolMethod::AutoRelease

## 핵심 구조
- **UActorComponent 오버라이드**: `BeginPlay()`, `EndPlay(EEndPlayReason::Type)`.
- **UPROPERTY (동작 옵션)**:
  - `FName TargetMeshComponentName = NAME_None` — NAME_None 이면 첫번째 UMeshComponent 자손 (StaticMeshComponent 또는 SkeletalMeshComponent) 자동 선택. 명시 이름 시 `Owner->FindComponentByName` 매칭.
  - `int32 LoadPriority` (0~200) — FStreamableManager 우선순위.
- **BP 노출 델리게이트**: `FMCOnSocketBindingsSpawnedDelegate OnSocketBindingsSpawned`.
- **BP API**:
  - `DeactivateAllSpawnedNiagaras()` — 활성 Niagara Deactivate + 핸들 reset. 재 BeginPlay 시 다시 spawn.
  - `RespawnBindings()` — 외부 호출로 재시도 (메시 swap 후).
- **protected**: `UMeshComponent* FindTargetMeshComponent() const`, `RequestBindingsAsyncLoad()`, `HandleBindingsLoaded()`.
- **private 핸들·캐시**:
  - `TSharedPtr<FStreamableHandle> LoadHandle` — Pin 보관 의무.
  - `UPROPERTY(Transient) TObjectPtr<UMCNiagaraSocketBindings> CachedBindings` — BeginPlay 시 한 번 캡처, 메시가 도중에 swap 돼도 일관 처리.
  - `UPROPERTY(Transient) TObjectPtr<UMeshComponent> CachedTargetMesh` — Tick/콜백 안 매번 검색 회피.
  - `TArray<TWeakObjectPtr<UNiagaraComponent>> SpawnedNiagaras` — Pool AutoRelease 가 lifetime 관리.
  - `TWeakObjectPtr<AActor> CachedOwner`.

## 따르는 패턴
- 6대 Component 의무 → [[concepts/Component-Policies-6]]
- TSoftObjectPtr 비동기 로드 + Handle Pin → [[concepts/Asset-Loading-Policy]] §2 단계 5
- Profiling Scope (TRACE_CPUPROFILER_EVENT_SCOPE) → [[concepts/Profiling-Scope-Rule]]
- silent return 금지 / MC_LOGRET_IF_NULL → [[concepts/MC-Asset-Validation-Policy]]
- SpawnSystemAttached + ENCPoolMethod::AutoRelease → [[sources/ue-niagara-skill]]
- 동작 단계 (헬퍼 위임): BeginPlay → Owner UMeshComponent 찾기 → GetBindingsFromMesh → RequestLoad (한 핸들) → 콜백 → SpawnAllNow → EndPlay 시 Cancel + DeactivateAll.
- 디자이너 워크플로 (Editor): StaticMesh.uasset 안 UAssetUserData 로 binding 정의 → 본 컴포넌트가 자동으로 spawn (별도 BP/C++ 수정 불필요).

## ⚠ 함정
- **얇은 래퍼 — 동등 헬퍼 사용**: UMCNiagaraSocketPreviewSubsystem (Editor) 도 동일 헬퍼 사용 — DRY 의무. 헬퍼 변경 시 양쪽 영향.
- **Soft 컴포넌트 + 본 Spawner 동시 부착 race**: Soft 컴포넌트가 메시 로드 전이면 GetStaticMesh()==nullptr → AssetUserData 추출 실패. 본 Spawner 단독 사용 시 Owner 가 일반(non-Soft) StaticMeshComponent 라야 안전. UMCSoftStaticMeshComponent 사용 시 `bAutoSpawnSocketNiagara=true` 로 대체.
- **Pool AutoRelease 보관 = TWeak**: 별도 UPROPERTY 마커 불필요 (TWeakObjectPtr 자체가 GC-aware), 단 lifetime 은 Pool 이 결정.

## 횡단 정책 준수
> 적용: [[ue-cross-cutting-policies/index]] §3 (Component 행). raw 미마운트 — 추출 본문 근거, 미확인은 ❓.

| 정책 | 적용 | 근거 / 위반·미확인 | 신뢰도 |
|---|---|---|---|
| 11 asset-loading | ✅ | `TSoftObjectPtr` 비동기 + `LoadHandle` Pin + Transient TObjectPtr 캐시 + EndPlay Cancel = 정책 정합(헤더 명시). → [[ue-cross-cutting-policies/11_AssetLoadingPolicy]] | 🟢 |
| 12 asset-opt | ✅ | Niagara — **EffectType 지정 + Quality Scaling**(12 §5) + Pool `ENCPoolMethod::AutoRelease`(헤더 명시) 대상. EffectType 설정 ❓(자산 측). → [[ue-cross-cutting-policies/12_AssetOptimizationPolicy]] | 🟡 |
| 10 component | ✅ | CachedOwner TWeak, Transient TObjectPtr, Niagara TWeak(Pool 관리) = 6대 정합(헤더 명시). → [[ue-cross-cutting-policies/10_ComponentPolicies]] | 🟢 |
| 07 profiling | ✅ | 헤더 주석 `TRACE_CPUPROFILER_EVENT_SCOPE` 의무 명시(콜백). | 🟢 |
| 09 global-iterator | ➖ | 미사용. | 🟢 |

## 연관 entity
- [[entities/MCSoftStaticMeshComponent]] — 동등 동작의 자체 처리 경로 (Soft 페어).
- [[entities/MCNiagaraSocketBindings]] — 본 컴포넌트가 BeginPlay 시 추출하는 메타.
- [[entities/MCNiagaraSocketPreviewSubsystem]] — Editor 측 동일 헬퍼 재사용 (DRY 페어).
- MCNiagaraSocketBindingHelpers (미인덱싱).
