---
title: "횡단 정책 인덱스 (UE Cross-Cutting Policies)"
kind: policy-index
source: mcwiki — UE Knowledge Vault (MCP)
source_version: "미러 원본 UE 버전 (본 미러는 5.5.4 계열 — 재미러 시 갱신)"
project_version: "→ project.config.md 의 UE_VERSION (대상 프로젝트 버전)"
mirrored: 2026-05-31
confidence: 🔴 INFERRED   # 정책 본문 = 일반 UE 지식. 대상 프로젝트(project.config.md) 검증 전.
last_updated: 2026-05-31
---

# 횡단 정책 인덱스 (Cross-Cutting Policies)

> 본 폴더(`ue-cross-cutting-policies/`)는 **mcwiki 에서 사용 중인 횡단 정책을 KMCKnowledge 로 미러한 것**이다.
> 카테고리(Actor/Component/...)는 "이 클래스가 *무엇인가*"를 가르고, 횡단 정책은 "어떤 카테고리든 코드를 짤 때 *공통으로 지켜야 할 것*"을 가른다. 즉 **카테고리와 직교(orthogonal)** 한다.
> 본 인덱스는 정책 폴더의 진입점 — ingest 와 생성기 locate 가 "이 entity 에 어떤 정책이 걸리는가"를 여기서 판정한다. (`index.md` ↔ entity, `categories.md` ↔ 카테고리 와 같은 역할.)

---

## 0. 성격과 경계 (먼저 읽기)

| 구분 | 본 폴더의 정책 | 본 저장소 entity/catalog |
|---|---|---|
| 무엇 | UE 일반 규약 (모든 UE 프로젝트 공통) | **대상 프로젝트 고유** 구현 지식 |
| 출처 | mcwiki 미러 (일반 지식) | 대상 프로젝트 코드 추출 |
| 기본 신뢰도 | 🔴 (일반 지식, 프로젝트 검증 전) | 🟢/🟡 (코드 근거) |
| 누가 SSOT | **원본은 mcwiki** — 본 폴더는 작업 편의용 사본 | 본 저장소 |

> **프로젝트 중립 원칙**: 본 폴더는 **일반 UE 지식**이라 *어느 프로젝트에도 종속되지 않는다*. 대상 프로젝트명·엔진 버전 등 인스턴스 값은 본문에 박지 않고 **`project.config.md`(PROJECT_NAME / UE_VERSION)** 를 단일 진실원으로 참조한다. (본 인스턴스 = KMCProject / UE 5.7.4.) 다른 프로젝트가 이 폴더를 베이스로 재사용할 때 본문 수정 없이 그대로 가져갈 수 있어야 한다.
>
> ⚠ **경계 주의 (CLAUDE.md §2.1 연계)**: 정책 *본문*은 mcwiki 가 단일 진실원인 **일반 지식**이다. 본 폴더는 ingest 가 매번 mcwiki 를 왕복하지 않도록 둔 **로컬 사본**일 뿐, 정책 본문을 이 저장소의 "프로젝트 지식"으로 승격하지 않는다.
> 반대로, **"이 대상 프로젝트 클래스가 정책을 지키는가/어기는가"** 라는 *판정 결과*는 코드 근거에 기반한 **프로젝트 지식**이며, 그것은 각 entity 의 `횡단 정책 준수` 섹션에 🟢/🟡/❓ 로 기록된다.

---

## 1. Provenance & Staleness (정합성 경고)

- **미러 원본 버전**: mcwiki (UE Knowledge Vault MCP). 본문 곳곳에 `5.5.4 트리 검증` 표기 → 본 미러는 **UE 5.5.4 계열**. (이 값은 미러 자체의 속성 — 재미러 시 갱신.)
- **대상 프로젝트 버전**: `project.config.md` 의 `UE_VERSION` 을 따른다 (본 인스턴스 = **UE 5.7.4**). → 미러 원본과 **버전 차이가 있으면 staleness 검토 대상**.
- **staleness 처리**: 버전 차이로 일부 정책이 stale 일 수 있다 (예: 5.x Nanite/WorldPartition/Enhanced Input/TObjectPtr 표준화). [[ue-cross-cutting-policies/18_ModelEvolutionAudit]] 의 감사 기준으로 분기별 재검토. 정책을 대상 프로젝트에 적용해 충돌·불일치가 나오면 즉시 감사 트리거.
- **미러 일자**: 2026-05-31. mcwiki 원본 갱신 시 본 사본은 자동 동기화되지 않음 — 재미러 책임은 운영자.

### 알려진 import 아티팩트 (교정 이력 + 잔존)

mcwiki 에서 미러하며 딸려온 링크 아티팩트. **✅ 교정됨 / ⏳ 미러 큐 / 📐 정규화 키로 해소** 로 구분.

- ✅ **deep 링크 경로 (10·11)** — `10_ComponentPolicies`·`11_AssetLoadingPolicy` 본문이 `./references/*Deep.md` 를 가리키던 것을 실제 위치 `./deep/*Deep.md` 로 일괄 교정(2026-05-31). 타깃 `deep/ComponentPoliciesDeep.md`·`deep/AssetLoadingDeep.md` 실재 확인.
- ✅ **12 deep (AssetOptimizationDeep)** — `deep/AssetOptimizationDeep.md` 추가 미러 완료(2026-05-31, 509줄). §1~§5 헤더(SkeletalMesh Bone LOD / StaticMesh LOD / Actor Merging / Audio Culling / Niagara Quality Scaling) 앵커가 12 본문 링크 5개와 일치 확인. 12 본문의 ❓ 주석 제거(verbatim 복귀). frontmatter `name: asset-optimization-deep`.
- ⏳ **결번 정책** — `02_VerificationLog` `03_WikiHarness` `04_OverrideIndex` `05_EditorOnlyIndex` `06_InvalidationHotspots` `08`/`13` 미러 안 됨. mcwiki 횡단 인덱스 중 일부(메타·하네스)만 가져옴. 필요 시 추가 미러.
- 📐 **19 frontmatter 없음** — `19_ExternalSourcesGuide.md` 는 `name:` 프런트매터 부재 → 본 인덱스가 슬러그 `external-sources-guide` 부여(§2.2).

### 링크 정규화 키 (본문 verbatim 보존 — 경로는 아래 규약으로 해석)

정책 본문은 mcwiki 미러라 **verbatim 보존**한다. 따라서 본문 안의 아래 경로는 *재작성하지 않고* 다음 규약으로 해석한다 — ingest·locate 가 링크를 따라갈 때 이 키를 적용.

| 본문에 나오는 형태 | 실제 의미 | 따라가는 법 |
|---|---|---|
| `../skills/<X>` (예: `../skills/Components/references/ActorComponent.md`) | **mcwiki 소스 트리** 경로 (KMCKnowledge 에 없음) | mcwiki MCP `read_page` 로 조회. 로컬 파일 아님. |
| `NN_*.md` 중 **미러된 것** (07·09·10·11·12·14~19) | 본 폴더의 형제 정책 | `[[ue-cross-cutting-policies/NN_*]]` 로 로컬 resolve |
| `NN_*.md` 중 **비미러** (02·03·04·05·06·08·13) | mcwiki 횡단 인덱스의 미미러 정책 | mcwiki 참조 또는 미러 큐(위) — 로컬에 없음 |
| `./deep/*Deep.md` | 본 폴더 `deep/` 의 progressive-disclosure 본문 | 로컬 resolve (3종 모두 실재: Component·AssetLoading·AssetOptimization) |
| `CLAUDE.md §x` (정책 본문 내) | **mcwiki 쪽** CLAUDE/하네스 문서 | 본 저장소 루트 CLAUDE.md 와 혼동 금지 — mcwiki 메타 |
| `<외부>/...` (14 TaskHandoff) | 사용자 외부 작업 폴더 placeholder | 의도된 자리표시자 — 교정 대상 아님 |

> 왜 본문을 직접 안 고치나: `../skills/*` 등은 KMCKnowledge 에 대응 파일이 없어 로컬 경로로 "교정"이 불가능하다(애초에 mcwiki 자산). 수백 개 링크를 재작성하면 미러 충실성(§0)이 깨지고 오류 위험만 커진다. 그래서 **로컬에 실파일이 있는 deep 링크만 교정**하고, 나머지는 본 키로 해석한다.

---

## 2. 정책 로스터

### 2.1 entity 급 정책 (entity 에 직접 부착 — `policy_refs` 대상)

> ingest 가 각 entity 를 평가하고 `횡단 정책 준수` 섹션에 기록하는 5종.

| ID | 슬러그 (`policy_refs` 값) | 파일 | 한 줄 규약 |
|---|---|---|---|
| 07 | `profiling-scope-rule` | [[ue-cross-cutting-policies/07_ProfilingScopeRule]] | 매 프레임·타이머·바인딩 UFunction·시간 람다 **첫 줄에 프로파일링 스코프**. |
| 09 | `global-iterator-policy` | [[ue-cross-cutting-policies/09_GlobalIteratorPolicy]] | `TActorIterator`/`TObjectIterator`/`TObjectRange`/`TActorRange` **사용 금지**(등록 패턴 대체, 최후의 수단만). |
| 10 | `component-policies` | [[ue-cross-cutting-policies/10_ComponentPolicies]] | 컴포넌트 **6대 의무**(Mobility / NewObject·Outer / GC(UPROPERTY+TObjectPtr) / GetOwner 캐싱 / Tick OFF 기본 / CDO 안전). |
| 11 | `asset-loading-policy` | [[ue-cross-cutting-policies/11_AssetLoadingPolicy]] | Soft/Hard Reference + 비동기 로드 + SpawnActor 히칭 방지 4단 (Constructor 로드 금지). |
| 12 | `asset-optimization-policy` | [[ue-cross-cutting-policies/12_AssetOptimizationPolicy]] | 자산 최적화 5대(SkeletalMesh Bone LOD / StaticMesh LOD·Nanite / Actor Merging / Audio Culling / Niagara Scalability). |

### 2.2 메타·워크플로우 정책 (entity 에 부착하지 않음)

> 코드를 *짜는 방식*·*검증하는 방식*을 규율. 생성기(design/implement)·리뷰어(review/validate)·ingest 워크플로우가 참조. entity `policy_refs` 에는 넣지 않는다.

| ID | 슬러그 | 파일 | 역할 |
|---|---|---|---|
| 14 | `task-handoff-template` | [[ue-cross-cutting-policies/14_TaskHandoffTemplate]] | 멀티 세션 작업 인계(`_HANDOFF_*.md`). context reset > compaction. |
| 15 | `evaluator-recipe` | [[ue-cross-cutting-policies/15_EvaluatorRecipe]] | Generator/Evaluator 분리 — 회의적 평가 표준. (review-ue-cpp / validate-compliance 연계) |
| 16 | `policy-priority` | [[ue-cross-cutting-policies/16_PolicyPriority]] | 정책 충돌 시 우선순위(Tier 1 빌드 → 2 GC/메모리 → ...). **정책 적용 중 모순 나면 여기로.** |
| 17 | `quality-criteria` | [[ue-cross-cutting-policies/17_QualityCriteria]] | 코드 품질 4기준(Performance/Memory/Network/Maintainability) + 측정 채널. |
| 18 | `model-evolution-audit` | [[ue-cross-cutting-policies/18_ModelEvolutionAudit]] | 정책 staleness 분기 감사. **본 폴더 버전 차이(§1) 관리 주체.** |
| 19 | `external-sources-guide` | [[ue-cross-cutting-policies/19_ExternalSourcesGuide]] | 위키에 없을 때 외부 소스 우선순위·인용. (본 저장소 locate fallback §3.2 와 유사) |

### 2.3 deep (progressive disclosure 본문)

| 파일 | 모(母) 정책 |
|---|---|
| `deep/ComponentPoliciesDeep.md` | 10 ComponentPolicies (6대 정책 상세 코드/함정/결정 트리) |
| `deep/AssetLoadingDeep.md` | 11 AssetLoadingPolicy (FStreamableManager / AssetManager / PreLoad 5대 / 함정 12종) 상세 |
| `deep/AssetOptimizationDeep.md` | 12 AssetOptimizationPolicy (§1~§5: Bone LOD / StaticMesh LOD·Nanite / Actor Merging / Audio Culling / Niagara Scaling) 상세 |

---

## 3. 카테고리 × entity 급 정책 적용 매트릭스 (SSOT)

> ingest 는 entity 의 **카테고리 행**을 보고, ✅/△ 인 정책을 평가해 `횡단 정책 준수` 섹션에 적는다.
> **범례** — ✅ 항상 평가 · △ 조건부(아래 §4 트리거 충족 시) · ➖ 보통 해당 없음(단, "없음 확인"도 🟢 로 1줄 기록).

| 카테고리 | 07 Profiling | 09 GlobalIter | 10 Component6 | 11 AssetLoad | 12 AssetOpt |
|---|---|---|---|---|---|
| **Actor** | ✅ Tick/Timer | ✅ | ➖¹ | ✅ Spawn/asset | △ 메시 소유 시 |
| **Component** | ✅ TickComponent | ✅ | ✅ **핵심** | ✅ asset 멤버 | △ Mesh/Niagara/Audio 컴포넌트 |
| **Subsystem** | △ Tickable/FTSTicker | ✅ **관리자=최우선** | ➖¹ | ✅ Spawn/로드 | ➖ |
| **Interface** | ➖ | ➖ | ➖ | ➖ | ➖ |
| **DataTable** | ➖ | ➖ | ➖ | △ 행이 asset 참조 | △ 참조 자산 |
| **AssetUserData** | ➖ | ➖ | ➖ | △ 참조 asset Soft | △ Mesh/Niagara 메타 |
| **Asset** | ➖ | ➖ | ➖ | ✅ 자산 참조 | ✅ Mesh/VFX/Audio 자산 |
| **GraphNode** | △ 런타임 실행 노드 | △ | ➖ | ✅ Parts_* 메시/머티/VFX 참조 | △ 참조 자산 |
| **BlueprintLibrary** | ➖ 정적 | ✅ 정적 헬퍼 전역검색 위험 | ➖ | △ 로드 헬퍼면 | ➖ |
| **Widget** | ✅ NativeTick | △ | ➖² | ✅ Texture/Font/Brush/BP클래스 Pre-load | △ UI 텍스처 |

¹ **10번 주석**: 6대 정책은 *컴포넌트 전용*이 원칙. 단 하위 규칙 중 **GC 방어(UPROPERTY+TObjectPtr)·NewObject Outer·CDO 안전**은 UObject 를 생성·보유하는 Actor/Subsystem 에도 준용 가능(이 경우 🟡 로 "준용" 명시).
² **Widget**: UMG 위젯은 별도 라이프사이클(CreateWidget/BindWidget). 컴포넌트 6대는 직접 적용 안 됨.

---

## 4. 트리거 시그널 (ingest 가 △/✅ 를 켜는 코드 신호)

| 정책 | 트리거 (헤더/구현 grep) |
|---|---|
| 07 Profiling | `Tick(`·`TickComponent(`·`NativeTick(`·`NativeUpdateAnimation(`·`SetTimer`·`FTSTicker`·`OnRep_`·`AddDynamic`/바인딩 UFUNCTION·시간 기반 람다 |
| 09 GlobalIterator | `TActorIterator`·`TObjectIterator`·`TObjectRange`·`TActorRange` (1건이라도 = 위반 후보) |
| 10 Component6 | `category: Component` (= `UActorComponent`/`USceneComponent`/`UAnimInstance` 자손) |
| 11 AssetLoading | asset 멤버: `UStaticMesh*`·`USkeletalMesh*`·`UMaterial*`·`UTexture*`·`USoundBase*`·`UNiagaraSystem*`·`TSubclassOf<>`·`TSoftObjectPtr<>`·`TSoftClassPtr<>` / 또는 `SpawnActor`·`LoadObject`·`LoadSynchronous` 호출 |
| 12 AssetOpt | 위 asset 중 **Mesh/VFX/Audio** 자산을 소유·참조 (LOD/Nanite/Niagara EffectType 관련) |

> 트리거가 전혀 없으면 ➖(해당 없음) 으로 1줄 기록 — 빈칸 금지(CLAUDE.md §2.5).

---

## 5. 링크·표기 규약

- **frontmatter `policy_refs:`** — entity 에 적용되는 **entity 급 정책의 슬러그** 리스트만. 예:
  ```yaml
  policy_refs:
    - component-policies
    - profiling-scope-rule
    - asset-loading-policy
  ```
  (➖ 해당 없음 정책은 `policy_refs` 에 넣지 않고, 본문 표에서만 ➖ 로 1줄.)
- **본문 wiki 링크** — `[[ue-cross-cutting-policies/10_ComponentPolicies]]` (파일 기준, 접두 번호 포함).
- **준수 신뢰도** — 🟢 코드 근거 명확 · 🟡 추론 · ❓ entity/코드 근거로 판단 불가(미확인, 사용자 검토 대상). 정책 *본문* 자체의 🔴 와 구분.
- **근거 출처 마커 (§6.1 직접 코드 확인과 짝)** — 준수 근거가 *어디서* 왔는지 병기:
  - 🟢 **VAULT** — entity 페이지(정제된 KB)에 근거 있음.
  - 🔍 **RAW** — entity 없음/부족 → **raw(KMCProject 코드) 직접 확인 완료**. 근거 칸에 `경로:line` 명시.
  - 🔍❓ **RAW-PENDING** — raw 직접 확인이 *필요하나 아직 안 함*(raw 미마운트 등). 사용자 검토 대상.
  - 🔴 **INFERRED** — raw 도 없어 정책 일반 지식으로만 메움(프로젝트 선례 없음).

---

## 6. ingest·생성기에서의 사용

- **ingest (코드→저장소)**: entity 작성 시 §3 매트릭스로 적용 정책을 고른 뒤, §4 트리거를 코드에서 확인해 `횡단 정책 준수` 섹션을 채운다. 절차는 `.claude/skills/ingest-kmc/references/cross-cutting-policies.md` 참조.
- **생성기 locate (저장소→계획)**: entity 의 `policy_refs` + `횡단 정책 준수` 를 읽어, **그 클래스를 수정·확장할 때 강제할 정책**을 안다. 위반(❌)·미확인(❓) 항목은 plan 체크포인트의 집중 검토 대상.
- **충돌**: 정책끼리 모순 시 [[ue-cross-cutting-policies/16_PolicyPriority]] 우선순위 트리.

### 6.1 정책·entity 둘 다 없는 케이스 — 직접 코드 확인(raw fallback) 체크 (의무)

> **추측 금지 원칙(CLAUDE.md §2.2)의 구체화.** 어떤 클래스/요구사항에 대해 **(a) 적용 entity 페이지가 없고 (b) §3 매트릭스로도 정책 판정이 안 서면**, KB 만으로 답하지 말고 **반드시 raw(KMCProject 코드)를 직접 확인**한 뒤, 그 사실을 마커로 남긴다.

**판정 순서 (CLAUDE.md §3.2 locate 4단계와 동일 축):**

```
적용 entity 있나?
├─ 예 → entity 의 policy_refs + §6 사용 (출처 🟢 VAULT)
└─ 아니오 → §3 매트릭스로 카테고리 정책 판정되나?
            ├─ 예 → 정책 적용. 단 "이 클래스가 실제로 지키나"는 raw 확인 필요
            │        ├─ raw 확인함 → 🔍 RAW  (근거 `경로:line`)
            │        └─ raw 못 봄  → 🔍❓ RAW-PENDING (사용자 검토)
            └─ 아니오(정책도 entity 도 없음) → 🚨 직접 코드 확인 의무
                     ├─ raw grep·읽기로 실제 구현 확인 → 🔍 RAW (근거 명시)
                     └─ raw 에도 선례 없음 → 🔴 INFERRED (mcwiki 일반 지식, 사용자 검토)
```

**기록 규약:**
- locate-report / entity §횡단 정책 준수 / plan 의 해당 항목에 위 마커(🟢 / 🔍 / 🔍❓ / 🔴)를 **반드시** 표기.
- 🔍 RAW 는 **무엇을 봤는지 증거**를 남긴다 — `KMCProject/.../Foo.h:123` 또는 grep 패턴. "직접 봤다"는 주장만으로는 🔍 를 못 붙인다.
- 🔍❓ 와 🔴 는 plan 체크포인트의 **집중 검토 대상**(🔴/❓ 보고 규약에 합류).
- raw 가 마운트 안 돼 직접 확인 자체가 불가하면, 그 사실을 🔍❓ 로 명시하고 "raw 마운트 필요" 를 사용자에게 보고 — 임의 추정으로 메우지 않는다.

> 요지: **"정책도 없고 entity 도 없다 = 모른다"** 가 아니라 **"raw 를 직접 봐야 한다"** 는 신호다. 직접 봤으면 🔍 로, 못 봤으면 🔍❓ 로 — KB 공백을 추측이 아니라 *명시된 확인 상태*로 메운다.

---

## 7. 변경·감사 이력

- **2026-05-31** — mcwiki 횡단 정책 11종(+deep 2) KMCKnowledge 미러. entity 급 5 / 메타 6 분류. 카테고리×정책 매트릭스(§3) 수립. entity 양식에 `policy_refs` + `횡단 정책 준수` 섹션 도입(STRUCTURE.md §4). ingest-kmc 스킬 동기화. Component 카테고리 파일럿 마이그레이션.
- **2026-05-31 (import 아티팩트 교정)** — deep 링크 경로 10·11 교정(`references/`→`deep/`). 12 deep(AssetOptimizationDeep) 미러 누락 ❓ 표시 + 미러 큐 등록. 링크 정규화 키(§1) 수립 — `../skills/*`=mcwiki, 비미러 NN_(02·03·04·05·06·08·13) 명시. 본문 verbatim 보존 원칙 확정.
- **2026-05-31 (AssetOptimizationDeep 미러)** — `deep/AssetOptimizationDeep.md`(509줄) 추가 미러 완료. 12 deep 링크 5개 앵커 일치 확인, ⏳→✅ 승격, 12 본문 ❓ 주석 제거(verbatim 복귀). deep 본문 3종(Component·AssetLoading·AssetOptimization) 전부 실재. 미러 큐 잔여 = 결번 정책(02·03·04·05·06·08·13)만.
- **2026-05-31 (직접 코드 확인 체크 도입)** — §5 근거 출처 마커(🟢/🔍/🔍❓/🔴) + §6.1 "정책·entity 둘 다 없는 케이스 → raw 직접 확인 의무" 규약 신설. CLAUDE.md §3.2 locate·ingest 스킬과 동기화.
- 차기 감사: [[ue-cross-cutting-policies/18_ModelEvolutionAudit]] §분기 감사 — 5.5.4→5.7.4 버전 차이 항목 우선.
