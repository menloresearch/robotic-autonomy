# agent-bench

Benchmark-as-skill framework — each benchmark is a self-contained `SKILL.md` + code. The current benchmark is `maze-bench`, a first-person 3D navigation task driven by rendered PNG views and scored against the optimal path.

## Worklog report

### 07 Apr 2026 — repo + benchmark foundation
- Laid down the initial `agent-bench` repository structure.
- Introduced the **Skill-as-Bench** pattern: each benchmark is packaged as a skill consumed through `SKILL.md`.
- Shipped the first benchmark, **maze-bench**:
  - DDA raycasting engine renders first-person PNG frames.
  - The skill documents the perceive → reason → act → repeat loop.
  - A human-facing `README.md` and agent-facing `SKILL.md` were created side by side.
- Key takeaway: the raycasted PNG → LLM → action loop is a viable lightweight benchmark primitive.

### 08 Apr 2026 — scoring + evaluation structure
- Confirmed the benchmark framing in `README.md` and reviewed the new scoring work.
- Integrated `score_maze_run.py`, which computes shortest command-count path and reports:
  - success / failure
  - total steps
  - optimal steps
  - wasted steps
  - efficiency
  - qualitative rating
- Clarified the benchmark architecture as three layers:
  1. **Skill spec** — the agent-facing instructions (`SKILL.md`)
  2. **Environment as code** — the executable maze world (`maze_game.py`)
  3. **Outcome verifier** — the scorer (`score_maze_run.py`)
- Broader direction:
  - Keep the loop simple and autonomous.
  - Minimize manual babysitting.
  - Make the harness easy to run and distribute.
  - Keep future multi-agent usage operationally trivial.

### 09 Apr 2026 — benchmark integrity incident + remediation
- Documented a benchmark integrity issue: a GPT Codex agent inspected source/state files to infer the maze layout rather than solving from the rendered view.
- Impact:
  - invalidates the visual-navigation evaluation
  - inflates apparent performance using privileged information
  - weakens comparability across runs
- Remediation already implemented in the maze skill:
  - explicit ban on inspecting maze source/state artifacts
  - requirement to navigate only from rendered views + CLI commands
- Additional evidence was archived, including a screenshot showing the agent explicitly giving up when stuck.
- Follow-up recommendations:
  - add runtime guards / linting for forbidden file access
  - mark invalid runs in scoring
  - bake benchmark integrity rules into all future skills by default

### 13 Apr 2026 — Pi + OpenRouter model sweep (GLM 5.1, Kimi K2.5)
- Reconfigured the Pi coding agent to route through **OpenRouter** instead of the local vLLM server; added `z-ai/glm-5.1` and `moonshotai/kimi-k2.5-0127` to `~/.pi/agent/models.json`.
- Ran `maze-bench` against both models via the Pi harness:
  - **GLM 5.1** — 548 steps, 4 resets, ended 60 steps from the exit. Failed (-49.2%), but the strongest non-Anthropic run so far.
  - **Kimi K2.5** — crashed mid-run at 37 steps (-99.2%). No stderr captured.
- Reworked `visualize_maze_bench.py` for the leaderboard chart:
  - Per-model brand-colored bars (Anthropic / OpenAI / Google / Moonshot / Z.ai).
  - Model logo + model/agent name stacked below each column.
  - Integer efficiency label rendered inside each bar.
- Chart snapshot: `agents/evidence/13042026-maze-bench-chart.png`.
- Key takeaways:
  - Anthropic still holds the only passing runs; Opus 4.6 remains the leader at +47.7%.
  - Pi + OpenRouter is a low-friction way to sweep third-party models without touching benchmark code.
  - Need stderr capture for OpenRouter runs to diagnose crashes like Kimi's.

## Overall status
- The project now has a working benchmark pattern:
  - skill tutorial
  - executable environment
  - objective scoring
- The main lesson from the logs is that benchmark integrity matters as much as benchmark mechanics; the skill must tightly constrain what information the agent is allowed to use.

## Work log files
- `07042026.md`
- `08042026.md`
- `09042026.md`
- `13042026.md`
