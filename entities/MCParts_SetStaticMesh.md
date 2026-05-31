---
title: "UMCParts_SetStaticMesh"
kind: entity
category: GraphNode
base_class: UMCParts_SetMesh
module: MCPLAYMODULE
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCPlayModule/MCParts/MCPartsAsset.h
  - KMCProject/MCPlayModule/MCParts/MCPartsAsset.cpp
vault_refs: []
last_ingested: 2026-05-29
---

# UMCParts_SetStaticMesh

## 한 줄 정의
Parts Static Mesh 설정 노드 — `UMCParts_SetMesh` 자손. `UStaticMesh*` + 런타임 Dynamic Material 캐시.

## 소속 / 상속
- 카테고리: GraphNode
- `UCLASS(BlueprintType, Blueprintable)` `class MCPLAYMODULE_API UMCParts_SetStaticMesh : public UMCParts_SetMesh`

## 핵심 구조
- **UPROPERTY**:
  - `EditAnywhere BlueprintReadWrite UStaticMesh* pStaticMesh` — **명칭 패턴**: `p` prefix (Hungarian) — 컨벤션 불일치.
  - `Transient TArray<UMaterialInstanceDynamic*> DynamicMaterial` — 런타임 머티리얼 캐시.
- 생성자 + `virtual FString GetDescription() const override`.
- `TArray<UMaterialInstanceDynamic*> GetOrCreateDynamicMaterial()` — lazy 생성.
- `#if WITH_EDITOR`:
  - `virtual UObject* GetProperty() override`, `virtual UMaterialInterface* GetNodeMaterial() override`.

## 따르는 패턴
- Loader 가 본 노드를 만나면 `pStaticMesh` 를 SocketName 위치에 부착. GetOrCreateDynamicMaterial 가 머티리얼 인스턴스 생성.
- WITH_EDITOR property/material hook — 디테일 패널 customization 페어 (MCEditorModule).

## ⚠ 함정
- **`pStaticMesh` 명칭 컨벤션 불일치** — UE 표준 `StaticMesh` 권장. BP 노출 변경 시 redirector.
- **DynamicMaterial 가 raw 포인터 TArray** — TObjectPtr 미적용. Transient 라 직렬화는 안 되지만 GC 마커는 UPROPERTY 가 담당.
- 베이스 함정 적용 ([[entities/MCParts_SetMesh]]).

## 연관 entity
- [[entities/MCParts_SetMesh]] — 직접 베이스.
- [[entities/MCParts_SetSkeletalMesh]] / [[entities/MCParts_SetSkinnedMesh]] — 동등 자손.
