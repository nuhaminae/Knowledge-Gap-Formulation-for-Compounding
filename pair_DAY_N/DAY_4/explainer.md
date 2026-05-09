# Explainer — Fixing Position Bias in an LLM Judge with Order-Swapped Evaluation

---

[LinkedIn Link](https://www.linkedin.com/pulse/explainer-fixing-position-bias-llm-judge-evaluation-nuhamin-alemayehu-yjqwe)

---

## The Question

> How can we implement position bias mitigation in our judge system, such as by modifying the judge prompt to evaluate pairs in both orders, A-first and B-first, and only count wins that hold across both evaluations?

This is a strong question because it moves beyond “LLM judges can be biased” and asks for an implementation-level fix.

In pairwise LLM judging, the model sees two candidate answers and chooses the better one. The problem is that the judge may prefer a candidate because of where it appears in the prompt, not because of its content. If the judge tends to favor the first answer, then Candidate A can win when shown first and lose when shown second. That is position bias.

For a benchmark or ablation harness, this matters because it can inflate the measured improvement of a method. If the trained method is always placed in slot A and the baseline is always placed in slot B, then some “wins” may be slot wins, not method wins.

## What the Current System Already Handles

The current judge-filter scaffold already addresses several important risks. It prevents preference leakage by requiring the generator model and judge model to differ. It also scores task quality across explicit dimensions such as input coherence, ground-truth verifiability, and rubric-application clarity. It includes duplicate comparison logic so near-identical candidates do not silently pollute the dataset.

Those are useful safeguards, but they do not address position bias. A content-based rubric can still be applied inconsistently if the model has an order preference. A no-same-model rule prevents self-preference, but it does not prevent “first-answer preference.” Duplicate detection prevents contamination, but it does not prove that the judge’s pairwise decision is stable under swapped order.

So the missing mitigation is not another general instruction like “be unbiased.” The missing mitigation is a protocol.

## The Core Fix: Order-Swapped Judging

The judge should evaluate every pair twice:

```markdown
Pass 1:
A = candidate_1
B = candidate_2

Pass 2:
A = candidate_2
B = candidate_1
````

Then the system compares the two results.

If candidate_1 wins in both presentations, count candidate_1 as the stable winner.

If candidate_2 wins in both presentations, count candidate_2 as the stable winner.

If the winner flips when the order flips, mark the pair as unstable.

If the judge returns a tie or malformed judgment in either order, mark the pair as inconclusive.

The key rule is:

```markdown
Only count wins that survive the order swap.
```

This turns position bias from an invisible risk into a measurable field in the evaluation output.

## Why Prompting Alone Is Not Enough

You can add a prompt instruction such as:

```markdown
Do not prefer Candidate A or Candidate B based on position.
Judge only the content.
```

That instruction is good, but it is not sufficient. The model may still have a learned tendency to prefer the first or second answer, especially when the candidates are close in quality.

The stronger fix is behavioral: force the judge to make the same decision under both orders. If the judge cannot do that, the evaluation should not pretend the result is stable.

In other words:

```markdown
Prompt instruction reduces bias.
Order-swapped evaluation detects bias.
Stable-win filtering mitigates bias in reported metrics.
```

## How to Implement It

The clean implementation is to add a wrapper around pairwise judging:

```python
def judge_pair_order_invariant(task, candidate_a, candidate_b):
    forward = judge_pair(task, candidate_a, candidate_b, order="original")
    reverse = judge_pair(task, candidate_b, candidate_a, order="swapped")

    forward_winner = map_winner_to_candidate_id(forward, original_order=True)
    reverse_winner = map_winner_to_candidate_id(reverse, original_order=False)

    if forward_winner == reverse_winner:
        return {
            "decision": "stable_win",
            "winner": forward_winner,
            "position_bias_flag": False,
            "forward": forward,
            "reverse": reverse,
        }

    return {
        "decision": "unstable",
        "winner": None,
        "position_bias_flag": True,
        "forward": forward,
        "reverse": reverse,
    }
```

The important part is that the reverse result must be mapped back to the original candidate IDs. If the swapped prompt says “A wins,” that means the original Candidate B won.

The output should include trace fields like:

```json
{
  "task_id": "T-017",
  "candidate_a_id": "trained_judge",
  "candidate_b_id": "baseline",
  "forward_order_winner": "candidate_a",
  "reverse_order_winner": "candidate_a",
  "stable_winner": "candidate_a",
  "position_bias_flag": false,
  "counted_in_delta": true
}
```

For an unstable pair:

```json
{
  "task_id": "T-018",
  "forward_order_winner": "candidate_a",
  "reverse_order_winner": "candidate_b",
  "stable_winner": null,
  "position_bias_flag": true,
  "counted_in_delta": false
}
```

## How This Changes Delta A/B

This mitigation should be applied before computing ablation deltas.

Instead of:

```markdown
Delta A = all trained judge wins - baseline wins
```

use:

```markdown
Delta A = stable trained judge wins - stable baseline wins
```

and report:

```markdown
unstable_pair_rate
position_bias_flag_count
stable_win_rate
inconclusive_rate
```

This makes the result more conservative but more trustworthy. If the headline number drops after order swapping, that is not a failure. It means the original result was partly supported by presentation effects.

## What This Teaches

Position bias is not solved by trusting the judge harder. It is solved by making the judge’s decision testable under a symmetry condition.

The symmetry condition is simple:

> If Candidate X is truly better than Candidate Y, Candidate X should win whether it appears first or second.

This is the same logic as a unit test. The model is allowed to judge, but the harness checks whether the judgment is invariant to a change that should not matter.

## Recommended Grounding Edit

The best repo change is to add order-swap support to the judge filter or ablation harness.

Concretely:

1. add a pairwise judge wrapper that evaluates A/B and B/A;
2. map swapped labels back to original candidate IDs;
3. write both raw judge outputs to the log;
4. add `position_bias_flag`;
5. count only stable wins in Delta A/B;
6. report unstable-pair rate in the final ablation report.

This turns position bias from a known limitation into a measured and mitigated property of the evaluation system.
