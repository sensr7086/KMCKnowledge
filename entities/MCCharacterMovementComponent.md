---
title: "UMCCharacterMovementComponent"
kind: entity
category: Component
base_class: UCharacterMovementComponent
module: MCPLAYMODULE
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCPlayModule/Actor/Component/MCCharacterMoveComponent.h
  - KMCProject/MCPlayModule/Actor/Component/MCCharacterMoveComponent.cpp
vault_refs: []
policy_refs:
  - component-policies
  - profiling-scope-rule
last_ingested: 2026-05-29
---

# UMCCharacterMovementComponent

## 한 줄 정의
UCharacterMovementComponent 확장 — Custom Movement Mode 로 Climbing 추가, OnLanding 델리게이트 노출.

## 소속 / 상속
- 대분류: Component
- `UCLASS(meta = (BlueprintSpawnableComponent))` `class MCPLAYMODULE_API UMCCharacterMovementComponent : public UCharacterMovementComponent`
- 함께 정의: `UENUM(BlueprintType) enum class EMCCustomMovementMode : uint8 { CUSTOM_NONE, CUSTOM_CLIMB, CUSTOM_MAX }`.

## 핵심 구조
- **UCharacterMovementComponent 오버라이드**:
  - `virtual void InitializeComponent() override`
  - `virtual void PhysCustom(float deltaTime, int32 interations) override`
  - `virtual void CalcVelocity(float, float, bool, float) override`
  - `virtual float GetMaxSpeed() const override`
  - `virtual void BeginDestroy() override`
  - `virtual void ProcessLanded(const FHitResult& Hit, float remainingTime, int32 Iterations) override`
- **OnLanding 델리게이트**:
  - `FLanding& GetLanding() { return OnLandingEvent; }`
  - protected: `FLanding OnLandingEvent`
- **Climb 파라미터 (UPROPERTY EditAnywhere BlueprintReadWrite, Category="Climb")**:
  - `float CollsionCapsuleRadius = 50.0f` (sic Collsion)
  - `float CollsionCapsuleHalfHeight = 70.0f`
  - `float ClimbMaxSpeed = 120.0f`, `float ClimbingSnapSpeed = 15.0f`, `float ClimbDistFormSurface = 5.0f` (sic Form), `float ClimbingRotateSpeed = 3.0f`
  - `float ClimbingCollisionShrinkAmount = 5.0f`, `float FloorCheckDistance = 90.0f`
  - `float MinHorizontalDegreesToStartClimb = 25.0f`, `float WallDeteatDistance = 150.0f` (sic Deteat)
- **Climb 상태**:
  - `TArray<FHitResult> CurrentClimbWalls`
  - `FCollisionQueryParams ClimbQueryParam`
  - `bool IsWantToClimb`, `bool IsClimbDash`, `bool IsHitDirectionMove`, `bool IsForceOverlabFall` (sic Overlap)
  - `FVector ClimbVelocityDirection / CurrentClimbingNormal / CurrentClimbingPosition / CurrentClimbSweepNormal / CurrentClimbAmount / ClimbMoveDirection`
  - `EMCCustomMovementMode CustomMovementFlag = EMCCustomMovementMode::CUSTOM_NONE`
- **public Climb API**:
  - `void AWakeClimb()` (sic Awake)
  - `bool IsClimbing() const`, `bool IsClimbingDash()`
  - `void CancleClimbing()` (sic Cancle)
  - `void SetCustomMovement(EMovementMode, EMCCustomMovementMode)`
  - `void SetClimbVelocity(const FVector&)`, `void SetClimbMove(const FVector&)`
- **protected Climb 구현**:
  - `void PhysClimb(float dt, int32 iteraction)`
  - 정찰 / 표면 계산: `SweepAndLevelDetect()`, `ComputeClimbingSurface(float dt)`, `CanFacingSurface(float Steep) const`, `HeadHeightTrace(float dist) const`
  - 레지 처리: `TryClimbUpLegde(FVector& end_point, bool force_ledge=false) const` (sic Legde), `CanClimbLedge`, `HasClimbReachedEdge`, `CanMoveToLedgeClimbLocation`, `CanMoveToLedgeClimbLocationRecursive`, `IsClimbEndLocationWalkable`
  - 종료 판정: `ShouldStopClimb() const`, `ClimbFloor() const`, `CheckClimbFloor(FHitResult&) const`
  - 회전·이동·스냅: `GetClimbRotation(float dt) const`, `MoveAlongClimbSurface(float dt)`, `SnapToClimbSurface(float dt)`

## 따르는 패턴
- UE 표준 Custom Movement Mode 확장 — `PhysCustom` override 로 사용자 정의 물리 단계 삽입 (🟢 — override 선언).
- `ProcessLanded` override + `OnLandingEvent` 브로드캐스트 → 외부 UMCMoveComponent 가 Land() 트리거에 구독 (🟡 — 페어 추론).
- Climb 파라미터 전부 EditAnywhere → 디자이너 튜닝 가능.

## ⚠ 함정
- 명칭 오타 다수: `Collsion`, `Form`, `Deteat`, `Overlab`, `AWake`, `Cancle`, `Legde`, `iteraction` — BP 노출 시 redirector 필요할 수 있음. 향후 정리 시 일괄 변경 + redirector 등록.
- `PhysCustom` 단계 안에서 `ProcessLanded` 호출 race 가능 (UE 표준 함정 — 🟡 추론).
- Climb 상태 변수가 매우 많아 `CancleClimbing` 시점에 모두 리셋 안 하면 stale 상태 진입 가능.

## 횡단 정책 준수
> 적용: [[ue-cross-cutting-policies/index]] §3 (Component 행). raw 미마운트 — 추출 본문 근거, 미확인은 ❓.

| 정책 | 적용 | 근거 / 위반·미확인 | 신뢰도 |
|---|---|---|---|
| 10 component | ✅ | `UCharacterMovementComponent` 자손 — GC/CDO 준용. Climb 상태 다수 = 값 타입(GC 부담 낮음). | 🟡 |
| 07 profiling | ✅ | `PhysCustom`/`PhysClimb` 매 프레임 물리 — 스코프 유무 ❓. → [[ue-cross-cutting-policies/07_ProfilingScopeRule]] | ❓ |
| 11 asset-loading | ➖ | asset 멤버 없음. | 🟢 |
| 12 asset-opt | ➖ | 자산 미소유. | 🟢 |
| 09 global-iterator | ➖ | 미사용. | 🟢 |

> 네트워크 예측(Replicated Movement) 정합성은 [[ue-cross-cutting-policies/16_PolicyPriority]] Tier3(네트워크) 영역 — entity 정책 외.

## 연관 entity
- [[entities/MCMoveComponent]] — GetMovement() 페어, OnLandingEvent 구독자.
- [[entities/MCCharacter]] — Owner Character (CharacterMovement 표준).
- FLanding 델리게이트 (MCDelegate — 미인덱싱).
- EMCCustomMovementMode (본 파일 정의).
