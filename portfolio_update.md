# Portfolio Update — How Week 12 Improved My Week 3, 10 and 11 Work

Week 12 turned previous projects from working demos into more defensible FDE artifacts. The five grounding commits collectively improved three things: mechanism attribution, evaluation reliability, and production diagnosability.

## The Conversion Engine

My Week 10 Conversion Engine already implemented the core sales-automation path: classify a prospect reply, coordinate HubSpot, generate a Cal.com booking link, draft a response, and send through Resend. The weakness was observability. A failed scheduling flow could collapse into “agent failed,” without showing whether the fault came from the LLM, orchestration, a bad tool argument, HubSpot, Cal.com, Resend, stale CRM state, or missing fallback logic.

The Week 12 grounding commits added a typed failure-attribution layer. I added schemas for per-turn traces, persistent JSONL trace writing, typed tool-result wrappers, traced HubSpot/Cal.com/Resend wrappers, confidence and alternatives in reply classification, and a failure-attribution resolver. I also updated the probe library with trace-resolvable labels. This means a warm-lead scheduling failure can now be debugged as a causal sequence rather than a generic outcome.

For an FDE manager, this matters because production agents fail at interfaces. The improved Conversion Engine now shows that I understand not only how to wire tools together, but how to make those tool interactions auditable.

## Tenacious-Bench and the DPO Judge

My Week 11 project built a Tenacious-specific benchmark and trained a DPO judge. The headline result was strong: the fine-tuned judge outperformed the prompted judge on strict pairwise accuracy. But before Week 12, my documentation treated that mostly as a metric improvement.

The grounding commits updated the decision memo and model card to explain what DPO changed compared with prompting. The prompted judge had the rubric in context, while DPO moved preference pressure into LoRA adapter weights through chosen/rejected log-prob margins against a frozen reference model. I now frame the result as a likely decision-boundary and calibration shift, while explicitly naming overfitting and benchmark-style memorization as risks to test.

For an FDE manager, this matters because I am not overclaiming “fine-tuning worked.” I can explain the mechanism, name the alternative hypotheses, and list the diagnostics needed to validate the claim.

## Document Intelligence Refinery

The Document Intelligence Refinery grounding work improved my tool-router design. The repo exposed FastText, Layout-Aware, and Vision-Augmented extraction strategies, but the observed run collapsed to Vision. Week 12 clarified that the missing concept was Gate 2: post-extraction quality validation.

I added or planned changes around output-quality scoring, extractor confidence remeasurement, model schema updates, and extraction rules. Instead of treating triage confidence as extraction quality, the system now has a path to measure whether an extractor actually produced usable blocks, bounding boxes, tables, and provenance.

For an FDE, this shows I can reason about cost-quality routing. I do not simply reach for the strongest tool. I design a router that can justify escalation.

## Evaluation Reliability

Across the Week 11 judge work, I also added a clearer position-bias mitigation pattern: pairwise judge results should be order-swapped. A stable content preference should survive both A/B and B/A presentation. This protects Delta A/B reporting from inflated slot effects.

For an FDE, this matters because evaluation artifacts are often more fragile than model code. I now know how to turn known LLM-judge biases into harness-level controls.

## Overall portfolio effect

Together, the five grounding commits make my portfolio more credible in three ways:

1. **Traceability:** The Conversion Engine now has a design for step-level causal failure attribution.
2. **Calibration:** The Tenacious-Bench judge documentation now explains DPO as boundary/margin behavior, not just a scoreboard win.
3. **Tool routing:** The Document Intelligence Refinery now distinguishes pre-extraction triage from post-extraction quality.
4. **Evaluation integrity:** The judge pipeline now has a pattern for order-invariant comparison.
5. **Production judgment:** Each project now names its remaining risks instead of hiding them behind aggregate metrics.

The most important change is that I can now defend the “why” behind the systems I built. I can explain when a model is responsible, when the scaffold is responsible, when the tool is responsible, and when the evaluation harness itself may be misleading. That is the difference between a prototype and FDE-grade engineering.
