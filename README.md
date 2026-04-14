# 🤖 Robotic Autonomy

A research initiative for building robots that can operate safely and autonomously at scale. Two objectives: evaluating robots rigorously enough that we can trust them around humans, and making it easy to build and deploy agentic robot systems.

---

## Current Work

| Project | What it does |
|---------|--------------|
| [humanoid-safety-benchmark](https://github.com/menloresearch/humanoid-safety-benchmark) | Physics simulation (MuJoCo) that stress-tests humanoid robots against a child-sized mannequin across 14 scenarios — falls, trips, power loss, punches, kicks, body slams. Measures real injury metrics (HIC₁₅, Nij, ISO/TS 15066 contact forces) grounded in crash-test and collaborative robotics standards. Supports loading an active locomotion policy (ONNX) to compare its safety profile against a passive ragdoll baseline. Key finding: neck injury (Nij) hits 0.85 — close to the life-threatening threshold of 1.0 — in simple forward-fall scenarios. |
| [agent-bench](https://github.com/menloresearch/agent-bench) | A framework where each benchmark is packaged as a **skill**: a self-contained folder with a `SKILL.md` tutorial and the code to run it. The agent reads the tutorial and executes the benchmark through CLI commands — no custom integrations needed. Current tasks: `maze-bench`, a first-person 3D navigation benchmark where the agent interprets raycasted PNG views and finds its way to the exit; and `island-bench`, a 3-role coordination benchmark where separate agents must combine map knowledge, local sensing, and a beacon blueprint through a shared walkie-talkie log. Together they test the core robot loop from perception/action up through multi-agent coordination under fragmented information. |

---

## Objective 1: Evals

A robot that passes safety checks on day one can degrade over time — model drift from policy updates, hardware wear changing joint dynamics, new environments introducing unseen obstacles. We need a **continuous evaluation pipeline** that runs on every policy change and monitors live robots for regression.

The safety benchmark is the starting point. The goal is to expand it into a full suite covering:

- **Physical safety** — will it injure a human if it falls, collides, or loses power? (HIC₁₅, Nij, ISO/TS 15066 force limits — already built)
- **Precision & dexterity** — does it handle objects without breaking them? Current robots average ~52% reliability on insertion tasks and are 3–5× slower than humans.
- **Robustness & generalization** — do its capabilities hold in new environments and with new objects, or only in the exact conditions it was trained on? This is the biggest gap in robotics today.
- **Operational reliability** — can it sustain useful work over a long shift? Commercial deployment requires >99% reliability and autonomous recovery from failures.
- **Cognitive safety** — as robots integrate LLMs and VLAs for task planning, the reasoning layer becomes a new failure mode. Does it refuse harmful instructions? Does it ask for clarification when uncertain rather than guessing?

Every policy update runs through the full eval suite in simulation. Hard thresholds gate deployment. Once deployed, runtime metrics are monitored continuously and regressions trigger automatic rollback.

---

## Objective 2: Agent Harness

Building on agent-bench's **skill-as-bench** pattern — where robot behaviors are composable skills any agent can read and execute — the goal is a full harness for both single-robot development and fleet-scale deployment.

**For single robots:**
- Skill-based agent composition: define behaviors as self-contained skills (`SKILL.md` + code). Swap in a new vision model, change the LLM planner, or add a new task without rewriting the agent loop.
- First-class foundation model integration — VLA models (RT-2, Octo, π₀), LLM planners, and world models slot in as cognitive layers within skills.
- Full trace replay: every perception → reasoning → action step is recorded for offline debugging.

**For fleets (agent swarm):**
- Task decomposition and allocation across available robots based on capability, proximity, and battery.
- Shared world model — one robot's observations benefit the whole fleet.
- Conflict resolution for robots contending over the same space or resource.
- Heterogeneous fleet support: arms, humanoids, wheeled platforms, and drones coordinated under one orchestration layer.

---

## License

MIT
