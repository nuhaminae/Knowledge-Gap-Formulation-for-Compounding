# Sources — Attention Sinks, Long-Prefix Behavior, and Judge Calibration

## Source 1 — Efficient Streaming Language Models with Attention Sinks

**Citation:**  
Guangxuan Xiao, Yuandong Tian, Beidi Chen, Song Han, Mike Lewis. “Efficient Streaming Language Models with Attention Sinks.” arXiv:2309.17453, 2023.

**Link:** [https://arxiv.org/abs/2309.17453](https://arxiv.org/abs/2309.17453)

**Why I used it:**  
This paper gives the mechanism behind “attention sinks.” It shows that initial tokens can receive strong attention scores even when they are not semantically important. This matters for my question because my prompted judge begins with a long rubric. If the first tokens frame the model as a strict QA auditor, they may have an outsized influence on the final `good` vs. `bad` decision.

**Key idea I used:**  
Early prompt tokens can become persistent attention anchors. Therefore, the opening section of a judge prompt is not neutral; it can shape the model’s decision behavior throughout inference.

---

## Source 2 — Lost in the Middle: How Language Models Use Long Contexts

**Citation:**  
Nelson F. Liu, Kevin Lin, John Hewitt, Ashwin Paranjape, Michele Bevilacqua, Fabio Petroni, Percy Liang. “Lost in the Middle: How Language Models Use Long Contexts.” Transactions of the Association for Computational Linguistics, 2024.

**Link:** [https://arxiv.org/abs/2307.03172](https://arxiv.org/abs/2307.03172)

**Why I used it:**  
This paper shows that models do not use all parts of a long context equally. Performance can change depending on where the relevant information appears. This matters because my prompted judge places the rubric first, examples next, and the candidate output late in the prompt.

**Key idea I used:**  
The candidate output may not receive the same practical weight as the opening rubric or the final verdict position, especially in a long judge prompt.

---

## Source 3 — Judging the Judges: A Systematic Study of Position Bias in LLM-as-a-Judge

**Citation:**  
Lin Shi, Chiyu Ma, Wenhua Liang, Weicheng Ma, Soroush Vosoughi. “Judging the Judges: A Systematic Study of Position Bias in LLM-as-a-Judge.” arXiv:2406.07791, 2024.

**Link:** [https://arxiv.org/abs/2406.07791]([https://arxiv.org/abs/2406.07791])

**Why I used it:**  
This paper studies systematic position bias in LLM judges. It supports the broader point that LLM-as-a-judge outputs are not purely objective judgments of content; they can be shaped by where content appears in the prompt and by the prompt’s structure.

**Key idea I used:**  
Judge prompts should be evaluated as mechanisms, not just as instructions. If candidate position or rubric ordering changes the outcome, that is a real inference-time calibration issue.

---

## Source 5 — My Week 11 Evaluation Artifacts

**Files / artifacts:**

- `src/evaluation/eval_prompted_judge.py`
- `reports/prompted_judge_metrics.json`
- `reports/ablation_results.json`
- `reports/evaluation_metric_comparison.json`
- `reports/final_report_v2.md`

**Why I used them:**  
These artifacts show the observed behavior that motivated the question: the prompted judge achieved perfect precision but lower recall, meaning it caught every bad output but over-rejected some good ones.

**Key result used:**

```markdown
Prompted judge:
accuracy = 88.46%
precision = 100.00%
recall = 76.92%
strict_pairwise_accuracy = 76.92%
rank-based pairwise_accuracy = 100.00%
```
