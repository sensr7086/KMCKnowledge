---
title: "Actor 대분류"
kind: catalog
category: Actor
last_ingested: 2026-05-29
---

# Actor 대분류

> KMCProject 의 `A...` 액터 (ACharacter / AActor / APawn / ACameraActor / APhysicsVolume 자손) + Actor 보조 UObject (Camera Mode 등). Subsystem 은 [[catalogs/subsystem]], 인터페이스는 [[catalogs/interface]] 참조.

## 베이스
- 🟢 [[entities/MCPooledActor]] — `AMCPooledActor : AActor, IMCPoolableInterface`. 풀링 표준 베이스. Tick OFF / Hidden / NoCollision 으로 시작.

## 구현체
- 🟡 [[entities/MCCharacter]] — `ACharacter, IMCActorInterface`. 메인 게임플레이 캐릭터 (MCPlayModule).
- 🟡 [[entities/MCCamera]] — `ACameraActor`. 카메라 액터 placeholder.
- 🟡 [[entities/MCWaterVolume]] — `APhysicsVolume, IInterface_PostProcessVolume`. 물 볼륨 + PostProcess.
- 🟢 [[entities/KMCProjectCharacter]] — `ACharacter`. UE 템플릿 기본 (MCProjectModule). SpringArm + FollowCamera + Enhanced Input.
- 🟡 [[entities/KMCProjectGameMode]] — `AGameModeBase`. UE 템플릿 기본 (MCProjectModule, MinimalAPI).

## 보조 객체 (Camera mode UObject — `UCLASS(Abstract)` 베이스 + 자손)
- 🟡 [[entities/MCCameraBase]] — `UCLASS(Abstract) : UObject`. Start/Update/Finish + UpdateTransform/UpdateCollision hook.
- 🟡 [[entities/MCCameraPlayable]] — `UMCCameraBase` 자손. ViewTarget weak 추적 + Angle/Distance third-person 위치.

## 페어 카탈로그
- [[catalogs/subsystem]] — Actor 관리자 (UMCActorSpawnSubsystem, UMCCameraSubSystem).
- [[catalogs/interface]] — Actor 가 implements 하는 인터페이스 (IMCActorInterface, IMCPoolableInterface, IMCSpatialQueryFilterable).

## 공통 함정
- **인터페이스 다단 상속 동반**: 구현체가 인터페이스 함께 구현 (IMCActorInterface / IMCPoolableInterface / IInterface_PostProcessVolume).
- **얇은 헤더의 함정**: AMCCharacter / AMCCamera / UMCCameraBase / UMCCameraPlayable 는 헤더 주석 거의 없어 confidence 🟡.
- **풀링 식별**: `IsValid(Actor)` 만으로 풀 상태 판별 불가 — 명시 플래그 `bIsPoolActive` 의무.
- **Camera mode 의 `MCPLAYMODULE_API` 누락**: 모듈 boundary 넘는 직접 참조 시 link error 가능.
- **Update 의 내부 분기 순서 강제**: UMCCameraBase::Update → UpdateTransform → UpdateCollision. 자손이 Update 자체 override 시 순서 깨질 가능성.
