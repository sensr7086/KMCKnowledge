---
title: "UMCActorComponent"
kind: entity
category: Component
base_class: UActorComponent
module: MCPLAYMODULE
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCPlayModule/Actor/Component/MCActorComponent.h
  - KMCProject/MCPlayModule/Actor/Component/MCActorComponent.cpp
vault_refs: []
last_ingested: 2026-05-29
---

# UMCActorComponent

## 한 줄 정의
KMCProject 의 모든 MC 컴포넌트가 상속하는 베이스 — `Init/Update/End` 라이프사이클 훅 + `IMCActorInterface` 포인터 보관 + `MCCOMPONENT_DEF` 타입 식별자.

## 소속 / 상속
- 대분류: Component
- `UCLASS(Blueprintable)` `class MCPLAYMODULE_API UMCActorComponent : public UActorComponent`
- 매크로: `MCCOMPONENT_DEF(UMCActorComponent, EMCComponentType, EMCComponentType::None)` — MC 전용 컴포넌트 타입 식별자.
- 라이프사이클: 표준 UActorComponent (BeginPlay → TickComponent → BeginDestroy) + 추가 hook(Init/Update/End/TickFired/EndTickFired).

## 핵심 구조
- **생성자**: `UMCActorComponent(const FObjectInitializer&)`
- **UActorComponent 오버라이드**:
  - `virtual void TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction*) override`
  - `virtual void BeginPlay() override`
  - `virtual void BeginDestroy() override`
- **가상 hook (자손 override)**:
  - `virtual void Init() {}` — 초기 진입
  - `virtual void Update(float delta) {}` — 갱신
  - `virtual void End() {}` — 종료 정리
  - `virtual void TickFired()` — Tick 진입 hook
  - `virtual void EndTickFired()` — Tick 종료 hook
- **인터페이스 연결**:
  - `void SetInterface(IMCActorInterface*)` — Owner Actor 와의 통신 채널.
  - protected: `IMCActorInterface* GetInterface()`
  - private: `IMCActorInterface* Interface` — raw 포인터(스마트포인터 아님 — Owner 라이프사이클과 동기 가정, 🟡).

## 따르는 패턴
- Owner Actor 와 컴포넌트 간 통신 = 인터페이스 경유 (IMCActorInterface) — Actor 의 구체 타입 의존 회피 (🟡 추론 — SetInterface API 로 추론).
- Init/Update/End — 표준 UActorComponent 의 BeginPlay/Tick/EndPlay 위에 얇은 abstraction layer (🟡 추론).
- MCCOMPONENT_DEF 매크로 — 모든 MC 컴포넌트가 자기 EMCComponentType 식별자 노출 (Type-tagged Component Registry 패턴, 🟡).

## ⚠ 함정
- 본 클래스 헤더의 일부 한국어 주석은 CP949 인코딩 mojibake 상태(`MC���� ������Ʈ Ÿ�Ե����ο�`) — UTF-8 으로 재인코딩 시 의도 손상 가능. CLAUDE.md(KMCProject) 명시 — 임의 재인코딩 금지.
- `Interface` 가 raw 포인터 — Owner Actor 가 먼저 사라지면 dangling 가능. SetInterface 호출 측이 라이프사이클 보장 책임.
- BeginPlay/BeginDestroy override 시 Super 호출 누락 시 BP 노출/Garbage Collection 에 영향 (🟡 추론).

## 연관 entity
- [[entities/MCActorInterface]] — `SetInterface(this)` 페어, AttachedComponent 맵의 value 타입.
- [[entities/MCBouyancyComponent]] — 직접 자손
- [[entities/MCMoveComponent]] — 직접 자손
- [[entities/MCPartsLoaderComponent]] — 직접 자손
- EMCComponentType (KMCProject/MCPlayModule/MCTypeDef/MCComponentTypeDef.h — 미인덱싱)
- EMCMoveMode/MoveSpeed/MoveFlag (PlayEnum — 미인덱싱)
