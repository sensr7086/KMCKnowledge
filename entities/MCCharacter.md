---
title: "AMCCharacter"
kind: entity
category: Actor
base_class: ACharacter
module: MCPLAYMODULE
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCPlayModule/Actor/Character/MCCharacter.h
  - KMCProject/MCPlayModule/Actor/Character/MCCharacter.cpp
vault_refs: []
last_ingested: 2026-05-29
---

# AMCCharacter

## 한 줄 정의
KMCProject 의 메인 게임플레이 캐릭터. `ACharacter` + `IMCActorInterface` 구현. Parts 시스템(UMCPartsLoaderComponent)의 BP 친화 진입점 제공.

## 소속 / 상속
- 대분류: Actor
- `UCLASS(BlueprintType, meta = (BlueprintSpawnableComponent))`
- `class AMCCharacter : public ACharacter, public IMCActorInterface`
- 다단 상속 동반: 구현 인터페이스 `IMCActorInterface` (별도 페이지 미인덱싱)
- 라이프사이클: 표준 ACharacter — Constructor → BeginPlay → BindComponent/BindEvent → ...

## 핵심 구조
- 생성자 2종:
  - `AMCCharacter()`
  - `AMCCharacter(const FObjectInitializer& ObjectInitializer)`
- 오버라이드:
  - `virtual void BeginPlay() override`
- 가상 (자손 hook):
  - `virtual void BindComponent()` — 컴포넌트 바인딩 단계
  - `virtual void BindEvent()` — 이벤트 바인딩 단계
- **BP 친화 API (Parts 시스템)**:
  - `UFUNCTION(BlueprintCallable) void AddPartsObject(const FName& _part_name, TSoftObjectPtr<UMCPartsAsset> _partsAsset)`
  - `UFUNCTION(BlueprintCallable) void VisiblePartsObject(const FName& _part_name, bool _bVisible)`
- API 표면이 매우 슬림 — 실제 Move/Camera/Parts 로직은 컴포넌트(UMCMoveComponent, UMCPartsLoaderComponent 등)와 서브시스템(UMCCameraSubSystem 등)에 위임 (추론, 🟡).

## 따르는 패턴
- Component 합성 아키텍처 — Character 본체는 얇은 facade, 책임은 UMCActorComponent 자손에 분산 (🟡 추론 — 주석 근거 없음. UMCMoveComponent/UMCPartsLoaderComponent 등의 존재로부터 유추).
- 인터페이스 다단 상속 동반 — Component 가 Actor 와 통신 시 IMCActorInterface 경유.

## ⚠ 함정
- 주석/근거 부족 — 본 클래스는 헤더 주석이 없어 설계 의도·함정 기록이 추출되지 않음. confidence 🟡 의 사유.
- BindComponent/BindEvent 는 가상이지만 final 보장 없음 → 자손 override 가 부모 호출 안 하면 컴포넌트 바인딩 누락 가능 (🟡 추론).

## 연관 entity
- [[entities/MCActorInterface]] — 구현 인터페이스 (Component 컬렉션 노출).
- [[entities/MCActorComponent]] — IMCActorInterface 연결의 컴포넌트 측 사용처.
- [[entities/MCMoveComponent]]
- [[entities/MCPartsLoaderComponent]]
- [[entities/MCAnimInstance]] — Character 의 SkeletalMesh animation runtime.
- [[entities/MCPartsAsset]] — AddPartsObject 의 자산 인자.
