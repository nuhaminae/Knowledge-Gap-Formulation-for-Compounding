# Portfolio Update — From Working Demos to Defensible FDE Systems

This update summarises how my grounding commits from **Knowledge Gap Formulation for Compounding** improved my earlier portfolio projects: **The Conversion Engine**, **Tenacious-Bench and the DPO Judge**, and **Document Intelligence Refinery**. The caveat is that I worked across **seven unique questions total**: four gaps I named and three peer gaps I researched, with the first day spent researching my own question.

## The Conversion Engine

**The Conversion Engine** already had a functional sales-automation path: classify a prospect reply, coordinate HubSpot, generate a Cal.com booking link, draft a response, and send through Resend. The weakness was not capability; it was diagnosability. If a warm-lead scheduling flow failed, the system could not clearly say whether the model, scaffold, tool arguments, external API, CRM state, fallback policy, or final delivery caused the failure.

The grounding commits added a step-level failure-attribution layer. I added typed trace schemas, JSONL trace persistence, typed tool-result wrappers, traced service wrappers for HubSpot/Cal.com/Resend, confidence and alternatives in reply classification, and a failure-attribution resolver. I also updated the probe library with trace-resolvable failure labels.

For a hiring manager, the signal is that I can build and instrument an agentic workflow, not just demo one. The system now moves closer to production-grade observability: every failed business outcome can be tied to a responsible stage.

## Tenacious-Bench and the DPO Judge

**Tenacious-Bench and the DPO Judge** produced a strong headline result: the DPO-trained judge outperformed the prompted judge. Before the grounding work, that result was reported mostly as a metric improvement. The Week 12 work made the interpretation more rigorous.

I updated the model card and decision memo to explain what DPO changed compared with prompting. Prompting placed the rubric in context; DPO moved the preference signal into LoRA adapter weights through chosen/rejected log-prob margins against a frozen reference model. The result is now framed as likely decision-boundary movement and calibration improvement, not as unsupported proof that the model learned fully general Tenacious-specific reasoning.

I also clarified the prompted-judge side of the project: long rubric prefixes can affect binary calibration. A strict rubric may help catch bad outputs while also over-rejecting acceptable ones. That means prompted baselines require prompt-layout and threshold diagnostics, not just a single reported score.

For a hiring manager, the signal is that I can distinguish a leaderboard improvement from a mechanism-level claim. I also know what diagnostics remain: reward-margin distributions, per-source-mode accuracy, failed-pair inspection, threshold sweeps, and same-base prompted comparison.

## Document Intelligence Refinery

**Document Intelligence Refinery** had a multi-strategy extraction architecture: FastText/pdfplumber, Layout-Aware/Docling, and Vision-Augmented extraction. The risk was tool-selection collapse: every document could end up at Vision without proof that cheaper tools failed.

The grounding work clarified the three-gate router model. Gate 1 chooses an initial tool from the document profile. Gate 2 measures the actual extraction output. Gate 3 accepts or escalates based on evidence. This led to changes around output-quality schemas, post-extraction quality scoring, extractor confidence remeasurement, and configurable quality thresholds.

For a hiring manager, the signal is cost-quality judgment. I am not simply calling the strongest or most expensive tool. I am designing a router that can justify escalation.

## Evaluation integrity

The week also improved how I think about LLM-as-a-judge systems. A pairwise judge can be biased by position, length, or self-preference. The key mitigation I added to my canon is order-swapped judging: evaluate A/B and B/A, map labels back to original candidate IDs, and only count stable wins.

For a hiring manager, the signal is that I can protect evaluation metrics from artifacts. I now treat the harness as part of the system, not an afterthought.

## Portfolio-level improvement

Collectively, these grounding commits improve my portfolio in five ways:

1. **Traceability:** The Conversion Engine now has a causal trace design for model/tool/scaffold failures.
2. **Mechanism clarity:** Tenacious-Bench and the DPO Judge now explains why DPO may have improved behavior.
3. **Prompt calibration:** The prompted judge is now treated as a system that needs layout and threshold diagnostics.
4. **Cost-aware routing:** Document Intelligence Refinery now distinguishes pre-extraction triage from post-extraction quality.
5. **Evaluation rigor:** Judge evaluations now include concrete bias-mitigation patterns.

The strongest signal across the portfolio is that I can revisit shipped work, find the hidden mechanism gap, research it, teach it, and then edit the artifact. That is the FDE habit I developed through **Knowledge Gap Formulation for Compounding**.
