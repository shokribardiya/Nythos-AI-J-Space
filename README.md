# J Space for Local AI Model in LM Studio

AI J Space
# NYTHOS

## External Global Workspace for Local AI

> **Memory is not intelligence. Context is not memory. A model is not a system.**

Nythos is a lightweight, local-first runtime that gives local language models a persistent external workspace:

**memory → salience → bounded workspace → conflict checking → context compilation → model interaction**

It is intentionally not a model, not a replacement for LM Studio, and not a claim to reproduce internal neural mechanisms such as Claude's J-space.

The current implementation is a single Python file using the standard library and SQLite. It communicates with LM Studio through a local MCP/loopback architecture and is designed to avoid model lifecycle control, arbitrary shell execution, network listeners, and invasive modification of LM Studio.

---

## Why Nythos exists

A strong local model is still operating inside an environment.

Without persistent state, it forgets project decisions.

Without context selection, it receives too much irrelevant history.

Without provenance, a generated statement can quietly become a "fact".

Without conflict detection, contradictory memories can coexist unnoticed.

Without bounded workspace management, context becomes a landfill.

Nythos focuses on that missing layer.

### The architecture

```text
                         USER
                           │
                           ▼
                   ┌──────────────┐
                   │ Task / Intent │
                   └──────┬───────┘
                          ▼
                   ┌──────────────┐
                   │   NYTHOS     │
                   │ Memory       │
                   │ State        │
                   │ Workspace    │
                   │ Context      │
                   └──────┬───────┘
                          ▼
                 ┌──────────────────┐
                 │ Local AI Runtime │
                 │   LM Studio      │
                 └──────┬───────────┘
                        ▼
                 Local model
```

In the broader Angra ecosystem:

```text
                 ┌──────────────────────┐
                 │        NYTHOS        │
                 │ Memory / State       │
                 │ Workspace / Context  │
                 └──────────┬───────────┘
                            │
                            ▼
                 ┌──────────────────────┐
                 │        ANGRA         │
                 │ Routing / Relay      │
                 │ Collaboration        │
                 │ Verification         │
                 └──────────┬───────────┘
                            │
                 ┌──────────┼───────────┐
                 ▼          ▼           ▼
              Primary     Critic    Specialist
               model       model       model
```

Nythos answers:

- What do we know?
- What has been verified?
- What happened before?
- What is relevant right now?
- What is the current task state?

Angra answers:

- Which model should contribute?
- Should another model critique the result?
- Should we escalate effort?
- How should evidence and corrections move between models?

---

# Core design

## 1. Persistent memory

Memories are typed and provenance-aware.

Current memory types include:

- episodic
- semantic
- procedural
- decision
- evidence
- observation
- uncertainty

Memory states:

- candidate
- active
- verified
- stable
- superseded
- archived

A core rule is:

```text
MODEL OUTPUT != TRUTH
```

Model-created memories begin as untrusted/candidate information. A model cannot silently promote its own output to a verified user fact.

The implementation also deduplicates memories by normalized content hash and records corroboration.

---

## 2. Salience and bounded workspace

Nythos does not dump every stored memory back into every request.

It ranks memories using a deterministic, explainable scoring system based on factors such as:

- relevance
- goal match
- importance
- confidence
- recency
- usage
- novelty
- contradiction

The default workspace is intentionally small and bounded.

```text
Default workspace: 24
Hard maximum:       64
Candidate pool:    400
```

These are engineering limits, not claims about any model's internal capacity.

---

## 3. Conflict detection

Contradiction is first-class state.

Nythos detects conflicts such as:

```text
"LM Studio feature X is supported."

vs.

"LM Studio feature X is unavailable."
```

or numeric/value mismatches.

Trusted user/verified memories are not automatically degraded merely because a lower-trust model claim contradicts them.

Important conflicts require user authority to resolve.

---

## 4. Context compilation

The model-facing context is assembled from:

```text
Long-term memory
       ↓
Current task
       ↓
Salience
       ↓
Conflict filtering
       ↓
Evidence
       ↓
Recent observations
       ↓
Active workspace
       ↓
Compact context packet
```

The point is not "more context".

The point is **better context**.

---

## 5. Sessions and continuity

A session keeps:

- active goal
- workspace snapshot
- observations
- durable memories

The goal is to let a local model continue useful work without replaying the entire transcript every time.

---

# Safety philosophy

Nythos is deliberately conservative around the host machine.

It does not:

- load models automatically
- unload models automatically
- download models
- silently switch the selected model
- patch LM Studio binaries
- patch Electron bundles
- open network ports
- execute model-generated shell commands
- provide generic shell access
- provide unrestricted filesystem access
- treat model output as trusted system instructions
- store hidden/private chain-of-thought

The only external configuration operation is the explicitly backed-up and ownership-checked LM Studio `mcp.json` integration.

The local data store uses SQLite with:

- foreign keys
- WAL
- `synchronous=FULL`
- busy timeout
- transaction rollback
- atomic file replacement

---

# LM Studio integration

Nythos is designed around LM Studio's supported local integration model rather than modifying its internals.

The runtime can expose itself as an MCP server over STDIO:

```bash
python nythos.py --mcp
```

The MCP process uses:

- stdout for protocol only
- stderr for diagnostics

The integration is intentionally loopback/local.

---

# The most important scientific boundary

Nythos is an **external software-level analog of a global workspace architecture**.

It does not access internal neural activations.

It does not inspect hidden model states.

It does not implement Claude's internal J-space.

It stores observable information such as:

- generated conclusions
- task state
- tool results
- evidence
- decisions
- uncertainty
- runtime statistics

This distinction matters.

A software workspace around a model is useful.

Pretending it is the model's internal activation space would just be marketing.

---

# Current implementation test

The uploaded baseline was executed directly with:

```bash
python nythos.py self-test
```

Result:

```text
18 / 21 tests passed
```

Passed areas include:

- atomic write and recovery
- database schema
- installer rollback
- JSON-RPC parsing
- logo rendering
- MCP end-to-end tools
- MCP handshake
- memory poisoning / duplicate handling
- security capability restrictions
- path validation
- request IDs
- runtime stdout purity
- salience and workspace
- state transitions
- tool routing
- transactions
- validation limits
- workspace conflict filtering

Three tests currently fail:

```text
conflict_detection
installer_merge_and_uninstall
sessions_and_consolidation
```

So this baseline should **not** be described as fully production-ready yet.

The self-test is deliberately model-free. A green infrastructure test is not the same thing as proving that a model became better.

---

# Model comparison

## Important methodological note

The table below is **not** a fabricated "Nythos benchmark".

The current `nythos.py` baseline performs model-free runtime tests and does not contain a directly comparable cross-model benchmark result set. Therefore, the public model numbers below are shown as **external reference benchmarks**, while Nythos suitability is an engineering interpretation rather than a measured score.

The clean comparison we still need to run is:

```text
RAW MODEL
   vs
MODEL + NYTHOS
   vs
MODEL + NYTHOS + ANGRA
```

under the same tasks, quantization, context, temperature, hardware, and evaluation harness.

---

## Public benchmark snapshot

| Model | Important public result | What it suggests for Nythos |
|---|---:|---|
| Claude Sonnet 5.5 | Terminal-Bench 4.0: **70.6%** | Extremely strong agentic/coding reference; proprietary/cloud model |
| Claude Fable 5.1 | Terminal-Bench 4.0: **55.8%**; CursorBench 3.2: **73.4%** | Long-horizon coding/research reference; proprietary/cloud model |
| GPT-OSS 20B | SWE-Bench Verified: **60.7%** at high reasoning | Particularly interesting local reasoning/coding candidate |
| Gemma 4 12B | LiveCodeBench v6: **77.1%**; GPQA: **82.3%** | Strong local multimodal/agentic candidate at laptop scale |
| Gemma 4 E4B | LiveCodeBench v6: **52.0%**; GPQA: **58.6%** | Efficiency-oriented local worker / specialist |
| GLM-4.6V-Flash | **9B**, **128k context**, vision, reasoning, native multimodal function calling | Excellent local visual specialist candidate |

These numbers come from different benchmark suites and evaluation settings. They must not be averaged into one fake leaderboard.

---

# What the numbers actually mean

### Sonnet 5.5

Anthropic reports:

- Terminal-Bench 4.0: 70.6%
- CursorBench 4.0: 55.5%
- Humanity's Last Exam with tools: 64.5%
- OSWorld 2.1 partial: 80.1%
- Chartography: 61.6%

Anthropic also emphasizes adaptive effort, long-horizon work, efficient tool use and fewer steps on some tasks.

That makes Sonnet 5.5 an excellent **behavioral reference ceiling** for the Nythos architecture.

It is not, however, a local LM Studio model.

### Fable 5.1

Anthropic reports:

- Terminal-Bench-Science 0.1: 52.6%
- Terminal-Bench 4.0: 55.8%
- GDPval-AA v2: 1853
- OSWorld 2.0 partial: 77.9%
- OSWorld 2.0 strict: 41.7%
- Humanity's Last Exam with tools: 65.0%
- AutomationBench: 31.4%
- CursorBench 3.2.0: 73.4%

Anthropic describes Fable 5.1 as the same underlying model as Mythos 5.1 with different safeguards.

Again, this is a **frontier reference**, not a model we should pretend can simply be dropped into LM Studio.

### GPT-OSS 20B

OpenAI reports the open-weight gpt-oss-20b model at high reasoning with:

- AIME 2025 with tools: 98.7%
- GPQA Diamond: 71.5%
- MMLU: 85.3%
- SWE-Bench Verified: 60.7%
- Tau-Bench Retail: 54.8%
- Codeforces with tools: 2516 Elo

It is explicitly designed for local / low-latency use and exposes adjustable reasoning effort.

This makes it one of the most interesting candidates for an experiment like:

```text
GPT-OSS 20B
      +
Nythos
      +
Angra
```

### Gemma 4

Google's published Gemma 4 results show a very strong local story.

For Gemma 4 12B:

- AIME 2026: 88.3%
- LiveCodeBench v6: 77.1%
- GPQA Diamond: 82.3%
- MMMU Pro: 73.8%
- τ2-bench Retail: 85.5%

For Gemma 4 E4B:

- AIME 2026: 42.5%
- LiveCodeBench v6: 52.0%
- GPQA Diamond: 58.6%
- MMMU Pro: 52.6%
- τ2-bench Retail: 57.5%

Google explicitly positions Gemma 4 as a family designed for efficient deployment, including laptop-class hardware.

That makes 12B a strong general local candidate and E4B an attractive lightweight specialist.

### GLM-4.6V-Flash

Z.ai describes GLM-4.6V-Flash as a 9B model optimized for local deployment and low latency with:

- 128k context
- vision input
- reasoning
- native multimodal function calling

That is a different strength profile from GPT-OSS 20B.

It is especially interesting as a visual specialist inside Angra rather than as the universal primary model.

---

# Proposed Nythos/Angra model roles

Instead of asking:

> Which model is #1?

the architecture asks:

> Which model is best at this job?

A practical local configuration might look like:

```text
PRIMARY
GPT-OSS 20B
Reasoning / coding

CRITIC
Gemma 4 12B
Alternative reasoning / verification

VISION SPECIALIST
GLM-4.6V-Flash
Screenshots / images / documents

LIGHTWEIGHT SPECIALIST
Gemma 4 E4B
Fast local side tasks

REFERENCE CEILINGS
Claude Sonnet 5.5
Claude Fable 5.1
```

The last two are reference systems for capability comparison, not local LM Studio participants.

---

# The experiment that matters

The real Nythos study should not be:

```text
Model A > Model B
```

It should be:

```text
                    SAME TASK SET
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
       RAW MODEL      + NYTHOS      + NYTHOS+ANGRA
          │              │              │
          └──────────────┼──────────────┘
                         ▼
                 SAME EVALUATOR
```

Measure:

- task success
- first-pass success
- repair rate
- tool calls
- total tokens
- reasoning tokens when exposed
- TTFT
- tokens/second
- context size
- memory retrieval precision
- contradiction rate
- verification score
- recovery after restart
- long-task completion

Only then can we honestly answer whether Nythos actually makes local models better.

---

# Roadmap

### Phase 1
Make the existing runtime fully green.

```text
21/21 self-test
```

### Phase 2
Add reproducible local model evaluation.

### Phase 3
Measure:

```text
RAW
vs
NYTHOS
vs
NYTHOS + ANGRA
```

### Phase 4
Add long-horizon task state and checkpoints.

### Phase 5
Add adaptive effort and semantic relay.

### Phase 6
Add verifier + dissent engine.

### Phase 7
Publish the benchmark data, including failures.

---

# Philosophy

Nythos is not trying to prove that a small model secretly contains a frontier model.

That would be nonsense.

The more interesting question is:

> **How much of the frontier-model experience comes from the model itself, and how much comes from the system around it?**

Nythos is an experiment in answering that question.

**One model gives you intelligence.**

**A good runtime gives that intelligence memory, context, state, tools and continuity.**

**Angra gives multiple models a way to cooperate.**

The goal is not to turn a local model into Claude.

The goal is to build a local environment in which capable open models can operate at a much higher level than a blank chat window would suggest.

---

## Status

**Experimental / active development**

Current baseline:

```text
Infrastructure self-test: 18/21
Model benchmark suite:    not yet run end-to-end
LM Studio live launch:    not verified by self-test
```

That is the honest status.

And honesty is considerably easier to maintain than a fake leaderboard.
