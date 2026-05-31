---
title: "UMCActorBlueprintLibrary"
kind: entity
category: BlueprintLibrary
base_class: UBlueprintFunctionLibrary
module: MCPLAYMODULE
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCPlayModule/BlueprintLib/MCActorBlueprintLibrary.h
  - KMCProject/MCPlayModule/BlueprintLib/MCActorBlueprintLibrary.cpp
vault_refs: []
last_ingested: 2026-05-29
---

# UMCActorBlueprintLibrary

## 한 줄 정의
Actor 헬퍼 정적 라이브러리 — `ActorJumpStart`(점프 시작) + `ActorOnGround`(접지 검사) 2 함수.

## 소속 / 상속
- 카테고리: BlueprintLibrary
- `UCLASS()` `class MCPLAYMODULE_API UMCActorBlueprintLibrary : public UBlueprintFunctionLibrary`

## 핵심 구조
- **BP API (static)**:
  - `UFUNCTION(BlueprintCallable, Category="MCActor", meta=(WorldContext="WorldContextObject")) static void ActorJumpStart(AActor* pTargetActor)`.
  - `UFUNCTION(BlueprintPure, Category="MCActor", meta=(WorldContext="WorldContextObject")) static bool ActorOnGround(AActor* pTargetActor)`.

## 따르는 패턴
- WorldContext meta — World 접근 가정 (🟡 추론 — pTargetActor 만으로 충분할 수도, meta 가 의도된 안전장치인지 불명확).
- ActorJumpStart 는 [[entities/MCMoveComponent]] 의 LuanchCharacter(sic) / [[entities/MCCharacterMovementComponent]] 의 점프 API 호출 추론 (🟡 — .cpp 확인 필요).

## ⚠ 함정
- **`pTargetActor` 명칭 컨벤션**: `p` prefix (Hungarian) — UE 표준 (`TargetActor`) 와 불일치. BP 노출 변경 시 redirector.
- **헤더 정보 극소** — 함수 시그니처만. confidence 🟡 사유.
- 베이스 함정 적용 ([[catalogs/blueprintlibrary]] §공통 함정).

## 연관 entity
- [[entities/MCCharacter]] — pTargetActor 의 주 대상.
- [[entities/MCMoveComponent]] / [[entities/MCCharacterMovementComponent]] — 점프·접지 페어.
