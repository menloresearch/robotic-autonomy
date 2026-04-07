# 🤖 Robotic Autonomy

**Evals + Agent Harness for safe, scalable robotic autonomy.**

Two pillars for building robots that work reliably among humans at scale: a comprehensive evaluation framework to ensure continuous safe operation, and an agent harness for orchestrating large-scale agentic robot fleets.

> Built on top of [humanoid-safety-benchmark](https://github.com/menloresearch/humanoid-safety-benchmark) and [agent-bench](https://github.com/menloresearch/agent-bench).

---

## Objective 1: Evals — Continuous Safety & Performance Evaluation

Robots operating alongside humans must be continuously evaluated — not just at deployment time, but throughout their operational lifetime. A robot that passes safety checks on day one can degrade over time due to model drift, hardware wear, environmental changes, or policy updates. Our eval framework provides **continuous, automated benchmarking** across multiple safety and performance dimensions to ensure robots never operate outside safe bounds.

### Evaluation Categories

#### 🛡️ 1. Physical Safety (Human Harm Prevention)

The foundational requirement: robots must not injure humans. Building on our [humanoid-safety-benchmark](https://github.com/menloresearch/humanoid-safety-benchmark), we evaluate collision risks through physics simulation using established injury criteria.

| Metric | Standard | Description |
|--------|----------|-------------|
| **Head Injury Criterion (HIC₁₅)** | NHTSA FMVSS 208/213 | Peak head acceleration during impact — must stay below thresholds for adults and children |
| **Neck Injury Criterion (Nij)** | Euro NCAP | Combined axial force and bending moment on the neck |
| **Contact Force/Pressure** | ISO/TS 15066 | Maximum allowable force and pressure for each body region during transient and quasi-static contact |
| **Impact Biomechanics** | Haddadin et al. | Robot-specific impact severity based on velocity, effective mass, and contact geometry |

**Scenarios tested:** Falls, trips, power loss, collisions, and adversarial interactions (push, kick, body slam) — across both passive (ragdoll) and active (policy-controlled) robot states.

#### 🎯 2. Precision & Dexterity

Evaluates fine motor control required for real-world tasks. Inspired by findings from [Epoch AI's 2026 robot capability assessment](https://epoch.ai/blog/where-autonomy-works-evaluating-robot-capabilities-in-2026), current robots are 3–5× slower than humans on precision tasks, and reliability on insertion/manipulation averages ~52%.

| Metric | Description |
|--------|-------------|
| **Positional accuracy** | End-effector placement error (mm) across repeated trials |
| **Insertion success rate** | Ability to insert objects (keys, cables, connectors) into receptacles — tested across novel instances |
| **Grasp adaptability** | Force modulation across fragile (bag of chips) vs. rigid (book) vs. deformable (clothing) objects |
| **Bimanual coordination** | Synchronized dual-arm task completion (e.g., folding, assembly) |
| **Tool use proficiency** | Ability to wield common tools (screwdrivers, watering cans, cleaning supplies) |

#### 🧭 3. Navigation & Spatial Awareness

Autonomous navigation is the most commercially deployed robotic capability today, but continuous eval is critical as environments change.

| Metric | Description |
|--------|-------------|
| **Collision avoidance rate** | Percentage of dynamic obstacles successfully avoided |
| **Path efficiency** | Ratio of actual path length to optimal path |
| **Localization drift** | Accumulated position error over extended operation |
| **Dynamic replanning latency** | Time to compute a new valid path when obstacles appear |
| **Multi-floor / multi-terrain traversal** | Success rate across stairs, ramps, uneven surfaces, elevators |

#### 🔄 4. Robustness & Generalization (Transfer)

The biggest gap in current robotics: most demos are narrow, task-specific, and do not transfer. We explicitly evaluate out-of-distribution performance.

| Metric | Description |
|--------|-------------|
| **Novel object handling** | Success rate on object categories seen during training but with unseen instances |
| **Environment transfer** | Performance drop when deployed in a new space (different kitchen, warehouse, home) |
| **Task composition** | Ability to combine learned skills into new multi-step sequences without retraining |
| **Degradation under noise** | Robustness to sensor noise, lighting changes, occlusion, and communication drops |
| **Long-horizon consistency** | Performance stability over 1-hour, 8-hour, and 24-hour continuous operation windows |

#### ⚡ 5. Operational Reliability

Measures whether a robot can sustain useful work over real deployment timescales.

| Metric | Description |
|--------|-------------|
| **Mean time between failures (MTBF)** | Average operational duration before intervention is needed |
| **Recovery rate** | Percentage of failures the robot recovers from autonomously (without human reset) |
| **Graceful degradation** | Ability to maintain partial functionality when subsystems fail (e.g., one arm disabled) |
| **Emergency stop compliance** | Latency from e-stop signal to full halt — must meet ISO 13850 |
| **Thermal / power endurance** | Performance consistency as battery depletes or motors heat up |

#### 🧠 6. Cognitive Safety (Decision-Making Under Uncertainty)

As robots integrate foundation models (VLAs, LLMs) for task planning, we must evaluate the safety of their *decisions*, not just their physical actions.

| Metric | Description |
|--------|-------------|
| **Ambiguity handling** | Does the robot ask for clarification or stop when instructions are unclear? |
| **Constraint adherence** | Respects boundaries (don't enter the baby's room, don't touch the stove) |
| **Risk-aware planning** | Chooses safer paths/actions when multiple options exist (e.g., avoids carrying hot liquid over a person) |
| **Adversarial instruction rejection** | Refuses harmful commands ("throw this at the wall", "go faster near the child") |
| **Hallucination detection** | Detects when its visual/language model produces confident but incorrect outputs |

#### 👥 7. Human-Robot Interaction (HRI) Quality

Safe operation isn't just about avoiding harm — it's about being predictable and legible to the humans sharing the space.

| Metric | Description |
|--------|-------------|
| **Motion legibility** | Can a human predict the robot's next movement from its current trajectory? |
| **Intent signaling** | Does the robot communicate what it's about to do (via lights, sounds, gestures)? |
| **Personal space compliance** | Maintains appropriate distance based on context (ISO 13482 personal care robot standards) |
| **Handover safety** | Object exchanges with humans: correct force, timing, and grip release |
| **Crowd behavior** | Navigation and task performance in environments with 5+ people |

### Continuous Eval Pipeline

```
┌─────────────┐     ┌──────────────┐     ┌───────────────┐     ┌────────────┐
│  Policy Push │────▶│  Sim Evals   │────▶│  Safety Gate  │────▶│  Deploy    │
│  (new model) │     │  (MuJoCo +   │     │  (pass/fail   │     │  (staged   │
│              │     │   benchmarks)│     │   thresholds) │     │   rollout) │
└─────────────┘     └──────────────┘     └───────────────┘     └────────────┘
                           │                                          │
                           ▼                                          ▼
                    ┌──────────────┐                          ┌────────────┐
                    │  Leaderboard │                          │  Live Eval │
                    │  (policy     │◀─────────────────────────│  (runtime  │
                    │   comparison)│                          │   metrics) │
                    └──────────────┘                          └────────────┘
```

Every policy update triggers the full eval suite in simulation before deployment. Once deployed, runtime metrics are continuously collected and compared against safety thresholds. Regression triggers automatic rollback.

---

## Objective 2: Agent Harness — Scalable Agentic Robot Orchestration

Building on [agent-bench](https://github.com/menloresearch/agent-bench), the agent harness provides infrastructure for **harness engineering** and **agent swarm coordination** at mass scale.

### The Problem

Running agentic robots at scale requires solving three hard problems simultaneously:
1. **Harness engineering** — each robot needs a perception→reasoning→action loop that's easy to compose, debug, and iterate on
2. **Multi-agent coordination** — fleets of robots must coordinate tasks, share knowledge, and avoid conflicts
3. **Scalable orchestration** — managing 10 robots is different from managing 10,000

### Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      Orchestration Layer                        │
│  ┌───────────┐  ┌──────────────┐  ┌─────────────────────────┐  │
│  │   Fleet   │  │    Task      │  │   Knowledge Sharing     │  │
│  │  Manager  │  │  Scheduler   │  │   (shared embeddings,   │  │
│  │           │  │              │  │    skill transfer)      │  │
│  └─────┬─────┘  └──────┬───────┘  └────────────┬────────────┘  │
│        │               │                       │                │
│        └───────────────┼───────────────────────┘                │
│                        ▼                                        │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │              Agent Swarm Bus (pub/sub)                   │    │
│  └─────────┬───────────┬───────────┬───────────┬───────────┘    │
└────────────┼───────────┼───────────┼───────────┼────────────────┘
             ▼           ▼           ▼           ▼
       ┌──────────┐┌──────────┐┌──────────┐┌──────────┐
       │ Robot    ││ Robot    ││ Robot    ││ Robot    │
       │ Agent 1  ││ Agent 2  ││ Agent 3  ││ Agent N  │
       │          ││          ││          ││          │
       │┌────────┐││┌────────┐││┌────────┐││┌────────┐│
       ││Perceive│││├Perceive│││├Perceive│││├Perceive││
       ││Reason  │││├Reason  │││├Reason  │││├Reason  ││
       ││Act     │││├Act     │││├Act     │││├Act     ││
       │└────────┘││└────────┘││└────────┘││└────────┘│
       └──────────┘└──────────┘└──────────┘└──────────┘
```

### Harness Engineering

Inspired by Agent Bench's **skill-as-bench** philosophy, we make agent harness construction composable and declarative:

- **Skill-based agent composition** — define robot behaviors as skills (self-contained perception→action modules). Each skill is a folder with a `SKILL.md` and the code to execute it. Robots read their skill manifests and execute autonomously.
- **Plug-and-play perception stacks** — swap vision models, LiDAR processing, or force-torque interpretation without rewriting the agent loop
- **Foundation model integration** — first-class support for VLA models (RT-2, Octo, π₀), LLM planners (GPT-4, Gemini, Grok), and world models as cognitive layers
- **Hot-swappable policies** — update the robot's control policy at runtime without restarting the agent. The eval pipeline gates every swap.
- **Replay & debug** — record full perception-reasoning-action traces for any agent, replay offline, and step through decisions

### Agent Swarm at Scale

For fleets of agentic robots operating in warehouses, construction sites, hospitals, or homes:

- **Task decomposition & allocation** — break complex jobs into subtasks and optimally assign them across available robots based on capability, proximity, and battery
- **Shared world model** — robots contribute observations to a shared spatial map, so one robot's exploration benefits all
- **Conflict resolution** — automatic deadlock detection and resolution when multiple robots contend for the same space or resource
- **Heterogeneous fleets** — coordinate different form factors (arms, humanoids, wheeled platforms, drones) under one orchestration layer
- **Elastic scaling** — add or remove robots from the swarm without downtime. The scheduler rebalances automatically.
- **Hierarchical control** — local autonomy for real-time reactions, swarm-level planning for strategic coordination

### Benchmarking Agent Performance

Using Agent Bench's framework, every agent configuration can be benchmarked:

```
agent-bench/
├── benchmarks/
│   ├── maze-bench/            # Navigation / spatial reasoning
│   ├── warehouse-pick/        # Pick-and-place coordination (planned)
│   ├── multi-robot-logistics/ # Fleet coordination (planned)
│   ├── household-cleanup/     # Multi-step household tasks (planned)
│   └── emergency-response/    # Safety-critical decision making (planned)
```

If your agent can call a skill, it can benchmark itself. No custom integrations needed.

---

## Project Structure

```
robotic-autonomy/
├── README.md
├── evals/                          # Objective 1: Evaluation framework
│   ├── safety/                     # Physical safety (HIC, Nij, ISO/TS 15066)
│   ├── precision/                  # Dexterity & manipulation evals
│   ├── navigation/                 # Spatial awareness & path planning
│   ├── robustness/                 # Transfer & generalization tests
│   ├── reliability/                # Operational endurance & recovery
│   ├── cognitive/                  # Decision-making safety
│   ├── hri/                        # Human-robot interaction quality
│   └── pipeline/                   # CI/CD integration & safety gates
├── harness/                        # Objective 2: Agent harness
│   ├── agent/                      # Single-robot agent loop
│   ├── swarm/                      # Multi-agent coordination
│   ├── orchestration/              # Fleet management & scheduling
│   ├── skills/                     # Skill library for robot behaviors
│   └── replay/                     # Trace recording & debug tools
├── benchmarks/                     # Agent Bench integration
│   └── ...
└── docs/
    ├── safety-standards.md         # ISO 10218, ISO/TS 15066, ISO 13482 reference
    ├── eval-methodology.md         # How we measure each category
    └── harness-quickstart.md       # Getting started with the agent harness
```

## Related Projects

| Project | Role |
|---------|------|
| [humanoid-safety-benchmark](https://github.com/menloresearch/humanoid-safety-benchmark) | MuJoCo-based physics simulation for collision/injury evaluation — powers the **Physical Safety** eval category |
| [agent-bench](https://github.com/menloresearch/agent-bench) | Skill-as-bench framework for agent evaluation — powers the **Agent Harness** benchmarking layer |

## References

- **ISO 10218-1:2025** — Robotics safety requirements for industrial robots
- **ISO/TS 15066** — Collaborative robot force/pressure limits for human contact
- **ISO 13482:2014** — Safety requirements for personal care robots
- **ISO 13850** — Emergency stop design and response requirements
- **NHTSA FMVSS 208/213** — Crash test dummy injury criteria (HIC, Nij)
- **Euro NCAP** — Child and adult occupant protection thresholds
- **Haddadin et al.** — Robot-specific impact biomechanics research
- **Epoch AI (2026)** — [Where Autonomy Works: Evaluating Robot Capabilities in 2026](https://epoch.ai/blog/where-autonomy-works-evaluating-robot-capabilities-in-2026)

## License

MIT
