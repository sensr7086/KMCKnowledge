# 횡단 정책 평가 (ingest 4단계 필독)

entity 작성 시 적용되는 **횡단 정책(cross-cutting policies)** 을 평가해 frontmatter `policy_refs` + 본문 §6 `횡단 정책 준수` 를 채우는 방법.

> **단일 진실원 = 저장소 루트의 [[ue-cross-cutting-policies/index]]**. 본 문서는 ingest 측 수행 절차·표기법·예시. 매트릭스가 갱신되면 index 가 우선이다.
> **정책 본문**은 `ue-cross-cutting-policies/<NN_*>.md` — mcwiki 미러(일반 UE 지식, 기본 🔴, **프로젝트 중립**). **여기서 하는 일**은 "이 대상 프로젝트 클래스가 그 정책을 지키는가"를 코드 근거로 판정해 기록하는 것(프로젝트 지식). 대상 프로젝트명·엔진 버전은 `project.config.md` 참조.

---

## 0. 부착 대상 = entity 급 5종만

| ID | 슬러그 (`policy_refs` 값) | 한 줄 |
|---|---|---|
| 07 | `profiling-scope-rule` | Tick/타이머/바인딩 UFunction/시간 람다 첫 줄 프로파일링 스코프 |
| 09 | `global-iterator-policy` | TActorIterator/TObjectIterator/TObjectRange/TActorRange 금지 |
| 10 | `component-policies` | 컴포넌트 6대 의무 (Mobility/NewObject/GC/GetOwner 캐싱/Tick OFF/CDO) |
| 11 | `asset-loading-policy` | Soft/Hard ref + 비동기 로드 + SpawnActor 히칭 방지 |
| 12 | `asset-optimization-policy` | Mesh/VFX/Audio 자산 LOD·Nanite·Culling·Scalability |

**메타 정책(14~19)은 entity 에 부착하지 않는다.** 생성기·리뷰 워크플로우가 참조하는 작업 방식 규약이다(충돌 해결 16 / 품질 17 / 평가 15 / 인계 14 / 감사 18 / 외부소스 19).

---

## 1. 적용 판정 — 카테고리 × 정책 매트릭스 (요약)

> 권위본 = [[ue-cross-cutting-policies/index]] §3. 아래는 작업용 사본. ✅ 항상 평가 · △ 조건부(트리거) · ➖ 보통 없음.

| 카테고리 | 07 | 09 | 10 | 11 | 12 |
|---|---|---|---|---|---|
| Actor | ✅ | ✅ | ➖¹ | ✅ | △ |
| Component | ✅ | ✅ | ✅ | ✅ | △ |
| Subsystem | △ | ✅ | ➖¹ | ✅ | ➖ |
| Interface | ➖ | ➖ | ➖ | ➖ | ➖ |
| DataTable | ➖ | ➖ | ➖ | △ | △ |
| AssetUserData | ➖ | ➖ | ➖ | △ | △ |
| Asset | ➖ | ➖ | ➖ | ✅ | ✅ |
| GraphNode | △ | △ | ➖ | ✅ | △ |
| BlueprintLibrary | ➖ | ✅ | ➖ | △ | ➖ |
| Widget | ✅ | △ | ➖² | ✅ | △ |

¹ 6대 중 GC/NewObject Outer/CDO 하위 규칙은 UObject 생성·보유 Actor/Subsystem 에 **준용 가능**(🟡 "준용" 명시).
² UMG 위젯은 별도 라이프사이클 — 컴포넌트 6대 직접 적용 안 함.

---

## 2. △/✅ 확정 — 트리거 시그널 (코드 grep)

| 정책 | 트리거 |
|---|---|
| 07 | `Tick(` `TickComponent(` `NativeTick(` `NativeUpdateAnimation(` `SetTimer` `FTSTicker` `OnRep_` 바인딩 UFUNCTION·시간 람다 |
| 09 | `TActorIterator` `TObjectIterator` `TObjectRange` `TActorRange` (1건이라도 위반 후보) |
| 10 | category=Component (UActorComponent/USceneComponent/UAnimInstance 자손) |
| 11 | asset 멤버(`UStaticMesh*`/`USkeletalMesh*`/`UMaterial*`/`UTexture*`/`USoundBase*`/`UNiagaraSystem*`/`TSubclassOf`/`TSoftObjectPtr`/`TSoftClassPtr`) 또는 `SpawnActor`/`LoadObject`/`LoadSynchronous` |
| 12 | 위 중 **Mesh/VFX/Audio** 자산 소유·참조 |

- 트리거 있음 → ✅. 조건부였는데 트리거 없음 → ➖.
- 코드(raw)가 없어 트리거 확인 불가 → **❓(미확인)**. 지어내지 말 것(§2.5).

---

## 3. 본문 §6 `횡단 정책 준수` 양식

entity 급 5종을 **모두** 표에 적되(빈칸 금지 §2.5), 적용 열로 구분한다. `policy_refs` 에는 ✅·△ 행의 슬러그만.

```markdown
## 횡단 정책 준수
> 적용 판정: [[ue-cross-cutting-policies/index]] §3 (카테고리 <X> 행). 준수 근거 = 코드/추출 본문.

| 정책 | 적용 | 근거 / 위반·미확인 | 신뢰도 |
|---|---|---|---|
| 10 component | ✅ | ... | 🟢/🟡/❓ |
| 07 profiling | ✅ | ... | 🟢/🟡/❓ |
| 11 asset-loading | △→✅/➖ | ... | ... |
| 09 global-iterator | ➖ | 전역 이터레이터 미사용 | 🟢 |
| 12 asset-opt | ➖ | Mesh/VFX/Audio 자산 아님 | 🟢 |
```

### 신뢰도 (준수 판정)
- 🟢 — 코드/추출 본문에 **명확한 근거** (예: `bCanEverTick=false` 확인).
- 🟡 — 코드 구조에서 **추론** (주석 근거 없음) / 또는 6대 하위규칙 "준용".
- ❓ — entity·코드 근거로 **판단 불가** (raw 미마운트 등). 사용자 검토 대상. 체크포인트 보고.
- 위반은 근거 칸에 **❌ + 무엇을 어겼는지** 명시 (생성기 안전장치).

### 근거 출처 마커 (직접 코드 확인 — index §5/§6.1 과 동일 축)
준수 근거가 *어디서* 왔는지 함께 표시한다. ingest 는 raw 를 읽는 워크플로우이므로 보통 🟢/🔍 가 정상이고, raw 를 못 본 항목만 🔍❓ 로 남긴다.
- 🟢 **VAULT** — 이미 정제된 entity/추출 본문 근거.
- 🔍 **RAW** — entity 없음/부족 → **raw 직접 grep·읽기로 확인 완료**. 근거 칸에 `경로:line` 또는 grep 패턴 증거 의무. (주장만으로는 🔍 금지.)
- 🔍❓ **RAW-PENDING** — raw 확인이 필요하나 못 함(미마운트 등). ❓ 의 한 갈래 — 체크포인트 집중 검토.
- 🔴 **INFERRED** — raw 에도 선례 없어 정책 일반 지식으로만 메움.

> **정책·entity 둘 다 없는 케이스**(이 클래스가 어느 정책에도 안 걸리고 선례 entity 도 없음): KB 로 답하지 말고 raw 를 직접 확인해 🔍 로 남긴다. raw 가 없으면 🔍❓ + "raw 마운트 필요" 보고. 상세 결정 트리 = [[ue-cross-cutting-policies/index]] §6.1.

> 정책 *본문* 의 🔴(일반 지식 출처)와, 여기 *준수 판정* 의 신뢰도(🟢/🟡/❓)는 다른 축이다. 혼동 금지.

### policy_refs 규약
```yaml
policy_refs:          # ✅·△ 적용 정책 슬러그만. ➖ 는 넣지 않음(본문 표에만)
  - component-policies
  - profiling-scope-rule
```

### 충돌
정책끼리 모순(예: 11 비동기 로드 vs 즉시 Spawn 요구)이면 [[ue-cross-cutting-policies/16_PolicyPriority]] Tier(빌드 > GC/메모리 > 네트워크 > 성능 > 유지보수) 우선순위로 해소하고 근거 칸에 명시.

---

## 4. 예시

### 4.1 Component — `UMCActorComponent` (UActorComponent 베이스)

```markdown
## 횡단 정책 준수
> 적용 판정: [[ue-cross-cutting-policies/index]] §3 (카테고리 Component 행).

| 정책 | 적용 | 근거 / 위반·미확인 | 신뢰도 |
|---|---|---|---|
| 10 component | ✅ | GC: `Interface` = raw 포인터(UPROPERTY 아님) — 6대 §3 위반 후보 ❌(단 Owner 라이프사이클 동기 가정). GetOwner 캐싱: 미확인. Tick: 베이스 기본값 확인 필요. → [[ue-cross-cutting-policies/10_ComponentPolicies]] | 🟡 |
| 07 profiling | ✅ | `TickComponent` 존재 — 첫 줄 프로파일링 스코프 유무 ❓(코드 확인 필요). → [[ue-cross-cutting-policies/07_ProfilingScopeRule]] | ❓ |
| 11 asset-loading | ➖ | asset 멤버 없음(인터페이스 포인터만). | 🟢 |
| 12 asset-opt | ➖ | 자산 미소유. | 🟢 |
| 09 global-iterator | ➖ | 전역 이터레이터 미사용. | 🟢 |
```
→ `policy_refs: [component-policies, profiling-scope-rule]`

### 4.2 Interface — `IMCActorInterface`

```markdown
## 횡단 정책 준수
> 적용 판정: [[ue-cross-cutting-policies/index]] §3 (카테고리 Interface 행) — 전 정책 ➖. 계약만 정의.

| 정책 | 적용 | 근거 | 신뢰도 |
|---|---|---|---|
| 07~12 | ➖ | 인터페이스는 구현이 없음 — 정책은 **구현 클래스**(예: AMCCharacter)에서 평가. | 🟢 |
```
→ `policy_refs:` 없음(생략) 또는 빈 리스트.

> 인터페이스는 정책 평가를 **구현체로 위임**한다. "구현체에서 평가" 한 줄만 남기면 빈칸 금지(§2.5) 충족.
