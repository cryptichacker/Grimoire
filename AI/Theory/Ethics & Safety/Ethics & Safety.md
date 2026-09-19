---
tags: [ai, theory, safety, alignment, security, ethics]
type: cheatsheet
---

# Ethics & Safety

## Up
- [[Theory]]

The risks of deploying LLMs and how to mitigate them — spanning **alignment** (does it do what we want?), **security** (can it be abused?), and **societal** concerns (bias, privacy). The bridge between [[Theory]] and the adversarial work in [[Hacking]].

---

## Alignment
- **Goal:** models that are **Helpful, Honest, Harmless (HHH)** and pursue intended goals.
- **Outer vs inner alignment** — is the *objective* right, and does the model *internalize* it (vs. reward-hacking / [[Training & Alignment|proxy gaming]])?
- **Techniques:** RLHF, DPO, **Constitutional AI** (rules-based self-critique), red-teaming, interpretability.
- **Longer-term worries:** deception, power-seeking, scalable oversight (supervising models smarter than us), specification gaming.

---

## Security threats (LLM-specific)

| Threat | What it is |
|---|---|
| **Prompt injection** | malicious instructions in user input **or retrieved/tool content** hijack the model |
| **Indirect injection** | payload hidden in a web page/doc/email the agent reads |
| **Jailbreaking** | prompts that bypass safety (role-play, obfuscation, many-shot) |
| **Data exfiltration** | trick the model/agent into leaking secrets, context, or files |
| **Insecure output handling** | trusting model output → XSS/SQLi/RCE downstream |
| **Excessive agency** | over-permissioned tools → real-world damage |
| **Training-data poisoning** | corrupt data to implant backdoors/bias |
| **Model/prompt theft** | extraction of weights or system prompts |
| **Supply chain** | malicious models, skills, MCP servers ([[nvidia-skillspector]]) |

> This maps to the **OWASP Top 10 for LLM Applications** — cross-link [[Hacking]] → *Web LLM attacks* (PortSwigger).

### Defenses
Treat all tool/retrieved content as **untrusted** · least-privilege tools + human-in-the-loop for risky actions · input/output **guardrails** (Llama Guard, NeMo, Guardrails AI) · sandboxing · output validation before use · red-team continuously · don't put secrets in prompts.

---

## Societal & ethical concerns
- **Bias & fairness** — models inherit training-data bias; can amplify stereotypes. Mitigate: data curation, evals across groups, fairness metrics.
- **Privacy** — memorization/leakage of PII; **PII redaction**, data governance, differential privacy, respecting consent.
- **Misinformation** — hallucinations + scalable persuasion, deepfakes.
- **Copyright & data provenance** — training-data rights, attribution, licensing.
- **Environmental cost** — training/inference energy & water.
- **Labor & economic impact** — automation, displacement.
- **Transparency** — model cards, disclosure that content is AI-generated, watermarking.

---

## Governance & standards
- **NIST AI Risk Management Framework**, **EU AI Act** (risk tiers), **ISO/IEC 42001**.
- **OWASP Top 10 for LLMs**, **MITRE ATLAS** (adversarial ML threat matrix).
- Org practices: **red teams, model cards, evals gating release, incident response, usage policies.**

## Key terms
- **HHH** · **red-teaming** · **guardrails** · **prompt injection (direct/indirect)** · **jailbreak** · **reward hacking** · **model card** · **RAI (Responsible AI)**.

## Related
- [[Training & Alignment]] — how alignment is trained in
- [[Evaluation]] — safety evals & red-teaming
- [[Hacking]] → Web LLM attacks · [[Agents]] — where agency risk is real · [[nvidia-skillspector]]
