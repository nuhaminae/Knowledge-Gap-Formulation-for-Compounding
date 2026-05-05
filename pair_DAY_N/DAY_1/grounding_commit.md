# Grounding Commit — Prompted Judge Rubric Bias

## Original Gap

Last week, I reported that the prompt-engineered judge helped but was too conservative. It rejected all bad outputs, but it also rejected 6 of 26 good outputs.

Before this, my explanation was mostly descriptive:

```text
Prompting helped, but DPO was better calibrated.
```

That was true, but incomplete. I could not yet explain whether the prompted judge’s conservatism came from:

1. The base model itself.
2. The `threshold = 0.0` binary decision rule.
3. The long failure-heavy rubric prefix.
4. Candidate-output position in the prompt.
5. Few-shot example ordering.
6. Some combination of the above.

The week's question helped me name a specific inference-time mechanism: the prompt layout itself may have biased the judge toward over-rejection.

## Portfolio Artifacts This Improves

This grounding work improves the following Week 11 artifacts:

1. `src/evaluation/eval_prompted_judge.py`
2. `configs/eval_config.yaml`
3. `reports/final_report_v2.md`
4. `models/model_card.md`
5. `reports/blog_post.md`

## Concrete Edit 1 — Add Prompt Variants to `eval_prompted_judge.py`

I will add a prompt-ablation mode to the prompted judge evaluator.

### Current behavior

The evaluator uses one prompt template:

```markdown
long rubric
few-shot examples
metadata
task input
candidate output
Verdict:
```

### New behavior

The evaluator should support multiple templates:

```python
PROMPT_VARIANTS = {
    "current_long_rubric": build_judge_prompt_current,
    "short_neutral": build_judge_prompt_short_neutral,
    "candidate_first": build_judge_prompt_candidate_first,
    "balanced_good_bad": build_judge_prompt_balanced,
    "goodness_first": build_judge_prompt_goodness_first,
    "rubric_after_candidate": build_judge_prompt_rubric_after_candidate,
}
```

Each variant should run over the same held-out pointwise examples.

---

## Concrete Edit 2 — Add Config Support

I will add a `prompt_ablation` section to `configs/eval_config.yaml`.

```yaml
prompt_ablation:
  enabled: true
  variants:
    - current_long_rubric
    - short_neutral
    - candidate_first
    - balanced_good_bad
    - goodness_first
    - rubric_after_candidate
  metrics:
    - accuracy
    - precision
    - recall
    - f1
    - false_negatives
    - strict_pairwise_accuracy
    - mean_score_margin
```

This makes prompt calibration reproducible rather than an informal observation.

---

## Concrete Edit 3 — Add Prompt-Ablation Outputs

The updated evaluator should write:

```text
reports/prompt_ablation/current_long_rubric_metrics.json
reports/prompt_ablation/short_neutral_metrics.json
reports/prompt_ablation/candidate_first_metrics.json
reports/prompt_ablation/balanced_good_bad_metrics.json
reports/prompt_ablation/goodness_first_metrics.json
reports/prompt_ablation/rubric_after_candidate_metrics.json
reports/prompt_ablation/prompt_ablation_summary.csv
reports/prompt_ablation/prompt_ablation_summary.json
```

Each metrics file should include:

```json
{
  "variant": "candidate_first",
  "accuracy": 0.0,
  "precision": 0.0,
  "recall": 0.0,
  "f1": 0.0,
  "false_positives": 0,
  "false_negatives": 0,
  "strict_pairwise_accuracy": 0.0,
  "mean_score_margin": 0.0,
  "mean_true_good_margin": 0.0,
  "mean_true_bad_margin": 0.0
}
```

The most important field is:

```text
false_negatives
```

because the observed failure was over-rejection of good outputs.

---

## Concrete Edit 4 — Add a Prompt-Calibration Section to the Final Report

I will update `reports/final_report_v2.md` with a section like this:

```markdown
## Prompted-Judge Calibration Risk

The prompt-engineered judge achieved 100% precision but only 76.92% recall. This indicates a conservative binary gate: it rejected all bad outputs but falsely rejected 6 of 26 good outputs. One plausible cause is the inference-time structure of the prompt. The judge used a long failure-heavy rubric before every candidate output, which may have made the model overweight risk criteria and under-accept acceptable-but-imperfect responses.

To test this, I added a prompt-layout ablation over the same held-out examples. The variants compare the current long rubric against shorter, balanced, candidate-first, and rubric-after-candidate prompts. The key metric is false-negative rate on true-good outputs.
```

---

## Concrete Edit 5 — Add a Warning to `models/model_card.md`

I will update the model card with:

```markdown
### Prompt-Only Judge Calibration Warning

The prompt-engineered baseline was highly conservative: it reached 100% precision but only 76.92% recall on held-out pointwise evaluation. This suggests that long, failure-heavy judge rubrics can over-reject acceptable outputs. Users should not assume that adding a longer rubric always improves judge reliability. Prompt-only judges should be calibrated on held-out data, with explicit tracking of false positives, false negatives, and strict pairwise accuracy.
```

---

## Concrete Edit 6 — Add a Better Explanation to the Blog Post

I will revise the blog post’s “Prompting vs DPO” section.

### Old explanation

```text
Prompting helped, but DPO was better.
```

### New explanation

```text
The prompted judge was not simply weaker. It was differently calibrated. It ranked all pairs correctly, but as a binary gate it was too conservative. The likely failure mode is that a long, failure-heavy rubric made the model search aggressively for disqualifying flaws. DPO tuning improved the acceptance boundary: it preserved strong rejection of bad outputs while accepting more valid good outputs.
```

---

## Experiment Plan

I will run the same held-out set through each prompt variant.

### Held-out data

```text
tenacious_bench/held_out/held_out.jsonl
```

### Metrics

```text
accuracy
precision
recall
F1
false positives
false negatives
strict pairwise accuracy
mean score margin
true-good margin distribution
true-bad margin distribution
```

### Hypothesis

If the long rubric prefix is causing over-rejection, then at least one of these variants should reduce false negatives:

1. `short_neutral`
2. `candidate_first`
3. `balanced_good_bad`
4. `goodness_first`

without causing a large increase in false positives.

### Falsification condition

The hypothesis is weakened if all prompt variants keep the same false-negative pattern, or if false negatives only improve by tuning the threshold. In that case, the root issue is likely not attention sinks or long-prefix layout, but binary threshold calibration or base-model uncertainty.

---

## What I Understand Now

Before this question, I treated the prompt-engineered judge as a simple baseline. Now I understand it as an inference-time system whose behavior is shaped by prompt layout, token position, and log-prob margins.

The key insight is:

```text
Prompting does not just tell the judge what to evaluate.
Prompting also changes the judge's calibration.
```

A long rubric can make the model safer in one direction but worse in another. In my case, the long rubric likely helped the prompted judge catch bad outputs, but may also have contributed to rejecting acceptable good outputs.

---

## Grounding Commit Summary

The grounding commit will:

1. Add prompt-template variants to `eval_prompted_judge.py`.
2. Add prompt-ablation config fields to `eval_config.yaml`.
3. Generate prompt-ablation reports under `reports/prompt_ablation/`.
4. Update `final_report_v2.md` with a prompted-judge calibration section.
5. Update `models/model_card.md` with a prompt-only judge warning.
6. Update `reports/blog_post.md` to explain the difference between ranking and binary gate calibration.

---

## Expected Diff

```text
modified: src/evaluation/eval_prompted_judge.py
modified: configs/eval_config.yaml
modified: reports/final_report_v2.md
modified: models/model_card.md
modified: reports/blog_post.md
new file: reports/prompt_ablation/prompt_ablation_summary.json
new file: reports/prompt_ablation/prompt_ablation_summary.csv
```

---

## Done Criteria

This grounding is complete when:

1. The prompt-ablation code runs on the 52 held-out pointwise examples.
2. Each prompt variant produces a metrics file.
3. The summary table shows false positives, false negatives, recall, precision, and strict pairwise accuracy.
4. The final report explains whether rubric layout contributed to over-rejection.
5. The model card warns that prompt-only judges require calibration, especially with long safety-heavy rubrics.
