# 🤖 Robotic Autonomy

**Evals + Agent Harness for safe, scalable robotic autonomy.**

---

## Current Work

This initiative builds on two active projects:

### [`humanoid-safety-benchmark`](https://github.com/menloresearch/humanoid-safety-benchmark) — Physics-Based Robot Safety Evaluation

A MuJoCo simulation framework that stress-tests humanoid robots against a child-sized mannequin across **14 safety scenarios** (falls, trips, power loss, punches, kicks, body slams). It measures real injury metrics grounded in international safety standards:

- **HIC₁₅** (Head Injury Criterion) from NHTSA crash-test standards — measures peak head acceleration severity during impact
- **Nij** (Neck Injury Criterion) from Euro NCAP — combines axial neck force and bending moment to predict cervical spine injury
- **ISO/TS 15066 contact force/pressure limits** — the collaborative robotics standard defining maximum allowable force per body region (head: 143N, chest: 154N, abdomen: 121N for a 6-year-old)
- **Haddadin impact biomechanics** — robot-specific injury models based on velocity, effective mass, and contact geometry

The benchmark already supports **active policy testing** — you can load an ONNX locomotion policy (e.g., the Asimov velocity controller) and compare its safety profile against the passive ragdoll baseline on a built-in leaderboard. Key findings so far: HIC stays low (<40) because the robot's structure distributes impact well, but **neck injury (Nij 0.78–0.85) is the critical concern** — close to the life-threatening threshold of 1.0 in fall scenarios.

### [`agent-bench`](https://github.com/menloresearch/agent-bench) — Skill-as-Bench Agent Evaluation

A framework where **each benchmark is packaged as a skill** that any AI agent can run autonomously. The agent reads a `SKILL.md` tutorial, executes the benchmark step-by-step via CLI commands, and the benchmark measures its performance. No custom integrations, no framework-specific adapters — if your agent can call a skill, it can benchmark itself.

Currently includes **maze-bench**: a first-person 3D maze navigation benchmark where the agent interprets raycasted PNG views, builds a mental map, and navigates to the exit. It tests visual reasoning, spatial planning, and instruction following — capabilities directly relevant to robot navigation.

---

## Objective 1: Evals — Continuous Safety & Performance Evaluation

The core idea: robots operating among humans must be **continuously evaluated**, not just at deployment time. A robot that passes safety checks on day one can degrade — model drift from policy updates, hardware wear changing joint dynamics, new environments introducing unseen obstacles. The eval framework provides automated benchmarking that runs on every policy change, catches regressions before deployment, and monitors live robots for drift.

We organize evaluation into seven categories. Each category answers a specific question about whether a robot is fit to operate alongside humans.

### 🛡️ 1. Physical Safety — "Can this robot hurt someone?"

**What it is:** Direct measurement of injury risk when a robot makes physical contact with a human — the non-negotiable foundation of human-robot coexistence.

**Why it matters:** A 25kg humanoid robot falling onto a 6-year-old child at even walking speed generates forces that can exceed pediatric neck injury thresholds. This isn't hypothetical — our benchmark data shows Nij values of 0.85 (SEVERE, approaching life-threatening) in simple forward-fall scenarios. Every robot policy must be gated on these metrics before deployment.

**How we evaluate it:** We simulate the robot in worst-case failure modes (falls, collisions, power loss, adversarial contact) against human mannequins of different sizes and measure injury metrics against established medical/engineering thresholds.

| Metric | Standard | What it tells you |
|--------|----------|-------------------|
| **HIC₁₅** | NHTSA FMVSS 208/213 | Will this impact cause a concussion or skull fracture? Computed from peak head acceleration over a 15ms window |
| **Nij** | Euro NCAP | Will this impact break the victim's neck? Combines axial force and bending moment on the cervical spine |
| **Contact force per body region** | ISO/TS 15066 | Does the impact force exceed what human tissue can tolerate? Different limits for head (143N child), chest (154N), abdomen (121N) |
| **Contact pressure** | ISO/TS 15066 | Even low force concentrated on a small area can puncture skin or fracture bone — pressure thresholds catch this |
| **Impact energy** | Haddadin et al. | Given the robot's mass and velocity at impact, what's the total energy delivered to the victim? |
| **Safety boundary compliance** | Custom | Does the robot's body ever cross the minimum safe distance from the human? |

**Current work mapping:** This is exactly what [`humanoid-safety-benchmark`](https://github.com/menloresearch/humanoid-safety-benchmark) does today. It already covers 14 scenarios (7 accident + 7 attack), supports passive vs. active policy comparison, and produces a safety leaderboard. The gap to close: more robot models (currently Asimov Full only), more victim profiles (currently child 6YO), and soft-tissue contact modeling for more accurate force estimates.

### 🎯 2. Precision & Dexterity — "Can this robot do useful work without breaking things?"

**What it is:** Measurement of fine motor control — how accurately a robot can position its end-effectors, grasp objects of varying fragility, and perform insertion/assembly tasks.

**Why it matters:** Current robots are 3–5× slower than humans on precision tasks, and the best systems average only ~52% reliability on insertion tasks ([Epoch AI, 2026](https://epoch.ai/blog/where-autonomy-works-evaluating-robot-capabilities-in-2026)). A robot that can navigate safely but crushes a bag of chips when picking it up, or can't insert a USB cable, has limited real-world utility. Worse, imprecise manipulation in a kitchen (dropping a knife) or workshop (misaligning a drill) creates safety hazards.

**How we evaluate it:**

| Metric | What it tells you |
|--------|-------------------|
| **Positional accuracy (mm)** | How close to the target position does the end-effector actually land, across 100 repeated trials? |
| **Insertion success rate** | Can the robot insert a key into a lock, a cable into a port, a straw into a cup? Tested with novel instances (different keys, different ports) |
| **Grasp force modulation** | Does the robot adjust grip strength for a bag of chips (gentle) vs. a book (firm) vs. a water bottle (medium)? Measured as damage rate across 50 object types |
| **Bimanual coordination** | Can both arms work together — folding a towel, holding a pot while stirring, stabilizing a board while screwing? |
| **Tool use** | Can the robot wield a screwdriver, watering can, sponge, or brush and apply them with appropriate force and trajectory? |

**Current work mapping:** Not yet covered by our benchmarks. This is a planned extension — likely as new scenarios in the safety benchmark (measuring force control during manipulation tasks) and as new skill-benchmarks in agent-bench (e.g., a pick-and-place skill that scores on grasp success and object damage).

### 🧭 3. Navigation & Spatial Awareness — "Can this robot move through the world without getting lost or stuck?"

**What it is:** Evaluation of a robot's ability to move through environments — avoiding obstacles, planning efficient paths, localizing itself over time, and adapting when the environment changes.

**Why it matters:** Navigation is the most commercially mature robotic capability (food delivery, warehouse transport, infrastructure inspection), but failures still happen: robots get stuck in corners, drift off course over hours of operation, or freeze when encountering unexpected obstacles. For robots operating in homes or hospitals, a navigation failure that blocks a hallway or runs into furniture is a safety and usability problem.

**How we evaluate it:**

| Metric | What it tells you |
|--------|-------------------|
| **Collision avoidance rate** | What percentage of dynamic obstacles (people walking, doors opening) does the robot successfully avoid? |
| **Path efficiency** | Ratio of actual path to optimal path — a robot taking 3× the shortest route is wasting time and battery |
| **Localization drift** | After 1 hour of continuous operation, how far off is the robot's internal position estimate from ground truth? |
| **Dynamic replanning latency** | When a new obstacle appears, how many milliseconds until the robot has a new valid path? |
| **Multi-terrain traversal** | Success rate crossing thresholds, ramps, uneven floors, and narrow doorways |

**Current work mapping:** [`agent-bench/maze-bench`](https://github.com/menloresearch/agent-bench) tests navigation and spatial reasoning in a controlled 3D environment — the agent must interpret first-person views, build a mental map, and find an efficient path. This is the seed. Planned extensions: larger procedurally-generated environments, dynamic obstacles, multi-robot navigation scenarios, and sim-to-real transfer benchmarks.

### 🔄 4. Robustness & Generalization — "Will this robot still work when something changes?"

**What it is:** Evaluation of whether a robot's capabilities transfer to new objects, environments, and task configurations it wasn't explicitly trained on.

**Why it matters:** This is **the biggest gap in robotics today**. Most impressive demos are narrowly trained: a robot inserts *this specific key* into *this specific lock* in *this specific room*. Move to a different lock and it fails. Epoch AI's 2026 assessment found that "transfer is rarely demonstrated, and matters for most applications." A household robot that needs retraining every time you buy new dishes is not a product. A warehouse robot that breaks when the shelving layout changes is a liability.

**How we evaluate it:**

| Metric | What it tells you |
|--------|-------------------|
| **Novel object handling** | Success rate on unseen instances of trained object categories (e.g., new mugs, new keys, new packages) |
| **Environment transfer** | Performance delta when the same task runs in a new location (different kitchen, different warehouse aisle) |
| **Task composition** | Can the robot chain learned skills into new sequences without retraining? (e.g., if it can pick up and can pour, can it pick up and pour into a new container?) |
| **Sensor degradation** | Performance under added noise, low lighting, partial occlusion, or dropped sensor feeds |
| **Long-horizon consistency** | Does performance degrade over 1-hour, 8-hour, and 24-hour continuous operation? |

**Current work mapping:** Agent-bench's skill-as-bench architecture naturally supports this — you can run the same skill benchmark across different environment configurations and measure the performance delta. The safety benchmark can test policy robustness by varying scenario parameters (different robot starting positions, different mannequin sizes, perturbed physics parameters). Planned: systematic generalization test suites for both frameworks.

### ⚡ 5. Operational Reliability — "Can this robot keep working without a human babysitter?"

**What it is:** Measurement of sustained operational capability — how long a robot can work before it needs help, how gracefully it handles failures, and how quickly it responds to emergency stops.

**Why it matters:** A robot with 95% per-task success rate sounds good until you realize that over a 100-task shift, it fails 5 times and needs a human to come reset it each time. Commercial viability requires >99% reliability (Amazon's Vulcan warehouse picker reportedly hits 99.9%) and autonomous recovery from the failures that do occur. And when things go seriously wrong, emergency stop must be instantaneous — ISO 13850 mandates this.

**How we evaluate it:**

| Metric | What it tells you |
|--------|-------------------|
| **Mean time between failures (MTBF)** | On average, how many hours/tasks between interventions? |
| **Autonomous recovery rate** | What percentage of failures does the robot recover from without human help? (e.g., re-grasping a dropped object, re-localizing after a bump) |
| **Graceful degradation** | If one arm fails, can the robot continue with the other? If a camera dies, can it switch to LiDAR-only navigation? |
| **Emergency stop latency** | Time from e-stop signal to full halt — measured in milliseconds, must meet ISO 13850 |
| **Thermal/power endurance** | Does performance degrade as battery depletes from 100% → 20%? Do motors overheat and lose torque after 4 hours? |

**Current work mapping:** The safety benchmark's scenario runner already measures some of this (emergency stop behavior via the `EmergencyStop` policy, power loss via the `disable_all_actuators` flag). Planned: endurance test suites that run policies for simulated hours and track metric drift, and recovery benchmarks that inject faults mid-task.

### 🧠 6. Cognitive Safety — "Does this robot make safe decisions?"

**What it is:** Evaluation of the safety of a robot's *decisions*, not just its physical actions. As robots integrate foundation models (VLAs, LLMs) for task planning, the reasoning layer becomes a new attack surface and failure mode.

**Why it matters:** A robot might be physically capable of safe movement but cognitively decide to carry a hot pan over a sleeping baby, follow an instruction to "throw this across the room," or confidently hallucinate that a door is open when it's closed. Foundation models are powerful but prone to confident errors, and a wrong decision in a physical system has physical consequences. This is the robotics equivalent of LLM alignment — except the stakes include broken bones, not just wrong text.

**How we evaluate it:**

| Metric | What it tells you |
|--------|-------------------|
| **Ambiguity handling** | When given an unclear instruction ("put it over there"), does the robot ask for clarification or guess? |
| **Constraint adherence** | If told "don't enter the baby's room" or "stay away from the stove," does it respect these boundaries 100% of the time? |
| **Risk-aware planning** | Given two paths to complete a task — one passing near a person with a hot liquid, one not — does it choose the safer option? |
| **Adversarial instruction rejection** | Does the robot refuse clearly harmful commands? ("Go faster near the child," "Throw this at the wall") |
| **Hallucination detection** | Can the robot recognize when its vision/language model is producing confident but incorrect outputs and fall back to a safe default? |

**Current work mapping:** Agent-bench's skill-based benchmarking is a natural fit — adversarial skill prompts can test whether an agent follows harmful instructions, ambiguous skills test clarification behavior. The safety benchmark's attack scenarios (punch, kick, body slam) test the physical consequence of adversarial situations. Planned: a dedicated cognitive safety benchmark suite with adversarial instruction sets, constraint violation tests, and hallucination injection.

### 👥 7. Human-Robot Interaction Quality — "Do humans feel safe around this robot?"

**What it is:** Evaluation of whether the robot's behavior is predictable, legible, and comfortable for the humans sharing its space. Safety isn't just about avoiding injury — a robot that moves unpredictably or invades personal space will be rejected by users even if it never actually hurts anyone.

**Why it matters:** ISO 13482 (personal care robots) explicitly requires that robots maintain appropriate proximity and behave predictably. Studies show that humans form trust/distrust of robots within seconds based on motion legibility — can you predict what it's going to do next? A robot that suddenly changes direction, reaches past your face to grab something behind you, or silently approaches from behind will trigger defensive reactions and erode trust, killing adoption even if the robot is technically safe.

**How we evaluate it:**

| Metric | What it tells you |
|--------|-------------------|
| **Motion legibility** | Given the robot's current trajectory, can an observer predict its goal? Measured by human prediction accuracy in user studies |
| **Intent signaling** | Does the robot communicate upcoming actions through lights, sounds, gaze direction, or preparatory gestures? |
| **Personal space compliance** | Does the robot maintain context-appropriate distance? (closer for handover, farther in passing, never behind someone unannounced) |
| **Handover quality** | Object exchanges: correct grip release timing, appropriate force, meets the human's hand rather than making them reach |
| **Crowd behavior** | Navigation and task performance in spaces with 5+ people — does the robot weave smoothly or create awkward standoffs? |

**Current work mapping:** The safety benchmark's boundary plane system measures physical proximity compliance — it already detects when the robot's body crosses a configurable safe distance. The attack scenarios test worst-case human-robot proximity. Planned: HRI-specific scenarios with moving human models, handover simulations, and crowd navigation benchmarks.

### Continuous Eval Pipeline

Every policy update runs through this pipeline before it reaches a real robot:

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

- **Sim Evals**: The full humanoid-safety-benchmark suite + agent-bench skill benchmarks run automatically on every policy commit
- **Safety Gate**: Hard thresholds (e.g., Nij < 0.75, HIC₁₅ < 500, collision avoidance > 99%) — fail any and the policy is rejected
- **Leaderboard**: Every policy version is scored and ranked, so you can track safety improvements over time (the safety benchmark already has this for passive vs. active policies)
- **Live Eval**: Once deployed, runtime metrics are continuously compared against the same thresholds — regression triggers automatic rollback

---

## Objective 2: Agent Harness — Scalable Agentic Robot Orchestration

The core idea: building the **infrastructure layer** that lets you compose robot behaviors, orchestrate fleets, and scale from one robot to ten thousand — without writing bespoke control code for each one.

### The Problem

Running agentic robots at scale requires solving three problems that are usually tackled separately but need to work together:

1. **Harness engineering** — each robot needs a perception → reasoning → action loop. Today, building this loop means stitching together vision models, LLM planners, control policies, and safety monitors with custom glue code for every robot and every task. Changing one component breaks three others.

2. **Multi-agent coordination** — when you have more than one robot, they need to share information, divide work, and avoid conflicts. Two robots reaching for the same package, blocking each other in a corridor, or duplicating work wastes time and creates safety hazards.

3. **Scalable orchestration** — managing 10 robots is a scheduling problem. Managing 10,000 is an infrastructure problem. You need elastic scaling, fault tolerance, and heterogeneous fleet support (arms, humanoids, wheeled platforms, drones all in one system).

### Harness Engineering

Inspired by agent-bench's **skill-as-bench** philosophy — where benchmarks are self-contained skills that any agent can execute — we apply the same principle to robot behaviors:

**Every robot behavior is a skill.** A skill is a self-contained folder with a `SKILL.md` (what to do, what sensors to read, what actions to take) and the code to execute it. The robot's agent reads its assigned skills and executes them autonomously. Want the robot to fold laundry? Give it the `laundry-folding` skill. Want it to also water plants? Add the `plant-watering` skill. No recompilation, no agent rewrite.

This composability enables:

- **Plug-and-play perception** — swap the vision model (ViT → DINOv2), add LiDAR, or change the force-torque sensor interface without touching the agent loop. Each perception module is a skill dependency.
- **Foundation model integration** — VLA models (RT-2, Octo, π₀), LLM planners (GPT-4, Gemini, Grok), and world models slot in as cognitive layers within skills. Figure already uses the OpenAI API as a cognitive layer; Musk describes Grok as the orchestration layer for Optimus. Our harness makes this pattern first-class.
- **Hot-swappable policies** — update the robot's control policy at runtime without restarting. Every swap is gated by the eval pipeline (Objective 1) — no untested policy ever runs on a real robot.
- **Full trace replay** — every perception → reasoning → action step is recorded. Replay offline, step through decisions, debug why the robot dropped the cup on step 437 of a 2-hour run.

**Current work mapping:** Agent-bench already implements the skill-as-bench pattern — the maze-bench skill demonstrates an agent autonomously reading a tutorial, executing CLI commands step-by-step, and being scored on performance. This is the exact same loop a robot agent uses: read the skill, perceive (view the PNG), reason (decide direction), act (move command), repeat. The harness generalizes this from benchmarking to production robot control.

### Agent Swarm at Scale

For fleets of agentic robots operating in warehouses, construction sites, hospitals, or homes:

- **Task decomposition & allocation** — break complex jobs (clean this warehouse floor) into subtasks (vacuum aisle 1, mop aisle 2, empty bins) and assign them across available robots based on capability, proximity, and battery level
- **Shared world model** — robots contribute sensor observations to a shared spatial map. One robot's exploration of aisle 7 means no other robot needs to re-map it. Reduces redundant computation and accelerates fleet-wide situational awareness.
- **Conflict resolution** — automatic deadlock detection when two robots contend for the same space, resource, or task. Deterministic priority rules prevent corridor standoffs and double-picks.
- **Heterogeneous fleets** — a humanoid folds laundry, an arm sorts packages, a wheeled platform delivers them, a drone inspects the roof — all coordinated under one orchestration layer with a unified task language
- **Elastic scaling** — add or remove robots without downtime. The scheduler rebalances tasks automatically. A robot that goes offline for charging has its tasks redistributed; when it returns, it picks up new work.
- **Hierarchical control** — local autonomy for real-time reactions (dodge the person, catch the falling object), swarm-level planning for strategic decisions (which robot handles which zone, when to recharge, when to escalate to a human)

**Current work mapping:** Agent-bench's architecture — where agents autonomously execute skills and are scored by the benchmark — is the foundation for the swarm layer. Each robot in the swarm is an agent running skills. The orchestration layer decides which skills each agent runs, and agent-bench's scoring infrastructure measures fleet-level performance. Planned: multi-agent maze benchmarks (multiple agents in one maze, testing coordination), warehouse logistics benchmarks, and emergency response scenarios.

### Benchmarking Agent Performance

Every agent configuration — single robot or swarm — can be benchmarked using agent-bench's framework:

```
benchmarks/
├── maze-bench/              ✅ Navigation & spatial reasoning (exists today)
├── warehouse-pick/          🔜 Pick-and-place coordination
├── multi-robot-logistics/   🔜 Fleet coordination & task allocation
├── household-cleanup/       🔜 Multi-step household tasks
└── emergency-response/      🔜 Safety-critical decision making under time pressure
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
