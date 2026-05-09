# Sources — System-Prompt Constraints, Recency, and Repair Loops

## Source 1 — Lost in the Middle: How Language Models Use Long Contexts

**Citation:**  
Nelson F. Liu, Kevin Lin, John Hewitt, Ashwin Paranjape, Michele Bevilacqua, Fabio Petroni, Percy Liang. “Lost in the Middle: How Language Models Use Long Contexts.” Transactions of the Association for Computational Linguistics, 2024.

**Link:** [Lost in the Middle: How Language Models Use Long Contexts](https://arxiv.org/abs/2307.03172)

**Why I used it:**  
This paper helps explain why the follow-up rewrite prompt may have worked better than the original system-prompt constraint. The paper studies how language models use long contexts and finds that performance can change depending on where relevant information appears in the input. It reports that models often perform best when relevant information appears near the beginning or end of the context, and worse when the relevant information is buried in the middle.

**Key idea I used:**  
The SOC-01 repair turn placed the exact violation and correction request immediately before the next assistant response. That made the honesty constraint more locally salient than an abstract instruction sitting earlier in the conversation. This supports the explanation that the fix worked partly because the correction was recent, concrete, and placed near the point of generation.

---

## Source 2 — The Instruction Hierarchy: Training LLMs to Prioritize Privileged Instructions

**Citation:**  
Eric Wallace, Kai Xiao, Reimar Leike, Lilian Weng, Johannes Heidecke, Alex Beutel. “The Instruction Hierarchy: Training LLMs to Prioritize Privileged Instructions.” arXiv:2404.13208, 2024.

**Link:** [The Instruction Hierarchy: Training LLMs to Prioritize Privileged Instructions](https://arxiv.org/abs/2404.13208)

**Why I used it:**  
This paper helps distinguish between formal instruction priority and actual model behaviour. It argues that one source of prompt-injection vulnerability is that models may treat system prompts, developer text, user text, and third-party text as similar-priority natural language unless they are trained to respect an explicit instruction hierarchy.

**Key idea I used:**  
The system prompt should have higher formal authority, but that does not mean it acts like a hard runtime rule. SOC-01 shows that an early system-level honesty rule can still lose to competing generation pressures. The follow-up user turn did not “outrank” the system prompt; it made the system rule concrete and easier for the model to apply during the rewrite.

---

## Source 3 — Sharing the Latest Model Spec

**Citation:**  
OpenAI. “Sharing the latest Model Spec.” OpenAI, February 12, 2025.

**Link:** [Sharing the latest Model Spec](https://openai.com/index/sharing-the-latest-model-spec/)

**Why I used it:**  
The Model Spec explains the intended chain of command for model behaviour: platform, developer, and user instructions are prioritized in a defined order. OpenAI describes this chain of command as a way to balance user and developer control within platform-level boundaries.

**Key idea I used:**  
This source clarified that the SOC-01 fix should not be explained as “user prompts are stronger than system prompts.” The correct explanation is more subtle: system prompts have higher intended authority, but they are still implemented through model behaviour rather than an external symbolic rule engine. A later user correction can be more effective because it is specific, local, and turns the task into violation-specific repair.

---

## Source 4 — Peer Project Artifact: SOC-01 Probe and Phase 5 Repair Loop

**Files / artifacts:**

- `probes/probe_library.md`
- `agent/agent_core.py`
- SOC-01: Signal Over-Claiming probe 01
- Phase 5: post-generation honesty repair / rewrite loop

**Why I used it:**  
These artifacts provide the concrete failure that motivated the question. SOC-01 tests whether the Conversion Engine overclaims weak hiring signals. The specific failure was that the model generated unsupported language such as “you are rapidly scaling your team” even though the system prompt instructed it not to overclaim. The Phase 5 fix added a post-generation scan and rewrite loop that named the violation and requested a corrected rewrite.

**Key idea I used:**  
The important difference was not simply “first prompt failed, second prompt worked.” The first attempt was open-ended sales generation under an abstract honesty constraint. The second attempt was violation-specific repair: it named the exact unsupported phrase, explained why it was wrong, and requested safer ask-language. This changed the model’s local task and made the honesty rule easier to apply.

---

## Source 5 — My Explainer Artifact

**File / artifact:**

- `explainer.md`

**Why I used it:**  
The explainer turns the peer’s observation into a mechanistic account. It reframes SOC-01 from “prompt-only constraint proved insufficient” to a more precise failure mode: abstract system-prompt constraints are soft probabilistic priors and can be outweighed by sales-copy generation pressures unless backed by a verifier or repair loop.

**Key result used:**

```markdown
SOC-01:
- Weak signal: exactly 2 open Python roles
- velocity_label = "low"
- Forbidden behaviour: assertive growth or hiring claims
- Failure: model generated “you are rapidly scaling your team”
- Fix: post-generation repair turn named the violation and requested a rewrite
```
