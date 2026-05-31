---
title: "UMCCameraSubSystem"
kind: entity
category: Subsystem
base_class: UTickableWorldSubsystem
module: MCPLAYMODULE
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCPlayModule/Actor/Camera/MCCameraSubSystem.h
  - KMCProject/MCPlayModule/Actor/Camera/MCCameraSubSystem.cpp
vault_refs: []
last_ingested: 2026-05-29
---

# UMCCameraSubSystem

## 한 줄 정의
카메라 모드 호스트 World Subsystem — `EMCCameraMode → UMCCameraBase` 맵으로 카메라 모드 인스턴스 보관 + 활성 `AMCCamera` 추적 + Transform 쿼리 API.

## 소속 / 상속
- 대분류: Actor (역할 기준 — 카메라 액터·모드 관리. ViewTarget 변경 등 액터 라이프사이클 영역.)
- `UCLASS()` `class MCPLAYMODULE_API UMCCameraSubSystem : public UTickableWorldSubsystem`
- 함께 정의: `UENUM(BlueprintType) enum class EMCCameraMode : uint8 { None, Playable }`.

## 핵심 구조
- **USubsystem Interface**:
  - `virtual void Initialize(FSubsystemCollectionBase&) override`
  - `virtual void Deinitialize() override`
  - `virtual void BeginDestroy() override`
- **Tickable**:
  - `virtual void Tick(float DeltaTime) override`
  - `virtual TStatId GetStatId() const override`
  - `virtual bool IsTickable() const override { return true; }`
- **BP API**:
  - `UFUNCTION(BlueprintCallable) void SetCameraMode(EMCCameraMode)` — 활성 모드 전환.
  - `UFUNCTION(BlueprintCallable) void AddMainCamera(AActor* pViewTarget)` — MainCamera 등록 (이름은 Add 지만 단일 슬롯, 🟡).
  - `UFUNCTION(BlueprintPure) FTransform GetCameraTransform(EMCCameraMode)` — 모드별 Transform 쿼리.
  - `UFUNCTION(BlueprintPure) FTransform GetCurrentCameraTransform()` — 현재 활성 모드의 Transform.
- **protected 상태**:
  - `TMap<EMCCameraMode, TStrongObjectPtr<UMCCameraBase>> CameraMode` — 모드별 카메라 인스턴스. **Strong** ref (TStrongObjectPtr) — 자산 GC 방지.
  - `TWeakObjectPtr<AMCCamera> MainCamera` — 활성 카메라 액터 (Weak 보관).
  - `EMCCameraMode CurrentMode = EMCCameraMode::None`.

## 따르는 패턴
- 모드 enum + 모드별 인스턴스 컬렉션 — 전형적 카메라 모드 매니저.
- TStrongObjectPtr 사용 — UPROPERTY 가 아닌 GC-aware Strong reference. Subsystem 자체가 GC 추적되므로 멤버 TStrongObjectPtr 가 안전 (🟡 추론 — UPROPERTY 보다 우회적, 의도 불명확).
- AddMainCamera 가 `AActor*` 받고 내부에서 AMCCamera 만 보관 — Cast<AMCCamera> 후 MainCamera 갱신 가정 (🟡 — .cpp 미확인).

## ⚠ 함정
- **헤더 주석 부재** — 모드 전환 정책·MainCamera 등록 정책이 헤더에 명시 안 됨. confidence 🟡 사유.
- **TStrongObjectPtr vs UPROPERTY**: UPROPERTY 가 아닌 TStrongObjectPtr 멤버 — GC 마커로 동작하지만 reflection 시스템에 노출 안 됨. 디버그 Details 패널에서 보이지 않고, BP 노출 불가. 의도된 선택인지 .cpp/사용처에서 확인 필요 (🟡).
- **`AddMainCamera` 이름의 의미 모호** — 단일 슬롯 갱신이라면 `SetMainCamera` 가 의도 명확.
- **`IsTickable() const { return true; }` 무조건**: CurrentMode==None 일 때도 Tick 발화 — Tick 본체에서 가드 의무.
- **PIE 다중 인스턴스** — World 별 별도 Subsystem.

## 연관 entity
- [[entities/MCCamera]] — MainCamera 의 type (AMCCamera).
- [[entities/MCCameraBase]] — TMap value 베이스 (Strong 보관).
- [[entities/MCCameraPlayable]] — `EMCCameraMode::Playable` 인스턴스.
- EMCCameraMode (본 파일 정의).
