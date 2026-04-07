# 🤖 Robotic Autonomy

A research initiative for building robots that can operate safely and autonomously at scale. Two objectives: evaluating robots rigorously enough that we can trust them around humans, and making it easy to build and deploy agentic robot systems.

---

## Current Work

### [humanoid-safety-benchmark](https://github.com/menloresearch/humanoid-safety-benchmark)

Physics simulation (MuJoCo) that stress-tests humanoid robots against a child-sized mannequin across 14 scenarios — falls, trips, power loss, punches, kicks, body slams. Measures real injury metrics (HIC₁₅, Nij, ISO/TS 15066 contact forces) grounded in crash-test and collaborative robotics standards. Supports loading an active locomotion policy (ONNX) to compare its safety profile against the passive ragdoll baseline. Key finding so far: neck injury (Nij) is the critical concern, hitting 0.85 — close to the life-threatening threshold of 1.0 — in simple forward-fall scenarios.

### [agent-bench](https://github.com/menloresearch/agent-bench)

A framework where each benchmark is packaged as a **skill**: a self-contained folder with a `SKILL.md` tutorial and the code to run it. The agent reads the tutorial and executes the benchmark through CLI commands — no custom integrations needed. Currently includes `maze-bench`, a first-person 3D navigation benchmark where the agent interprets raycasted PNG views and finds its way to the exit. This tests the same core loop a real robot uses: perceive, reason, act, repeat.

---

## Objective 1: Evals

We need a continuous evaluation pipeline that can answer a simple question before any policy touches a real robot: **is this safe to deploy?**

That means running automated benchmarks on every policy update — not just for physical safety (will it hurt someone if it falls?) but across the full picture of what makes a robot trustworthy in the real world: how precisely it handles objects, how well it navigates, whether its capabilities hold up in new environments, how reliably it keeps working over a long shift, whether its decisions are safe, and whether humans feel comfortable around it.

The safety benchmark is the starting point. The goal is to expand it into a full suite covering all of these dimensions, with hard pass/fail thresholds that gate deployment — and continuous live monitoring once a robot is in the field so regressions get caught before they cause harm.

---

## Objective 2: Agent Harness

We need a better way to build and run agentic robots — both for individuals developing a single robot and for teams running fleets.

Agent-bench's skill-as-bench pattern is the core idea: robot behaviors defined as composable skills that any agent can read and execute, without bespoke glue code for every task and every robot. The goal is to generalize this into a full harness — easy skill composition for single robots, and a coordination layer for swarms: task allocation, shared world models, conflict resolution, and support for heterogeneous fleets (arms, humanoids, wheeled platforms, drones) under one orchestration layer.

---

## License

MIT
