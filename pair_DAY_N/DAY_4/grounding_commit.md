# Grounding Commit

**Artifact updated:** [methodology.md](/home/bethel/Documents/10academy/Sales-Evaluation-Bench-and-Aligning-the-Conversion-Engine/methodology.md)

I added a new `Position Bias Mitigation Plan` section to the methodology so the repo now names an implementation-level fix for LLM-as-a-judge order effects. The edit records the exact protocol the explainer argued for: judge each pair in both orders, map the reverse result back to the original candidate IDs, count only stable wins, and log unstable or inconclusive comparisons explicitly.

This grounding change improves the Week 11 portfolio because it turns a known evaluation limitation into a specific design commitment tied to Tenacious-Bench artifacts. Instead of saying the judge should "be unbiased," the methodology now states how the harness should detect and mitigate first-answer preference before computing Delta A/B. That makes the benchmark write-up more technically defensible and aligns the portfolio with the Week 12 standard that every explainer should pay back into an existing shipped artifact.
