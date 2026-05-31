---
title: "UMCCameraBase"
kind: entity
category: Actor
base_class: UObject
module: MCPLAYMODULE
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCPlayModule/Actor/Camera/MCCameraMode.h
  - KMCProject/MCPlayModule/Actor/Camera/MCCameraMode.cpp
vault_refs: []
last_ingested: 2026-05-29
---

# UMCCameraBase

## 한 줄 정의
카메라 모드의 추상 베이스 UObject — `Start / Update / Finish` + `UpdateTransform / UpdateCollision` 가상 hook. UMCCameraSubSystem 의 `TMap<EMCCameraMode, TStrongObjectPtr<UMCCameraBase>>` 의 베이스.

## 소속 / 상속
- 대분류: Actor (역할 기준 — 카메라 액터·모드 관리 보조)
- `UCLASS(Abstract)` `class UMCCameraBase : public UObject` (모듈 API 표기 없음)
- 라이프사이클: UMCCameraSubSystem 이 모드 보관 → SetCameraMode(EMCCameraMode) 시 Start / Tick 시 Update / 모드 전환 시 Finish.

## 핵심 구조
- **추상 가상 hook (default = no-op, 자손 override)**:
  - `virtual void Start() {}` — 모드 진입.
  - `virtual void SetViewTarget(AActor* pTarget) {}` — ViewTarget 등록.
  - `virtual void Update(float _dt)` — 내부 호출 `UpdateTransform(_dt)` + `UpdateCollision(_dt)`.
  - `virtual void Finish() {}` — 모드 종료.
- **API**: `FTransform GetTransformUpdate() { return CameraTransform; }` — 갱신된 Transform 조회.
- **protected hook**:
  - `virtual void UpdateTransform(float _dt) {}` — 위치/회전 갱신.
  - `virtual void UpdateCollision(float _dt) {}` — 충돌 처리.
- **protected 상태**: `FTransform CameraTransform`.

## 따르는 패턴
- Abstract base + 가상 hook 패턴 — 자손이 override 로 구체 구현.
- Update 내부 `UpdateTransform` + `UpdateCollision` 분리 — 위치/회전 갱신과 충돌 처리의 책임 분리.
- UMCCameraSubSystem 의 `TStrongObjectPtr<UMCCameraBase>` 보관 — UPROPERTY 가 아님 (Subsystem 자체 GC 추적). 단일 인스턴스 모드별 캐싱.

## ⚠ 함정
- **헤더 주석 부재** — 설계 결정·vault 링크 0개. confidence 🟡 사유.
- **`UCLASS(Abstract)` 만 — 모듈 API 매크로 (`MCPLAYMODULE_API`) 없음**: 외부 모듈(MCEditorModule) 에서 본 클래스 직접 참조 시 link error 가능 (🟡 추론).
- **Update 의 내부 분기 — `UpdateTransform → UpdateCollision` 순서 강제**: 충돌 처리가 Transform 갱신 후 발화. 자손이 Update 자체 override 시 순서 깨질 가능성.
- **CameraTransform 갱신 책임**: 자손의 `UpdateTransform` 안에서 명시 mutate 의무. default no-op 이므로 자손 override 안 하면 Transform 변화 없음.

## 연관 entity
- [[entities/MCCameraSubSystem]] — 본 베이스의 호스트 (TMap value).
- [[entities/MCCameraPlayable]] — 직접 자손.
- [[entities/MCCamera]] — MainCamera 액터 (Subsystem 의 ViewTarget 대상).
