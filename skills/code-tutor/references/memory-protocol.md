# Memory Protocol — 파일 스키마와 갱신 규칙

## 전역/로컬 경계

| 위치 | 기준 | 파일 |
|---|---|---|
| 전역 `~/.claude/memory/code-tutor/` | **프로젝트가 바뀌어도 재사용되는 지식** — CS 이론, 프레임워크, 언어 문법, 라이브러리, 디자인 패턴 | learning-state.md, knowledge-graph.md, bookmarks.md |
| 프로젝트 `<프로젝트>/.claude/tutor/` | **이 코드베이스 고유 정보** — 프로젝트 구조·파일 위치 | learning-report.md, codebase-index.md |

전역 파일에 프로젝트 파일 경로를 적지 않는다. 프로젝트 파일에 개념 설명을 길게 적지 않는다(이름과 링크만).

세션의 탐색 흐름·"③" 목록·"돌아가자" 복귀 지점은 **대화 컨텍스트가 보유**하므로 파일로 남기지 않는다(별도 세션 파일 없음). 디스크에는 durable한 학습 변화만 기록한다.

## 갱신 타이밍

| 시점 | 갱신 대상 | 내용 |
|---|---|---|
| 주제 전환 시 (flush) | knowledge-graph.md, learning-state.md | 연상 엣지 추가, 새 키워드 🔴/🟡 등록, 전이 반영 |
| 연결 발견 시 | knowledge-graph.md | 새 개념이 기존 학습과 닿으면 연상 엣지 추가 |
| Feynman·캘리브레이션 통과 시 | learning-state.md | 해당 키워드 🔵 승급 (틀리면 강등) |
| `/code-tutor end` | 전부 | learning-state 일괄 갱신, learning-report append, bookmarks 정리 — 메인 대화가 사건 목록을 정리해 `tutor-scribe`(haiku)에 위임 |

매 턴 파일을 쓰지 않는다. 전역 파일은 주제 전환·연결 발견 시에만 flush 하므로, 컨텍스트 압축 손실 창은 "직전 주제 하나"로 제한된다.

## 항목 단위: 키워드

learning-state의 항목은 **한 번의 설명으로 온전히 다룰 수 있는 최소 키워드**다.

- 좋은 항목: `@Transactional propagation`, `Isolation Level`, `pool.query vs pool.connect`, `Dirty Read`
- 금지 항목: `Transaction`, `Connection Pool`, `Spring` 같은 큰 주제 — 어디까지 이해했는지 특정할 수 없다
- 큰 주제는 knowledge-graph.md의 트리 루트/중간 노드로만 존재한다.
- **큰 주제의 진척도는 저장하지 않는다.** 대화로 큰 개념의 이해도를 판정하는 것은 오류 소지가 크다. Learning Mode에서 knowledge-graph의 하위 키워드 이해도를 집계해 파생적으로만 보여준다 (예: `Transaction: 하위 키워드 7개 중 🟡 3, 🟢 1`).
- 설명 중 큰 주제를 다뤘다면, 실제로 설명한 하위 키워드들로 쪼개서 등록한다.

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
> 항목은 최소 키워드 단위. 큰 주제(Transaction, Connection Pool 등)는 등록 금지 — knowledge-graph에만.

## Language
<!-- 예: Optional.orElseThrow, Stream.collect, ESM export/import -->

## Framework
<!-- 예: @Transactional propagation, useEffect 의존성 배열 -->

## CS
<!-- 예: Isolation Level, B+Tree 리프 노드, TCP handshake -->

## Library
<!-- 예: pool.query vs pool.connect, axios interceptor -->

## Pattern
<!-- 예: Repository Pattern의 인터페이스 분리, DTO 변환 위치 -->
```

항목 형식: `- <키워드> <이모지> (YYYY-MM-DD)` — 한 줄, 설명 없이.

### knowledge-graph.md (전역)

**이것은 선수지식 트리가 아니라 연상 그래프(associative graph)다.** "무엇을 알아야 이걸 이해하나"의 하향식 분해가 아니라, **"내가 실제로 만진 것들이 어떻게 서로 닿는가"를 아래에서 위로 축적**한 것이다. 코딩 학습은 흩어진 섬이므로, 이 그래프의 가치는 그 섬들을 잇는 데 있다.

트리 형태로 적되 엣지는 선수관계가 아니라 **개념적 인접**을 뜻한다. 특히 **인스턴스→패턴** 링크를 적극적으로 만든다: "이 코드는 패턴 P의 사례다 → P가 내가 배운 어디에 또 나오나".

```markdown
# Knowledge Graph

<!-- 연상 그래프: 실제로 만진 개념들이 어떻게 닿는가. 개념 이름만, 프로젝트 파일 경로 금지.
     엣지 = 개념적 인접(선수관계 아님). 인스턴스→패턴 링크를 적극 추가. -->

## Database
Connection Pool
├── pool.query vs pool.connect
│   └── (연결) withTransaction 콜백 패턴 — 같은 커넥션 보장이라는 같은 문제
└── Transaction
    ├── Isolation Level
    └── MVCC
```

- 새 개념은 실제로 닿는 기존 노드에 엣지로 붙인다. 어디에도 안 닿으면 새 루트(새 섬)로 둔다.
- 세션·프로젝트를 넘어 닿으면 그 엣지를 반드시 추가한다 — 이것이 섬을 잇는 핵심 작업.
- `(연결)` 주석처럼 왜 닿는지 한 줄 근거를 남기면 나중에 소급 상기(spaced resurfacing)에 쓸 수 있다.

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
