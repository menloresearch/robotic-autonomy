# Interesting Work Notes

Scratchpad for research papers, blog posts, repos, and random ideas worth remembering.
No strict format — drop a title, link, and a one-liner on why it's interesting.

---

<!-- Template:
## [Title](url)
> One-liner on why this is interesting.
*Added: DD Mon YYYY*
-->

---

## [GLM-5.1 — zai-org](https://huggingface.co/zai-org/GLM-5.1)
> Next-gen flagship from ZhipuAI focused on long-horizon agentic engineering; key differentiator is it *doesn't plateau* — keeps improving over hundreds of rounds and thousands of tool calls where other models exhaust their strategies early.

**Why it matters:**
- 754B params, BF16, released on HF under `zai-org/GLM-5.1`
- SWE-Bench Pro: **58.4** (beats GLM-5 55.1, Claude Opus 4.6 57.3)
- NL2Repo (repo generation): **42.7** — big jump from GLM-5's 35.9
- Terminal-Bench 2.0: **63.5** (66.5 with Claude Code scaffold)
- CyberGym: **68.7** vs GLM-5's 48.3 — huge leap on cybersecurity agent tasks
- Deployment: SGLang ≥0.5.10, vLLM ≥0.19.0, KTransformers, Transformers
- Paper: [arXiv 2602.15763](https://arxiv.org/abs/2602.15763)

*Added: 08 Apr 2026*

---

## [AURA: Always-On Understanding and Real-Time Assistance via Video Streams](https://arxiv.org/abs/2604.04184)
> End-to-end streaming VideoLLM that watches a live video feed continuously and answers open-ended questions *and* fires proactive responses — no decoupled trigger pipeline needed.

**Why it matters:**
- Submitted 5 Apr 2026 (brand new); authors from CUHK / Hongsheng Li's group
- Solves the key gap in existing streaming VideoLLMs: they either do captioning-only narration or rely on a separate trigger model — AURA unifies both in one model
- Integrates: context management + data construction + training objectives + deployment opt — all end-to-end
- Runs real-time demo with ASR + TTS at **2 FPS on 2× 80G GPUs**
- SOTA on streaming video benchmarks
- Model + inference framework released: [aurateam/AURA](https://huggingface.co/aurateam/AURA), demo site: [aurateam2026.github.io](https://aurateam2026.github.io)
- Relevance to robotics: "always-on" continuous perception + proactive response is exactly the loop a robot agent needs for long-horizon tasks

*Added: 08 Apr 2026*

