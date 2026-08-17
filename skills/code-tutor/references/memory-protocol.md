# Memory Protocol — 파일 스키마와 갱신 규칙

## 전역/로컬 경계

| 위치 | 기준 | 파일 |
|---|---|---|
| 전역 `~/.claude/memory/code-tutor/` | **프로젝트가 바뀌어도 재사용되는 지식** — CS 이론, 프레임워크, 언어 문법, 라이브러리, 디자인 패턴 | learning-state.md, bookmarks.md |
| 프로젝트 `<프로젝트>/.claude/tutor/` | **이 코드베이스 고유 정보** — 프로젝트 구조·파일 위치 | learning-report.md, codebase-index.md |

전역 파일에 프로젝트 파일 경로를 적지 않는다. 프로젝트 파일에 개념 설명을 길게 적지 않는다(이름과 링크만).

세션의 탐색 흐름·"③" 목록·"돌아가자" 복귀 지점은 **대화 컨텍스트가 보유**하므로 파일로 남기지 않는다(별도 세션 파일 없음). 디스크에는 durable한 학습 변화만 기록한다.

## 갱신 타이밍

| 시점 | 갱신 대상 | 내용 |
|---|---|---|
| 주제 전환 시 (flush) | learning-state.md | 새 키워드 🔴/🟡 등록, `#주제` 태그, 전이 반영 |
| 연결 발견 시 | 노트 `related` | 두 개념이 실제로 닿고 노트로 승격돼 있으면 note `related`에 엣지 추가 |
| Feynman·캘리브레이션 통과 시 | learning-state.md | 해당 키워드 🔵 승급 (틀리면 강등) |
| `/code-tutor end` | 전부 | learning-state 일괄 갱신, learning-report append, bookmarks 정리 — 메인 대화가 사건 목록을 정리해 `tutor-scribe`(haiku)에 위임 |

매 턴 파일을 쓰지 않는다. 전역 파일은 주제 전환·연결 발견 시에만 flush 하므로, 컨텍스트 압축 손실 창은 "직전 주제 하나"로 제한된다.

## 항목 단위: 키워드 (+ 주제 태그)

learning-state의 항목(=이해도 점수를 매기는 노드)은 **한 번의 설명으로 온전히 다룰 수 있는 최소 키워드**다. 큰 주제는 점수 노드가 될 수 없다 — 어디까지 이해했는지 특정할 수 없기 때문이다. 대신 **주제 태그**와 **그래프 허브 노드**로 큰 주제를 붙잡는다.

- 좋은 항목: `@Transactional propagation`, `Isolation Level`, `pool.query vs pool.connect`
- 점수 금지: `Transaction`, `Connection Pool`, `Spring` 같은 큰 주제를 **하나의 이해도 항목으로** 등록하지 않는다.

큰 주제를 붙잡는 두 방법 (작은 단위의 정밀함은 유지하면서):

1. **주제 태그** — 각 항목 끝에 `#큰주제`를 붙인다: `@Transactional propagation 🟡 (2026-07-15) #Transaction`. 여러 개 가능: `#Transaction #Spring`. 태그는 Obsidian tag와 호환되고 grep으로 필터된다.
2. **노트의 허브 역할** — 큰 주제가 노트로 승격되면 그 노트가 하위 키워드들을 `related`로 잇는 허브가 된다. (승격 전에는 `#주제` 태그만으로 묶음.)

→ 이렇게 하면 `#Transaction`으로 필터해 **큰 주제 단위의 진척을 집계**할 수 있다 (Learning 모드 "주제별 진척": 태그로 묶어 이모지 카운트). 큰 주제 자체의 진척도를 하나의 값으로 *저장하지는* 않는다 — 항상 하위 항목에서 파생 집계한다.

## 이해도 전이 기준

전이는 **판정이 아니라 관찰 가능한 사건**으로만 일어난다. "이해한 것 같다"는 인상으로 올리지 않는다.

| 단계 | 의미 | 전이 사건 |
|---|---|---|
| 🔴 | 등장만 함 | "더 알아보기" 목록에 이름이 나옴 |
| 🟡 | 설명 받음 | 이 키워드가 응답의 주제로 다뤄짐 |
| 🟢 | 예제·적용 확인 | 예제 코드를 봤거나, 이 키워드를 전제로 스스로 후속 질문을 함 |
| 🔵 | 검증됨 | Feynman 체크·캘리브레이션 퀴즈 통과, 또는 사용자가 스스로 정확히 설명함 |

- **Probe(사전 진단)**: 채점형 객관식 측정 결과를 반영 — 맞힘 🟢, 설명만 받고 미검증 🟡, 틀림·"모르겠다" 🔴. (상세는 modes.md §0)
- 사용자 자기보고("이건 이미 알아") → 즉시 🟢, Feynman으로 검증되면 🔵.
- **강등 (캘리브레이션)**: 주기적 확인 질문에 틀리면 한 단계 내린다(🔵→🟢, 🟢→🟡). 🟢 이상 키워드를 다시 기초부터 물어도 🟡로 내린다. 강등은 실패가 아니라 정직한 재조정이다. (상세는 modes.md)

---

## 파일 템플릿

### learning-state.md (전역)

카테고리는 Language / Framework / CS / Library / Pattern 5개를 기본으로 하고 필요 시 추가한다.

```markdown
# Learning State

> 🔴 등장만 함 → 🟡 설명 받음 → 🟢 예제·적용 확인 → 🔵 검증됨(Feynman)
> 항목은 최소 키워드 단위 + 주제 태그(#큰주제). 큰 주제 자체는 점수 노드로 등록 금지.

## Language
<!-- 예: - ESM export/import 🟢 (2026-07-14) #ESM -->

## Framework
<!-- 예: - @Transactional propagation 🟡 (2026-07-15) #Transaction #Spring -->

## CS
<!-- 예: - Isolation Level 🟡 (2026-07-15) #Transaction -->

## Library
<!-- 예: - pool.query vs pool.connect 🟢 (2026-07-15) #ConnectionPool -->

## Pattern
<!-- 예: - withTransaction 콜백 패턴 🔵 (2026-07-15) #Transaction -->
```

항목 형식: `- <키워드> <이모지> (YYYY-MM-DD) #<주제> [#<주제2> …]` — 한 줄, 설명 없이.
- **주제 태그**는 큰 주제(집계 단위)를 가리킨다. 최소 1개, 여러 개 가능. Obsidian tag·grep과 호환.
- 카테고리 섹션(Language/…)은 "지식의 종류", 태그는 "주제 클러스터" — 서로 직교하므로 둘 다 쓴다.

### 개념 그래프 = 노트 `related` (별도 파일 없음)

개념 그래프에는 **별도 파일을 두지 않는다.** 개념 관계(엣지)는 내보낸 노트의 `related` [[링크]]에 담기고(whiteboarding.md 노트 스키마), 노드 인벤토리는 **learning-state의 키워드 + `#주제` 태그**가 대신한다.

- **그래프 뷰(HTML)**는 노트에서 렌더한다(graph-html-maker.md). Obsidian 네이티브 그래프뷰도 노트 `[[링크]]`로 자연 생성.
- **아직 노트로 승격 안 된 키워드**는 learning-state에 `#태그`로만 존재(엣지 없음). 중요해지면 노트로 승격하며 그때 `related` 엣지를 갖는다.
- connection-first·소급 상기는 `#태그` 클러스터 + 노트 `related` 이웃 + 질문 시점 추론으로 관련 학습을 찾는다.

**엣지 = 실제 개념 관계만 (개념도 원칙)** — 노트 `related`에 적용:
- 진짜 개념적·구조적 관계일 때만. **co-occurrence**(같은 세션·코드베이스에 함께 등장)만으로는 잇지 않는다.
- **코드베이스 구현 링크**("이 코드가 패턴 P의 사례")는 개념도에 넣지 **않는다** — 노트 `source`/`## 코드에서`로 분리. 개념도는 코드베이스와 무관하게 성립하는 관계만.
- **간접 관계를 직접 잇지 않는다**: A—중간개념 M—B면 `A—M`, `M—B`로. 직접 `A—B` ✗. (예: RAG—`fail-closed`—Humble이지 RAG—Humble이 아님)

### bookmarks.md (전역)

```markdown
# Bookmarks

<!-- 나중에 학습할 주제. 어디서 나왔는지 맥락 한 줄 -->

- [ ] Spring Proxy — @Transactional 학습 중 파생 (2026-07-14)
- [ ] MVCC — SELECT FOR UPDATE에서 파생 (2026-07-14)
```

학습 완료 시 체크하고 learning-state로 옮긴다.

> **세션 상태 파일은 두지 않는다.** 탐색 흐름·"③" 목록·복귀 지점은 대화 컨텍스트가 보유하므로 디스크에 중복 저장하지 않는다. 섬형 사용에서는 이전 세션을 선형으로 이어가지 않으므로 세션 재개 기능도 불필요하다.

### learning-report.md (프로젝트)

세션마다 아래 블록을 append 한다.

```markdown
## 세션 YYYY-MM-DD

### 학습 경로
@Transactional → Transaction Manager → MVCC

### 새로 배운 것
- Transaction Manager 🟡 → 🟢

### 아직 부족한 부분
- Isolation Level 🔴 — Deep Dive에서 언급만 됨

### 다음 추천
- Spring Proxy (bookmarks 등록됨)
```

### codebase-index.md (프로젝트)

init이 전체를 채우고, lazy indexing은 탐색한 영역만 append 한다. 두 경로 모두 같은 스키마.

```markdown
# Codebase Index

- 갱신: YYYY-MM-DD
- 기술 스택: <언어, 프레임워크, 빌드 도구>
- 커버리지: 전체 스캔(init) | 부분 (탐색된 영역만)

## 레이어 구조
Frontend(src/components) → API(server/routes) → Service(server/services) → DB(server/db)

## 주요 진입점
- 프론트: src/main.jsx
- 서버: server/index.js

## 영역별 인덱스
<!-- lazy indexing은 여기에 영역 단위로 append -->

### server/routes
- users.js — 유저 CRUD 엔드포인트 (탐색: 2026-07-14)
```
