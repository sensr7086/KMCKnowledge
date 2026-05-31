---
title: "UMCMeshProxyDataAsset"
kind: entity
category: Asset
base_class: (UPrimaryDataAsset — 주석 처리, 미컴파일)
module: MCPLAYMODULE
confidence: 🔴 INFERRED
source_paths:
  - KMCProject/MCPlayModule/Actor/Proxy/MCMeshProxyDataAsset.h
vault_refs: []
last_ingested: 2026-05-29
---

# UMCMeshProxyDataAsset

## 한 줄 정의
**현재 컴파일 안 되는 placeholder** — 헤더 본문이 통째로 `/* ... */` 주석. 의도는 mesh proxy 머티리얼 배열을 보관하는 UPrimaryDataAsset. [[entities/MCMeshProxyUserAssetData]] 와 페어 placeholder.

## 소속 / 상속
- 카테고리: Asset (의도된 분류)
- **현재 상태**: 헤더 8~19 행 전체가 `/* ... */` 블록 주석. UCLASS / GENERATED_BODY / UPROPERTY 모두 비활성.
- 의도된 본문 (주석 안 보존):
  ```cpp
  UCLASS(MinimalAPI)
  class UMCMeshProxyDataAsset : public UPrimaryDataAsset
  {
      GENERATED_BODY()
  public:
      UPROPERTY(EditAnywhere, Category = MCMeshProxyDataAsset)
      TArray<UMaterialInterface*> ProxyMaterial;
  };
  ```
- 헤더 주석 mojibake (CP949): `Runtime/Engin/UProxyAssetUserData...` — 표준 `UProxyAssetUserData` 와 유사한 의도 추정.

## 핵심 구조
- (의도 — 주석 안):
  - `UPROPERTY(EditAnywhere) TArray<UMaterialInterface*> ProxyMaterial`.
- **현재 실제 노출 멤버 0** — 클래스 자체 없음.

## 따르는 패턴
- (의도) UPrimaryDataAsset 자손 — Asset Manager 통합 가능.
- (의도) [[entities/MCMeshProxyUserAssetData]] 와의 페어 — DataAsset 이 prop 정의, UserAssetData 가 인스턴스 측 데이터 보관 (🔴 추론).

## ⚠ 함정
- **컴파일 비활성**: 본 파일 include 후 심볼 참조 시 link error. 현재 다른 KMCProject 코드 참조 0건 (🟡 — 본 배치에서 grep 안 함).
- **MCMeshProxyUserAssetData 와 동일 패턴 — 페어 placeholder**: 두 파일 모두 본문 주석. 함께 복구 / 함께 삭제 결정 필요.
- **`MinimalAPI` specifier** — 모듈 외부 export 최소화.
- **분류 보류 사유**: 클래스 본문 없으므로 의도와 다를 가능성. 본문 복구 시 본 페이지 갱신 필요.

## 연관 entity
- [[entities/MCMeshProxyUserAssetData]] — 페어 placeholder (동일 상태).
- UE 표준 `UProxyAssetUserData` (Runtime/Engine — 본 파일 출처 추정).
