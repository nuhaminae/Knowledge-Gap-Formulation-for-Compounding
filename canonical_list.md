# Canonical List — Papers, Tools, and Patterns for FDE-Grade AI Systems

This is my annotated contribution to the cohort canon: the sources, tools, and engineering patterns I would recommend other Forward-Deployed Engineers read, run, and reuse.

## Papers and primary sources

### 1. Lost in the Middle: How Language Models Use Long Contexts

**Link:** [https://arxiv.org/abs/2307.03172]

**Why it belongs in the canon:**  
This paper explains why context position matters. It helped me reason about long judge rubrics, candidate-output placement, and why a late correction turn can behave differently from an early standing instruction.

**FDE takeaway:**  
Prompt layout is an inference-time control surface. Do not assume a model uses every part of a long prompt equally.

---

### 2. The Instruction Hierarchy: Training LLMs to Prioritize Privileged Instructions

**Link:** [https://arxiv.org/abs/2404.13208]

**Why it belongs in the canon:**  
This paper clarifies the gap between intended instruction priority and actual model behaviour. It was useful for explaining why system prompts should be authoritative but still do not behave like hard validators.

**FDE takeaway:**  
Instruction hierarchy is learned behaviour, not a symbolic enforcement layer. Safety-critical rules still need validators, filters, and repair loops.

---

### 3. OpenAI Model Spec

**Link:** [https://openai.com/index/sharing-the-latest-model-spec/]

**Why it belongs in the canon:**  
The Model Spec gives a practical chain-of-command framing for platform, developer, and user instructions.

**FDE takeaway:**  
Use the chain of command to reason about prompt design, but do not confuse instruction priority with execution guarantees.

---

### 4. Direct Preference Optimization

**Link:** [https://arxiv.org/abs/2305.18290]

**Why it belongs in the canon:**  
DPO is central to understanding Week 11-style judge training. It explains how chosen/rejected preference pairs can update model behaviour without training a separate reward model.

**FDE takeaway:**  
Prompting puts preferences in context; DPO moves preference pressure into weights. Always check whether gains reflect preference learning, calibration, or overfitting.

---

### 5. SimPO: Simple Preference Optimization with a Reference-Free Reward

**Link:** [https://arxiv.org/abs/2405.14734]

**Why it belongs in the canon:**  
SimPO gives a clean way to think about reference-free preference learning, length-normalized rewards, beta scaling, and target reward margins.

**FDE takeaway:**  
Beta, margin, and threshold are different levers. Do not tune them interchangeably.

---

### 6. Hugging Face TRL Documentation: CPO / SimPO Trainers

**Link:** [https://huggingface.co/docs/trl/]

**Why it belongs in the canon:**  
The documentation matters because many repos describe a preference method at the concept level but implement a slightly different trainer in code.

**FDE takeaway:**  
Always compare the claimed training objective with the actual trainer and config. “SimPO-style” and “CPOTrainer with beta” are not automatically the same.

---

### 7. LoRA: Low-Rank Adaptation of Large Language Models

**Link:** [https://arxiv.org/abs/2106.09685]

**Why it belongs in the canon:**  
LoRA is the practical bridge between full fine-tuning and lightweight adaptation. It is essential for understanding adapter-based judges.

**FDE takeaway:**  
A small adapter can shift behaviour without rewriting the whole model. But small adapters also need margin and overfitting diagnostics.

---

### 8. Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena

**Link:** [https://arxiv.org/abs/2306.05685]

**Why it belongs in the canon:**  
This is one of the foundational practical references for LLM-as-a-judge evaluation, including known judge biases.

**FDE takeaway:**  
LLM judges are useful evaluators, but they are not neutral instruments. Bias checks belong in the harness.

---

### 9. Position Bias Studies in LLM-as-a-Judge

**Example link:** [https://arxiv.org/abs/2406.07791]

**Why it belongs in the canon:**  
Position bias is one of the easiest LLM-judge bugs to miss and one of the easiest to mitigate.

**FDE takeaway:**  
Pairwise judging should run A/B and B/A. Count only stable wins or report instability.

---

### 10. Who&When / Multi-Agent Failure Attribution Literature

**Why it belongs in the canon:**  
The key idea is that multi-agent failures require identifying both the responsible component and the failed step.

**FDE takeaway:**  
An agent trace should identify where the failure happened, not merely that the final outcome failed.

## Tools worth learning

### 1. Hugging Face TRL

Use it for DPO, CPO, SimPO-style experiments, and reward-margin diagnostics. It is especially useful when the question is whether a model learned a preference or merely shifted a threshold.

### 2. Unsloth

Useful for low-cost LoRA and DPO fine-tuning in constrained environments such as Colab. The FDE skill is knowing when the training stack actually matches the method described in the report.

### 3. Langfuse or OpenTelemetry

Use these for span-level observability. For agent systems, logging the final answer is not enough. You need model stage, tool stage, runtime, retry, and outcome spans.

### 4. JSONL trace logs

A simple but powerful pattern. Every run should produce an auditable artifact that can be inspected, diffed, and linked to claims in a memo.

### 5. Pydantic schemas

Use typed models for traces, tool results, output quality, and business outcomes. Pydantic turns “logs” into contracts.

### 6. Bootstrap / paired evaluation scripts

Use these for agent and judge evaluations. A single aggregate number is rarely enough; confidence intervals and paired comparisons are often the difference between evidence and storytelling.

## Engineering patterns worth reusing

### Pattern 1 — Order-swapped pairwise judging

Run judge comparisons in both A/B and B/A order. Count only stable wins.

**Use when:** Reporting pairwise win rates, Delta A/B, or ablation improvements.

---

### Pattern 2 — Gate 2 post-extraction validation

Do not escalate tools based only on pre-extraction profile. After each extractor runs, measure actual output quality.

**Use when:** Routing among cheap parser, layout parser, and expensive vision/OCR fallback.

---

### Pattern 3 — Failure-attribution trace graph

Every agent turn should log model decision, tool arguments, tool result, fallback, and business outcome.

**Use when:** A business process spans multiple tools such as CRM, calendar, email, payment, or database.

---

### Pattern 4 — Verifier + repair loop

For unsupported claims or policy-sensitive outputs, generate, verify, name the violation, repair, and re-check.

**Use when:** System prompts alone are too soft for the risk.

---

### Pattern 5 — Reward-margin diagnostics

After preference training, inspect train/dev/held-out margins rather than only final accuracy.

**Use when:** Evaluating DPO, SimPO, CPO, ORPO, or small adapter training.

---

### Pattern 6 — Threshold sweep before retraining

If a judge is biased toward PASS or FAIL, inspect the score/margin distribution and sweep thresholds before changing weights.

**Use when:** A model ranks well but binary decisions are miscalibrated.

---

### Pattern 7 — Per-source-mode evaluation

Break accuracy by source: trace-derived, programmatic, synthetic, adversarial, or real user logs.

**Use when:** You need to distinguish generalization from style memorization.

---

### Pattern 8 — Prompt-layout ablation

Test long rubric, short rubric, candidate-first, rubric-after-candidate, and balanced examples.

**Use when:** A prompted judge or agent behaves too strict, too lenient, or overly sensitive to framing.

---

## Final canon principle

The most reusable lesson is this:

> Every FDE system needs a mechanism-level story and an artifact that can prove it.

If the claim is “DPO improved the judge,” show margins.  
If the claim is “Vision was necessary,” show failed cheaper attempts.  
If the claim is “the agent booked the meeting,” show the trace.  
If the claim is “the judge preferred A,” show it survived A/B and B/A.
