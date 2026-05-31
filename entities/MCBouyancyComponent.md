---
title: "UMCBouyancyComponent"
kind: entity
category: Component
base_class: UMCActorComponent
module: MCPLAYMODULE
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCPlayModule/Actor/Component/MCBouyancyComponent.h
  - KMCProject/MCPlayModule/Actor/Component/MCBouyancyComponent.cpp
vault_refs: []
last_ingested: 2026-05-29
---

# UMCBouyancyComponent

## 한 줄 정의
부력 시뮬레이션 컴포넌트 — 등록된 `UPrimitiveComponent` 들의 수면 높이 대비 깊이를 계산해 위쪽 힘을 가산.

## 소속 / 상속
- 대분류: Component
- `UCLASS(Blueprintable, meta = (BlueprintSpawnableComponent))` `class MCPLAYMODULE_API UMCBouyancyComponent : public UMCActorComponent`
- 매크로: `MCCOMPONENT_DEF(UMCBouyancyComponent, EMCComponentType, EMCComponentType::MCBouyancy)`

## 핵심 구조
- **UMCActorComponent hook 오버라이드**:
  - `virtual void Init() override`
  - `virtual void Update(float delta) override`
  - `virtual void End() override`
- **API**:
  - `void Fire()` — 부력 시뮬 시작 (🟡 추론 — 함수명 기반).
  - `void UpdateSimualtion(TWeakObjectPtr<UPrimitiveComponent> _wp_comp, float _tick)` — 개별 컴포넌트 갱신.
- **private**: `float AdjustForce(float _water_height, float _my_height, float thinkness, float _exp)` — 깊이·두께·지수 기반 힘 계산.
- **UPROPERTY**:
  - `Transient TArray<UPrimitiveComponent*> AdjustBouyancy` — 부력 적용 대상 컴포넌트 목록.
  - `EditAnywhere BlueprintReadWrite float FluidDensity = 1000.0f` — 유체 밀도.
  - `EditAnywhere BlueprintReadWrite float DispalcementRto = 2.0f` — (sic) Displacement Ratio.
  - `EditAnywhere BlueprintReadWrite float PointThickness = 10.0f`
  - `EditAnywhere BlueprintReadWrite float DebugWaterHeight = 100.0f`
  - `EditAnywhere BlueprintReadWrite float Pow = 1.5f` — 힘 지수.
- **private 상태**: `bool bSimualtionBouyancy = false`, `bool IsUp = false`.

## 따르는 패턴
- UMCActorComponent Init/Update/End 라이프사이클 활용 (🟢 — override 선언).
- TWeakObjectPtr 로 PrimitiveComponent 보관 — 컴포넌트 동적 소멸 시 안전 (🟢 — UpdateSimualtion 시그니처).
- DebugWaterHeight UPROPERTY 존재 → AMCWaterVolume/UMCWaterPlaneComponent 없이도 단독 디버그 가능 (🟡 추론).

## ⚠ 함정
- `AdjustBouyancy` 가 raw `UPrimitiveComponent*` TArray (TWeak 아님) — 컴포넌트 동적 소멸 시 dangling 가능. UPROPERTY(Transient) 라 직렬화는 안 되지만 GC 추적은 안전 (UPROPERTY 자체가 GC 마커).
- `DispalcementRto` 오타 (Displacement) — 명명 일관성 깨짐. BP 노출 이름도 같이 변경하려면 redirector 등록 필요.
- 본 클래스 헤더의 일부 한국어 주석은 CP949 mojibake 상태 — 임의 재인코딩 금지.

## 연관 entity
- [[entities/MCActorComponent]] — 직접 베이스.
- [[entities/MCWaterPlaneComponent]] — 수면 높이 제공 가능성 (🟡 추론).
- [[entities/MCWaterVolume]] — 수중 영역 감지 페어 (🟡 추론).
