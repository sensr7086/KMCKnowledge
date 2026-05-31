---
title: "IMCActorInterface"
kind: entity
category: Interface
base_class: (UInterface)
module: MCPLAYMODULE
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCPlayModule/Actor/Interface/MCActorInterface.h
vault_refs: []
last_ingested: 2026-05-29
---

# IMCActorInterface

## 한 줄 정의
Actor 가 MC 컴포넌트 컬렉션을 노출하는 인터페이스 — `EMCComponentType → UMCActorComponent` 맵 + 템플릿 `AppendComponent<T>` / `GetComponent<T>` 헬퍼.

## 소속 / 상속
- 대분류: Component (주 사용처 = Component 관리 — edge-cases.md "주로 쓰이는 대분류"). 인터페이스 자체는 Actor 가 implements 하지만, API 표면이 Component 관리에 집중.
- `UINTERFACE(MinimalAPI) class UMCActorInterface : public UInterface` (reflection 호스트)
- `class IMCActorInterface` (실제 native 인터페이스, `GENERATED_BODY()`)
- 함께 사용: `EMCComponentType` (MCComponentTypeDef.h — 미인덱싱), `MCCOMPONENT_DEF(...)` 매크로 (모든 MC 컴포넌트가 `GetType()` static 멤버 제공).
- BlueprintNativeEvent 패턴 아님 — 일반 C++ 인터페이스 (BP 호출용 UFUNCTION 없음).

## 핵심 구조
- **Owner / ID API**:
  - `void SetInterfaceOwner(TWeakObjectPtr<UObject>)`, `TWeakObjectPtr<UObject> GetInterfaceOwner()`
  - `uint64 GetActorID()`, `void SetActorID(uint64)` — `ActorUUID`
- **Component 컬렉션 API (템플릿, inline)**:
  - `template<class COMPONENT> void AppendComponent()` — 기존 동일 type 컴포넌트 있으면 UnregisterComponent + DestroyComponent + Remove 후 `NewObject<UMCActorComponent>(Owner, COMPONENT::StaticClass())` + SetActive + SetInterface(this) + RegisterComponent. `#if WITH_EDITOR` 에서 `CreationMethod=EComponentCreationMethod::Instance` + `AddOwnedComponent` 페어.
  - `template<class COMPONENT> COMPONENT* GetComponent()` — 맵에서 type 키로 lookup 후 Cast.
- **편의 캐스트**:
  - `AActor* GetActorPtr() { return Cast<AActor>(GetInterfaceOwner().Get()); }`
  - `template<class TypeClass> TypeClass* GetClass() { return Cast<TypeClass>(GetInterfaceOwner().Get()); }`
- **protected 상태**:
  - `TMap<EMCComponentType, TWeakObjectPtr<UMCActorComponent>> AttachedComponent` — 타입별 단일 인스턴스(중복 add 시 기존 destroy).
  - `TWeakObjectPtr<UObject> Owner`
  - `uint64 ActorUUID`

## 따르는 패턴
- Component 동적 부착·교체 표준 패턴 (Unregister → Destroy → NewObject(Outer=Owner) → RegisterComponent). 🟢 — UFUNCTION 매핑은 없지만 표준 UE API 사용.
- TWeakObjectPtr 보관 — 컴포넌트 동적 소멸 안전.
- BP 노출 없음 — 본 인터페이스는 C++ 측 전용. BP 노출이 필요하면 별도 BlueprintFunctionLibrary 로 wrap.
- 동일 type 중복 add 시 *교체* 정책 — 같은 EMCComponentType 의 컴포넌트는 1개로 강제.

## ⚠ 함정
- **TMap value 가 TWeak**: 컴포넌트 동적 소멸 시 nullptr 반환 — GetComponent<T>() 결과를 즉시 nullptr 체크해야.
- **AppendComponent 의 `Cast<AActor>(GetInterfaceOwner())`** — `.Get()` 누락된 채로 `Cast` 호출. TWeakObjectPtr 의 암묵 변환에 의존 — UE 버전에 따라 컴파일/동작 차이 가능 (🟡 추론).
- **`UnregisterComponent + DestroyComponent`** 순서 의무. 반대 순서는 dangling render thread access 위험 (🟡 추론).
- **`#if WITH_EDITOR` 분기**: CreationMethod 가 Editor only 설정. Cooked 빌드에서는 BP 자손이 Append 한 컴포넌트가 Instance 로 표시 안 됨 — Details 패널 인지 차이 가능.
- **주석 없음**: 본 인터페이스의 설계 의도·예제 시나리오는 헤더에 명시 안 됨. 본 페이지의 패턴 설명은 코드 구조 기반 추론.

## 연관 entity
- [[entities/MCActorComponent]] — `SetInterface(this)` 페어, value 타입.
- [[entities/MCCharacter]] — 대표적인 implements (ACharacter + IMCActorInterface).
- EMCComponentType (MCComponentTypeDef — 미인덱싱).
- `MCCOMPONENT_DEF` 매크로 (MCComponentTypeDef — 미인덱싱).
