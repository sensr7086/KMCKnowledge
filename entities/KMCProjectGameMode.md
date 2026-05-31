---
title: "AKMCProjectGameMode"
kind: entity
category: Actor
base_class: AGameModeBase
module: KMCPROJECT
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCProjectModule/Base/KMCProjectGameMode.h
  - KMCProject/MCProjectModule/Base/KMCProjectGameMode.cpp
vault_refs: []
last_ingested: 2026-05-29
---

# AKMCProjectGameMode

## 한 줄 정의
UE 템플릿 기본 GameMode — `AGameModeBase` 자손. `UCLASS(minimalapi)`. 생성자만 헤더 노출 (.cpp 에서 DefaultPawnClass 등 설정 가정).

## 소속 / 상속
- 카테고리: Actor
- 모듈: **KMCPROJECT**
- `UCLASS(minimalapi)` `class AKMCProjectGameMode : public AGameModeBase`

## 핵심 구조
- 생성자만: `AKMCProjectGameMode()`.
- 헤더에 멤버·메서드 없음 — .cpp 에서 DefaultPawnClass / PlayerControllerClass 등 설정 (🟡 추론).

## 따르는 패턴
- UE ThirdPersonGameMode 템플릿 — 생성자에서 `DefaultPawnClass = BP_ThirdPersonCharacter.StaticClass()` 같은 BP 클래스 설정 표준.
- `UCLASS(minimalapi)` — 모듈 외부 export 최소화 (자손 + 기본 API 만).

## ⚠ 함정
- **헤더 정보 극소** — DefaultPawnClass 설정·실제 게임 룰 명시 없음 — .cpp 측 확인 필요 (🟡).
- **minimalapi 의 의미**: 다른 모듈에서 본 GameMode 직접 참조 시 link error 가능. BP 자손 또는 ConstructorHelpers 로 우회.

## 연관 entity
- [[entities/KMCProjectCharacter]] — DefaultPawnClass 후보.
