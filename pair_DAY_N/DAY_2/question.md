# Question - Agent and tool-use internals

On my peer's [Conversion Engine project](https://github.com/YohannesDereje/The-Conversion-Engine), his `agent_core.py` Phase 5 (~lines 461-503), **Signal Over-Claiming, Probe 01** (SOC-01) proved that embedding the honesty constraint in the system prompt was not sufficient — the model generated "you are rapidly scaling your team" despite an explicit instruction not to. The fix was to inject a follow-up user turn mid-conversation naming the specific violation and requesting a rewrite, and that second attempt passed.

My peer cannot explain why: at the token level, what is different about a constraint carried in the system prompt vs the same constraint arriving as a new user turn after the initial generation? Is this recency, role-tag weighting, or something about how instruction-tuned models process competing signals during decoding?

Knowing this would let him revise `probes/probe_library.md` SOC-01 from "prompt-only constraint proved insufficient" — which is just an observation — to an actual mechanistic explanation of the failure mode.
