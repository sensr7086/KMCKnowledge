---
title: "IMCSpatialQueryFilterable"
kind: entity
category: Interface
base_class: (UInterface)
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/BlueprintLib/MCSpatialQueryFilterable.h
vault_refs:
  - sources/ue-coreuobject-interface
  - sources/ue-spatialpartition-toctree2
  - synthesis/mc-actor-spawn-subsystem-implementation
  - concepts/Profiling-Scope-Rule
last_ingested: 2026-05-29
---

# IMCSpatialQueryFilterable

## 한 줄 정의
`UMCActorSpawnSubsystem::GetActorsInRadius` 결과를 *액터 측에서* 추가 필터링하는 인터페이스. 액터 자신만 알 수 있는 게임 상태 (dead / invulnerable / phase) 검사용.

## 소속 / 상속
- 대분류: Actor (Actor 가 implements — 액터 측 필터 hook)
- `UINTERFACE(MinimalAPI, Blueprintable, BlueprintType) class UMCSpatialQueryFilterable : public UInterface`
- `class MCPLAYMODULE_API IMCSpatialQueryFilterable`
- BlueprintNativeEvent 패턴 (C++ 디폴트 + BP override).

## 핵심 구조
- **단일 API**:
  - `UFUNCTION(BlueprintNativeEvent, BlueprintCallable, Category="MC|SpatialQuery") bool CanBeSpatialQueryResult(AActor* Instigator) const`
  - 디폴트 구현: `virtual bool CanBeSpatialQueryResult_Implementation(AActor* Instigator) const { return true; }` — 모든 액터 통과.
- **사용처**: `UMCSpatialQueryLibrary::GetActorsInRadius(... bRequireInterface=true)` 호출 시 발화 (🟡 추론 — 헤더 주석의 사용처 명시).

## 따르는 패턴
- BlueprintNativeEvent 패턴 (C++ 디폴트 + BP override) → [[sources/ue-coreuobject-interface]] §5
- TOctree2 박스 prefilter + 정밀 거리 → [[sources/ue-spatialpartition-toctree2]] §2.5
- 게임 중 반경 쿼리 사용 사례 → [[synthesis/mc-actor-spawn-subsystem-implementation]] §4.2
- UFunction 바인딩된 핸들러 첫 줄 스코프 의무 → [[concepts/Profiling-Scope-Rule]]
- 도입 시점: 2026-05-13 — UMCActorSpawnSubsystem 의 TOctree2 통합 + 콘솔 명령 4종 + 디버그 시각화 시점.

## ⚠ 함정
- **§7.1 함정 #3 (vault) — `meta=(CannotImplementInterfaceInBlueprint="false")`**: UHT 가 "false" 문자열을 truthy 로 평가해 BP implement 차단 → BlueprintNativeEvent 거부. **meta 자체를 두지 말 것.** Blueprintable specifier 단독으로 BP implement 허용. 기존 MCPoolableInterface 동일 패턴. → [[synthesis/mc-actor-spawn-subsystem-implementation]] §7.1 #3.
- **MinimalAPI 의 의미**: interface 자체는 export 안 함 (자손 + Execute_* 만). 헤더 주석 명시.
- Instigator 가 nullptr 가능 — 디폴트 구현은 무시(true), 자손이 override 시 nullptr 가드 권장 (🟡).

## 연관 entity
- [[entities/MCActorSpawnSubsystem]] — GetActorsInRadius 호출자 (🟡 — bRequireInterface 옵션 위치는 본 Subsystem 또는 페어 라이브러리, 본 배치에선 확인 안 함).
- [[entities/MCSpatialQueryLibrary]] — `FMCSpatialQueryFilter.bRequireFilterableInterface=true` 시 본 인터페이스 체크.
- 예: 죽은 NPC 제외 / 무적 액터 제외 / Phase 진행 중인 적만 노출.
