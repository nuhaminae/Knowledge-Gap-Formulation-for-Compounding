# Sign-Off — Week 12 Day 3

**Project:** Tenacious-Bench — DPO Judge Post-Training Analysis
**Asker:** Eyobed Feleke
**Gap closure status:** CLOSED
**Date:** 2026-05-07

---

Before reading the explainer I could report the result — strict pairwise accuracy
improved from 76.92% to 96.15% after DPO training on 65 preference pairs — but I could
not explain what DPO mechanistically changed. I knew the prompted judge had the full
rubric in context and still over-rejected good outputs. I knew DPO used `chosen` and
`rejected` pairs. What I did not understand was why writing preferences into the adapter
weights produced a different outcome than giving the model the same preferences as a
prompt, or how 65 pairs could generalize reliably to the held-out benchmark.

After reading the explainer, I can now explain the gap. Prompting puts the rules in
context at inference time but leaves the base model's decision boundary unchanged. The
base model carries an inherited conservatism — a prior toward caution — and the rubric
prompt compounds it rather than corrects it. DPO writes the preference signal directly
into the weights via the log-prob margin between `chosen` and `rejected` relative to the
frozen reference model. The reference model acts as a regularizer: the update is not
"be more confident in general" but "shift your preference specifically toward the
acceptable side of the boundary, relative to where you started." That is what removed
the over-conservatism the prompt could not fix.

I also understand now why 65 pairs was sufficient. DPO does not teach the model language
or judgment from scratch — the base model was already capable. The 65 pairs recalibrated
an existing boundary to my benchmark's specific threshold. The efficiency comes from the
base model's prior capability, not from the dataset size.

I have four concrete diagnostics I can now run before claiming the result generalizes:
train vs dev reward margins to check for overfitting, held-out margin distribution to
confirm calibration improved, per-source-mode accuracy to detect style memorization, and
a comparison against a prompted judge on the same base model to isolate DPO's weight-level
contribution. I can now write a mechanistic account of the 76.92% → 96.15% result in
`reports/final_report_v2.md` and document what the DPO adapter actually encodes in
`models/model_card.md`, rather than reporting the number without an explanation.
