# Thread — Can a Long Judge Rubric Make an LLM Over-Reject Good Outputs?

## Post 1

Last week, my prompt-engineered judge had a strange pattern:

- 100% precision
- 76.92% recall
- 100% rank-based pairwise accuracy
- 76.92% strict pairwise accuracy

It caught every bad output, but rejected 6 of 26 good outputs.

So I asked: was the judge actually weak, or did my long rubric make it too conservative?

---

## Post 2

The judge prompt had a long prefix before every candidate output:

```text
You are a quality-assurance judge.

A GOOD output should...
A BAD output should...
Examples...
Now evaluate this candidate.
Candidate output:
...
Verdict:
````

That prefix is not neutral.

At inference time, the model processes the whole prompt before deciding whether the next token should be `good` or `bad`.

## Post 3

The first mechanism is **attention sinks**.

Early tokens in a prompt can become stable attention anchors. If the first part of the prompt frames the task as “catch every possible failure,” the model may behave like a strict compliance auditor.

That is useful for catching bad outputs.

But it can also shift the decision boundary from:

```text
safe and acceptable → good
```

to:

```text
not perfect → bad
```

---

## Post 4

The second mechanism is **long-prefix behavior**.

Models do not always use long contexts evenly. In my judge prompt, the rubric came first, examples came next, and the actual candidate appeared late.

That matters because good Tenacious responses are often subtle:

```text
“I can’t tell from the public signal alone...”
```

or:

```text
“I don’t want to overstate capacity...”
```

Bad responses often have louder cues: fake urgency, unsupported claims, broken links, or prompt-injection compliance.

---

## Post 5

This explains the Bench project's result better:

The prompted judge was not simply bad.

It was **well-ranked but poorly calibrated as a binary gate**.

It knew which response was better, but the long failure-heavy rubric may have made it reluctant to call acceptable responses `good`.

That is why strict pairwise accuracy mattered more than rank accuracy.

---

## Post 6

The next experiment is a prompt-layout ablation.

Run the same 52 held-out examples through:

1. current long rubric
2. short neutral rubric
3. candidate-first prompt
4. balanced good/bad rubric
5. goodness-first prompt
6. rubric-after-candidate prompt
7. threshold-tuned variant

Measure false negatives, recall, precision, strict pairwise accuracy, and `logprob(good) - logprob(bad)`.

---

## Post 7

The lesson:

Prompting is not just “giving rules.”

Prompting changes inference-time calibration.

A long safety-heavy rubric can make an LLM judge safer in one direction but worse in another: great at rejecting bad outputs, too strict about accepting good ones.

For my Tenacious Bench project, this gives a sharper explanation of why DPO tuning beat the prompted judge.

---

## Post 8

Concrete portfolio update:

I will add prompt-ablation support to `eval_prompted_judge.py`, update the model card with a prompt-only judge calibration warning, and revise the final report to separate:

```text
ranking ability
```

from:

```text
deployment-style binary gate calibration
```

That distinction is now the real lesson.
