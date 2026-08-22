# Whiteboarding & Note Export

세션 중 설명 내용을 Obsidian에 **실시간으로 적어** SVG·mermaid가 효과적으로 시각화되게 하고, 소주제가 끝나면 영구 노트로 승격한다.

**활성 조건**: `~/.claude/memory/code-tutor/config.md`가 있고 경로가 설정돼 있을 때만. 없으면 이 기능은 조용히 비활성(사용자에게 경로 설정을 한 번 안내만).

---

## 1. 실시간 화이트보드 (whiteboard.md)

`config.md`의 화이트보드 파일에 **터미널 답변의 전문(full text)을 시각화해 담는다.** 요약본이 아니라 **설명 전체**를, Obsidian에서 잘 읽히도록 구조와 시각 자료를 입혀 붙인다.

- **세션 시작**: 파일을 새 세션 헤더로 **초기화**한다(이전 세션 스크래치는 지운다).
  ```markdown
  # Whiteboard — 2026-08-18 세션: <주제>

  > Claude Code 설명 전문 + 시각화
  ```
- **전문 첨부 (핵심 원칙)**: 터미널에 낸 설명을 **빠짐없이** 화이트보드에도 남긴다. 단, 그대로 복붙이 아니라 **읽기 좋게 재배치**한다:
  - **명확한 섹션·단락 구분** — `##`/`###` 헤딩으로 논리 단위를 나누고, 긴 문단은 쪼갠다. 한눈에 스캔되도록.
  - **텍스트로 그린 타임라인·흐름 → mermaid로 승격** — 트랜잭션 인터리빙·시퀀스는 `sequenceDiagram`, 구조·파이프라인은 `graph`.
    - **mermaid 안전 규칙(파서 오류 방지)**: `participant … as` 별칭과 메시지 텍스트에 **괄호 `()`·`=`·`;` 를 넣지 않는다.** 값은 콤마·공백으로 풀어 쓰고(`id=1` → `id 1`, `COUNT(*)` → `COUNT`), 격리 수준·경고 같은 라벨은 `Note over`로 뺀다. (이게 sequenceDiagram이 깨지는 가장 흔한 원인.)
  - **표·코드블록은 그대로 유지** (Obsidian이 렌더). SQL·코드 예시는 fenced 코드블록으로.
  - **기하·공간·연속량**은 SVG로(§3, svg-maker), `![[파일.svg]]` 임베드.
- 목표: **화이트보드만 봐도 그 세션의 설명이 완결적으로, 시각적으로** 남는다.
- 화이트보드는 스크래치다 — 소주제가 노트로 승격되면 해당 블록은 정리(삭제)해도 된다.

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
