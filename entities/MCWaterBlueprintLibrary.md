---
title: "UMCWaterBlueprintLibrary"
kind: entity
category: BlueprintLibrary
base_class: UBlueprintFunctionLibrary
module: MCPLAYMODULE
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCPlayModule/BlueprintLib/MCWaterBlueprintLibrary.h
  - KMCProject/MCPlayModule/BlueprintLib/MCWaterBlueprintLibrary.cpp
vault_refs: []
last_ingested: 2026-05-29
---

# UMCWaterBlueprintLibrary

## 한 줄 정의
물 표면 파동 계산 정적 라이브러리 — 파동 높이·근접 삼각형 점 계산 헬퍼 5종. [[entities/MCWaterVolume]] / [[entities/MCWaterPlaneComponent]] / [[entities/MCBouyancyComponent]] 시각화 페어.

## 소속 / 상속
- 카테고리: BlueprintLibrary
- `UCLASS()` `class MCPLAYMODULE_API UMCWaterBlueprintLibrary : public UBlueprintFunctionLibrary`

## 핵심 구조
- **5 BP API (모두 static, meta=(WorldContext))**:
  1. `GetWaterPlaneWaveHeightClosetPoint(const FTransform& water_tm, const FVector& location, float time, float section_size, float mesh_size, const TArray<FWaveParam>& param, bool drawDebug=false) → FVector` — 근접 segment 의 파동 높이 (디버그 옵션).
  2. `ClosetPointOnTriangleToPoint(const FVector& point, point_a, point_b, point_c, FVector& out) → int32` — 삼각형 위 근접점 (out 반환 + 결과 종류 int).
  3. `GetWaterPlaneWaveHeight(const FVector& water_plane_location, const FVector& location, float time, float wave_scale, float wave_speed, float wave_height) → FVector` — 단순 사인파 (4 파라미터).
  4. `GetWaterPlaneWaveHeightV2(const FVector& water_plane_location, const FVector& location, float time, float wave_speed, FVector4 wave_param) → FVector` — V2 (4 파라미터 vector4 packing).
  5. `CheckWaterAndLocation(const FVector& location, float up_tolerance, float down_tolerance) → bool` — 위치 ± tolerance 안 물 존재 검사.

## 따르는 패턴
- `FWaveParam` (MCCoreStruct.h, 미인덱싱) 사용 — 다중 파동 합성 가정 (🟡 추론).
- V2 는 V1 의 단순화 (`FVector4` 로 wave 파라미터 packing).
- ClosetPointOnTriangleToPoint 의 int 반환 — 0/1/2/3 = vertex/edge/face/none 같은 분류 가정 (🟡 — .cpp 확인 필요).

## ⚠ 함정
- **명칭 오타**: `Closet` (의도 = Closest). BP 노출 변경 시 redirector.
- **`drawDebug` 인자가 함수 매개변수**: 디자이너가 매 호출 토글 가능. 의도된 설계지만 BP 그래프 시 매번 false 전달이 복잡함 (Detail Panel 기본값 활용).
- **CheckWaterAndLocation 의 World 접근 방식 불명확**: meta=(WorldContext) 이지만 인자에 WorldContextObject 없음 — UE 가 자동 추가하는 보이지 않는 인자 패턴 (🟡 추론).
- **5 함수의 좌표계 일관성**: water_tm / water_plane_location / location 의 좌표 기준이 World / Local 어느 쪽인지 명시 없음 (🟡).
- 베이스 함정 적용.

## 연관 entity
- [[entities/MCWaterVolume]] — 물 볼륨.
- [[entities/MCWaterPlaneComponent]] — 시각화 컴포넌트.
- [[entities/MCBouyancyComponent]] — 부력 계산 페어.
- FWaveParam / FWaveInfo (MCCoreStruct / MCWaterVolume — 미인덱싱).
