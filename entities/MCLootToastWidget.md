---
title: "UMCLootToastWidget"
kind: entity
category: Widget
base_class: UUserWidget
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/MCLoot/UI/MCLootToastWidget.h
  - KMCProject/MCPlayModule/MCLoot/UI/MCLootToastWidget.cpp
vault_refs:
  - entities/UUserWidget
  - concepts/UMG-Super-Call-Convention
last_ingested: 2026-05-29
---

# UMCLootToastWidget

## 한 줄 정의
루팅 결과 토스트/팝업 위젯 베이스(UUserWidget Abstract) — C++ 는 결과 데이터 전달만, 실제 시각화는 BP 위젯이 OnShowResult 에서 구성.

## 소속 / 상속
- 대분류: Widget (신설 — 2026-05-29, [[categories]] §활성 카테고리)
- `UCLASS(Abstract, Blueprintable)` `class MCPLAYMODULE_API UMCLootToastWidget : public UUserWidget`
- 충족 REQ: REQ-24, REQ-25. 설계 출처: `_workspace/02_ue-architect_design.md §2.7`.
- C++ 는 결과 데이터 전달만 책임. 실제 시각화("+ HP 포션 ×2" 등)는 BP 위젯이 OnShowResult 에서 구성. (근거: `entities/UUserWidget "BP 위젯 베이스, CreateWidget<>로 생성, BindWidget"; concepts/UMG-Super-Call-Convention`)
- `Abstract` — 직접 인스턴스화 차단, BP 자손 필수.

## 핵심 구조
- **`void ShowResult(const TArray<FMCLootResultItem>& Items)`** `UFUNCTION(BlueprintCallable, Category="MC|LootUI")` — C++ 진입점(REQ-25). 내부에서 `OnShowResult(Items)` 호출해 시각화 위임. 빈 결과(전부 꽝)여도 그대로 전달(BP 가 "획득 없음" 토스트 구성 가능).
- **`void OnShowResult(const TArray<FMCLootResultItem>& Items)`** `UFUNCTION(BlueprintImplementableEvent, Category="MC|LootUI")` — BP 위젯에서 구현. Items 를 텍스트/리스트로 구성, 애니메이션 재생 등. (REQ-24)
- `.cpp` 본체는 `ShowResult` → `OnShowResult` 단순 전달 한 줄.

## 따르는 패턴
- BP 위젯 베이스, CreateWidget<>로 생성, BindWidget → [[entities/UUserWidget]]
- C++ 진입점(ShowResult) → BP 구현 이벤트(OnShowResult) 위임 — UMG Super 호출 컨벤션 계열 → [[concepts/UMG-Super-Call-Convention]]
- 데이터/시각화 분리 — C++ 는 `TArray<FMCLootResultItem>` 전달만, 레이아웃/애니메이션은 BP.
- [[entities/MCLootableComponent]] `OnLootCompleted` 델리게이트가 본 위젯 `ShowResult` 의 1차 구독 배선 대상(REQ-24).

## ⚠ 함정
- **★ 빌드 영향 (사용자 수동)**: 이 클래스(UUserWidget)는 UMG/Slate/SlateCore 모듈 의존 필요 — `MCPlayModule.Build.cs` 에 `"UMG", "Slate", "SlateCore"` 추가 필요. 헤더 주석이 "직접 수정 금지 지시 → 보고"로 명시. **현재 Build.cs 미반영이면 컴파일 실패** — 사용자 확인 필요 항목.
- **Abstract — 직접 사용 불가**: BP 자손을 만들어 OnShowResult 를 구현해야 동작. C++ 단독으로는 시각화 없음.
- 빈 결과도 전달됨 — BP 측이 `Items.Num()==0` 분기로 "획득 없음" 처리 책임.

## 연관 entity
- [[entities/MCLootableComponent]] — `OnLootCompleted(TArray<FMCLootResultItem>)` broadcast 가 본 위젯 ShowResult 로 배선.
- [[entities/MCData_LootEntry]] — `FMCLootResultItem`(ItemId/Count/Rarity) 입력 타입, Rarity 는 토스트 색상 표시용.
- [[entities/MCLootRoller]] — `FMCLootResultItem` 생산자.
