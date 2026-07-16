# Memory Protocol — 파일 스키마와 갱신 규칙

## 전역/로컬 경계

| 위치 | 기준 | 파일 |
|---|---|---|
| 전역 `~/.claude/memory/code-tutor/` | **프로젝트가 바뀌어도 재사용되는 지식** — CS 이론, 프레임워크, 언어 문법, 라이브러리, 디자인 패턴 | learning-state.md, knowledge-graph.md, bookmarks.md |
| 프로젝트 `<프로젝트>/.claude/tutor/` | **이 코드베이스 고유 정보** — 파일 위치, 세션 흐름, 프로젝트 구조 | current-session.md, learning-report.md, codebase-index.md |

전역 파일에 프로젝트 파일 경로를 적지 않는다. 프로젝트 파일에 개념 설명을 길게 적지 않는다(이름과 링크만).

## 갱신 타이밍

| 시점 | 갱신 대상 | 내용 |
|---|---|---|
| 매 턴 (응답 직후) | current-session.md | 탐색 노드 1줄 append + 방금 제시한 "더 알아보기" 번호 목록 교체 |
| 주제 전환 시 | knowledge-graph.md, learning-state.md | 개념 연결 추가, 새 개념 🔴 또는 🟡로 등록 |
| Feynman 체크 통과 시 | learning-state.md | 해당 개념 🔵로 승급 |
| `/code-tutor end` | 전부 | learning-state 일괄 갱신, learning-report append, bookmarks 정리, current-session 완료 표시 — 메인 대화가 사건 목록을 정리해 `tutor-scribe`(haiku)에 위임 |

매 턴 갱신은 current-session.md **하나만**. 전역 파일을 매 턴 고치지 않는다 (토큰 낭비).

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
| 🔵 | 검증됨 | Feynman 체크 통과 또는 사용자가 스스로 정확히 설명함 |

- 사용자 자기보고("이건 이미 알아") → 즉시 🟢, Feynman으로 검증되면 🔵.
- 강등: 사용자가 🟢 이상 키워드를 다시 기초부터 물으면 🟡로 내린다.

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

```markdown
# Knowledge Graph

<!-- 개념 간 연결. 트리 형식, 개념 이름만. 프로젝트 파일 경로 금지 -->

## Database
SQL
├── Index
│   └── B+Tree
└── Transaction
    ├── Lock
    └── MVCC

## Spring
@Transactional
├── Proxy
│   └── AOP
└── Transaction Manager
```

새 연결은 기존 트리에 붙이고, 어느 트리에도 안 붙으면 새 루트를 만든다.

### bookmarks.md (전역)

```markdown
# Bookmarks

<!-- 나중에 학습할 주제. 어디서 나왔는지 맥락 한 줄 -->

- [ ] Spring Proxy — @Transactional 학습 중 파생 (2026-07-14)
- [ ] MVCC — SELECT FOR UPDATE에서 파생 (2026-07-14)
```

학습 완료 시 체크하고 learning-state로 옮긴다.

### current-session.md (프로젝트)

```markdown
# Current Session

- 시작: YYYY-MM-DD HH:mm
- 상태: 진행 중 | 완료

## 탐색 흐름
<!-- 매 턴 1줄 append. 형식: 모드 | 주제 | 코드 위치 -->
1. Quick | @Transactional | src/service/OrderService.java:42
2. Expand | Transaction Manager | (개념)
3. Quick | UserController | src/api/UserController.java:15   ← 새 갈래

## 직전 "더 알아보기" 목록
<!-- 매 턴 교체. 번호 참조("③") 복원용 -->
① Spring Proxy
② AOP
③ Isolation Level

## 보류 중인 갈래
<!-- "원래 이야기로 돌아가자" 복귀 지점 -->
- @Transactional (2번 노드에서 이탈) — 남은 항목: ①③
```

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
