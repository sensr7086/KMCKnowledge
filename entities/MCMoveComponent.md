---
title: "UMCMoveComponent"
kind: entity
category: Component
base_class: UMCActorComponent
module: MCPLAYMODULE
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCPlayModule/Actor/Component/MCMoveComponent.h
  - KMCProject/MCPlayModule/Actor/Component/MCMoveComponent.cpp
vault_refs: []
policy_refs:
  - component-policies
  - profiling-scope-rule
last_ingested: 2026-05-29
---

# UMCMoveComponent

## 한 줄 정의
이동 상태 머신 컴포넌트 — Move Mode/Speed/Flag 를 enum 으로 보관하고 변화 시 델리게이트 브로드캐스트.

## 소속 / 상속
- 대분류: Component
- `UCLASS(Blueprintable, meta = (BlueprintSpawnableComponent))` `class MCPLAYMODULE_API UMCMoveComponent : public UMCActorComponent`
- 매크로: `MCCOMPONENT_DEF(UMCMoveComponent, EMCComponentType, EMCComponentType::MCMovemoent)` (오타: Movemoent).

## 핵심 구조
- **UMCActorComponent hook 오버라이드**:
  - `virtual void Init() override`
  - `virtual void Update(float delta) override`
  - `virtual void End() override`
- **상태 갱신**:
  - `void UpdateMoveMode()`
  - `void UpdateMoveSpeed()`
  - `void Land()` — 착지 처리.
  - `void LuanchCharacter()` — (sic) Launch Character.
- **상태 setter / 델리게이트 핸들**:
  - `void SetMoveSpeed(EMCMoveSpeed)`, `void SetMoveMode(EMCMoveMode)`
  - `void AddMoveFlag(EMCMoveFlag)`, `void RemoveMoveFlag(EMCMoveFlag)`, `bool HasMoveFlag(EMCMoveFlag)`, `void SetMoveFlag(EMCMoveFlag)`
  - `EMCMoveMode GetMoveMode() { return Movemode; }`
  - `FOnMoveMode& GetMoveModeEvent()`, `FOnMoveSpeed& GetMoveSpeedEvent()`, `FOnMoveFlags& GetMoveFlagEvent()`
- **UMCCharacterMovementComponent 접근**: `UMCCharacterMovementComponent* GetMovement()`.
- **protected 상태**:
  - `EMCMoveMode Movemode = EMCMoveMode::Ground`
  - `EMCMoveSpeed MoveSpeed = EMCMoveSpeed::Walk`
  - `EMCMoveFlag MoveFlag = EMCMoveFlag::None`
  - `FOnMoveMode OnMovemode`, `FOnMoveSpeed OnMoveSpeed`, `FOnMoveFlags OnMoveFlag`
- **protected 핸들**: `FDelegateHandle LandingHandle` — UMCCharacterMovementComponent::GetLanding() 구독 핸들 (🟡 추론).

## 따르는 패턴
- 상태 머신 + 변화 알림 (FOnXxx 델리게이트) — 외부 UI/Anim 시스템이 폴링 없이 상태 변화 수신 (🟡 추론).
- UMCCharacterMovementComponent 와의 페어 — Get 접근으로 캡슐화 (🟢 — API 선언).
- LandingHandle 보관 → UMCCharacterMovementComponent::OnLandingEvent 구독 패턴 (🟡 추론 — Delegate Handle 멤버 존재).

## ⚠ 함정
- 명칭 오타 누적: `Movemoent`, `Luanch`, `Movemode` — UFUNCTION 으로 BP 노출 시 redirector 필요할 수 있음.
- FDelegateHandle 의 unbind 누락 → End() 단계에서 명시 해제 안 하면 stale callback 가능 (🟡 추론).

## 횡단 정책 준수
> 적용: [[ue-cross-cutting-policies/index]] §3 (Component 행). raw 미마운트 — 추출 본문 근거, 미확인은 ❓.

| 정책 | 적용 | 근거 / 위반·미확인 | 신뢰도 |
|---|---|---|---|
| 10 component | ✅ | 베이스 [[entities/MCActorComponent]] 6대 적용. 델리게이트 멤버(OnMovemode 등) GC: UPROPERTY 마커 ❓. | 🟡 |
| 07 profiling | ✅ | `Update(delta)` 경유 매 프레임 상태 갱신 — 스코프 유무 ❓. → [[ue-cross-cutting-policies/07_ProfilingScopeRule]] | ❓ |
| 11 asset-loading | ➖ | asset 멤버 없음(enum/델리게이트). | 🟢 |
| 12 asset-opt | ➖ | 자산 미소유. | 🟢 |
| 09 global-iterator | ➖ | 미사용. | 🟢 |

## 연관 entity
- [[entities/MCActorComponent]] — 직접 베이스.
- [[entities/MCCharacterMovementComponent]] — GetMovement() 페어.
- [[entities/MCCharacter]] — Owner Character.
- EMCMoveMode/EMCMoveSpeed/EMCMoveFlag (PlayEnum — 미인덱싱).
- FOnMoveMode/FOnMoveSpeed/FOnMoveFlags (MCDelegate — 미인덱싱).
