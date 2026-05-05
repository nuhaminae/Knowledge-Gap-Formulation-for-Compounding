# Signoff — Prompted Judge Rubric Bias

## Question

My Tenacious Bench prompted judge used a long rubric before every candidate output. How do attention sinks and long-prefix behavior affect judge calibration at inference time, and could the rubric prefix itself bias the model toward over-rejecting good outputs?

## Gap-Closure Judgment

**Status:** Closed.

## What I Understand Now

Before this question, I understood the prompted judge mostly as a baseline: it was the same base model as the DPO judge, but prompted with a rubric instead of fine-tuned. I could report the result — 100% precision, 76.92% recall, and 76.92% strict pairwise accuracy — but I could not explain why it behaved that way beyond saying it was “too conservative.”

I now understand that the prompted judge’s behavior is not only determined by the rubric content, but also by the **inference-time structure** of the prompt. A long rubric prefix can shape the model’s next-token distribution before it ever reaches the candidate output. If that prefix is failure-heavy, the model may become excellent at detecting bad outputs while becoming too strict about what counts as good.

## The Mechanism That Landed

The key mechanism is that a long judge prompt is not neutral context. It affects calibration through:

1. **Attention sinks:** early prompt tokens can become persistent attention anchors.
2. **Long-prefix position effects:** the model may not use every part of a long context equally.
3. **Rubric framing:** a failure-heavy rubric can shift the task from “is this acceptable?” to “can I find any flaw?”
4. **Log-prob calibration:** the model may rank the chosen output above the rejected output while still not giving the word `good` enough probability to cross the binary decision threshold.

This explains my bench project's prompted judge result more precisely. The prompted judge was not simply weak; it was **differently calibrated**. It ranked pairs well, but as a deployment-style binary gate it over-rejected acceptable good outputs.

## What Changed in My Understanding

I now distinguish between two evaluation questions:

```text
Does the judge know which response is better?
````

and:

```text
Does the judge accept the good response and reject the bad one?
```

My prompted judge did well on the first question but worse on the second. That means the failure was not necessarily comprehension. It may have been prompt-layout and threshold calibration.

## Concrete Portfolio Edit

This closes the gap enough for me to make the following The Bench project's portfolio edits:

1. Add prompt-template ablations to `src/evaluation/eval_prompted_judge.py`.
2. Add a prompt-ablation section to `configs/eval_config.yaml`.
3. Generate `reports/prompt_ablation/` outputs comparing:

   * current long rubric
   * short neutral rubric
   * candidate-first prompt
   * balanced good/bad rubric
   * goodness-first prompt
   * rubric-after-candidate prompt
4. Update `reports/final_report_v2.md` to explain that the prompted judge’s lower recall may come from conservative rubric calibration.
5. Update `models/model_card.md` with a warning that prompt-only judges should be calibrated on false positives, false negatives, and strict pairwise accuracy — not just accuracy or ranking.

## Final Signoff

This gap is closed because I can now explain the prompted judge’s over-rejection behavior as a plausible inference-time prompt-layout effect, not just as a vague limitation of prompting. The next step is empirical: run the proposed prompt-ablation variants and verify whether shortening, balancing, or repositioning the rubric reduces false negatives without increasing false positives.
