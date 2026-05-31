# KMCKnowledge Index

> Last ingested: 2026-05-29 · **94 entities / 10 catalogs** · raw = `E:\MCProject\KMCProject` (mode: A — 참조만, Cowork 폴더로 마운트)
> 매 ingest 후 자동 갱신. 생성기 locate 단계의 진입점.
> **카테고리 시스템**: 단일 진실원 = [[categories]] (활성 10 + 예약 1). 본 인덱스는 활성 카테고리만 enumerate.
> **횡단 정책 레이어**: 2026-05-31 도입. SSOT = [[ue-cross-cutting-policies/index]]. entity 양식에 `policy_refs`+§6 추가(STRUCTURE.md §4). Component 카테고리 파일럿 마이그레이션 완료, 그 외 카테고리는 단위별 진행 예정.

---

## 저장소 역할
- 본 저장소 = **KMCProject 전용 지식** (이 프로젝트는 *실제로* 이렇게 짜여 있다)
- UE 일반 지식·함정은 **mcwiki** (MCP) 참조 — vault_refs 로 링크

## 신뢰도 범례
- 🟢 VAULT · 🟡 PARTIAL · 🔴 INFERRED

## 횡단 정책 (cross-cutting — 카테고리와 직교)
- 진입점·SSOT: [[ue-cross-cutting-policies/index]] (mcwiki 미러 / 원본 5.5.4 · 프로젝트 5.7.4)
- entity 급 5종(07 profiling / 09 global-iterator / 10 component / 11 asset-loading / 12 asset-opt)을 entity frontmatter `policy_refs` + 본문 §6 `횡단 정책 준수` 에 기록. 적용 매트릭스 = 정책 index §3.
- 메타 정책 6종(14~19)은 생성기·리뷰 워크플로우용 — entity 에 부착 안 함. 운영 규약: CLAUDE.md §2.7.

---

## 대분류 (catalogs)

### Actor — [[catalogs/actor]] (8)
- 🟢 [[entities/MCPooledActor]] · 🟡 [[entities/MCCharacter]] · 🟡 [[entities/MCCamera]] · 🟡 [[entities/MCWaterVolume]] · 🟡 [[entities/MCCameraBase]] · 🟡 [[entities/MCCameraPlayable]] · 🟢 [[entities/KMCProjectCharacter]] · 🟡 [[entities/KMCProjectGameMode]]

### Component — [[catalogs/component]] (12)
- 🟡 [[entities/MCActorComponent]] · 🟡 [[entities/MCAnimInstance]] · 🟡 [[entities/MCMoveComponent]] · 🟡 [[entities/MCCharacterMovementComponent]] · 🟢 [[entities/MCSoftSkeletalMeshComponent]] · 🟢 [[entities/MCSoftStaticMeshComponent]] · 🟡 [[entities/MCWaterPlaneComponent]] · 🟡 [[entities/MCPartsLoaderComponent]] · 🟡 [[entities/MCBouyancyComponent]] · 🟢 [[entities/MCStaticMeshNiagaraSpawnerComponent]] · 🟢 [[entities/MCLootableComponent]] · 🟢 [[entities/MCInventoryComponent]]

### Subsystem — [[catalogs/subsystem]] (6)
- 🟢 [[entities/MCGameSubsystem]] · 🟢 [[entities/MCActorSpawnSubsystem]] · 🟡 [[entities/MCStorySubSystem]] · 🟡 [[entities/MCCameraSubSystem]] · 🟢 [[entities/MCNiagaraSocketPreviewSubsystem]] · 🟢 [[entities/MCLootSubsystem]]

### Interface — [[catalogs/interface]] (4)
- 🟡 [[entities/MCActorInterface]] · 🟢 [[entities/MCPoolableInterface]] · 🟢 [[entities/MCSpatialQueryFilterable]] · 🟢 [[entities/MCComboPreviewVisitor]]

### DataTable — [[catalogs/datatable]] (5)
- 🟢 [[entities/MCDataBase]] · 🟢 [[entities/MCTableManager]] · 🟢 [[entities/MCTableRegistry]] · 🟢 [[entities/MCData_Sheet1]] · 🟢 [[entities/MCData_LootEntry]] (EMCLootRarity·FMCLootResultItem 동거)

### AssetUserData — [[catalogs/assetuserdata]] (3)
- 🟢 [[entities/MCNiagaraSocketBindings]] · 🟢 [[entities/MCHitBoneCurveUserData]] · 🔴 [[entities/MCMeshProxyUserAssetData]]

### Asset — [[catalogs/asset]] (6)
- 🟡 [[entities/MCStoryAsset]] · 🟡 [[entities/MCPartsAsset]] · 🟡 [[entities/MCStoryBoard]] · 🟢 [[entities/MCComboAsset]] · 🔴 [[entities/MCMeshProxyDataAsset]] · 🟢 [[entities/MCLootTableAsset]] (CP-1 Option B)

### GraphNode — [[catalogs/graphnode]] (45)
**Story (12)**: 🟡 [[entities/MCStory_Graph]] · 🟢 [[entities/MCStory_Node]] · 🟡 [[entities/MCStory_DecortorBase]] · 🟡 [[entities/MCStory_Entry]] · 🟡 [[entities/MCStory_Condition]] · 🟢 [[entities/MCStory_DecoratorCondition]] · 🟡 [[entities/MCStory_Sequence]] · 🟡 [[entities/MCStory_Action]] · 🟡 [[entities/MCStory_Comment]] · 🟡 [[entities/MCStory_ActionAddParts]] · 🟡 [[entities/MCStory_ActionVisibleParts]] · 🟡 [[entities/MCStory_ActionRunStroy]]
**Parts (20)**: 🟡 [[entities/MCParts_Graph]] · 🟢 [[entities/MCParts_Node]] · 🟡 [[entities/MCParts_Comment]] · 🟡 [[entities/MCParts_MeshParamter]] · 🟡 [[entities/MCParts_SetMesh]] · 🟡 [[entities/MCParts_SetStaticMesh]] · 🟡 [[entities/MCParts_SetSkeletalMesh]] · 🟡 [[entities/MCParts_SetSkinnedMesh]] · 🟡 [[entities/MCParts_AddVirtualSocket]] · 🟡 [[entities/MCParts_AddNiagara]] · 🟡 [[entities/MCParts_AddSubStaticMesh]] · 🟡 [[entities/MCParts_AddSubSkeletalMesh]] · 🟡 [[entities/MCParts_AddDecal]] · 🟡 [[entities/MCParts_SetOverlayMaterial]] · 🟡 [[entities/MCParts_SkinMeshDeformer]] · 🟡 [[entities/MCParts_Constraint]] · 🟡 [[entities/MCParts_AddSplineCurve]] · 🟡 [[entities/MCParts_Parameter]] · 🟡 [[entities/MCParts_MaterialParameter]] · 🟡 [[entities/MCParts_TransformParameter]]
**Combo (13)**: 🟢 [[entities/MCComboBinding]] · 🟢 [[entities/MCComboTrack]] · 🟢 [[entities/MCComboSection]] · 🟢 [[entities/MCComboAudioTrack]] · 🟢 [[entities/MCComboAudioSection]] · 🟢 [[entities/MCComboInputTrack]] · 🟢 [[entities/MCComboInputSection]] · 🟢 [[entities/MCComboMontageTrack]] · 🟢 [[entities/MCComboMontageSection]] · 🟢 [[entities/MCComboNotifyTrack]] · 🟢 [[entities/MCComboNotifySection]] · 🟢 [[entities/MCComboTransformTrack]] · 🟢 [[entities/MCComboTransformSection]]

### BlueprintLibrary — [[catalogs/blueprintlibrary]] (4)
- 🟡 [[entities/MCActorBlueprintLibrary]] — Actor 점프/접지.
- 🟢 [[entities/MCSpatialQueryLibrary]] — Octree + Cone + LOS 필터 5종 + FMCSpatialQueryFilter USTRUCT.
- 🟡 [[entities/MCWaterBlueprintLibrary]] — 파동 높이 계산 5종.
- 🟢 [[entities/MCLootRoller]] — 결정적 루팅 굴림 single static Roll (FRandomStream, 보장/가중치 2단계).

### Widget — [[catalogs/widget]] (1) **NEW**
- 🟢 [[entities/MCLootToastWidget]] — `UUserWidget` Abstract. 루팅 결과 토스트 베이스, ShowResult → BP OnShowResult 위임. (Build.cs UMG/Slate/SlateCore 추가 = 사용자 수동)

---

## 예약된 대분류 ([[categories]] 참조)
- **Editor** — MCEditorModule (다수, 정책 결정 필요 — 단일 Editor 카테고리 vs Editor.* 서브카테고리)

---

## 미인덱싱 큐

### MCEditorModule → Editor (예약 신설 시점 정책 결정 필요)
- **MCStoryEditor**: F\*AssetAction / F\*EditorApplication / F\*EditorApplicationMode / UMCStoryAssetFactory / U\*GraphSchema / U\*GraphNode 자손 8 / F\*Tab\*Factory 4 / S\*GraphNode 자손 5 / F\*ConnectionDrawingPolicy 1.
- **MCPartsEditor**: 유사 패턴, 노드 19+ / S\*GraphNode 자손 13.
- **MCComboEditor**: F\*AssetAction / F\*EditorApplication / S\*OutlinerView / S\*TrackArea / S\*Timeline / S\*PreviewSceneViewport 등 다수.
- **HitBoneCurveEditor**: F\*EditorMenu / SMCHitBoneCurveEditor.
- **MCDataTableAuto**: UMCDataTableAutoSubsystem(UEditorSubsystem) + FMCDataTableAutoMcpServer / FMCDataTableAutoClaudeProcess / FMCDataTableAutoMcwikiResolver / SMCDataTableAutoWidget / UMCDataTableAutoSettings(UDeveloperSettings) 등.
- **MCMaterialAuto**: 유사 패턴 + UMCMaterialAutoLibrary(UBlueprintFunctionLibrary).
- **DetailView**: F\*Details 자손 다수 (IDetailCustomization / IPropertyTypeCustomization).
- **Subsystem**: SMCBindingsListWidget.

---

## 다음 배치 (Story → Parts → Combo → 소묶음 → **Editor**)

- **마지막**: MCEditorModule (Editor 카테고리 정책 결정 후 진행). 예상 entity 70+ — 단일 Editor catalog vs Editor.GraphNode/Editor.Schema/Editor.Widget 등 서브카테고리 분리 결정 필요.
- **이 에이전트 안 완주 약속 — Editor 진행 후 완료**.
