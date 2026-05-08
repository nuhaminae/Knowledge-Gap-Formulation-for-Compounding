# Sources — Position Bias Mitigation in LLM-as-a-Judge

## Source 1 — Judging the Judges: A Systematic Study of Position Bias in LLM-as-a-Judge

**Citation:**  
Lin Shi, Chiyu Ma, Wenhua Liang, Weicheng Ma, Soroush Vosoughi. “Judging the Judges: A Systematic Study of Position Bias in LLM-as-a-Judge.” arXiv:2406.07791, 2024.

**Link:** [Judging the Judges: A Systematic Study of Position Bias in LLM-as-a-Judge](https://arxiv.org/abs/2406.07791)

**Why I used it:**  
This paper directly studies position bias in LLM-as-a-judge systems. It defines position bias as a judge’s tendency to favor answers based on their position in the prompt rather than their content, and introduces metrics such as repetition stability, position consistency, and preference fairness.

**Key idea I used:**  
A pairwise judge decision is not fully reliable unless it is tested for stability under answer-position changes. This supports the order-swapping mitigation: evaluate A/B and B/A, then only count wins that persist across both orders.

---

## Source 2 — Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena

**Citation:**  
Lianmin Zheng, Wei-Lin Chiang, Ying Sheng, Siyuan Zhuang, Zhanghao Wu, Yonghao Zhuang, Zi Lin, Zhuohan Li, Dacheng Li, Eric P. Xing, Hao Zhang, Joseph E. Gonzalez, Ion Stoica. “Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena.” NeurIPS 2023.

**Link:** [Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena](https://arxiv.org/abs/2306.05685)

**Why I used it:**  
This is one of the canonical LLM-as-a-judge papers. It shows that LLM judges can be useful scalable evaluators, but it also explicitly discusses limitations such as position bias, verbosity bias, and self-enhancement bias.

**Key idea I used:**  
LLM judges are useful but not automatically neutral. A benchmark that uses an LLM judge should include bias controls, especially when pairwise preferences are used to report model or method improvements.

---

## Source 3 — Tool / Pattern Used: Order-Swapped Pairwise Judging

**Pattern:**  
Order-swapped pairwise evaluation.

**How it works:**

```markdown
1. Judge Candidate A vs Candidate B.
2. Judge Candidate B vs Candidate A.
3. Map the swapped labels back to original candidate IDs.
4. Count a winner only if the same original candidate wins in both orders.
5. Mark flipped decisions as unstable or position-bias-flagged.
```

**Why I used it:**
The asker’s repo already has a judge-filter scaffold and ablation harness, but the question identified that order swapping was missing. This pattern turns position bias from an invisible evaluation risk into a logged field.

**Key idea I used:**
A stable content preference should survive a presentation-order swap. If the winner changes only because the candidates changed slots, the pair should not be counted as reliable evidence for Delta A/B.
