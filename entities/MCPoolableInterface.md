---
title: "IMCPoolableInterface"
kind: entity
category: Interface
base_class: (UInterface)
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/Subsystem/MCPoolableInterface.h
vault_refs:
  - synthesis/actor-pool-reset-pattern
  - concepts/MC-Asset-Validation-Policy
  - sources/ue-significance-skill
  - synthesis/character-many-npc-5-fold-optimization
last_ingested: 2026-05-29
---

# IMCPoolableInterface

## 한 줄 정의
액터 풀 통합 hook 인터페이스 — `UMCActorSpawnSubsystem` 이 acquire/release 시점에 호출. `OnPoolActivate / OnPoolDeactivate / IsPoolActive` + Significance hook (`GetSignificanceTag / OnSignificanceChanged`).

## 소속 / 상속
- 대분류: Actor (Actor 가 implements — Pool 시스템의 액터 측 hook)
- `UINTERFACE(MinimalAPI, Blueprintable, BlueprintType) class UMCPoolableInterface : public UInterface`
- `class MCPLAYMODULE_API IMCPoolableInterface`
- BlueprintNativeEvent 패턴 (C++ 디폴트 구현 + BP override 가능).

## 핵심 구조
- **Pool hook (UFUNCTION BlueprintNativeEvent, BlueprintCallable)**:
  - `void OnPoolActivate(const FTransform& SpawnTransform)` — Subsystem 의 SetActorTransform/Hidden/Collision/Tick 토글 *직후* 호출. BP: `Event On Pool Activate`.
  - `void OnPoolDeactivate()` — Subsystem 의 Tick/Collision/Hidden 토글 *전* 호출. BP: `Event On Pool Deactivate`.
  - `bool IsPoolActive() const` — Subsystem 디버그/자기 진단용. AMCPooledActor 베이스는 `bIsPoolActive` 멤버 기반 default 구현 제공.
- **Significance hook (옵션 — `bEnableSignificanceManagement=true` 시 발화)**:
  - `FName GetSignificanceTag() const` — USignificanceManager Tag (그룹 식별자). NAME_None 이면 Subsystem 의 `DefaultSignificanceTag` 사용.
  - `void OnSignificanceChanged(float NewScore, float OldScore)` — score 0~1 매핑. BP: `Event On Significance Changed`. 자손 결정: Tick OFF/ON / VFX / SkeletalMesh URO Bucket / Audio Cull / Niagara Cull.

## 따르는 패턴
- 사용 방식 2 가지:
  - **(A) AMCPooledActor 베이스 상속** → 자동 구현 (기본 Tick/Hidden/Collision 토글).
  - **(B) 기존 액터** (ACharacter / APawn / AActor 자손) 가 interface 만 implement → 베이스 변경 없이 풀 통합.
- 라이프사이클 매트릭스 (Activate/Deactivate hook 의무) → [[synthesis/actor-pool-reset-pattern]] §2
- silent return 금지 → [[concepts/MC-Asset-Validation-Policy]]
- USignificanceManager + FManagedObjectInfo → [[sources/ue-significance-skill]] §2
- 누적 적용 §3 단계 → [[synthesis/character-many-npc-5-fold-optimization]] §3

## ⚠ 함정
- **§6 함정 #2 (vault) — 캐싱 weak ptr 무효화 누락**: `OnPoolActivate(Transform)` 에서 캐싱 weak ptr 의무 reset → [[synthesis/actor-pool-reset-pattern]] §6.
- **BlueprintPure 미지원**: UHT 가 Interface 함수의 BlueprintPure 거부. `IsPoolActive()` 같은 const 함수는 BP 에서 자동 Pure 처리(BlueprintPure specifier 없음, const + 반환값으로 자동 인식).
- **OnPoolActivate 의 SpawnTransform 인자는 참고용** — Subsystem 이 이미 Actor 에 적용 후 호출.
- **OnPoolDeactivate 의 정리 의무**: 외부 시스템 등록 콜백 unbind, 캐싱 ptr reset, 진행 중 비동기 핸들 cancel.

## 연관 entity
- [[entities/MCPooledActor]] — 베이스 클래스 구현 (사용 방식 A).
- [[entities/MCActorSpawnSubsystem]] — Implements 검사 + Activate/Deactivate 호출자.
