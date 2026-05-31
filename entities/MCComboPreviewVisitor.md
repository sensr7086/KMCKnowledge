---
title: "IMCComboPreviewVisitor"
kind: entity
category: Interface
base_class: (plain C++ interface)
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/MCCombo/MCComboPreviewVisitor.h
vault_refs:
  - synthesis/mc-combo-editor-phase-6-7-inhouse-master
last_ingested: 2026-05-29
---

# IMCComboPreviewVisitor

## 한 줄 정의
Sequencer-lite Preview Visitor — Combo Section 자손이 자기 type 의 `Visit<Type>Section` 메서드 호출. Cast 가 Section 자손 안으로 격리되어 새 type 추가 시 collector 측 코드 수정 의무 제거.

## 소속 / 상속
- 대분류: **Interface** (2026-05-29 배치 3 직후 Interface 카테고리 정식 신설 시 재분류. Combo 시스템 indexed 후 분류 한계 자연 해소).
- **UINTERFACE 아님** — plain C++ 인터페이스 (UObject reflection 미사용). 헤더 주석 "Slate 의존 회피 — 인터페이스 자체는 plain forward declare 만".
- `class MCPLAYMODULE_API IMCComboPreviewVisitor` — `virtual ~IMCComboPreviewVisitor() = default`.
- Phase 8 (2026-05-20) 도입.

## 핵심 구조
- **Visit 메서드 (default = no-op, 자손이 override)**:
  - `virtual void VisitMontageSection(UMCComboMontageSection*)` — Montage LoadSynchronous + Skeleton 검증 + cache 추가.
  - `virtual void VisitTransformSection(UMCComboTransformSection*)` — CachedTransformSections 추가.
  - `virtual void VisitGenericSection(UMCComboSection*)` — type-specific visit 매칭 안 된 자손 (Input/Audio/Notify/사용자 자손) 의 fallback. default no-op.
- **호출 패턴 (헤더 주석 예제)**:
  ```cpp
  class FMyPreviewCollector : public IMCComboPreviewVisitor { ... };
  FMyPreviewCollector Collector;
  for (auto& Section : Track->Sections)
      Section->AcceptPreviewVisitor(Collector);  // Section 자손이 type 별 Visit 호출
  ```

## 따르는 패턴
- **Visitor 패턴** — `SMCComboPreviewSceneViewport::LoadAssetPreview` L108-167 안 Track type 별 Cast 4개 산재를 해소. 새 Section type 추가 시 (a) Visitor 인터페이스에 `VisitX` 메서드 추가 + (b) Section 자손이 `AcceptPreviewVisitor` 안에서 `VisitX` 호출.
- **compile-time 정직성** — 새 type 의 Visit 누락 시 컴파일 에러 (interface 가 pure virtual 인 경우. 현재는 default no-op 이라 컴파일은 통과, 자손이 의도적으로 override 누락 가능).
- **Slate 의존 회피** — 인터페이스 자체는 plain forward declare 만. PreviewSceneViewport (Editor 측) 가 구현체.
- **Engine 미러**: `IMovieScenePlayer` (MovieScene 모듈) — runtime 인터페이스 + Editor 구현체 패턴.
- Phase 8 후속 권고 → [[synthesis/mc-combo-editor-phase-6-7-inhouse-master]] §13

## ⚠ 함정
- **default no-op 이지만 의도된 정직성과 트레이드오프**: 새 Section type 추가 시 `Visit<NewType>Section` 호출 누락 가능. pure virtual 로 강제하지 않은 선택 — 미구현 visitor 안전성 우선.
- **새 type-specific Section 자손 추가 시 절차**:
  1. 본 인터페이스에 `virtual void Visit<NewType>Section(...)` 추가 (default no-op 또는 pure virtual).
  2. Section 자손이 `AcceptPreviewVisitor` override 안에서 `Visitor.Visit<NewType>Section(this)` 호출.
- **UObject 인터페이스 아님** — Implements<>() 검사 / Execute_* 호출 불가. Section 자손에서 직접 가상 호출 (`Visitor.VisitXxx(this)`).

## 연관 entity
- [[entities/MCComboSection]] — 베이스 (AcceptPreviewVisitor 의 default 호출 대상).
- [[entities/MCComboMontageSection]] — VisitMontageSection 호출.
- [[entities/MCComboTransformSection]] — VisitTransformSection 호출.
- SMCComboPreviewSceneViewport (MCEditorModule — 미인덱싱, 본 visitor 의 구현체).
