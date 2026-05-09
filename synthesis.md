# Knowledge Gap Formulation for Compounding — Week Synthesis

This submission reflects the revised scope of the challenge and one caveat from my actual working pattern: I worked across **seven unique questions total**, not eight. I named four gaps from my own projects, but on the first day I worked directly on my own question rather than researching a separate peer question. That means the final week synthesis is best understood as: **four gaps I named, three peer gaps I researched, and one of my named gaps was self-researched on Day 1**.

The important outcome is still the same: I used the week to convert vague uncertainty into sharper questions, public explainers, peer signoffs, and concrete grounding commits. The projects most affected were **The Conversion Engine**, **Tenacious-Bench and the DPO Judge**, and **Document Intelligence Refinery**.

## The four gaps I named

### 1. Tenacious-Bench and the Prompted Judge — long rubric prefixes and calibration

The first gap I worked on was my own question. In **Tenacious-Bench and the DPO Judge**, the prompted judge used a long rubric before every candidate output. I wanted to understand whether that long rubric prefix could affect judge calibration at inference time. The prompted judge seemed strong at rejecting bad outputs, but it also risked over-rejecting acceptable ones.

The gap was not simply “do long prompts matter?” It was more specific: how do long-prefix behavior, attention sinks, and recency effects shape a judge’s binary accept/reject decision? I learned to separate pairwise ranking ability from deployment-style gate calibration. A model can rank two outputs reasonably while still being too conservative when forced to issue a binary decision. I also learned that a rubric is not neutral context. It shapes the local next-token distribution, especially when it emphasizes violations, risks, and failure conditions more than positive acceptance criteria.

The practical result was that I now treat prompted-judge strictness as something to measure, not assume. A good prompted-judge evaluation should include prompt-layout ablations, threshold sweeps, and examples that test whether good outputs are being rejected for being merely imperfect.

### 2. Document Intelligence Refinery — correct escalation versus tool-selection collapse

In **Document Intelligence Refinery**, I had multiple extraction strategies: FastText/pdfplumber, Layout-Aware/Docling, and Vision-Augmented extraction. But the recorded approach table showed the pipeline ending up at VisionAugmented. The gap was that I could not tell whether this meant correct escalation or bad tool-selection collapse.

My peer’s explanation gave me the load-bearing mechanism: **Gate 2**. Gate 1 chooses the initial extraction strategy based on the document profile. Gate 2 measures whether the chosen extractor actually produced usable output. Gate 3 accepts the output or escalates. Before this, I was over-relying on pre-extraction triage confidence. I now understand that triage confidence and extraction quality are different signals.

I grounded this in the repo by adding or planning output-quality schemas, extraction-quality utilities, updated extractors, and configurable extraction rules. The router should now be able to prove whether escalation was justified by measuring content blocks, bounding-box coverage, table extraction, provenance coverage, and validation failures.

### 3. Tenacious-Bench and the DPO Judge — what DPO changed that prompting did not

My next gap came from the DPO-trained judge. The DPO judge improved strict pairwise accuracy over the prompted judge, but I could only report the result, not explain the mechanism. I wanted to know whether DPO taught new Tenacious-specific judgment, shifted the decision boundary, improved calibration, or simply overfit to the small preference-pair distribution.

The peer explainer clarified the difference between rules in context and preferences in weights. Prompting leaves the base model’s internal boundary unchanged. DPO uses chosen/rejected preference pairs to shift model behavior through log-prob margins against a frozen reference model. The safest interpretation of my result is not that 65 pairs taught a fully general judge from scratch. It is that the base model already had relevant capability, and DPO calibrated where the acceptable-vs-bad boundary should sit for Tenacious-Bench.

I grounded this by updating the model card and decision memo. They now explain the result as likely decision-boundary movement and calibration improvement, while still naming overfitting and benchmark-style memorization as risks that require margin and per-source diagnostics.

### 4. The Conversion Engine — failure attribution in tool-use pipelines

My final named gap came from **The Conversion Engine**. The system could classify a reply, look up HubSpot, generate a Cal.com booking link, draft a response, and send through Resend. But if a warm-lead scheduling flow failed, I could not reliably say whether the failure came from the model, the orchestration layer, a tool argument, an external API, stale CRM state, missing fallback logic, or final delivery.

The explainer I received reframed this as a causal attribution problem. A tool-using agent should not only execute actions; it should produce a trace that explains each action. That trace needs model-visible state, selected action, tool arguments, tool results, errors, fallbacks, final response, and business outcome.

I grounded this through a commit series that added typed failure-attribution schemas, JSONL trace persistence, typed tool-result wrappers, traced HubSpot/Cal.com/Resend wrappers, confidence and alternatives in reply classification, a failure-attribution resolver, causal tracing in the Resend webhook, and trace-resolvable probe labels.

## The three peer gaps I researched

### 1. The Conversion Engine — system prompt constraints versus violation-specific repair

One peer’s question came from a **The Conversion Engine** probe where the model ignored an honesty rule in the system prompt and generated an unsupported claim such as “you are rapidly scaling your team.” The same model then corrected the issue when a follow-up user turn named the exact violation and requested a rewrite.

I explained this as a soft-instruction failure. A system prompt is a policy prior, not a hard validator. The follow-up repair turn worked because it was recent, concrete, diagnostic, and task-transforming. It did not merely repeat the rule; it identified the violation and converted the task from open-ended drafting into local repair. The production lesson is to use verifier-plus-repair loops for unsupported claims rather than relying on longer prompts.

### 2. Sales-Evaluation-Bench and Aligning the Conversion Engine — beta, CPO/SimPO, and PASS bias

Another peer’s question involved PASS bias on disqualification tasks. Her repo described the training as SimPO-style, but the actual implementation used TRL CPOTrainer with beta configured. The gap was whether to reduce false PASS decisions by changing beta, adding a SimPO-style target margin, or tuning the inference threshold.

I explained that beta is a global preference-pressure scale, not a PASS/FAIL threshold. A target margin controls how far the preferred answer should beat the rejected answer. The inference threshold controls when the final system is allowed to output PASS. The safe diagnostic order is: measure false PASS rate, inspect margins, sweep the threshold, add or tune a margin if true SimPO is enabled, and only then globally sweep beta.

### 3. LLM-as-a-Judge evaluation — position-bias mitigation

A peer gap involved position bias in LLM-as-a-judge evaluation. The question was how to prevent a pairwise judge from favoring Candidate A or Candidate B because of presentation order rather than content.

The mechanism I researched was order-swapped evaluation. Evaluate the pair in A/B order, then evaluate it in B/A order, map the swapped decision back to the original candidate IDs, and count only stable wins. If the winner flips, the pair is unstable and should be excluded or reported separately. This turns position bias from an invisible artifact into a measurable field.

## The most surprising thing I learned

The most surprising thing I learned is that many problems I initially described as “model problems” were actually interface, tracing, or evaluation-design problems.

A failed scheduling flow may be a Resend delivery failure, not a model failure. A Vision-heavy extraction pipeline may be missing post-extraction validation, not truly require Vision. A DPO win may be boundary calibration, not new reasoning. A prompted judge’s strictness may be prompt-layout bias. A pairwise judge result may be a slot artifact.

This is the strongest FDE lesson from **Knowledge Gap Formulation for Compounding**: production AI systems fail through interfaces. The model is only one component. The scaffold, tool schemas, trace logs, thresholds, fallbacks, and evaluation harness are equally load-bearing.

## Canonical reading list and tool list I contributed

The readings I contributed include *Lost in the Middle* for long-context effects, *The Instruction Hierarchy* and the OpenAI Model Spec for instruction priority, DPO and SimPO for preference-training mechanics, TRL documentation for trainer/config fidelity, LoRA for adapter mechanics, and LLM-as-a-judge papers for bias mitigation.

The tools and patterns I contributed include typed execution traces, JSONL trace logs, failure-attribution resolvers, order-swapped pairwise judging, Gate 2 post-extraction validation, reward-margin diagnostics, threshold sweeps, prompt-layout ablations, and per-source-mode evaluation.

## Final reflection

The week’s title, **Knowledge Gap Formulation for Compounding**, landed for me because the compounding came from the loop: name a gap, sharpen it with a peer, research or receive an explainer, sign off honestly, then edit a real artifact. I ended with fewer claims I could not defend and more mechanisms I could explain under review. That is the real portfolio improvement.
