---
title: "UMCAnimInstance"
kind: entity
category: Component
base_class: UAnimInstance
module: MCPLAYMODULE
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCPlayModule/Actor/Character/MCAnimInstance.h
  - KMCProject/MCPlayModule/Actor/Character/MCAnimInstance.cpp
vault_refs: []
policy_refs:
  - profiling-scope-rule
last_ingested: 2026-05-29
---

# UMCAnimInstance

## 한 줄 정의
KMCProject 의 캐릭터 AnimInstance 베이스 — UMCMoveComponent 의 Move Mode/Speed/Flag 델리게이트를 구독해 4개 BP 노출 상태 비트(Run/Walk/Air/Jump) 를 갱신. AnimGraph 가 이 비트를 읽어 상태 전이.

## 소속 / 상속
- 대분류: Component (UAnimInstance 자손 — SkeletalMesh 의 animation runtime)
- `UCLASS(Blueprintable, BlueprintType)` `class UMCAnimInstance : public UAnimInstance` (모듈 API 표기 없음)
- 라이프사이클: USkeletalMeshComponent 가 instantiate. NativeBeginPlay/Update/Uninitialize 표준 UAnimInstance.

## 핵심 구조
- **UAnimInstance 오버라이드**:
  - `virtual void NativeUninitializeAnimation() override`
  - `virtual void NativeBeginPlay() override`
  - `virtual void NativeUpdateAnimation(float DeltaSeconds) override`
  - `virtual void BeginDestroy() override`
- **Move 델리게이트 바인딩**:
  - `void BindMoveEvent(FOnMoveMode& _mode, FOnMoveSpeed& _speed, FOnMoveFlags _flag)` — UMCMoveComponent 의 3 델리게이트 구독. (3번째 인자 `FOnMoveFlags _flag` 가 reference 아님 — 🟡 일관성 미흡)
- **protected 콜백 (자손 override 가능)**:
  - `virtual void ChangeMoveMode(EMCMoveMode _mode)`
  - `virtual void ChangeMoveSpeed(EMCMoveSpeed _speed)`
  - `virtual void ChangeMoveFlag(EMCMoveFlag _flag)`
- **public 델리게이트 핸들**:
  - `FDelegateHandle MoveModeHandle`
  - `FDelegateHandle MoveSpeedandle` (sic — Speed_handle 의 오타)
  - `FDelegateHandle MoveFlagandle` (sic — Flag_handle 의 오타)
- **UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Anim")** — AnimGraph 전이 조건용 4 비트:
  - `bool Run = false`
  - `bool Walk = false`
  - `bool Air = false`
  - `bool Jump = false`

## 따르는 패턴
- UMCMoveComponent 의 [[entities/MCMoveComponent]] §델리게이트 페어 — `FOnMoveMode/FOnMoveSpeed/FOnMoveFlags` 3 델리게이트를 BindMoveEvent 안에서 구독 (🟢 — API 명시).
- AnimGraph 전이 조건은 BP 상태 비트 폴링 — `Run/Walk/Air/Jump` UPROPERTY 가 BlueprintReadWrite 로 BP 그래프에서 직접 사용 (🟢 — 명시).
- ChangeMoveMode/Speed/Flag 가 protected virtual — 자손이 override 로 비트 갱신 로직 커스터마이즈 가능 (🟡 추론 — default 동작은 .cpp 측에서 확인 필요).

## ⚠ 함정
- **헤더 주석 부재** — 설계 결정·델리게이트 unbind 정책 명시 없음. confidence 🟡 사유.
- **DelegateHandle unbind**: `BeginDestroy` 또는 `NativeUninitializeAnimation` 단계에서 해제 안 하면 stale callback 가능 (🟡 추론).
- **BindMoveEvent 의 3번째 인자 `FOnMoveFlags _flag`** (reference 아님): 다른 두 인자 (`FOnMoveMode&`, `FOnMoveSpeed&`) 와 일관성 결여. 복사 전달이면 원본 델리게이트와의 구독 관계가 깨질 가능성. 🟡 — 의도된 차이일 가능성도 있으나 헤더 정보로 판별 불가.
- **명칭 오타**: `MoveSpeedandle` / `MoveFlagandle` — 의도는 `MoveSpeedHandle` / `MoveFlagHandle`. BP 노출 변경 시 redirector 필요.
- **상태 4비트의 직교성**: `Run/Walk/Air/Jump` 가 mutually exclusive 인지 (Walk=true + Run=true 동시 가능?) — AnimGraph 측 정책에 의존, 헤더에 명시 안 됨.

## 횡단 정책 준수
> 적용: [[ue-cross-cutting-policies/index]] §3 (Component 행). raw 미마운트 — 추출 본문 근거, 미확인은 ❓.

| 정책 | 적용 | 근거 / 위반·미확인 | 신뢰도 |
|---|---|---|---|
| 07 profiling | ✅ | `NativeUpdateAnimation` 매 프레임 — 첫 줄 스코프 유무 ❓. 함정 "무거운 연산 금지" 와 정합. → [[ue-cross-cutting-policies/07_ProfilingScopeRule]] | ❓ |
| 10 component | ➖ | `UAnimInstance` — 컴포넌트 6대(Mobility/GetOwner/Component Tick) 직접 적용 아님. 단 **델리게이트 unbind = GC 안전 준용**(NativeUninitialize/BeginDestroy 해제, 함정 기재). | 🟡 |
| 11 asset-loading | ➖ | asset 멤버 없음. | 🟢 |
| 12 asset-opt | ➖ | 자산 미소유. | 🟢 |
| 09 global-iterator | ➖ | 미사용. | 🟢 |

## 연관 entity
- [[entities/MCMoveComponent]] — Move 델리게이트 (FOnMoveMode/Speed/Flag) 발신처.
- [[entities/MCCharacter]] — USkeletalMeshComponent Owner.
- [[entities/MCSoftSkeletalMeshComponent]] — `SoftAnimClass` UPROPERTY 의 후보 클래스 (🟡 — UMCAnimInstance 자손이 SoftAnimClass 에 들어갈 가능성).
- EMCMoveMode/EMCMoveSpeed/EMCMoveFlag (MCPlayEnum — 미인덱싱).
- FOnMoveMode/FOnMoveSpeed/FOnMoveFlags (MCDelegate — 미인덱싱).
