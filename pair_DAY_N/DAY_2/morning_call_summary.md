# Morning Call Summary

During the morning call, we each explained our Week 12 questions and clarified the gaps we were trying to investigate.

For my peer’s question, he walked me through the SOC-01 failure in detail. I summarized my understanding back to him, and we agreed that his question is about the timing and level of instruction injection. Specifically, he wants to understand why an honesty constraint embedded in the original system prompt was ignored during the first generation, while a later follow-up user turn that explicitly named the violation successfully triggered a corrected rewrite.

For my question, I explained that I found the Document Intelligence Refinery a better fit for “Agent and tool-use internals” theme because it contains a concrete multi-tool routing problem: deciding between FastText/pdfplumber, Layout-Aware/Docling, and Vision-Augmented extraction. We dicussed the key gaps:  understanding how an agentic extraction router distinguishes correct escalation from bad tool-selection collapse, and what evidence it should log to make each tool decision auditable.
