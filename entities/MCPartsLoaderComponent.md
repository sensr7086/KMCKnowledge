---
title: "UMCPartsLoaderComponent"
kind: entity
category: Component
base_class: UMCActorComponent
module: MCPLAYMODULE
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCPlayModule/Actor/Component/MCPartsLoaderComponent.h
  - KMCProject/MCPlayModule/Actor/Component/MCPartsLoaderComponent.cpp
vault_refs: []
policy_refs:
  - component-policies
  - asset-loading-policy
  - asset-optimization-policy
last_ingested: 2026-05-29
---

# UMCPartsLoaderComponent

## 한 줄 정의
UMCPartsAsset (Parts 그래프) 의 런타임 소비자 — 그래프를 walk 하면서 메시·소켓·Niagara·제약·Decal·오버레이 머티리얼을 대상 Actor 에 적용.

## 소속 / 상속
- 대분류: Component
- `UCLASS(Blueprintable, meta = (BlueprintSpawnableComponent))` `class MCPLAYMODULE_API UMCPartsLoaderComponent : public UMCActorComponent`
- 매크로: `MCCOMPONENT_DEF(UMCPartsLoaderComponent, EMCComponentType, EMCComponentType::MCPartsLoader)`

## 핵심 구조
- **UMCActorComponent hook 오버라이드**: `Init() / Update(float) / End()`
- **public API (Parts 적용 / 가시성 / 제거)**:
  - `void AddPartsLoadSync(const FName& part_name, const FName& asset_path)` — 경로 문자열 기반.
  - `void AddPartsLoadSync(const FName& part_name, TSoftObjectPtr<UMCPartsAsset> partsAsset)` — Soft 참조 기반.
  - `void VisibleParts(const FName& parts_name, bool bVisible)`
  - `void RemoveParts(const FName& parts_name)`
  - `void ClearAll()`
- **protected — Parts 그래프 walk 단계별 구현**:
  - `bool internal_LoadSync(const UMCPartsAsset*, FPartsInfomation&)`
  - `bool internal_LoadSync(UMCParts_Node*, const TMap<FGuid, UMCParts_Node*>& allNodes, FPartsInfomation&)`
  - `void Internal_Parameter(UMCParts_Node*, UMCParts_Node* param, FPartsInfomation&)`
  - `void LoadStaticMesh(UMCParts_Node*, const FName& socket, UStaticMesh*, FPartsInfomation&)`
  - `void LoadSkinnedMesh(UMCParts_Node*, USkeletalMesh*, FPartsInfomation&)`
  - `void AddVirtualSocket(UMCParts_Node* parent, UMCParts_Node*, const FName& socketname, const FName& parent_socket, const FTransform&, FPartsInfomation&)`
  - `void AddSubStaticMesh(..., UStaticMesh*, ...)`, `AddSubSkeletalMesh(..., USkeletalMesh*, ...)`
  - `void AddNiagara(..., UNiagaraSystem*, ...)`, `AddDecal(..., FPartsNodeDecalParam&, ...)`
  - `void SetOverlayMaterial(UMCParts_Node* parent, UMCParts_SetOverlayMaterial*, FPartsInfomation&)`
  - `void AddPhysicsConstraint(UMCParts_Node* parent, UMCParts_Node*, FPartsInfomation&)`
- **private**: `USkeletalMeshComponent* GetOwnerSkeletalMesh()`
- **protected 중첩 구조체 `FPartsInfomation`** (sic Infomation):
  - `TArray<TWeakObjectPtr<UMeshComponent>> TopAllMeshComponents` — 최상위 메시 리스트.
  - `TMap<FGuid, TWeakObjectPtr<UMeshComponent>> AllMeshComponents` — GUID→메시(노드별 새 메시 보관 가정, 🟡).
  - `TMap<FGuid, FMCRuntimeVirtualSocket> ArrVirualSocket` (sic Virual) — GUID→가상 소켓.
  - `TMap<FGuid, TWeakObjectPtr<UPhysicsConstraintComponent>> AllConstraintComponents`
  - `TMap<FGuid, FPartsRuntimeDynamicMaterial> AllDynamicMaterial`
  - `TArray<UActorComponent*> OtherComponents` — 그 외 컴포넌트, 여기서 정리(Release) 책임.
  - `TArray<FMCRuntimeReservedConstranit> RuntimeConstranit` (sic Constranit)
  - 메서드: `HasVirtualSocket(FGuid)`, `GetVirtualSocketPtr(FGuid)`, `ParsingMaterialForMesh(FGuid, UMeshComponent*)`, `GetPartsDynamicMaterial(FGuid)`, `MakeConstranit(const TMap<FGuid, UMCParts_Node*>&)`, `SetVisible(bool)`, `Release()`.
- **private 컨테이너**: `TMap<FName, FPartsInfomation> PartsContainer` — part_name → 적용 상태.

## 따르는 패턴
- Parts 그래프 → 런타임 컴포넌트 트리 매핑. GUID 기반 노드 추적 (🟢 — FGuid 키 TMap 들).
- TWeakObjectPtr 로 메시·constraint 컴포넌트 보관 — 외부 컴포넌트 동적 소멸 안전 (🟢 — TWeakObjectPtr 사용).
- TopAllMeshComponents / AllMeshComponents 분리 — 최상위와 전체 트리를 둘 다 관리 (🟡 추론).
- `Release()` 가 OtherComponents 정리 책임 — UPartsLoader 가 추가한 컴포넌트는 모두 등록되어야 ClearAll/RemoveParts 시 누락 없음 (🟡 추론).

## ⚠ 함정
- 명칭 오타 다수: `Infomation`, `Virual`, `Constranit` — 향후 정리 시 redirector 필요할 수 있음.
- AddPartsLoadSync 가 **sync** — Soft 자산 로드를 동기 처리 → 첫 호출 시 hitching 가능 (🟡 추론). Cooked 빌드 SpawnActor 히칭 함정(베이스 [[entities/MCSoftStaticMeshComponent]]/[[entities/MCSoftSkeletalMeshComponent]] 의 패턴) 과 상충.
- `AllMeshComponents` 가 raw 메시 컴포넌트들을 TWeak 로 보관 — UPROPERTY 마커 없음. UPartsLoader 가 NewObject 로 만든 컴포넌트라면 별도 UPROPERTY 멤버나 OtherComponents 등록으로 GC 방어 필요 (🟡 — Release() 가 이 역할 가정).
- 그래프 walk 중 노드 GUID 충돌 시 후속 노드가 앞 노드 매핑 덮어쓰기 가능 (🟡).

## 횡단 정책 준수
> 적용: [[ue-cross-cutting-policies/index]] §3 (Component 행). raw 미마운트 — 추출 본문 근거, 미확인은 ❓.

| 정책 | 적용 | 근거 / 위반·미확인 | 신뢰도 |
|---|---|---|---|
| 11 asset-loading | ✅ | `AddPartsLoadSync` 가 **동기 로드** — 첫 호출 hitching(함정 기재) → 11 정책 "동기 로드 회피"와 **상충 ❌**. 비동기(FStreamableManager) 전환 권장. → [[ue-cross-cutting-policies/11_AssetLoadingPolicy]] | 🟡 |
| 10 component | ✅ | 베이스 MCActorComponent 6대. 동적 부착 컴포넌트는 `OtherComponents`/`Release()` 정리 + 메시 TWeakObjectPtr 보관 — **NewObject 생성물 GC 마커(UPROPERTY) ❓**(§3 위반 후보). | 🟡 |
| 12 asset-opt | △ | 조립 파츠(메시/Decal/Niagara) 최적화는 **참조 자산 측**(각 자산 entity). | 🟡 |
| 07 profiling | △ | 조립은 이벤트성 추정 — `Update` 매 프레임 사용 여부 ❓. | ❓ |
| 09 global-iterator | ➖ | 미사용(GUID TMap 직접 보유). | 🟢 |

## 연관 entity
- [[entities/MCActorComponent]] — 직접 베이스.
- [[entities/MCCharacter]] — Owner 인 경우 GetOwnerSkeletalMesh().
- [[entities/MCPartsAsset]] — 본 컴포넌트가 런타임 소비하는 자산.
- [[entities/MCParts_Graph]] — 노드 컬렉션.
- [[entities/MCParts_Node]] — 노드 베이스.
- [[entities/MCParts_SetOverlayMaterial]] — SetOverlayMaterial 호출 대상.
- 그 외 Parts 노드 16개 — Walk 시 각각 핸들러 분기.
- FMCRuntimeVirtualSocket / FPartsRuntimeDynamicMaterial / FMCRuntimeReservedConstranit / FPartsNodeDecalParam (MCCoreStruct — 미인덱싱)
