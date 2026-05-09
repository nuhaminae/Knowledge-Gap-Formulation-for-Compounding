# Morning Call Summary

I have deliberated among four questions before landing on the attention sink angle of the inference time mechanics question.

To connect my Week 11's **Direct Preference Optimisation** (DPO) judge to the topic of **Inference-time mechanics**, I have looked at the scripts and artifacts I have produced and noticed that I have proved accuracy lift, but did not yet deeply justify **latency/cost mechanics** if the judge runs in the Week 10 Conversion Engine loop.

The Week 10 Tenacious challenge makes this especially relevant because it required p50/p95 latency, cost per lead, trace-backed claims, and channel-aware production behaviour for email/SMS/CRM/calendar workflows. My Week 11 judge improved strict pairwise accuracy, but my own final-report feedback said the production recommendation needed stronger quantitative grounding around cost and latency.

## Four questions that were up for deliberation

### Option A

In my Week 11 report, I recommended deploying the DPO judge in shadow mode, but I did not yet have a defensible inference-time model for the cost and latency of that judge. I know the held-out accuracy improved, but I cannot yet explain how much latency comes from prefill versus response scoring, whether the shared Tenacious prompt/rubric can be cached, or whether the policy/reference passes in DPO scoring double the cost. Closing this gap would let me revise the production-readiness and cost sections of my Week 11 final report, model card, and demo script.

In short:

How does Key-Value (KV) caching affect the actual cost and latency of my DPO judge when scoring Tenacious sales-agent outputs?

---

### Option B

When my Week 11 judge scores a pair of outputs for the same Tenacious prompt, can prefix/KV caching avoid recomputing the shared prompt tokens, and what exact latency savings should I expect for chosen-vs-rejected scoring on a 1B LoRA model?

---

### Option C

If the Week 11 DPO judge is added as a shadow-mode quality gate to the Week 10 email/SMS Conversion Engine, how should inference latency be decomposed into enrichment latency, prompt prefill, judge scoring, and decode/logprob time so that p50/p95 latency and cost-per-qualified-lead claims are traceable?

This connects more directly to Week 10’s trace-backed production requirements, where every number in the final memo had to resolve to trace files.

---

### Option D

My Week 11 prompted judge uses a long rubric before every candidate output. How do attention sinks and long-prefix behaviour affect judge calibration at inference time, and could the rubric prefix itself bias the model toward over-rejecting good outputs?

This is interesting because my prompted judge had perfect precision but lower recall, meaning it was too conservative.
