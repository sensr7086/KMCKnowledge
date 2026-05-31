---
title: "AMCCamera"
kind: entity
category: Actor
base_class: ACameraActor
module: MCPLAYMODULE
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCPlayModule/Actor/Camera/MCCamera.h
  - KMCProject/MCPlayModule/Actor/Camera/MCCamera.cpp
vault_refs: []
last_ingested: 2026-05-29
---

# AMCCamera

## 한 줄 정의
KMCProject 의 게임플레이 카메라 액터. `ACameraActor` 의 얇은 자손 — UMCCameraSubSystem 의 카메라 모드 시스템과 연결되는 placeholder.

## 소속 / 상속
- 대분류: Actor
- `UCLASS(Blueprintable)` `class MCPLAYMODULE_API AMCCamera : public ACameraActor`
- 라이프사이클: 표준 ACameraActor.

## 핵심 구조
- 오버라이드 (protected):
  - `virtual void BeginPlay() override`
- 그 외 멤버·함수 헤더 노출 없음. 추가 로직은 .cpp 또는 BP 자손에 있을 가능성 (🟡 추론).

## 따르는 패턴
- ACameraActor 기반 placeable camera (🟢 — 베이스 선언으로 확정).
- UMCCameraSubSystem 의 활성 카메라 등록 대상 (🟡 — UMCCameraSubSystem 헤더의 `class AMCCamera` 전방 선언 + Camera/Mode 디렉토리 구조로 추론).

## ⚠ 함정
- 주석·헤더 정보 극소 — 본 클래스는 거의 비어 있어 의도 추출 한계. 실제 사용 패턴은 UMCCameraSubSystem / UMCCameraBase / UMCCameraPlayable 인덱싱 후 명확해질 예정.

## 연관 entity
- [[entities/MCCameraSubSystem]] — MainCamera 보관자, 본 액터를 등록 대상으로 사용.
- [[entities/MCCameraBase]] — Camera 모드 베이스.
- [[entities/MCCameraPlayable]] — Playable 모드 구현.
