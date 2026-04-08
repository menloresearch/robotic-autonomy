---
name: repo-structure
description: Describes the structure of the robotic-autonomy repo — its topic folders, their purpose, and how to create and maintain work-log notes (ddmmYYYY.md format). Use this skill whenever you need to navigate the repo, add new work, or record progress for the day.
---

# Robotic Autonomy — Repo Structure

This repo is a research initiative for building, evaluating, and deploying autonomous robot systems. It has **two top-level objectives** (Evals and Agent Harness), each reflected in the folder layout below.

---

## Top-level Layout

```
robotic-autonomy/
├── README.md                  # Project overview, objectives, current work
├── evals/                     # Objective 1 — evaluation pipeline
├── agent-harness/             # Objective 2 — agent harness (single robot + fleet)
├── humanoid-safety/           # Sub-project: humanoid safety benchmark (MuJoCo)
├── agent-bench/               # Sub-project: agent-bench skill-as-bench framework
├── research/                  # Background reading, paper notes, references
└── .pi/skills/repo-structure/ # This skill
```

---

## Folder Details

### `evals/`
**Topic:** Continuous evaluation pipeline — runs on every policy change, monitors live robots.

Covers:
- Physical safety (HIC₁₅, Nij, ISO/TS 15066 contact force limits)
- Precision & dexterity (object manipulation, insertion tasks)
- Robustness & generalization (new environments, new objects)
- Operational reliability (long-shift endurance, failure recovery)
- Cognitive safety (LLM/VLA planner refusals, uncertainty handling)

Work logs go here as date-named files.

---

### `agent-harness/`
**Topic:** Skill-based agent composition for single robots and fleet-scale deployment.

Covers:
- Skill-based behavior composition (`SKILL.md` + code pattern)
- Foundation model integration (VLA: RT-2, Octo, π₀; LLM planners; world models)
- Full trace replay (perception → reasoning → action)
- Fleet: task decomposition, allocation, shared world model, conflict resolution
- Heterogeneous fleet coordination (arms, humanoids, wheeled, drones)

Work logs go here as date-named files.

---

### `humanoid-safety/`
**Topic:** Physics simulation (MuJoCo) stress-testing humanoid robots.

Covers work on the [`humanoid-safety-benchmark`](https://github.com/menloresearch/humanoid-safety-benchmark):
- Scenario design (falls, trips, power loss, punches, kicks, body slams)
- Injury metric implementation and tuning (HIC₁₅, Nij, ISO/TS 15066)
- ONNX locomotion policy loading and comparison vs. passive ragdoll
- Findings and regressions (e.g. Nij 0.85 in forward-fall scenarios)

Work logs go here as date-named files.

---

### `agent-bench/`
**Topic:** Benchmark-as-skill framework for testing robot agent loops.

Covers work on [`agent-bench`](https://github.com/menloresearch/agent-bench):
- New benchmark skills (e.g. `maze-bench`, navigation tasks)
- Raycasted visual input pipeline (PNG → LLM → action)
- Expanding the perceive → reason → act → repeat loop
- Agent performance analysis and scoring

Work logs go here as date-named files.

---

### `research/`
**Topic:** Background reading, paper summaries, and reference material.

Not tied to a single sub-project. Use it for:
- Paper notes (title + date in filename is fine, e.g. `rt2-notes-08042026.md`)
- Literature surveys
- Links and bookmarks worth preserving in the repo

Work logs / reading logs go here as date-named files.

---

## Work Log Convention

Every folder uses **date-named Markdown files** as work logs.

### Filename format

```
ddmmYYYY.md
```

| Part | Meaning | Example |
|------|---------|---------|
| `dd` | Day (zero-padded) | `08` |
| `mm` | Month (zero-padded) | `04` |
| `YYYY` | Four-digit year | `2026` |

Example for **8 April 2026**: `08042026.md`

### Suggested note template

```markdown
# <Topic Folder> — 08 Apr 2026

## What I did
-

## Findings / results
-

## Blockers
-

## Next steps
-
```

### Rules
1. One file per day per folder. If you work across multiple topics in a day, create a log in each relevant folder.
2. File lives directly inside the topic folder — no sub-directories needed for logs.
3. Keep logs factual and terse. Bullet points are preferred over prose.
4. If you reference code, link to the commit or file rather than pasting large snippets.

---

## Adding a New Topic Folder

1. Create the folder at the repo root with a short, lowercase, hyphenated name.
2. Add a one-line `README.md` inside it describing the topic.
3. Update the layout table in this SKILL.md.
4. Update the top-level `README.md` if the topic represents a significant new workstream.
