---
title: "UMCStory_DecortorBase"
kind: entity
category: GraphNode
base_class: UObject
module: MCPLAYMODULE
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCPlayModule/MCStory/MCStoryAsset.h
vault_refs: []
last_ingested: 2026-05-29
---

# UMCStory_DecortorBase

## 한 줄 정의
Story Decorator 의 추상 베이스 (Abstract Blueprintable) — `UMCStory_DecoratorCondition` 의 자식 컬렉션 요소. BP 가 `BeginDecorator` 이벤트로 조건 검사 시작 → `OnFinish(bool)` 로 결과 반환.

## 소속 / 상속
- 카테고리: GraphNode (Decorator 베이스 — Condition 자손이 보유)
- `UCLASS(Abstract, BlueprintType, Blueprintable)` `class MCPLAYMODULE_API UMCStory_DecortorBase : public UObject`
- **명칭 오타**: `DecortorBase` (의도 = DecoratorBase).

## 핵심 구조
- **virtual API**:
  - `virtual UWorld* GetWorld() const override`.
  - `virtual FString GetDescription() const { return TEXT("Decorator"); }`.
  - `virtual bool ComplateDecorator()` — DecoState == End 면 true (sic — Complete 의 오타).
- **BP API**:
  - `UFUNCTION(BlueprintCallable) void OnFinish(bool finish)` — DecoComplateCondition=finish + DecoState=End.
  - `UFUNCTION(BlueprintImplementableEvent) void BeginDecorator()` — BP 측 진입점.
- **C++**:
  - `void StartDecorator()` — DecoState=Start + BeginDecorator() 호출.
  - `bool ResultDecorator() { return DecoComplateCondition; }`.
- **protected 상태**:
  - `EMCStoryDecoState DecoState = EMCStoryDecoState::EMCStoryNone` — None / Start / End 3종.
  - `bool DecoComplateCondition = false` — OnFinish 결과 (sic — Complete 의 오타).

## 따르는 패턴
- BP 측 게임 로직 → BeginDecorator 에서 조건 검사 시작 → 결과 결정되면 OnFinish(true/false) 호출 → DecoState=End 전이.
- UMCStory_DecoratorCondition 이 Tick 마다 `ComplateDecorator()` 폴링 — 끝나면 결과(ResultDecorator) 로 분기.
- 시간 제한: DecoratorCondition 측의 `CheckDecoratorWaitTime` (-1=무한대) 가 폴링 타임아웃 결정.

## ⚠ 함정
- **명칭 오타 누적**: `DecortorBase` / `ComplateDecorator` / `DecoComplateCondition` — BP 노출 변경 시 redirector.
- **DecoState 가 UPROPERTY 아님** — 직렬화 안 됨. 의도된 transient 상태이지만 디버그/유닛 테스트에서 가시화 어려움.
- **OnFinish 호출 없이 BeginDecorator 가 끝나면 영구 Wait**: BP 측이 OnFinish 호출 의무 강제 안 됨. DecoratorCondition 의 `CheckDecoratorWaitTime` 타임아웃이 안전장치.
- **헤더 주석 부재** — confidence 🟡 사유.

## 연관 entity
- [[entities/MCStory_DecoratorCondition]] — 본 클래스 인스턴스 컬렉션 (`DecoratorClass[]`) 보유.
- [[entities/MCStory_Node]] — 동일 그래프 시스템.
