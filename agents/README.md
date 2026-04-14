# agent-bench

Benchmark-as-skill framework — each benchmark is a self-contained `SKILL.md` + code. Current benchmarks are `maze-bench`, a first-person 3D navigation task driven by rendered PNG views and scored against the optimal path, and `island-bench`, a fragmented-information multi-agent coordination task.

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

### 13 Apr 2026 — Pi + OpenRouter model sweep (GLM 5.1, Kimi K2.5) + human play mode
- I reconfigured the Pi coding agent to route through **OpenRouter** instead of the local vLLM server; added `z-ai/glm-5.1` and `moonshotai/kimi-k2.5-0127` to `~/.pi/agent/models.json`.
- I ran `maze-bench` against both models via the Pi harness:
  - **GLM 5.1** — 548 steps, 4 resets, ended 60 steps from the exit. Failed (-49.2%), but the strongest non-Anthropic run so far.
  - **Kimi K2.5** — crashed mid-run at 37 steps (-99.2%). No stderr captured.
- In `agent-bench`, my authored commit today was **`e60ffbf`** (`maze-bench: add human play mode`):
  - added `benchmarks/maze-bench/skills/maze-game/human_play.py`
  - updated `benchmarks/maze-bench/README.md` so humans can test the maze directly
- Bill's account appears in git history as **`bill_cache <108576106+billcache@users.noreply.github.com>`**.
  - Bill-authored commit today: **`df4e483`** — `Merge pull request #10 from menloresearch/feat/maze-human-play`
  - This indicates Bill merged the PR, while the feature implementation commit itself was authored by me.
- Chart snapshot: `agents/evidence/13042026-maze-bench-chart.png`.
- Key takeaways:
  - Anthropic still holds the only passing runs; Opus 4.6 remains the leader at +47.7%.
  - Pi + OpenRouter is a low-friction way to sweep third-party models without touching benchmark code.
  - Human play mode makes it easier to validate benchmark usability and create a baseline outside autonomous runs.
  - Need stderr capture for OpenRouter runs to diagnose crashes like Kimi's.

### 14 Apr 2026 — island-bench multi-agent benchmark
- Added **island-bench** as the second `agent-bench` task.
- The benchmark is designed for **3 separate agent sessions** with locked roles:
  - **Cartographer** — sees full topology
  - **Scavenger** — moves, collects resources, assembles, and triggers the beacon
  - **Engineer** — sees the beacon blueprint and required items
- Core benchmark idea:
  - each role has incomplete information
  - agents must coordinate through a shared `walkie-talkie.md`
  - success requires combining map knowledge, local sensing, and blueprint knowledge
- Shipped the full benchmark package:
  - human-facing benchmark docs in `benchmarks/island-bench/README.md`
  - a dedicated 3-window launch guide in `benchmarks/island-bench/RUN_3_WINDOWS.md`
  - agent-facing tutorial in `benchmarks/island-bench/skills/island-escape/SKILL.md`
  - benchmark engine, scoring, and communication helpers
- Also carried forward the benchmark-integrity rule from maze-bench by explicitly forbidding source/state inspection.
- Main takeaway: `agent-bench` now tests not just single-agent navigation, but **multi-role coordination under fragmented information**.

## Overall status
- The project now has a working benchmark pattern:
  - skill tutorial
  - executable environment
  - objective scoring
- The benchmark family has expanded from a **single-agent navigation** task (`maze-bench`) to a **multi-agent coordination** task (`island-bench`).
- The main lesson from the logs is that benchmark integrity matters as much as benchmark mechanics; the skill must tightly constrain what information the agent is allowed to use.

## Work log files
- `07042026.md`
- `08042026.md`
- `09042026.md`
- `13042026.md`
- `14042026.md`
