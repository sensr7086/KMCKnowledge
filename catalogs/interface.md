---
title: "Interface 대분류"
kind: catalog
category: Interface
last_ingested: 2026-05-29
---

# Interface 대분류

> KMCProject 의 인터페이스. UINTERFACE (UE reflection 통합) 과 plain C++ 인터페이스 양분. **카테고리 도입**: 2026-05-29 (배치 3 직후) — "주 사용 대분류" 강제 귀속 분류 한계 해소.

## UINTERFACE (BlueprintNativeEvent / BP implement 지원)
- 🟡 [[entities/MCActorInterface]] — Actor 가 implements. `EMCComponentType → UMCActorComponent` 맵 + 템플릿 `AppendComponent<T>` / `GetComponent<T>` 헬퍼.
- 🟢 [[entities/MCPoolableInterface]] — Actor 가 implements. Pool acquire/release hook + Significance hook. BlueprintNativeEvent 패턴.
- 🟢 [[entities/MCSpatialQueryFilterable]] — Actor 가 implements. Octree 결과의 액터 측 필터. `CanBeSpatialQueryResult(Instigator) const`.

## plain C++ 인터페이스 (UObject reflection 미사용)
- 🟢 [[entities/MCComboPreviewVisitor]] — Sequencer-lite Preview Visitor. Combo Section 자손이 자기 type 의 `Visit<Type>Section` 메서드 호출. Slate 의존 회피 (forward declare 만).

## 공통 함정
- **§7.1 함정 #3 — `meta=(CannotImplementInterfaceInBlueprint="false")` 금지**: UHT 가 "false" 문자열을 truthy 로 평가 → BP implement 차단 → BlueprintNativeEvent 거부. **meta 자체를 두지 말 것.** Blueprintable specifier 단독으로 BP implement 허용. → [[synthesis/mc-actor-spawn-subsystem-implementation]] §7.1 #3.
- **BlueprintPure 미지원** (UINTERFACE): UHT 가 Interface 함수의 BlueprintPure 거부. const 함수는 BP 에서 자동 Pure 처리 (BlueprintPure specifier 없음, const + 반환값으로 자동 인식).
- **MinimalAPI 의미**: interface 자체는 export 안 함 (자손 + `Execute_*` 만 export).
- **plain C++ 인터페이스 한계**: `Implements<>()` 검사 / `Execute_*` 호출 불가. 자손에서 직접 가상 호출 (`Visitor.VisitXxx(this)`). 예: IMCComboPreviewVisitor.
- **BlueprintNativeEvent 디폴트 구현 시그니처**: `_Implementation` 접미사 + virtual + return 표준. 자손 override 와 의도된 default 양분.
- **CP949 mojibake 주석**: 일부 인터페이스 헤더의 한국어 주석이 깨져 보일 수 있음 — CLAUDE.md(KMCProject) "임의 재인코딩 금지" 준수.
