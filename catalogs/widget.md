---
title: "Widget 대분류"
kind: catalog
category: Widget
last_ingested: 2026-05-29
---

# Widget 대분류

> KMCProject 의 `UUserWidget` / UMG 위젯 자손 — 런타임 UI 위젯 베이스 + 구현체. Slate `S...Widget`(MCEditorModule 의 편집기 위젯)은 본 카테고리가 *아님* (mcwiki 단일 진실원, [[categories]] §범위 밖 참조). **카테고리 도입**: 2026-05-29 (배치 8 MCLoot 직후).

## 베이스 / 추상
- 🟢 [[entities/MCLootToastWidget]] — `UMCLootToastWidget : UUserWidget` `UCLASS(Abstract, Blueprintable)`. 루팅 결과 토스트 베이스. C++ 진입점 `ShowResult` → BP `OnShowResult(BlueprintImplementableEvent)` 위임. 데이터/시각화 분리.

## 구현체
- (현재 없음 — BP 자손이 실제 구현. C++ 구현체 등장 시 등록.)

## 페어 카탈로그
- [[catalogs/component]] — UI 가 구독하는 런타임 컴포넌트 델리게이트 ([[entities/MCLootableComponent]] `OnLootCompleted`).
- [[catalogs/datatable]] — 위젯에 전달되는 결과 값 타입 (`FMCLootResultItem`, [[entities/MCData_LootEntry]] 동거).

## 공통 함정
- **UMG/Slate/SlateCore 모듈 의존 (빌드 영향)**: `UUserWidget` 자손은 `MCPlayModule.Build.cs` 에 `"UMG", "Slate", "SlateCore"` 추가 필요. 미반영 시 컴파일 실패. KMCProject Build.cs 직접 수정 금지 지시 → 사용자 보고 항목.
- **C++ 진입점 → BP 구현 이벤트 위임 패턴**: C++ `BlueprintCallable` 함수가 `BlueprintImplementableEvent`(또는 BlueprintNativeEvent)를 호출해 시각화를 BP 로 위임. 데이터는 C++, 레이아웃/애니메이션은 BP. → [[concepts/UMG-Super-Call-Convention]].
- **Abstract UUserWidget = 직접 사용 불가**: BP 자손 필수. `CreateWidget<>` 로 BP 클래스 생성. → [[entities/UUserWidget]].
- **Slate S-위젯과 혼동 금지**: 편집기 측 `S...Widget`(SCompoundWidget 등)은 MCEditorModule 소관 → mcwiki 참조, 본 카탈로그 아님.
