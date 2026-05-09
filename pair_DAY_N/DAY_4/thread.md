# Thread — Fixing Position Bias in an LLM Judge

---

[LinkedIn Thread Post](https://www.linkedin.com/posts/nuhamin-a-e-95n87a190_fixing-position-bias-in-an-llm-judge-https-ugcPost-7458778329500942336-CDoI?utm_source=share&utm_medium=member_desktop&rcm=ACoAACahjWcB-GSd6J8wRonZO0Rh5aMwXM4ESuc)

---

## Post 1

Pairwise LLM judges can look objective while still having a simple failure mode:

They may prefer the answer shown first.

That means a method can “win” because it was placed in slot A, not because it was better.

This is position bias.

## Post 2

The fix is not just telling the judge:

“Do not be biased.”

That helps, but it is still only a prompt instruction.

The stronger fix is procedural: evaluate every pair twice.

```markdown
Run 1: A = candidate 1, B = candidate 2
Run 2: A = candidate 2, B = candidate 1
```

## Post 3

Then map the swapped labels back to the original candidates.

If candidate 1 wins in both orders, count it as a stable win.

If candidate 2 wins in both orders, count it as a stable win.

If the winner flips when the order flips, mark the pair as unstable and do not count it as evidence.

## Post 4

This matters for ablations.

If the trained method is always shown first and the baseline is always shown second, Delta A can be inflated by slot preference.

So compute:

```markdown
stable Delta A = wins that survive A/B and B/A order swap
```

Also report:

```markdown
unstable pair rate
position bias flag count
inconclusive rate
```

## Post 5

The core principle:

A real content win should survive irrelevant presentation changes.

If a judge changes its winner only because the candidates swapped positions, the evaluation should not count that as a reliable win.

Order swapping turns position bias from an invisible risk into an auditable field.

## Post 6

The practical repo change:

Add an order-invariant judge wrapper that logs both judgments, maps labels back to original candidate IDs, sets `position_bias_flag`, and only counts stable wins in the ablation report.

Prompting asks the judge to be fair.

Order swapping checks whether it actually was.
