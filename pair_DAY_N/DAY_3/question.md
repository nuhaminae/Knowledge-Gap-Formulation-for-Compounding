# Question — Training and Post-Training Mechanics

In my Week 11 `Sales-Evaluation-Bench-trp` repo, the judge is trained on preference pairs where failing tasks use a correct `FAIL` verdict as `chosen` and an incorrect over-generous `PASS` verdict as `rejected`. The training script is named `train_simpo.py`, but it currently uses TRL `CPOTrainer` with `CPO_BETA = 2.0`, while the methodology describes the system as SimPO-style.

My sharpened question is:

> If the held-out judge shows PASS bias on disqualification tasks, should I treat that as a β problem, a missing SimPO target-margin problem, or an inference-threshold problem? More specifically, what does β actually control in the preference loss, how does it interact with length-normalized rewards and the implicit target reward margin, and what concrete configuration change would reduce false PASS decisions on disqualification tasks without collapsing the judge’s overall calibration?

## Why this is my gap

I understand that the trained judge improved overall held-out accuracy, but I do not yet understand whether the remaining PASS bias comes from training mechanics or inference calibration. In the repo, the training pairs for failing tasks are designed to teach the model that `FAIL` should beat `PASS`, but the current config exposes only `CPO_BETA = 2.0`. If I want SimPO-style margin control, I need to know whether to tune β, add `loss_type="simpo"` plus `gamma_beta_ratio`, change the PASS threshold, or collect more disqualification pairs.

## Specific artifact this improves

This question directly improves:

- `training/train_simpo.py`
- `training/hyperparameters.json`
- `training_data/build_simpo_pairs.py`
- `ablations/ablation_results.json`
- `scoring_evaluator.py`
- `memo.md`

## What would close the gap

A satisfying answer should explain what β scales in the CPO/SimPO preference objective, how length-normalized average log probability changes the reward being compared, what the SimPO target margin does, and how to distinguish three possible fixes:

1. tune β,
2. add or sweep SimPO `gamma_beta_ratio`,
3. adjust the inference PASS threshold.

It should end with a concrete diagnostic plan for PASS bias on `expected_pass=false` and disqualification-category tasks.

Project Link: [Sales Evaluation Bench](https://github.com/eyobed7b/Sales-Evaluation-Bench-trp)
