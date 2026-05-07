# Sources — β, SimPO Margin, and PASS Bias

## Source 1 — SimPO: Simple Preference Optimization with a Reference-Free Reward

**Citation:**  
Yu Meng, Mengzhou Xia, Danqi Chen. “SimPO: Simple Preference Optimization with a Reference-Free Reward.” arXiv:2405.14734, 2024.

**Link:** [SimPO: Simple Preference Optimization with a Reference-Free Reward](https://arxiv.org/abs/2405.14734)

**Why I used it:**  
This is the canonical paper for SimPO. It explains the two ideas that matter for the question: SimPO uses average log probability of the response as a reference-free implicit reward, and it adds a target reward margin to encourage stronger separation between winning and losing responses.

**Key idea I used:**  
Length normalization defines what reward is being compared: average token log-probability rather than total sequence log-probability. The target margin defines how much the chosen response should beat the rejected response by. This is directly relevant to PASS bias because a correct `FAIL` answer should not merely beat an incorrect `PASS` answer by a tiny amount.

---

## Source 2 — Hugging Face TRL CPO Trainer Documentation

**Citation:**  
Hugging Face TRL. “CPO Trainer.” Hugging Face Documentation.

**Link:** [CPO Trainer — Hugging Face TRL](https://huggingface.co/docs/trl/v0.18.1/en/cpo_trainer)

**Why I used it:**  
This source explains how TRL implements CPO and notes that SimPO is available through `CPOTrainer` by using `loss_type="simpo"` and `cpo_alpha=0.0`. It also explains that logged reward metrics include chosen rewards, rejected rewards, reward accuracies, and margins.

**Key idea I used:**  
The peer’s repo uses `CPOTrainer` with `CPO_BETA = 2.0`, so the explainer needed to distinguish current CPO behavior from true SimPO-style margin control. This source shows the concrete configuration path for SimPO-like training in TRL: use the CPO trainer with SimPO loss settings rather than assuming β alone provides target-margin behavior.

---

## Source 3 — Repo Artifact Pattern: PASS Bias Diagnostics

**Files / artifacts:**

- `training/train_simpo.py`
- `training/hyperparameters.json`
- `training_data/build_simpo_pairs.py`
- `ablations/ablation_results.json`
- `scoring_evaluator.py`

**Why I used it:**  
These repo files define the actual training setup and the failure mode. The training script uses `CPOTrainer` with `CPO_BETA = 2.0`; the hyperparameter file records this as a reference-free CPO substitute for SimPO; and the pair builder constructs failing tasks as correct `FAIL` verdicts paired against incorrect `PASS` verdicts.

**Key idea I used:**  
PASS bias on disqualification tasks should be measured directly. The useful tool/pattern is not just a β change; it is a diagnostic sweep:

```markdown
false PASS rate by category
PASS/FAIL margin distribution
overall accuracy
false FAIL rate on valid PASS tasks
threshold sweep
beta sweep
gamma_beta_ratio sweep
````

This pattern lets the asker determine whether the fix belongs in training loss configuration, SimPO margin control, or inference threshold calibration.
