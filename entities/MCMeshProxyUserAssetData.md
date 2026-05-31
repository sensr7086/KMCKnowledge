---
title: "UMCMeshProxyUserAssetData"
kind: entity
category: AssetUserData
base_class: (UAssetUserData — 주석 처리, 미컴파일)
module: MCPLAYMODULE
confidence: 🔴 INFERRED
source_paths:
  - KMCProject/MCPlayModule/Actor/Proxy/MCMeshProxyUserAssetData.h
vault_refs: []
last_ingested: 2026-05-29
---

# UMCMeshProxyUserAssetData

## 한 줄 정의
**현재 컴파일 안 되는 placeholder** — 헤더 본문이 통째로 `/* ... */` 주석 처리되어 실제 클래스 정의 없음. 의도된 형태는 mesh proxy 머티리얼 배열을 보관하는 UAssetUserData 자손.

## 소속 / 상속
- 대분류: DataTable (의도된 분류 — 에셋 류 / 머티리얼 배열 보관).
- **현재 상태**: 헤더 7~23 행 전체가 `/* ... */` 블록 주석. UCLASS / GENERATED_BODY / UPROPERTY 모두 비활성. `MCMeshProxyUserAssetData.generated.h` include 만 살아있음.
- 의도된 본문 (주석 안 보존):
  ```cpp
  UCLASS()
  class UMCMeshProxyUserAssetData : public UAssetUserData
  {
      GENERATED_BODY()
  public:
      UMCMeshProxyUserAssetData(const FObjectInitializer& ObjectInitializer) : Super(ObjectInitializer) {}
  public:
      UPROPERTY(EditAnywhere, Category=UMCMeshProxyUserAssetData)
      TArray<UMaterialInterface*> ProxyMaterial;
  };
  ```

## 핵심 구조
- (의도된 — 주석 안):
  - `UPROPERTY(EditAnywhere) TArray<UMaterialInterface*> ProxyMaterial` — proxy 머티리얼 배열.
- **현재 실제 노출 멤버 0** — 클래스 자체가 없음. `#include "...generated.h"` 만 살아 컴파일 통과 (해당 generated 파일은 UHT 가 빈 채로 생성).

## 따르는 패턴
- (의도) UAssetUserData 자손 — Mesh 자산에 ProxyMaterial 배열 첨부.
- (의도) UMCMeshProxyDataAsset (UPrimaryDataAsset, 미인덱싱) 와의 페어 — 본 UserAssetData 가 그 인스턴스 측 데이터 보관 가정 (🔴 추론).
- 가장 가까운 vault 참조 (인덱싱 안 — 추론):
  - `sources/ue-assetclasses-assetuserdata` — UAssetUserData 자손 표준.

## ⚠ 함정
- **컴파일 비활성**: 본 파일을 include 한 다른 코드가 `UMCMeshProxyUserAssetData` 심볼을 참조하면 link error. 현재 다른 KMCProject 코드에서 본 클래스 참조 0건 (🟡 — 본 배치에서 grep 안 함).
- **헤더의 mojibake 주석**: `// Runtime/Engin/UProxyAssetUserData ... ` (CP949 mojibake) — UE 표준 `UProxyAssetUserData` 와 유사한 의도였음을 시사 (🔴 추론).
- **분류 보류 사유**: 클래스 본문이 없으므로 진짜 의도와 다를 수 있음. 향후 본문 복구 시 본 페이지 갱신 필요. CLAUDE.md §5 "분류 불가/모호" 절차 대상.
- **`.cpp` 부재 가정**: 본 배치에서 cpp 미확인. 본문이 주석이라 cpp 도 비어있을 가능성 높음.

## 연관 entity
- UMCMeshProxyDataAsset (미인덱싱 — UPrimaryDataAsset, 동일 폴더 `Actor/Proxy/`). 페어 가정.
- UE 표준 `UProxyAssetUserData` (Runtime/Engine — 본 파일의 출처 추정).
