---
title: "BlueprintLibrary 대분류"
kind: catalog
category: BlueprintLibrary
last_ingested: 2026-05-29
---

# BlueprintLibrary 대분류

> KMCProject 의 `UBlueprintFunctionLibrary` 자손 — 정적 함수 모음. BP 측 전역 헬퍼 진입점. **카테고리 도입**: 2026-05-29 (배치 7 소묶음 직후).

## Actor 관련
- 🟡 [[entities/MCActorBlueprintLibrary]] — Actor 점프·접지 헬퍼 2종.
- 🟢 [[entities/MCSpatialQueryLibrary]] — Octree 결과 + 클래스/태그/Cone/LOS 필터 5종. [[entities/MCActorSpawnSubsystem]] / [[entities/MCSpatialQueryFilterable]] 페어.

## Water
- 🟡 [[entities/MCWaterBlueprintLibrary]] — 물 표면 파동 높이 계산·근접점 헬퍼 5종.

## Loot (MCLoot)
- 🟢 [[entities/MCLootRoller]] — 결정적 루팅 굴림 단일 static `Roll`. FRandomStream 시드 + 보장/가중치 2단계 알고리즘. TableManager FindRowAs 단일 진입(CP-1 Option B). [[entities/MCLootTableAsset]]/[[entities/MCData_LootEntry]] 페어.

## 페어 카탈로그
- [[catalogs/subsystem]] — MCSpatialQueryLibrary 가 MCActorSpawnSubsystem 1차 후보 fetch.
- [[catalogs/interface]] — MCSpatialQueryLibrary 가 IMCSpatialQueryFilterable 체크.
- [[catalogs/actor]] — Actor 액션 라이브러리.

## 공통 함정
- **모든 함수 static**: `UBlueprintFunctionLibrary` 자손은 static UFUNCTION 만 노출 가능. 멤버 변수 없음 (CDO 의 static behavior).
- **`WorldContext` meta 의무**: 정적 함수가 World 접근 필요 시 `meta=(WorldContext="WorldContextObject")` + 첫 인자 `const UObject* WorldContextObject` 패턴. BP 측이 자동으로 self 전달.
- **`AutoCreateRefTerm` meta**: `const FXxx&` 인자가 BP 측에서 wired 없이 default 값 사용 가능 — Filter struct 같은 옵션 인자에 유용.
- **`BlueprintPure=false` 명시**: 결과를 OutParam 으로 받으면서 bool 반환하는 패턴 — `BlueprintCallable, BlueprintPure=false` 가 의도된 명시 (UE 가 다중 outparam 시 자동 Pure 판정 회피).
- **silent return 금지** → [[concepts/MC-Asset-Validation-Policy]]: `MC_LOGRET_IF_*` 매크로로 early return + 로그.
- **TRACE_CPUPROFILER_EVENT_SCOPE 의무** (모든 진입점) → [[concepts/Profiling-Scope-Rule]].
- **Subsystem 의존 함수의 Editor preview / Commandlet 안전성**: Subsystem 미설치 World 에서 false 반환 + OutActors 비우기 (Soft fail). MCSpatialQueryLibrary 패턴.
- **Cone 검사**: `Normalize(Target - Origin) · Direction >= cos(HalfAngleDegrees)`. Direction 정규화 의무.
- **CP949 mojibake 주석**: 일부 라이브러리 헤더에 mojibake 가능 — 임의 재인코딩 금지.
