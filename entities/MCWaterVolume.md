---
title: "AMCWaterVolume"
kind: entity
category: Actor
base_class: APhysicsVolume
module: MCPLAYMODULE
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCPlayModule/Volume/MCWaterVolume.h
  - KMCProject/MCPlayModule/Volume/MCWaterVolume.cpp
vault_refs: []
last_ingested: 2026-05-29
---

# AMCWaterVolume

## 한 줄 정의
물(수면+수중) 볼륨 액터. `APhysicsVolume` + `IInterface_PostProcessVolume` — 수중 진입 시 PostProcess 적용 + Wave 시뮬레이션 파라미터 보유.

## 소속 / 상속
- 대분류: Actor
- `UCLASS()` `class AMCWaterVolume : public APhysicsVolume, public IInterface_PostProcessVolume`
- 다단 상속 동반: IInterface_PostProcessVolume (UE 엔진 표준 인터페이스)
- 함께 정의: `USTRUCT(BlueprintType) FWaveInfo` — Wave 시뮬 파라미터(DebugDraw, Param[], PlaneSize, SectionSize, WaterPlaneTransform) + IsEqual/IsValid 헬퍼.

## 핵심 구조
- **생성자**: `AMCWaterVolume(const FObjectInitializer& ObjectInitializer)`
- **UPROPERTY**:
  - `EditAnywhere BlueprintReadWrite FWaveInfo WaveData` — 파동 파라미터.
  - `interp Category=PostProcessVolumeSettings meta=(ShowPostProcessCategories) FPostProcessSettings UnderWaterSettings`
  - `EditAnywhere BlueprintReadWrite float Priority_UnderWater`
  - `interp ClampMin=0 UIMax=6000 float BlendRadius_UnderWater`
  - `interp BlueprintReadWrite UIMin=0 UIMax=1 float BlendWeight_UnderWater`
  - `EditAnywhere BlueprintReadWrite uint32 bEnabled_UnderWater :1`
  - `EditAnywhere BlueprintReadWrite uint32 bUnbound_UnderWater :1` — Infinite Extent (Unbound)
  - `EditAnywhere BlueprintReadWrite UMCWaterPlaneComponent* WaterPlaneComponent`
- **오버라이드**:
  - `virtual bool EncompassesPoint(FVector Point, float SphereRadius, float* OutDistanceToPoint) override`
  - `virtual FPostProcessVolumeProperties GetProperties() const override` — `bEnabled_UnderWater` / `bUnbound_UnderWater` / `BlendRadius_UnderWater` / `BlendWeight_UnderWater` / `Priority_UnderWater` / `&UnderWaterSettings` 패킹.
  - `virtual void Tick(float DeltaSeconds) override`
- **protected 오버라이드**:
  - `virtual void PostRegisterAllComponents() override`
  - `virtual void PostUnregisterAllComponents() override`
  - **`#if WITH_EDITOR`**: `virtual void PostEditChangeProperty(FPropertyChangedEvent&) override`
- **`#if DEBUG_POST_PROCESS_VOLUME_ENABLE`**: `virtual FString GetDebugName() const override { return GetName(); }`
- **API**:
  - `float GetSurfaceZ()`
  - `float GetHeight()`
  - `FVector GetWaterLocation(const FVector& _location)`
- **FWaveInfo 헬퍼**: `IsEqual(target, tolerance=100)`, `IsValid()`, `operator==(FVector)`.

## 따르는 패턴
- APhysicsVolume + IInterface_PostProcessVolume 결합 = 영역 진입 감지 + PostProcess 자동 블렌딩 (🟢 — 베이스 선언 + GetProperties() 구현으로 확정).
- WaterPlaneComponent 위임 — 시각화·메시 처리는 UMCWaterPlaneComponent (🟢 — UPROPERTY 멤버로 확정).

## ⚠ 함정
- 본 클래스에 명시 주석 없음 — Wave 시뮬 알고리즘·Bouyancy 컴포넌트와의 상호작용 의도는 .cpp 또는 UMCBouyancyComponent 측에서 확인 필요 (🟡).
- `bUnbound_UnderWater = true` 시 EncompassesPoint 가 항상 true 가 될 가능성 — 무한 영역(Infinite Extent). 디자이너 의도 외 활성화 주의.
- PostProcess 우선순위 충돌 — 같은 레벨에 다른 APostProcessVolume 들과 `Priority_UnderWater` 충돌 시 의도와 다른 블렌딩 가능.

## 연관 entity
- [[entities/MCWaterPlaneComponent]] — 시각화 컴포넌트.
- [[entities/MCBouyancyComponent]] — 부력 계산 컴포넌트 (Water plane 과 상호작용).
- FWaveInfo / FWaveParam (KMCProject/MCPlayModule/Core/MCCoreStruct.h — 미인덱싱).
- [[entities/MCWaterBlueprintLibrary]] — 파동 계산 BP 헬퍼.
