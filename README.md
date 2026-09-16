[한국어](README.ko.md) | [English](README.md) | [日本語](README.ja.md)

# LLM Orchestrator (working title)

> A development environment where multiple LLMs collaborate within a single workflow, with the system itself managing the information hand-off and context between them.

## Why

Using several LLMs at once during development usually ends up looking like this:

```
Ask LLM A → copy the result → paste into LLM B → copy B's review
→ paste back into A → compare results yourself → summarize what matters
→ paste into yet another AI …
```

Along the way you get context loss, missing information, stale versions being passed around,
copy-paste mistakes, distortion introduced by the human interpreting in the middle, and a lot of
unnecessary coordination overhead. Even an AI's own "export" feature can't guarantee it preserves
the original context 100%.

The question underneath all of this is simple:

> **"Why do I have to be the messenger between AIs?"**

This project starts from that question. Instead of a human shuttling information between AIs,
**the AIs exchange what they need directly and collaborate with each other, while the human owns
goal-setting, judgment, selection, and approval.**

## What this is

Not a Multi-LLM chat app, not just an AI coding tool. The goal is:

> An **LLM orchestration environment** that lets multiple LLMs collaborate inside one workflow, automating the context and information hand-off between them.

Development is the first use case, but the project isn't limited to it. The same orchestration
structure applies to any workflow that boils down to "gather from multiple sources, analyze and
verify, then let a human decide" — planning/DX, research, general office work, and more.

## Core principles

| # | Principle |
|---|---|
| 1 | Multi-LLM itself is not the goal — the point is automating coordination between AIs |
| 2 | Remove the step where a human relays information between AIs |
| 3 | Use a **Structured Shared Context** instead of just forwarding chat history |
| 4 | AIs should be able to review and revise each other's output directly |
| 5 | Cross-check is **verification**, not majority vote |
| 6 | Fact-checking cross-checks the **evidence itself**, not just the conclusion |
| 7 | Weight is Priority/Influence, not a Truth Score (`Weight ≠ Truth`) |
| 8 | Model selection always stays with the user |
| 9 | Provider and Model are treated as separate concepts |
| 10 | Model names are never hardcoded into core — **Dynamic Model Discovery** |
| 11 | Token/cost is something the user directly controls |
| 12 | Workflow intensity is adjustable: Quick / Review / Debate / Deep |
| 13 | **The human is always the final decision-maker** |
| 14 | The initial entry point is a VS Code Extension; Core is designed separately from it |
| 15 | Long-term, it should be able to grow into a standalone LLM workspace |

## Division of roles

**AI handles**: research · requirements analysis · planning · system design · implementation · code review · counter-argument · comparison · verification · proposing revisions · fact-checking

**The developer handles**: setting goals · deciding key requirements · trade-off judgment · comparing results · final selection · approval

```
AI opinion → developer review → selection → approval → applied
```

This isn't AI replacing the developer — it's removing the manual relay work required to use
multiple AIs at once.

## Development workflow (example)

```
Idea → Requirements → Research → Design → Architecture
→ Implementation Plan → Code → Review → Revision → Verification
```

The goal is one continuous LLM environment that spans from planning through development and
verification. Example flow:

1. Roughly describe the idea
2. LLMs flesh out requirements and surface ambiguities
3. Research relevant material
4. Multiple LLMs draft designs → review each other → compare trade-offs
5. The developer picks a design → it's saved to the Project Context
6. Implementation plan → code generation → another LLM reviews it → the original author LLM revises
7. Testing/verification → developer's final approval

## Cross-check: verification, not majority vote

Rather than showing multiple models' results side by side, the same issue is clustered together,
and each model's agree/disagree along with its reasoning is shown.

```
Issue #12 — service.py:47 — Potential None handling bug
  GPT     → Agree
  Claude  → Agree
  Gemini  → Disagree (reason: ...)
```

If two agree and one disagrees, that doesn't automatically make the majority correct. The final
call belongs to the human who has seen the evidence.

## Fact-checking: cross-check the evidence

Models' evidence — not just their conclusions — is cross-checked against each other. The system
never collapses the following four into a single score:

- Priority (configured priority)
- Confidence (the model's stated confidence)
- Evidence (actual supporting evidence)
- Consensus (whether models agree)

## Provider / Model / Dynamic Discovery

```
Provider Adapter → Model Discovery → Model Registry → Workflow
```

- **Provider** (OpenAI, Anthropic, Google …) and **Model** (GPT, Claude Sonnet, Claude Opus, Gemini …) are designed as separate concepts. The unit the user selects is the model.
- Model names are never hardcoded into the application. A Provider Adapter dynamically discovers what models are available, and Core treats them through a generic model abstraction.
- The goal is that a new model becomes available through discovery without requiring a core code change.

## Shared Project Context

Instead of forwarding chat history, a structured project context is maintained.

```
Project Context
├── Requirements
├── Constraints
├── Architecture
├── Decisions       (includes reasoning / model agree-disagree / approver)
├── Research
├── Source Evidence
├── Current Code
├── Open Issues
└── Change History
```

This structure becomes the AIs' shared context, and makes it possible to trace what was decided,
why, and who approved it.

## Workflow intensity (cost/latency control)

| Mode | Description |
|---|---|
| ⚡ Quick | 1 LLM, fast response, low cost |
| 🔍 Review | Generate → 2–3 LLMs review |
| ⚔️ Debate | Multiple models present views → rebut → re-examine |
| 🧠 Deep | Generate → Review → Debate → Revise → Re-review → Verify |

Running everything through Deep would be inefficient, so early on the user explicitly picks the
mode.

## Architecture direction

```
VS Code Extension
        │
        ▼
   Core Engine
        │
 ┌──────┼──────┐
 ▼      ▼      ▼
Model  Context Workflow
Registry Store  Engine
        │
        ▼
Provider Adapters
```

VS Code isn't the product itself — it's the first client. Core is designed separately from VS
Code from the start so it can later extend to CLI / Web / Desktop and beyond.

## Usage model (BYOK)

Users connect with their own API keys to each provider (Bring Your Own Key). This project isn't a
service that resells or proxies AI access — it's **software/a client that coordinates between
providers**.

## MVP scope (draft)

- VS Code Extension
- BYOK
- 2–3 providers
- Dynamic Model Discovery
- Shared Context
- Generate / Review / Revise workflow
- Human approval

## Growth path

```
Personal dev tool
   → Multi-LLM Development Workspace
   → LLM Collaboration Workspace
   → General LLM Integrated Workspace
   → B2B applications across IT / DX / manufacturing / research / planning
```

Designed on the hypothesis that only the UI and connectors need to change per industry, while Core
orchestration can be shared. Actual commercial viability still needs to be validated with real
customers.

## Current status

Early planning draft. Working through Problem Definition, MVP scope, Core Engine architecture
(Provider Adapter, Model Registry), and the Shared Project Context schema in sequence.

---

*This README is a draft and will keep being updated as the project progresses.*
