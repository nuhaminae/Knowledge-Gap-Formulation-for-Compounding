# Thread — β, Margin, and PASS Bias in a Preference-Trained Judge

## Post 1

A preference-trained judge can be accurate overall and still have a dangerous failure mode:

PASS bias.

That means the judge is too willing to mark flawed examples as acceptable — especially disqualification cases that should be rejected.

The question: is this a β problem, a margin problem, or a threshold problem?

---

## Post 2

β is not simply a “make the judge stricter” knob.

In CPO/SimPO-style preference training, β scales how sharply the loss reacts to the difference between chosen and rejected rewards.

Low β = softer preference pressure.  
High β = sharper pressure, but more risk of brittleness or calibration drift.

---

## Post 3

SimPO adds another idea: length-normalized reward.

Instead of rewarding total sequence log-probability, it uses average log-probability per token.

That matters because a short `PASS` should not automatically beat a longer but correct `FAIL` explanation just because it has fewer tokens.

---

## Post 4

The key missing lever for PASS bias may be the target reward margin.

A margin says:

```text
chosen should not merely beat rejected;
it should beat it by enough.
```

For disqualification tasks, that means correct `FAIL` verdicts should clearly beat incorrect `PASS` verdicts — not just barely outrank them.

## Post 5

So the safe tuning order is:

1. measure false PASS rate,
2. inspect margins on disqualification examples,
3. sweep the PASS threshold,
4. add or tune `gamma_beta_ratio`,
5. only then sweep β.

Do not crank β first unless you know the whole judge is under-separating.

---

## Post 6

The production lesson:

If the judge has PASS bias, the fix may not be “train harder.”

It may be:

```markdown
better margin,
better threshold,
or more disqualification pairs.
```

β scales the pressure.
Margin defines the required separation.
Threshold decides when PASS is allowed.
