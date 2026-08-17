# Whiteboarding & Note Export

세션 중 설명 내용을 Obsidian에 **실시간으로 적어** SVG·mermaid가 효과적으로 시각화되게 하고, 소주제가 끝나면 영구 노트로 승격한다.

**활성 조건**: `~/.claude/memory/code-tutor/config.md`가 있고 경로가 설정돼 있을 때만. 없으면 이 기능은 조용히 비활성(사용자에게 경로 설정을 한 번 안내만).

---

## 1. 실시간 화이트보드 (whiteboard.md)

`config.md`의 화이트보드 파일에 설명을 진행하면서 append 한다. 터미널 응답과 별개로, **시각 자료(mermaid/SVG)가 렌더되는 캔버스**다.

- **세션 시작**: 파일을 새 세션 헤더로 초기화한다.
  ```markdown
  # Whiteboard — 2026-08-17 세션

  ```
- **설명 중(append)**: 지금 다루는 개념 블록을 이어 쓴다. 터미널엔 요약, 화이트보드엔 시각 자료 포함 상세.
  ```markdown
  ## Java Stream 파이프라인

  > 컬렉션을 선언형 파이프라인(source → 중간연산 → 종료연산)으로 처리.

  ```mermaid
  graph LR
    S[List] --> F["filter()"] --> M["map()"] --> C["collect()"] --> R[Result]
  ```

  - `filter`는 지연(lazy), `collect`가 종료 연산에서 실제 실행
  - 코드: `src/service/OrderService.java:42`
  ```
- mermaid는 fenced ```mermaid``` 블록으로, SVG는 `![[파일명.svg]]` 임베드로 넣는다(§3).
- 화이트보드는 스크래치다 — 소주제가 끝나 노트로 승격되면 해당 블록은 정리(삭제)해도 된다.

## 2. 소주제 완료 → 영구 노트 승격 (note export)

한 소주제(예: Java Stream)의 학습이 마무리되면 화이트보드 블록을 **살아있는 영구 노트**로 승격한다.

- **위치**: `<vault>/CS정리/<대주제>/<소주제>.md`. 대주제 폴더는 config.md의 `#태그 → 폴더` 매핑으로 자동 결정.
- **살아있는 노트(living note)**: 같은 소주제를 다시 배우면 **새 파일이 아니라 기존 노트를 제자리에서 갱신**하고 `updated`만 바꾼다. 기존 내용과 병합(중복 제거), 새 내용 추가.
- 승격은 소주제가 끝났다고 판단될 때 또는 `/code-tutor end`에서 일괄 수행. vault에 파일을 쓰는 작업이므로 처음 한 번은 사용자에게 알린다.

### 노트 스키마

```markdown
---
title: Stream API
subject: 자바                    # 대주제(폴더)와 일치
tags: [Stream, Collection]      # learning-state의 #주제 태그 (# 없이)
keywords:                       # 이 노트가 담은 learning-state 키워드 + 이해도
  - "Stream.collect 🟢"
  - "Optional.orElseThrow 🟡"
related:                        # 개념 그래프의 엣지 → 위키링크(그래프 뷰·네이티브 그래프 생성)
  - "[[Optional]]"
  - "[[Collectors]]"
source: "aac/hub · src/service/OrderService.java:42"   # 코드 출처(어디서 나왔나)
sources:                        # 신뢰 가능한 외부 문서(검증용) — config.md의 신뢰 출처
  - "MDN: https://developer.mozilla.org/…/Array/reduce"
  - "Java SE Docs: java.util.stream.Stream"
created: 2026-08-17
updated: 2026-08-17
---

# Stream API

> 한 줄 요약.

## 핵심
- …

## 시각 자료
![[Stream-API-pipeline.svg]]   또는 인라인 ```mermaid```

## 코드에서
- `src/service/OrderService.java:42` — 이렇게 쓰임

## 연결
- [[Optional]] — 종료 연산이 빈 값일 수 있어 함께 등장
- [[Collectors]] — collect의 인자

## 출처
- [MDN — Array.prototype.reduce](https://developer.mozilla.org/…)
- Java SE Docs — `java.util.stream.Stream` (docs.oracle.com에서 확인)

## 이해도 체크
- [ ] Stream.collect를 한 문장으로 설명해보기
```

### 규칙

- `related`/`## 연결`의 `[[링크]]`가 **개념 그래프의 엣지 그 자체다** (별도 그래프 파일 없음). 노트끼리 링크가 생기므로 그래프 뷰(HTML)와 Obsidian 네이티브 그래프뷰가 여기서 만들어진다. **엣지 원칙(memory-protocol.md)** — 실제 개념 관계만, co-occurrence 금지, 코드베이스 구현 링크 제외, 간접 관계를 직접 잇지 않음. (근거 없는 링크를 넣으면 개념도가 오염된다.)
- `tags`/`keywords`는 learning-state와 동기화 — 노트는 표현(view)이고 **source of truth는 우리 메모리**다. 충돌 시 메모리가 우선.
- **`sources`(신뢰 출처)**: config.md의 신뢰 출처 표에서 확인 가능한 링크를 단다. 공식 문서 우선. **정확한 URL이 불확실하면 지어내지 말고** 검색 힌트로 대체한다 (예: "MDN에서 'reduce' 검색"). 코드 출처(`source`)와 외부 문서 출처(`sources`)는 별개다.
- 향후 가중치를 넣으면 frontmatter에 `weights:` 한 필드만 추가한다 (스키마 확장 지점).
- 기존 vault 노트 스타일(자유형식 `##` + 코드블록 + 출처 링크)과 이질감 없게. frontmatter만 추가 가치.

## 3. SVG/mermaid 배치

- 텍스트보다 그림이 나은 경우(§ svg-maker.md 판단) SVG를 만들어 `_attachments`에 저장하고 `![[파일.svg]]`로 임베드.
- 구조·흐름·관계는 mermaid, 기하·공간·커스텀 도식은 SVG (svg-maker.md 참조).
