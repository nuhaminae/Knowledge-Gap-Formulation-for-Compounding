# Question — Inference-Time Mechanics

My Week 11 prompted judge uses a long rubric before every candidate output. How do attention sinks and long-prefix behavior affect judge calibration at inference time, and could the rubric prefix itself bias the model toward over-rejecting good outputs?

## Specific artifact this improves

This question directly improves:

- `src/evaluation/eval_prompted_judge.py`
- `reports/prompted_judge_metrics.json`
- `reports/ablation_results.json`
- `reports/final_report_v2.md`
- `models/model_card.md`

The concrete edit I expect after the explainer is a prompted-judge prompt-ablation section that compares the current long rubric against shorter, balanced, and candidate-first prompt variants.

## What would close the gap

A satisfying answer should explain attention sinks, long-prefix behavior, and judge calibration in plain language; connect them to my prompted judge’s 100% precision / 76.92% recall pattern; and give me a runnable ablation plan that measures false negatives, good-vs-bad log-prob margins, and strict pairwise accuracy under different prompt layouts.
