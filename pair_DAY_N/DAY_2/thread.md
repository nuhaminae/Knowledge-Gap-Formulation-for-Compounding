# Thread — Why the Second Prompt Worked

**Link:** [LinkedIn Thread](https://www.linkedin.com/posts/nuhamin-a-e-95n87a190_why-the-second-prompt-worked-httpslnkdin-activity-7457919045963788289-3Nr9?utm_source=share&utm_medium=member_desktop&rcm=ACoAACahjWcB-GSd6J8wRonZO0Rh5aMwXM4ESuc)

**1/6**
In SOC-01, the Conversion Engine had an honesty rule in the system prompt: don’t overclaim weak hiring signals.

But the model still wrote: “you are rapidly scaling your team.”

Then a follow-up rewrite prompt named that exact violation — and the second attempt passed.

Why?

---

**2/6**
The key mistake is treating the system prompt like a hard validator.

It isn’t.

At generation time, the model is still predicting the next token from the whole conversation. The system prompt shifts probabilities, but it does not ban every violating phrase.

So “don’t overclaim” can still lose to sales-copy priors.

---

**3/6**
The first attempt was open-ended drafting:

“Write a persuasive sales email, while following these rules.”

That creates competing pressures: sound relevant, mention hiring context, be persuasive, and stay honest.

The phrase “rapidly scaling your team” was likely a high-probability B2B sales phrase.

---

**4/6**
The second attempt changed the task.

It did not just repeat the rule. It said, effectively:

“You violated the rule by saying ‘rapidly scaling your team.’ The evidence only shows 2 open Python roles and low hiring velocity. Rewrite without that claim.”

That is violation-specific repair.

---

**5/6**
So the answer is not simply “user prompts are stronger than system prompts.”

The system prompt has higher formal authority, but the follow-up turn had higher local salience: it was recent, concrete, diagnostic, and named the exact bad phrase.

It made the correct behaviour easier to predict.

---

**6/6**
The production lesson:

A system prompt is a policy prior.
A follow-up correction is a repair signal.
A real guardrail needs a verifier.

For SOC-01, the fix should be: draft → scan for unsupported claims → name the violation → rewrite → scan again before sending.
