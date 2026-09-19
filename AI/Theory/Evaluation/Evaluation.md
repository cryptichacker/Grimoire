---
tags: [ai, theory, evaluation, benchmarks]
type: cheatsheet
---

# Evaluation

## Up
- [[Theory]]

How you know a model (or an LLM app) is actually good — beyond vibes. Covers standard benchmarks, ways to judge open-ended output, and the failure modes (hallucination) you're guarding against.

---

## Levels of evaluation

| Level | Question | Methods |
|---|---|---|
| **Model** | is this LLM capable? | academic benchmarks, arenas |
| **System / app** | does *my* RAG/agent work? | task datasets, LLM-as-judge, human eval |
| **Production** | is it working *live*? | online metrics, tracing, user feedback |

---

## Common benchmarks

| Benchmark | Tests |
|---|---|
| **MMLU / MMLU-Pro** | broad knowledge across 57 subjects |
| **GPQA** | graduate-level "Google-proof" science reasoning |
| **HumanEval / MBPP** | code generation (pass@k) |
| **SWE-bench (Verified)** | fix real GitHub issues — agentic coding |
| **GSM8K / MATH / AIME** | grade-school → competition math |
| **HellaSwag / ARC / WinoGrande** | commonsense reasoning |
| **MT-Bench / Chatbot Arena** | chat quality; **Arena** = human pairwise Elo |
| **GAIA / WebArena / τ-bench** | agent & tool-use tasks |

⚠️ **Benchmark caveats:** **contamination** (test data leaked into training), overfitting to leaderboards, saturation (scores near 100%), and gaps vs. real-world use. Treat as directional, not gospel.

---

## Evaluating open-ended output

- **Reference-based** — compare to a gold answer: **BLEU/ROUGE** (overlap, weak for meaning), **BERTScore** (semantic), exact-match/F1.
- **LLM-as-a-judge** — a strong model scores/ranks outputs against a rubric. Scalable but has biases (position, verbosity, self-preference) → mitigate with pairwise + randomized order + rubrics.
- **Human evaluation** — gold standard for nuance; expensive; use rubrics + multiple raters (inter-rater agreement).
- **Task metrics** — for classification/extraction use precision/recall/F1 ([[ML Fundamentals]]).

---

## Hallucination
- Confident, fluent, **false** output. Types: **intrinsic** (contradicts the given context) vs **extrinsic** (unsupported by any source).
- **Reduce:** [[RAG|ground with retrieval]], demand citations, "say unknown if unsure," lower temperature, verification/critique passes, smaller claims.
- **Measure:** faithfulness/groundedness checks (RAGAS, TruLens), fact-verification, NLI-based entailment.

---

## Guardrails & safety evals
- **Input/output filters** — block unsafe requests/PII/toxicity; validate schema.
- **Red-teaming / jailbreak tests** — adversarial prompts (→ [[Ethics & Safety]], [[Hacking]] Web LLM attacks).
- Tools: **NeMo Guardrails, Guardrails AI, Llama Guard**.

---

## Evaluating LLM apps (practical loop)
1. Build a **test set** of representative inputs (+ expected traits).
2. Define **metrics** (accuracy, faithfulness, latency, cost, safety).
3. Automate with an **eval framework** (RAGAS, DeepEval, promptfoo, LangSmith, Braintrust).
4. **Trace** everything in prod; sample & review; add regressions to the test set.
5. Watch for **drift** and **regressions** on every prompt/model change.

## Key terms
- **pass@k** · **Elo (Arena)** · **contamination** · **LLM-as-judge** · **groundedness/faithfulness** · **eval set / golden dataset**.

## Related
- [[RAG]] — retrieval-specific metrics · [[Agents]] — agent benchmarks
- [[Ethics & Safety]] — red-teaming & guardrails
