---
title: "UMCPartsAsset"
kind: entity
category: Asset
base_class: UObject
module: MCPLAYMODULE
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCPlayModule/MCParts/MCPartsAsset.h
  - KMCProject/MCPlayModule/MCParts/MCPartsAsset.cpp
vault_refs: []
last_ingested: 2026-05-29
---

# UMCPartsAsset

## 한 줄 정의
Parts 그래프 자산 호스트 UObject — `UMCParts_Graph` (노드 컬렉션) + 프리뷰 메시·애니 보관. UMCPartsLoaderComponent 가 런타임 소비.

## 소속 / 상속
- 카테고리: Asset
- `UCLASS(BlueprintType)` `class MCPLAYMODULE_API UMCPartsAsset : public UObject`

## 핵심 구조
- **UPROPERTY**:
  - `UMCParts_Graph* Graph = nullptr` — 노드 컬렉션.
  - `EditAnywhere BlueprintReadOnly USkeletalMesh* PreviewMesh = nullptr` — 에디터 프리뷰.
  - `EditAnywhere BlueprintReadOnly UAnimSequence* PreviewAnimation = nullptr` — 에디터 프리뷰 애니.
- 메서드 0개 — 컬렉션 보관 전용.

## 따르는 패턴
- Story 자산과 동일 패턴 — Graph 보관 + Loader 가 워크. UPartsLoaderComponent::AddPartsLoadSync 가 본 자산을 입력으로 받음.
- PreviewMesh / PreviewAnimation 은 에디터 측 프리뷰 전용 (🟡 추론).

## ⚠ 함정
- **`UMCParts_Graph* Graph` raw 포인터** — TObjectPtr 미적용 UE 5.0+ 권장 패턴.
- **PreviewMesh / PreviewAnimation 이 Cooked 빌드에 포함**: BlueprintReadOnly 로 게임 코드에서 접근 가능 — 단순 프리뷰 목적이면 #if WITH_EDITORONLY_DATA 가드 권장 (🟡).

## 연관 entity
- [[entities/MCParts_Graph]] — 노드 컬렉션.
- [[entities/MCParts_Node]] — 노드 베이스.
- [[entities/MCPartsLoaderComponent]] — 런타임 소비자.
- [[entities/MCStory_ActionAddParts]] — Soft 참조로 본 자산 적용.
