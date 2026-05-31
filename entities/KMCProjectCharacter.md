---
title: "AKMCProjectCharacter"
kind: entity
category: Actor
base_class: ACharacter
module: KMCPROJECT
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCProjectModule/Base/KMCProjectCharacter.h
  - KMCProject/MCProjectModule/Base/KMCProjectCharacter.cpp
vault_refs: []
last_ingested: 2026-05-29
---

# AKMCProjectCharacter

## 한 줄 정의
UE 템플릿 기본 `ACharacter` 자손 — SpringArm + FollowCamera + Enhanced Input(Move/Look/Jump) + DefaultMappingContext. KMCProject 메인 모듈의 placeholder 캐릭터 (실제 게임플레이는 MCPlayModule 의 [[entities/MCCharacter]] 가 담당).

## 소속 / 상속
- 카테고리: Actor
- 모듈: **KMCPROJECT** (Default 메인 모듈)
- `UCLASS(config=Game)` `class AKMCProjectCharacter : public ACharacter`
- 함께 정의: `DECLARE_LOG_CATEGORY_EXTERN(LogTemplateCharacter, Log, All)`.

## 핵심 구조
- **UPROPERTY (VisibleAnywhere BlueprintReadOnly meta=(AllowPrivateAccess="true"))**:
  - `USpringArmComponent* CameraBoom` — 카메라 boom (캐릭터 뒤).
  - `UCameraComponent* FollowCamera` — Follow camera.
- **UPROPERTY (EditAnywhere BlueprintReadOnly meta=(AllowPrivateAccess="true") Category="Input")**:
  - `UInputMappingContext* DefaultMappingContext`.
  - `UInputAction* JumpAction / MoveAction / LookAction`.
- **생성자**: `AKMCProjectCharacter()`.
- **APawn 인터페이스**:
  - `virtual void SetupPlayerInputComponent(UInputComponent*) override` — Enhanced Input 바인딩.
  - `virtual void BeginPlay()` (override 없음 — 베이스 호출 가정).
- **protected**:
  - `void Move(const FInputActionValue&)` — 이동 입력.
  - `void Look(const FInputActionValue&)` — 시야 입력.
- **FORCEINLINE Getter**: `GetCameraBoom() const`, `GetFollowCamera() const`.

## 따르는 패턴
- UE ThirdPersonCharacter 템플릿 — SpringArm + FollowCamera + Enhanced Input 표준 골격.
- `AllowPrivateAccess="true"` 패턴 — private UPROPERTY 를 BP 가 ReadOnly 접근 가능.
- `config=Game` — *.ini 설정 가능.

## ⚠ 함정
- **본 액터는 KMCProject 모듈의 placeholder** — 실제 게임플레이 캐릭터는 MCPlayModule 의 [[entities/MCCharacter]] 가 담당. 두 액터의 역할 분리 인지 필요.
- **`virtual void BeginPlay()` 의 override 키워드 없음** — 헤더상 의도 명확하지 않음. .cpp 에서 Super::BeginPlay() 호출 가정.
- **`LogTemplateCharacter` 카테고리** — Template 이름 잔존. 프로젝트화 시 `LogKMCProjectCharacter` 등 명명 정리 권장.

## 연관 entity
- [[entities/MCCharacter]] — 실제 게임플레이 캐릭터 (MCPlayModule).
- [[entities/KMCProjectGameMode]] — DefaultPawnClass 후보.
