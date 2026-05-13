# AI 코딩 에이전트 메모리 도구 비교 리포트 v2
## Claude Code · Codex CLI · Hermes Agent 사용자를 위한 실전 가이드

> 비교 대상: **claude-mem · Supermemory · Honcho · Hindsight · seCall · agentmemory**
> 최종 업데이트: 2026-05-13
> 버전: v2.0 (v1 → v2 주요 변경 사항은 부록 참고)

---

## Executive Summary

2026년 5월 현재, AI 코딩 에이전트 메모리 시장은 **hook API 표준화**와 **로컬 자급 도구의 약진**이라는 두 흐름이 동시에 진행 중입니다. 이 리포트는 6개 주요 도구를 사용자의 에이전트 조합·하드웨어 환경·운영 모델에 따라 평가합니다.

### 한 줄 추천

| 당신은 | 가장 자연스러운 첫 선택 |
|---|---|
| Claude Code만, 가볍게 시작 | **Supermemory** (플러그인) |
| Claude Code, 로컬 100% | **claude-mem** 또는 **agentmemory** |
| Hermes Agent 메인 | **Honcho** (self-hosted) 또는 **agentmemory** |
| Codex CLI 메인 | **Hindsight** (공식 가이드) 또는 **agentmemory** |
| 세 에이전트 모두 사용 + 고사양 | **agentmemory** 단일 백본 (LongMemEval-S SOTA 주장) |
| Obsidian + 한국어 + 멀티 에이전트 아카이브 | **seCall** (보완재) |
| 팀 공유 필수 | **Supermemory** (cloud) 또는 **Honcho** (managed) |

### 핵심 변화 (2026 H1)

1. **Codex hooks engine 안정화** (v0.124.0, 2026-04-23): SessionStart/Stop hook이 1급 시민이 되면서 Claude/Hermes와 동등한 자동 메모리 통합이 가능해짐
2. **claude-mem Codex 공식 지원**: 멀티-IDE installer가 Codex CLI를 정식 타깃에 포함
3. **agentmemory의 등장**: Apache 2.0, SQLite 단일 파일, **LongMemEval-S 95.2% (벤더 주장 기준 SOTA)**, 3개 에이전트 모두 deep hook 통합
4. **Apple Silicon + MLX**: MoE 모델(Qwen 3.6 A3B 등)의 등장으로 dense 70B 의존성이 사라짐. 로컬 reflect가 **세션마다** 가능한 시대

---

## 1. 비교 방법론

### 평가 차원
- **자동성**: hook 자동 vs MCP 명시 호출 vs 수동 sync
- **에이전트 통합 깊이**: native hook → MCP → ingest-only
- **저장 위치**: 로컬 / self-host / 매니지드 클라우드
- **회상 vs 학습**: 단순 회상 / consolidation / reflection
- **운영 비용**: 셋업 시간 + 유지보수 시간 + 라이선스 비용 + LLM 호출 비용
- **데이터 통제**: 라이선스, 자체 호스팅 가능성, 외부 전송

### 검증 표기
- ✅ 1차 자료(GitHub README, 공식 docs)에서 직접 확인
- △ 검색 결과 또는 2차 소스
- ❓ 벤더 주장이며 독립 검증 자료 미확보 (예: agentmemory의 LongMemEval-S SOTA)

---

## 2. 도구 한 단락씩 (메타데이터 포함)

### 2.1 claude-mem (`thedotmack/claude-mem`)
- **46.1k ⭐ · Apache 2.0 · 로컬 SQLite + 벡터 DB**
- Claude Code 플러그인 (hook 기반) — Claude Agent SDK로 **AI 압축**, 다음 세션 자동 주입
- 멀티-IDE installer가 Claude Code / Gemini CLI / OpenCode / OpenClaw / **Codex CLI** 정식 지원 (2026-05 시점 ✅)
- Hermes도 README에 언급되나 실제 설치 명령은 비공식

### 2.2 Supermemory (`supermemoryai/claude-supermemory`)
- **상용 SaaS + Free tier · 클라우드 저장 · MIT 클라이언트**
- 3개 에이전트 모두 **native 통합** ✅ (2026-05 사용자 검증)
  - Claude Code: 플러그인 + hook
  - Codex CLI: 공식 통합 (Codex hooks engine 안정화 이후)
  - Hermes: built-in MemoryProvider
- 사용자/팀 컨테이너 분리, 클라우드 동기화
- 사실상 가장 매끄러운 cross-agent 통합 솔루션

### 2.3 Honcho (`plastic-labs/honcho`)
- **3.5k ⭐ · AGPL-3.0 · Python · Postgres+pgvector**
- 라이브러리 + 매니지드 클라우드($100 free credit), Docker self-host
- **"peer" 모델** — 사용자/에이전트를 동등하게, 비동기 reasoning loop이 대화에서 사실 추출
- 자체 발표 ❓ "Pareto Frontier of Agent Memory"
- Hermes의 native MemoryProvider. `honcho-self-hosted` 레포가 Hermes용으로 별도 존재 — 사실상 Hermes의 reference memory provider

### 2.4 Hindsight (`vectorize-io/hindsight`)
- **13.2k ⭐ · MIT · Python+TS · Docker / 임베디드 Python**
- ❓ LongMemEval 벤치마크 SOTA 자체 보고 (agentmemory 등장 전까지 가장 강력한 주장)
- **biomimetic 메모리**: World / Experiences / Mental Models + **Retain·Recall·Reflect** 3-op
- BM25 + 벡터 + graph + temporal **4-way 병렬 검색** → cross-encoder rerank
- Ollama / **LMStudio** 일급 지원
- Codex 공식 가이드 보유, Hermes native provider, Claude는 MCP/skill

### 2.5 seCall (`hang-in/seCall`)
- **265 ⭐ · AGPL-3.0 · Rust · 한국 개발자**
- Claude Code / Codex / Gemini / claude.ai / ChatGPT 세션 로그를 **로컬 통합 인덱싱** → Obsidian 호환 마크다운 위키 + Knowledge Graph
- BM25 + 벡터 하이브리드, **한국어 형태소 분석기 내장** (Lindera/Kiwi)
- MCP 서버 + Web UI + Obsidian 플러그인
- **자동 hook 아님** — `secall sync` 수동/cron
- 카테고리: "메모리 플러그인"이 아니라 **"AI 세션 영구 아카이브 + 통합 검색 시스템"**
- Hermes 정식 지원은 아직 없음 (커스텀 어댑터 필요)

### 2.6 agentmemory (`rohitg00/agentmemory`) ⭐ 신규
- **6.1k ⭐ · Apache 2.0 · TypeScript · v0.9.11 (2026-05-12, 매우 활발)**
- ❓ **LongMemEval-S 95.2%** 자체 보고 — 벤더 주장 기준 현재 SOTA
- **4-tier consolidation**: working / episodic / semantic / procedural — Honcho의 reasoning loop + Hindsight의 mental model을 하나로 통합한 형태
- **SQLite + 인메모리 벡터** — 외부 DB 의존 없음, `npx @agentmemory/agentmemory` 1줄 셋업
- 검색: BM25 + 벡터 + **knowledge graph** + RRF
- **3개 에이전트 모두 deep hook 통합** ✅
  - Claude Code: **12 hooks** + MCP + skills
  - Codex CLI: **6 hooks** + MCP + skills
  - Hermes: **6-hook memory provider plugin** + MCP
- 51개 MCP 도구, 내장 브라우저 뷰어
- LLM provider: Anthropic / MiniMax / Gemini / OpenRouter (no-op 기본 — LLM 없이도 동작)
- 임베딩: 로컬 `all-MiniLM-L6-v2` 내장 또는 OpenAI/Voyage/Cohere/Gemini

---

## 3. 종합 비교표

| 차원 | claude-mem | Supermemory | Honcho | Hindsight | seCall | **agentmemory** |
|---|---|---|---|---|---|---|
| 라이선스 | Apache 2.0 | 상용 | AGPL-3.0 | MIT | AGPL-3.0 | Apache 2.0 |
| 가격 | 무료 | 유료(+free) | 무료(self)/유료(cloud) | 무료 | 무료 | 무료 |
| GitHub ⭐ | 46.1k | — | 3.5k | 13.2k | 265 | 6.1k |
| 최신 릴리스 | 활발 | 활발 | v3.0.6 | v0.6.1 (2026-05) | 2026-05 | v0.9.11 (2026-05) |
| 저장소 | 로컬 SQLite | 클라우드 | Postgres | Postgres | SQLite + vault | **SQLite (단일)** |
| 외부 DB 의존 | ❌ | ✅ cloud | ✅ Postgres | ✅ Postgres | ❌ | **❌** |
| 캡처 방식 | hook | hook (3) | SDK | API/MCP/Codex hook | 수동 sync | **hook (3)** |
| 학습 레이어 | △ 압축 | ❌ raw | ✅ reasoning | ✅ Reflect | △ wiki | ✅ **4-tier consolidation** |
| 검색 방식 | 벡터 | 벡터 | 하이브리드 | **4-way + rerank** | BM25+벡터 | BM25+벡터+graph+RRF |
| 벤치마크 (자체보고) | — | — | ❓ Pareto Frontier | ❓ LongMemEval SOTA | — | ❓ **LongMemEval-S 95.2%** |
| 로컬 LLM 지원 | △ Claude SDK | ❌ cloud | △ provider 추가 | ✅ Ollama/LMStudio | ✅ LMS native | △ OpenRouter 경유 |
| 멀티-agent 인덱싱 | 단일 | 단일 | 단일 | 단일 | ✅ **5종 통합** | 단일 |
| 한국어 토큰화 | ❌ | ❌ | ❌ | ❌ | ✅ Kiwi/Lindera | ❓ 미확인 |
| Obsidian 통합 | ❌ | ❌ | ❌ | ❌ | ✅ 전용 플러그인 | ❌ |
| 내장 UI | ❌ | 대시보드 | △ | ❌ | ✅ Web UI | ✅ 브라우저 뷰어 |
| 셋업 난이도 | 1분 | 2분 | 30분 | 1시간 | 30분 | **1분 (npx 1줄)** |
| 운영 부담 | 낮음 | 낮음 | 중 (Postgres) | 높음 (Docker+PG) | 중 (cron) | **낮음** |

---

## 4. 에이전트별 1급 통합 매트릭스 (2026-05 기준)

| 에이전트 | claude-mem | Supermemory | Honcho | Hindsight | seCall | agentmemory |
|---|---|---|---|---|---|---|
| **Claude Code** | ✅ 플러그인 hook | ✅ **native 플러그인** | △ SDK/MCP | △ MCP/skill | △ MCP recall | ✅ **12 hooks** |
| **Hermes Agent** | △ 비공식 | ✅ **native provider** | ✅ **native provider** | ✅ **native provider** | ❌ | ✅ **6 hooks + provider** |
| **Codex CLI** | ✅ **정식 IDE 지원** | ✅ **native 통합** | △ MCP/SDK | ✅ **공식 가이드 (prefetch+retain hook)** | ✅ JSONL 정식 파서 | ✅ **6 hooks + skills** |

**범례**: ✅ = 자동/native 통합 / △ = MCP/API로 가능하나 명시 호출 / ❌ = 통합 없음 또는 보조 수단만

---

## 5. 핵심 차원에서의 도구 포지셔닝

### 자동성 (자동 hook ↔ 수동 호출)
```
완전 자동                                              완전 수동
←─────────────────────────────────────────────────→
agentmemory   Supermemory    Hindsight     seCall
claude-mem    Honcho(SDK)
```

### 회상 → 학습 (단순 키-밸류 ↔ 자체 학습)
```
키-밸류 회상                                          자체 학습
←─────────────────────────────────────────────────→
Supermemory  seCall(wiki)  claude-mem(압축)  Hindsight  agentmemory  Honcho
```

### 로컬 자급도 (클라우드 강제 ↔ 100% 로컬)
```
클라우드 강제                                         완전 로컬
←─────────────────────────────────────────────────→
Supermemory   Honcho(cloud)  Hindsight  Honcho(self)  claude-mem  agentmemory  seCall
```

---

# Part A — Claude Code 사용자

Claude Code는 hook 시스템과 플러그인 생태계가 가장 풍부합니다. 두 부류로 나눠 봅니다.

## A-1. 매니지드 클라우드 OK (가벼움 우선)

> 일반 노트북, 클라우드 비용 OK, 셋업 부담 최소

### ⭐ 추천: **Supermemory**

- 플러그인 1줄 + `SUPERMEMORY_CC_API_KEY` 환경변수만
- 자동 hook 캡처/주입, 사용자 신경 안 써도 됨
- Free tier 부담 적음, 팀 공유 필요시 확장 가능
- 단점: 클라우드 의존, raw turn 누적(가공 약함)

### 대안 1: **agentmemory** (로컬 무료 + 학습)
- 똑같이 1줄 (`npx @agentmemory/agentmemory`), 로컬 SQLite, 무료
- 4-tier consolidation으로 *학습*도 함
- 단점: v0.9.x이라 아직 신생, 한국어 토큰화 미검증

### 대안 2: **claude-mem** (로컬 + AI 압축)
- 완전 로컬, Apache 2.0, Claude SDK로 압축
- 단점: 압축에 Claude API 호출 비용, 학습은 안 함

---

## A-2. 로컬 LLM 가능한 고사양 사용자

> Apple Silicon 64GB+ / GPU 워크스테이션. **MLX/LMS 기반**

### ⭐⭐ 단일 백본 추천: **agentmemory**

이유:
- Claude Code에 **12 hooks** 통합 — 가장 깊음
- SQLite 단일 파일 + 인메모리 벡터 → Postgres/Redis 불필요
- 4-tier consolidation = 학습 레이어 내장
- LongMemEval-S 95.2% (벤더 보고)
- LMS의 OpenAI 호환 endpoint에 `OPENROUTER_BASE_URL`로 연결 가능

### ⭐ 야심찬 이중 레이어: **Supermemory + Hindsight**

```
[Claude Code]
    ├─ Supermemory (hook 자동, 매 turn 캡처)
    │     ↓ 단기 자동 비서
    └─ Hindsight (MCP, 깊은 회상 + Reflect)
          ↓ LMS Qwen 3.6 35B A3B MLX로 reflect
          ↓ 장기 학습 메모리
```

agentmemory가 시장에 안 나왔을 때 추천하던 구성. 지금은 agentmemory 단독이 같은 역할을 더 깔끔하게 수행함. **agentmemory 신뢰성이 검증되기 전 보수적 선택지**로만 유효.

### Obsidian/한국어 우선시: **seCall + agentmemory**

- seCall: 사람용 인터페이스(위키, Knowledge Graph, 한국어 검색)
- agentmemory: 에이전트용 메모리 백본
- 두 도구가 역할 안 겹침

---

# Part B — Hermes Agent 사용자

Hermes(NousResearch)는 "self-improving agent"가 슬로건이고, **메모리 시스템이 1급 시민**입니다.

## 핵심: native MemoryProvider 플러그인 시스템

빌트인 제공자:
```
honcho · mem0 · supermemory · byterover · hindsight ·
holographic · openviking · retaindb · agentmemory
```

→ 즉 Hermes 환경에선 **claude-mem과 seCall은 1급 통합 대상이 아닙니다**.

## 추천 트리

### ⭐⭐ Hermes 철학과 가장 잘 맞음: **Honcho (self-hosted)**
- Plastic Labs가 Hermes같은 에이전트를 위해 설계
- `honcho-self-hosted` 레포가 Hermes Agent용 reference
- peer 모델 + 비동기 reasoning loop = Hermes의 self-improving 철학과 정확히 맞물림
- 셋업: Docker Compose + `memory_provider: honcho` + endpoint

### ⭐⭐ SOTA 회상 + 학습 우선: **agentmemory**
- 6-hook memory provider plugin (Hermes 1급 통합)
- LongMemEval-S 95.2% (벤더 주장)
- 4-tier consolidation
- Apache 2.0이라 Honcho AGPL보다 자유로움

### ⭐ 가장 빠른 시작: **Supermemory** 또는 **Hindsight**
- Supermemory: 클라우드 매니지드, API 키만
- Hindsight: Docker + LMS, SOTA 회상 (단 LongMemEval에선 agentmemory가 위라고 주장)

## Hermes 사용자 결정 트리

```
Q1. 데이터 100% 로컬 통제 필요?
  Yes → agentmemory 또는 Honcho self-hosted
  No  → Supermemory (가장 매끄러움)

Q2. 학습/회고 vs 단순 회상?
  학습 → agentmemory / Honcho / Hindsight
  회상 → Supermemory / 더 가벼운 mem0

Q3. 라이선스 자유도가 중요?
  Apache 2.0 → agentmemory
  MIT → Hindsight
  AGPL 무관 → Honcho

Q4. LMS Qwen MLX로 로컬 reflect?
  Yes → Hindsight (LMS 1급) > agentmemory (OpenRouter 우회)
```

---

# Part C — Codex CLI 사용자

**2026년 초까지**: Codex는 "외부 메모리는 MCP로만"이라는 제약이 컸음.
**2026-04-23 이후 (v0.124.0)**: Codex hooks engine 안정화 → SessionStart/Stop hook이 1급 시민. 자동 메모리 통합 옵션이 폭발적으로 늘어남.

## Codex 기본 메모리 (Codex Memories)
- v0.124+ 기본 OFF, config로 켤 수 있음
- 세션 종료 후 비동기 추출, 시크릿 자동 redact, `codex resume`으로 transcript 이어가기
- 회상은 되지만 학습은 아님, 같은 머신/같은 사용자 한정

## 외부 메모리 옵션

### ⭐⭐ 자동 hook + SOTA: **agentmemory**
- Codex CLI에 **6 hooks + MCP + skills** 정식 통합
- 1줄 셋업, SQLite 로컬
- 학습 레이어 내장

### ⭐⭐ 공식 가이드 보유: **Hindsight**
- Vectorize 공식 블로그에 Codex 통합 가이드
- prefetch + retain의 2-point hook
- LMS Qwen MLX와 가장 매끄러움

### ⭐ Claude와 통합 사용: **claude-mem** (Codex 공식 지원!)
- 2026-05 현재 claude-mem installer가 Codex CLI를 multi-IDE 옵션에 포함
- Claude Code도 같이 쓰는 사람이면 셋업 통일 가능
- AI 압축의 강점이 Codex 세션에도 적용

### ⭐ 매끄러운 클라우드: **Supermemory**
- Codex native 통합 (사용자 검증)
- Claude/Hermes와 같은 store 공유

### ⭐ Obsidian + 멀티 에이전트 아카이브: **seCall**
- Codex CLI JSONL 1급 파서
- 자동 주입 안 됨(MCP recall 명시 호출)
- 장기 검색용으로 강력

## Codex 사용자 결정 트리

```
Q1. 자동 hook 메모리 + 학습 필요?
  Yes → agentmemory (1순위) 또는 Hindsight (안정성 + LMS 친화)
  
Q2. Claude Code도 사용?
  Yes → claude-mem 또는 Supermemory (양쪽 통일)
  
Q3. 데이터 100% 로컬?
  Yes → agentmemory / Hindsight self-host / seCall
  
Q4. Obsidian으로 영구 노트북?
  Yes → seCall (자동성 포기 감수)
```

---

# Part D — 멀티 에이전트 사용자 전략 (Claude + Codex + Hermes)

> 이 섹션은 v2에서 신규 추가. v1은 단일-에이전트 사용자만 가정했으나, 실전에선 셋 다 쓰는 사용자가 많아 별도 다룸.

## 핵심 질문: "백본을 통일할 것인가, 에이전트별 최적화할 것인가?"

### 시나리오 1: **단일 백본 통일**
- 모든 에이전트가 같은 메모리 store를 봄
- 교차 회상 가능: "Codex에서 풀었던 그 문제, Claude에서 다시 묻기"
- 후보:
  - **Supermemory** (cloud, 3-agent native) — 가장 매끄러움, 유료
  - **agentmemory** (local, 3-agent deep hook) — 무료, SOTA, 신생
- ❌ Honcho/Hindsight: Claude 통합이 MCP 수준이라 자동성 떨어짐

### 시나리오 2: **에이전트별 최적 도구**
- 각 에이전트의 hook 시스템에 가장 잘 맞는 도구 따로
- Claude → Supermemory, Codex → Hindsight, Hermes → Honcho
- 장점: 각 에이전트에서 최상의 경험
- 단점: 세 개 따로 관리, 교차 회상 안 됨

### 시나리오 3: **하이브리드 — 자동 백본 + 통합 아카이브**
- 자동 캡처: 각 에이전트별 최적 도구 (또는 통일된 단일 백본)
- 통합 아카이브: seCall이 모든 transcript를 Obsidian vault로 통합
- 자동성과 통합 검색 모두 확보
- 단점: 도구 2개 운영 부담

## 다중 hook 충돌 우려

복수 도구가 같은 hook(SessionStart/Stop)에 등록되면:
- 충돌은 안 남 (각자 독립 작동)
- 그러나 **컨텍스트 주입이 두 번** → 토큰 2배 + 노이즈
- **캡처 비용이 두 번** → 압축 LLM 호출 시 API 비용 가중

→ 백본은 단일이 정답. 통합 아카이브(seCall)는 캡처가 아닌 *수집/검색*이라 충돌 없음.

## 멀티 에이전트 사용자 결정 트리

```
Q1. 최우선 가치는?
  ┌─ 매끄러운 자동성     → Supermemory 단일 백본 (시나리오 1, cloud)
  ├─ 로컬 통제 + SOTA   → agentmemory 단일 백본 (시나리오 1, local)
  ├─ 각자 최적           → 시나리오 2
  └─ 통합 검색 + 자동성  → 시나리오 3 (자동 + seCall 보조)

Q2. 한국어 검색 빈도?
  많음 → seCall 보조 추가 필수 (시나리오 3)
  적음 → 단일 백본으로 충분

Q3. 사양 + 로컬 LLM 가능?
  Yes → agentmemory 또는 Hindsight + 로컬 reflect
  No  → Supermemory cloud
```

## 권장 멀티 에이전트 스택 (사양 무제한 + 로컬 우선 사용자)

```
백본:      agentmemory (1줄 셋업, 3-agent deep hook)
                ↑
                │ OpenRouter base URL 우회
                │
LLM:       LMStudio + Qwen 3.6 35B A3B MLX
                │
                ├─ agentmemory의 압축/요약
                ├─ seCall의 위키 생성
                └─ (선택) Hindsight reflect

선택 보조: seCall (Obsidian + 한국어 + 멀티 에이전트 아카이브)

선택 안전망: Supermemory (Claude Code 1개만 유지 — 익숙함 보존)
```

---

# Part E — 로컬 LLM 스택 (Apple Silicon)

> 사양이 받쳐주는 사용자만 해당. M1 Pro 16GB도 가능하나, M2 Max/M3 Max/M4/M5 Max 32GB+ 권장.

## MLX vs llama.cpp(Ollama): 왜 MLX인가

Apple Silicon에서 동일 모델 기준:
- MLX 추론 속도: llama.cpp Metal 대비 **20-40% 빠름**
- 메모리 효율: unified memory 직접 활용으로 모델 로딩 시간/RAM 점유 더 적음
- MoE 모델(Qwen A3B 등) 가속 효과 특히 큼

→ Apple Silicon 사용자는 **LMStudio + MLX**가 표준. Ollama는 GPU 워크스테이션 또는 Linux/Windows 사용자에게 권장.

## LMStudio 셋업 요점

```bash
# 1. LMS GUI에서 모델 다운로드
#    예: Qwen3.6-35B-A3B-Instruct-MLX (활성 3B param, 총 35B)
# 2. Developer 탭 → Local Server 시작 (기본 포트 1234)
# 3. OpenAI 호환 endpoint 확인
curl http://localhost:1234/v1/models

# 4. 임베딩 모델 별도 로드 (예: bge-m3, nomic-embed-text-v1.5)
#    같은 1234 포트에서 /v1/embeddings 자동 노출
```

## 도구별 LMS 연결

| 도구 | 연결 방법 |
|---|---|
| Hindsight | `HINDSIGHT_API_LLM_PROVIDER=lmstudio` (native) |
| seCall | `--backend lmstudio` (native) |
| agentmemory | `LLM_PROVIDER=openrouter` + `OPENROUTER_BASE_URL=http://localhost:1234/v1` (우회) |
| Honcho | `OPENAI_BASE_URL=http://localhost:1234/v1` (provider override) |
| Supermemory | N/A (클라우드 도구) |

## 추천 모델 매트릭스 (M5 Max / 128GB 기준)

| 용도 | 모델 | RAM | 처리 속도 | 비고 |
|---|---|---|---|---|
| **메인 추론** (압축/요약/wiki) | Qwen 3.6 35B A3B MLX | ~18GB | ~80-120 tok/s | MoE, 대부분 작업에 최적 |
| **임베딩** | BGE-M3 (MLX or GGUF) | ~2GB | — | 한국어 강함, 1024d |
| **빠른 분류** | Qwen 2.5 7B MLX | ~5GB | ~150 tok/s | 태그/제목 같은 마이크로 작업 |
| **깊은 reflect** (선택) | Qwen 2.5 72B MLX | ~45GB | ~15-25 tok/s | 월간 deep retrospective용 |
| **코드 회고** (선택) | DeepSeek-Coder 33B MLX | ~20GB | — | 코드 결정 분석 특화 |

세 개 동시 로드해도 약 60GB → 128GB의 절반 이하.

## MoE A3B vs Dense 70B — 워크플로 임팩트

| | Qwen 3.6 35B A3B (MLX) | Llama 3.3 70B (Ollama/Metal) |
|---|---|---|
| 토큰 처리 속도 | ~80-120 tok/s | ~10-20 tok/s |
| RAM 점유 | ~18GB | ~45GB |
| 품질 (체감) | ~30B급 dense | 70B급 |
| Reflect 1회 소요 | 1-3분 | 10-20분 |
| 가능한 빈도 | **세션마다** | 야간 배치만 |

→ MoE는 **워크플로를 바꿉니다**: reflect를 "야간 배치"가 아니라 "세션 종료마다" 돌릴 수 있게 됨. 메모리가 항상 최신 mental model.

---

# Part F — 의사결정 프레임

## 6.1 차원별 빠른 매칭

| 우선순위 | Claude Code | Hermes | Codex | 멀티 |
|---|---|---|---|---|
| 최단 셋업 | Supermemory | Supermemory | Hindsight | Supermemory |
| SOTA 회상 (벤더 보고 기준) | agentmemory | agentmemory | agentmemory | agentmemory |
| 자체 학습 | agentmemory | Honcho / Hindsight | agentmemory | agentmemory |
| 로컬 100% | agentmemory / claude-mem | Honcho self / agentmemory | Hindsight / agentmemory | agentmemory |
| 멀티 에이전트 영구 아카이브 | seCall | seCall(부분) | seCall | seCall |
| Obsidian 통합 | seCall | seCall | seCall | seCall |
| 팀 공유 | Supermemory | Honcho cloud | Supermemory | Supermemory |
| 한국어 검색 | seCall | seCall | seCall | seCall |
| 라이선스 자유 (Apache/MIT) | claude-mem / agentmemory / Hindsight | agentmemory / Hindsight | claude-mem / agentmemory / Hindsight | agentmemory / Hindsight |

## 6.2 자주 빠지는 함정

### 함정 1: "별 많은 게 좋다"
claude-mem 46k ⭐는 마케팅 효과가 크다. 실제 기능 깊이는 Hindsight(13k)/agentmemory(6.1k)/Honcho(3.5k)가 더 진지함. **별 수 ≠ 품질**.

### 함정 2: "벤더 벤치마크는 진실이다"
LongMemEval SOTA는 Hindsight도 주장했고 agentmemory도 주장 중. **독립 검증 없는 자체 보고는 마케팅 의도가 섞임**. 실제로는 본인 워크로드에서 직접 비교가 정답.

### 함정 3: "무료 = 좋다"
Honcho/Hindsight self-host는 무료지만 Postgres+Docker 유지보수 시간이 비용. **agentmemory는 SQLite로 이 함정을 깸**. Supermemory 유료가 "사람 시간" 기준 더 쌀 수 있음.

### 함정 4: "여러 개 다 쓰자"
메모리 백본은 *덜* 쓰는 게 핵심. 도구가 늘수록 어디 저장됐는지 헷갈리고, SessionStart 토큰만 늘어남. **백본은 하나, 보조는 역할이 안 겹치는 것만**.

### 함정 5: "로컬 LLM이 무조건 우월"
- 추론 작업(reflect, 요약, 분류): 로컬 MoE 35B-A3B로 충분히 SOTA에 가까움
- **임베딩 품질**: OpenAI text-embedding-3-large, Voyage 등 클라우드 모델이 *아직* 우위인 영역 있음 (특히 영어 외 언어)
- 현실적 최적: **추론 로컬 + 임베딩 하이브리드** (한국어는 BGE-M3 로컬이 충분)

### 함정 6: "Codex는 메모리 어렵다" (구식 정보)
v0.124+ hooks engine 안정화로 Claude Code와 거의 동등한 자동 메모리 가능. **2026 H1 이전 가이드는 outdated**.

## 6.3 솔직한 한 줄 결론

- **Claude Code 단독, 가벼움**: Supermemory로 시작 → 한계 보이면 agentmemory로 이동
- **Claude Code 단독, 고사양**: agentmemory 단독 (압축 LLM은 LMS Qwen MLX)
- **Hermes 메인**: Honcho self-hosted (철학 호환) 또는 agentmemory (성능)
- **Codex 메인**: agentmemory 또는 Hindsight (공식 가이드 있음)
- **셋 다 사용**: **agentmemory 단일 백본** + (필요시) seCall 보조
- **Obsidian/한국어/멀티-agent 아카이브**: seCall은 거의 모든 시나리오에서 *보완재*로 추가 가능

---

# Part G — 실전 구성 시나리오

> 사용자 페르소나별로 즉시 시작할 수 있는 풀-스택 4종.

## 시나리오 A: 가볍게 시작 (Claude Code 단독)

```
Supermemory (플러그인) - 끝.
```

- 셋업 5분
- 클라우드 의존 OK
- 비용: free tier 또는 월 소액

## 시나리오 B: agentmemory 단일 백본 (3 에이전트, 로컬 우선) ⭐ 권장

```
백본:    agentmemory (Claude + Codex + Hermes 모두 hook)
         - npx @agentmemory/agentmemory
         - SQLite, 로컬

LLM:     LMStudio + Qwen 3.6 35B A3B MLX
         - localhost:1234
         - agentmemory가 OpenRouter base URL 우회로 연결

임베딩:  LMS BGE-M3 (또는 agentmemory 내장 MiniLM)

(선택)   seCall (Obsidian 통합 + 한국어 검색 + 멀티 transcript 아카이브)
```

- 셋업: 1-2시간
- 운영: SQLite 백업 외 거의 무관
- 비용: 무료 (전기 + 디스크)

## 시나리오 C: Obsidian 영구 아카이브 + 매끄러운 자동성

```
자동 백본:  Supermemory (3 에이전트 native)
            - 클라우드, 익숙, 빠른 셋업

영구 아카이브: seCall + LMS Qwen 3.6 35B A3B MLX
              - 한국어 검색
              - Obsidian vault
              - 매시간 cron sync
```

- 셋업: 1시간
- 운영: cron + 월간 vault 정리
- 비용: Supermemory 구독 + 무료 로컬

## 시나리오 D: 풀스택 (이중 메모리 + 영구 아카이브)

```
실시간 자동:    Supermemory (Claude 익숙함 + 팀 공유 옵션)
                또는 agentmemory (로컬 SOTA)

학습 레이어:    agentmemory의 4-tier consolidation
                (또는 Hindsight Reflect + LMS Qwen MLX)

영구 아카이브:  seCall (모든 에이전트 transcript)

LLM 백엔드:    LMStudio + Qwen 3.6 35B A3B MLX
                + (선택) Qwen 2.5 72B MLX (월간 deep reflect)
                + BGE-M3 임베딩
```

- 셋업: 3-5시간
- 운영: 도구 3개, 캐던스 명확
- 비용: 무료 또는 Supermemory 소액
- **언제 정당화되는가**: 메모리 정확도가 비즈니스 가치(컨설팅, 연구, 코드베이스 대형)와 직결될 때

---

# Part H — 트렌드 & 향후 전망 (2026 H2 예측)

## 1. Hook API 표준화 → 도구 진입장벽 폭락
- 모든 주요 에이전트(Claude/Codex/Hermes/Gemini)가 hook 시스템 안정화
- 신규 메모리 도구가 빠르게 등장 (agentmemory가 그 사례)
- → 도구 시장은 **수렴**이 아닌 **분화** 단계. 기능 격차 빠르게 좁혀짐

## 2. MoE + 로컬 추론의 임팩트
- Qwen A3B 같은 MoE가 일반화 → reflect를 야간 배치에서 실시간으로
- 클라우드 메모리(Supermemory)의 차별점은 *데이터 통제*가 아닌 *팀 공유*로 좁아짐
- 자체 호스팅 도구의 매력 상승

## 3. "학습"의 표준화
- Reflect / Reasoning / Consolidation 등 도구마다 명칭 다르나 **본질은 같음**
- 2026 H2엔 4-tier(working/episodic/semantic/procedural)가 사실상 표준이 될 가능성
- 검증된 벤치마크(LongMemEval 등) 가 평가 지표로 자리잡는 중

## 4. Knowledge Graph의 부활
- agentmemory, seCall, Hindsight 모두 KG 도입
- 단순 벡터 검색의 한계(엔티티 관계 파악 실패)를 보완
- 2026 H2엔 KG 없는 메모리 도구는 경쟁력 떨어질 듯

## 5. Obsidian 통합의 의미
- 사람-AI 공동 노트북이 표준 인터페이스로 부상
- 메모리 도구가 단순 백엔드를 넘어 *인간이 읽고 편집 가능한 형태*로 출력하는 방향
- seCall이 선구적, 향후 다른 도구도 따라올 가능성

---

# 부록

## A. GitHub 링크

- claude-mem: <https://github.com/thedotmack/claude-mem>
- Supermemory (Claude 플러그인): <https://github.com/supermemoryai/claude-supermemory>
- Supermemory MCP installer: <https://github.com/supermemoryai/install-mcp>
- Honcho: <https://github.com/plastic-labs/honcho>
- honcho-self-hosted (Hermes용): <https://github.com/elkimek/honcho-self-hosted>
- Hindsight: <https://github.com/vectorize-io/hindsight>
- seCall: <https://github.com/hang-in/seCall>
- agentmemory: <https://github.com/rohitg00/agentmemory>

## B. 공식 가이드 및 docs

- claude-mem 설치 (Codex 포함): <https://docs.claude-mem.ai/installation>
- Codex MCP & hooks 공식 문서: <https://developers.openai.com/codex/mcp>
- Codex CLI 레퍼런스: <https://developers.openai.com/codex/cli/reference>
- Hindsight + Codex 통합 가이드: <https://hindsight.vectorize.io/blog/2026/04/08/adding-memory-to-codex-with-hindsight>
- Hermes Memory Provider 아키텍처: <https://hermes-agent.nousresearch.com/docs/developer-guide/memory-provider-plugin>

## C. v1 → v2 변경 이력

| 항목 | v1 (2026-05-13) | v2 (2026-05-13) |
|---|---|---|
| 비교 도구 수 | 5개 | **6개** (agentmemory 추가) |
| Supermemory Codex 통합 | MCP 수준으로 기술 | **native 통합으로 정정** |
| claude-mem Codex | "README만 있음" | **정식 IDE 지원으로 정정** |
| Codex hook engine | 불확실 | **v0.124.0 안정화 명시** |
| 로컬 LLM 권장 | Ollama Llama 3.3 70B | **LMS + Qwen 3.6 35B A3B MLX** |
| 멀티 에이전트 섹션 | 없음 | **Part D 신규** |
| 실전 시나리오 | 결론 한 줄 | **Part G 4종 풀스택 신규** |
| Executive Summary | 없음 | **신규** |
| 검증 표기 | 없음 | **✅ / △ / ❓ 도입** |
| 트렌드 분석 | 없음 | **Part H 신규** |

## D. 한계 및 면책

- **벤치마크 주장**: LongMemEval SOTA 등 일부 성능 지표는 벤더 자체 보고이며 독립 검증 자료가 부족합니다. 본인 워크로드에서 직접 비교가 권장됩니다.
- **빠른 변화**: 이 시장은 월 단위로 변합니다. 이 리포트는 2026-05-13 기준이며, 6개월 후엔 통합 매트릭스가 상당히 바뀔 가능성이 큽니다.
- **개인적 컨텍스트**: 본 리포트는 Apple Silicon + 고사양 사용자 + 한국어 워크플로 + 멀티-에이전트 사용자를 다소 우대하는 시각이 반영됐습니다. Linux GPU 워크스테이션이나 영어-only 사용자에겐 가중치가 달라질 수 있습니다.

---

*도구를 고르는 진짜 기준은 "내 워크플로에 자연스럽게 녹는가"입니다. 2026의 메모리 도구 시장은 기능 격차가 빠르게 좁혀지고 있어, 1년 전과 달리 **무엇을 고르느냐**보다 **무엇을 일관되게 쓰느냐**가 더 중요해졌습니다. 가장 매끄러운 것부터 시작해 실제 마찰을 느낀 다음 진화하세요.*

— *AI Memory Tools Comparison v2.0, 2026-05-13*
