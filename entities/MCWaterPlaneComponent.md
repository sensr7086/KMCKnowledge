---
title: "UMCWaterPlaneComponent"
kind: entity
category: Component
base_class: UMeshComponent
module: MCPLAYMODULE
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCPlayModule/Actor/Component/MCWaterPlaneComponent.h
  - KMCProject/MCPlayModule/Actor/Component/MCWaterPlaneComponent.cpp
vault_refs: []
policy_refs:
  - component-policies
  - profiling-scope-rule
  - asset-loading-policy
last_ingested: 2026-05-29
---

# UMCWaterPlaneComponent

## 한 줄 정의
물 수면 시각화 컴포넌트 — `UMeshComponent` 자손으로 자체 `FPrimitiveSceneProxy` / Bounds / Material 관리. AMCWaterVolume 에 부착되어 수면을 그리는 역할.

## 소속 / 상속
- 대분류: Component
- `UCLASS(Blueprintable, meta = (BlueprintSpawnableComponent))` `class MCPLAYMODULE_API UMCWaterPlaneComponent : public UMeshComponent`
- 매크로: `MCCOMPONENT_DEF(UMCWaterPlaneComponent, EMCComponentType, EMCComponentType::WaterPlane)`

## 핵심 구조
- **생성자**: `UMCWaterPlaneComponent(const FObjectInitializer&)`.
- **UActorComponent 오버라이드**:
  - `virtual void BeginPlay() override`
  - `virtual void TickComponent(float, ELevelTick, FActorComponentTickFunction*) override`
- **PrimitiveComponent / MeshComponent 오버라이드**:
  - `virtual FPrimitiveSceneProxy* CreateSceneProxy() override` — 자체 SceneProxy.
  - `virtual FBoxSphereBounds CalcBounds(const FTransform& LocalToWorld) const override`
  - `virtual int32 GetNumMaterials() const override`
  - `void GetUsedMaterials(TArray<UMaterialInterface*>& Out, bool bGetDebugMaterials=false) const override`
  - `virtual FMaterialRelevance GetMaterialRelevance(ERHIFeatureLevel::Type) const override`
- **API**: `float GetPlaneSize() const`, `float GetSectionSize() const`.
- **UPROPERTY**:
  - `Transient TArray<UPrimitiveComponent*> AdjustBouyancy` — 등록된 부력 대상.
  - `EditAnywhere BlueprintReadWrite TObjectPtr<UMaterialInterface> Material`
  - `EditAnywhere BlueprintReadWrite float PlaneSize = 100.0f`
  - `EditAnywhere BlueprintReadWrite float SectionSize = 32.0f`

## 따르는 패턴
- 자체 SceneProxy + CalcBounds — UMeshComponent 표준 확장 패턴 (🟢 — override 선언).
- AdjustBouyancy 멤버 → UMCBouyancyComponent 와의 페어 관계 (🟡 추론 — 이름 매칭).
- Plane + Section 분할 — 메시를 격자로 나눠 wave 시뮬레이션 적용 가능 (🟡 추론 — PlaneSize / SectionSize UPROPERTY).

## ⚠ 함정
- 본 클래스에 명시 주석 없음 — SceneProxy 의 구체 구현 의도(파동·셰이더 입력)는 .cpp 또는 머티리얼 측에서 확인 필요 (🟡).
- `AdjustBouyancy` 가 raw `UPrimitiveComponent*` TArray — UMCBouyancyComponent 의 동명 멤버와 혼동 주의. UPROPERTY(Transient) 가 GC 마커 역할.
- 일부 한국어 주석 CP949 mojibake — 임의 재인코딩 금지.

## 횡단 정책 준수
> 적용: [[ue-cross-cutting-policies/index]] §3 (Component 행). raw 미마운트 — 추출 본문 근거, 미확인은 ❓.

| 정책 | 적용 | 근거 / 위반·미확인 | 신뢰도 |
|---|---|---|---|
| 10 component | ✅ | `UMeshComponent` 자손 — 커스텀 SceneProxy/Bounds. GC: `Material` TObjectPtr, `AdjustBouyancy` UPROPERTY(Transient) 마커 ✓. Mobility ❓. | 🟡 |
| 07 profiling | ✅ | `TickComponent` 보유 — 매 프레임 수면 갱신 시 첫 줄 스코프 의무. 스코프 유무 ❓. → [[ue-cross-cutting-policies/07_ProfilingScopeRule]] | ❓ |
| 11 asset-loading | △ | `Material`(TObjectPtr) = **Hard 참조**(물 셰이더). Soft 전환 검토 후보. → [[ue-cross-cutting-policies/11_AssetLoadingPolicy]] | 🟡 |
| 12 asset-opt | ➖ | 수면=절차적 그리드(자산 LOD 아님). 그리드 해상도 성능은 자체 튜닝. | 🟡 |
| 09 global-iterator | ➖ | 미사용. | 🟢 |

> SceneProxy 렌더 스레드 안전(게임스레드 데이터 직접 접근 금지)은 렌더 규약 — entity 정책 외.

## 연관 entity
- [[entities/MCWaterVolume]] — Owner Actor, WaterPlaneComponent UPROPERTY 로 본 컴포넌트 보관.
- [[entities/MCBouyancyComponent]] — AdjustBouyancy 페어 (🟡 추론).
