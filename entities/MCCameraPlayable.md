---
title: "UMCCameraPlayable"
kind: entity
category: Actor
base_class: UMCCameraBase
module: MCPLAYMODULE
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCPlayModule/Actor/Camera/MCCameraMode.h
  - KMCProject/MCPlayModule/Actor/Camera/MCCameraMode.cpp
vault_refs: []
last_ingested: 2026-05-29
---

# UMCCameraPlayable

## 한 줄 정의
플레이어 카메라 모드 — `UMCCameraBase` 의 자손. ViewTarget 을 weak 보관 + 거리/각도 파라미터로 third-person 위치 계산 (🟡 추론).

## 소속 / 상속
- 대분류: Actor (베이스 [[entities/MCCameraBase]] 의 분류 승계)
- `UCLASS()` `class UMCCameraPlayable : public UMCCameraBase`
- UMCCameraSubSystem 의 `EMCCameraMode::Playable` 키에 대응하는 인스턴스.

## 핵심 구조
- **베이스 hook 오버라이드**:
  - `virtual void Start() override`
  - `virtual void Update(float _dt) override`
  - `virtual void Finish() override`
  - `virtual void SetViewTarget(AActor* pTarget) override`
- **protected 베이스 hook 오버라이드**:
  - `virtual void UpdateTransform(float _dt) override`
  - `virtual void UpdateCollision(float _dt) override`
- **protected 상태**:
  - `TWeakObjectPtr<AActor> ViewTarget` — 따라다닐 액터 (Weak — 액터 GC 안전).
  - `FVector Angle = FVector::ZeroVector` — 카메라 각도 (Pitch/Yaw/Roll 추정, 🟡 — 변수명만 — Rotation 의도일 가능성).
  - `float Distance = 500.0f` — ViewTarget 으로부터의 거리 (cm 단위 추정).

## 따르는 패턴
- 베이스 [[entities/MCCameraBase]] 의 `Update → UpdateTransform + UpdateCollision` 분리 패턴 승계 (🟢 — override 선언).
- ViewTarget Weak 보관 — 카메라가 추적 대상의 라이프사이클을 잡지 않음 (🟢 — TWeakObjectPtr 명시).
- 추정 동작: Update 시 ViewTarget 위치 + Angle 회전 + Distance 거리 로 CameraTransform 계산. UpdateCollision 에서 wall sweep / occlusion 처리 (🟡 추론 — .cpp 미확인).

## ⚠ 함정
- **헤더 주석 부재** — 설계 결정·알고리즘 명시 없음. confidence 🟡 사유.
- **`Angle` 의 타입이 `FVector`** — FRotator 가 표현상 더 명확. Pitch/Yaw/Roll 매핑 규칙 불명확.
- **ViewTarget 이 GC 되면**: TWeakObjectPtr → nullptr. Update 본문에서 IsValid 가드 의무 (🟡 추론 — .cpp 측 확인 필요).
- **Distance default 500.0f**: 단위·범위 명시 없음. ClampMin / UIMin 메타 없음.
- **`UCLASS()` 빈 specifier**: BlueprintType / Blueprintable 없음 — BP 에서 직접 변수 노출 불가, 자손 정의 불가 (🟡 추론).

## 연관 entity
- [[entities/MCCameraBase]] — 직접 베이스.
- [[entities/MCCameraSubSystem]] — `TMap<EMCCameraMode::Playable, TStrongObjectPtr<UMCCameraBase>>` 의 값으로 보관.
- [[entities/MCCamera]] — Subsystem 이 등록하는 ViewTarget 후보 (AMCCamera 가 카메라 액터지만 본 모드의 ViewTarget 은 *추적 대상* — 캐릭터일 가능성 더 큼, 🟡).
