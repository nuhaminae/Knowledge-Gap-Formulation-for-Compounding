# Grounding Commit

**Artifacts edited:** `probes/probe_library.md` — SOC-01 entry and new Mechanistic Notes section; `agent/agent_core.py` — module docstring, `_ASSERTIVE_CLAIM_RE` comment, Rule 5 comment block

**Commit link:** [https://github.com/YohannesDereje/The-Conversion-Engine/commit/eac8dfa6468e20a8b74ff50da31a56cf79ffed6a](https://github.com/YohannesDereje/The-Conversion-Engine/commit/eac8dfa6468e20a8b74ff50da31a56cf79ffed6a)

**What changed and why:**

`probe_library.md` previously recorded the SOC-01 failure as "Constraint is prompt-only (soft), not Python-enforced post-generation" — a correct observation but not an explanation. I could describe what happened but not why. The explainer taught me the mechanistic root cause: a system-prompt honesty rule is a soft logit influence, not a hard validator. During open-ended generation the model balances competing objectives simultaneously — write persuasive sales copy, sound relevant, infer company growth, follow the honesty constraint. When the first three objectives aligned on the high-probability sales phrase "you are rapidly scaling your team," the single honesty constraint lost locally despite being present in context. The constraint reduced the probability of overclaiming but did not eliminate it.

The follow-up user turn repair worked for a structurally different reason: it collapsed five competing objectives into one concrete task. By naming the exact forbidden phrase and requesting a rewrite, it converted the abstract policy prior into a violation-specific repair objective. The model was no longer doing open-ended generation — it was performing a correction on a labeled mistake, a pattern it has seen many times in training (assistant makes error → user points it out → assistant corrects). Recency and concreteness together made the honesty constraint locally dominant in a way it could never be in the system prompt.

The fix to `probe_library.md` replaces the observation with this mechanistic explanation and adds a Mechanistic Notes section that captures the correct production architecture: draft → extract factual claims → verify against evidence → name the violation → repair. This changes the framing of Phase 5 from "Python workaround for a prompt failure" to "a verifier that converts abstract policy into concrete, local, violation-specific repair signals" — which is why it works and what any engineer extending the claim-type coverage should understand before touching `_ASSERTIVE_CLAIM_RE`.

The fix to `agent_core.py` updates the Rule 5 comment block and module docstring with the same mechanistic reasoning, so the design rationale is visible at the code level rather than only in the probe library.
