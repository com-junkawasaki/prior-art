# TDCommons submission sheet — DP-2026-001

Ten separate records, **not one**. TDCommons indexes per record: the title, abstract
and keywords are what a search actually hits, so ten focused records are found where
one omnibus document is not.

## What TDCommons is, as verified 2026-08-19

| | |
|---|---|
| Cost | **Free.** Their FAQ: "Nothing! TDCommons is free to use." |
| Who may submit | Companies **and individuals** (recent records are individually authored) |
| Account | Required before uploading |
| Review | Yes — editorial review, revisions may be requested |
| Time to publish | Up to 72 hours after approval |
| Licence applied | **CC BY 4.0** (attribution required — slightly more restrictive than this bundle's CC0, and acceptable) |
| DOI | **No DOI.** A permanent sequential URL only (`tdcommons.org/dpubs_series/NNNNN`) |
| Indexing | Their FAQ: "The entire repository can be indexed and served using any search tool, including Google Patents." This is the reason to use it — it puts the disclosure in the non-patent-literature corpus examiners search |
| Series size | ~11,400 records as of 2026-08-19 |

Submit at: <https://www.tdcommons.org/cgi/ir_submit.cgi?context=dpubs_series>

Upload the **`.pdf`**. The `.md` and `.html` are kept alongside so the text stays diffable.

---

## 1. Record 02 — `DP-2026-001-02-capability-typed-actuation.pdf`

**Why this priority:** Highest novelty in the set, and the most actively filed area right now (agent / AI safety, least-privilege actuation). Submit this one first.

**Title** (copy-paste):

```
Actuation as a typed capability, such that an ungranted actuation cannot be expressed
```

**Abstract** (copy-paste):

```
Section 1 places a check between the actor and the hardware. That check can be bypassed if the
actor's language gives it *ambient authority* — the ability to reach a device, socket, file, or
driver directly, without asking. Every practical safety mechanism built on filtering an output
stream can be defeated by a component that writes to the device by another route. The actor's
program is written in a language, or a restricted subset of a language, in which the ability to
cause an externally observable effect is a value that must be received, not an operation that is
always available.
```

**Keywords** (copy-paste, semicolon-separated):

```
capability security; object capability; effect system; deny by default; sandboxing; WebAssembly;
ambient authority; robot actuation; least privilege; fail closed
```

---

## 2. Record 01 — `DP-2026-001-01-governor-gated-actuation.pdf`

**Why this priority:** Same field, equally active. Together with #2 it covers the whole 'model proposes, independent component permits' pattern.

**Title** (copy-paste):

```
Actuation gated by an independent governor, with the proposal and the permission held by separate parties
```

**Abstract** (copy-paste):

```
A robot controlled or advised by a learned model (a neural policy, a large language model, a
planner) can emit an actuation command that is physically unsafe. Filtering the model's output
inside the model, or inside the same program that generates it, does not bound the failure: the
component that decides what to do is the component that decides whether it is allowed. Two
separate software components with separate authority:
```

**Keywords** (copy-paste, semicolon-separated):

```
robot safety; governor; action admission; safety classification; audit ledger; telemetry proof;
human sign-off; mission bounding; autonomous systems; AI safety
```

---

## 3. Record 03 — `DP-2026-001-03-pre-dispatch-verified-choreography.pdf`

**Why this priority:** Drone light shows are a largely Chinese industry with active filing (formation safety, pre-flight validation). Narrow, concrete, easy for an examiner to match.

**Title** (copy-paste):

```
Multi-agent choreography verified in full before any vehicle is dispatched
```

**Abstract** (copy-paste):

```
A coordinated multi-vehicle motion — a drone light show, a warehouse fleet, a formation of
agricultural machines — is conventionally made safe by reactive collision avoidance during
flight. Reactive avoidance cannot give an advance guarantee, and it fails exactly when the
agents are densest. The entire choreography is precomputed as data and verified as a whole
before dispatch, so that dispatch is conditional on a completed proof rather than on runtime
reaction.
```

**Keywords** (copy-paste, semicolon-separated):

```
drone light show; swarm choreography; pre-flight verification; separation constraint; geofence;
multi-agent motion planning; UAV safety; formation flight
```

---

## 4. Record 06 — `DP-2026-001-06-single-dynamics-kernel.pdf`

**Why this priority:** Sim-to-real and unified physics kernels are heavily patented by robotics and simulation vendors.

**Title** (copy-paste):

```
One deterministic, dependency-free dynamics kernel serving simulation, training, and control
```

**Abstract** (copy-paste):

```
Robotics practice maintains at least three implementations of the same physics: one inside the
simulator, one inside the training environment, and one inside the controller's model. They
disagree, and the disagreement is discovered on the real robot. The simulators are additionally
platform-bound: they cannot run in a browser, in a serverless worker, or inside a verifier. A
single pure, deterministic, zero-dependency articulated-body dynamics kernel, written in a
portable source language and compiled to every target that needs it (server runtime, browser via
WebAssembly, native ahead-of-time binary, and a reference interpreter used as an oracle), such
that the same code answers all three roles.
```

**Keywords** (copy-paste, semicolon-separated):

```
articulated body dynamics; spatial vector algebra; recursive Newton-Euler; composite rigid body;
GJK; EPA; inverse kinematics; damped least squares; LQR; trajectory generation; deterministic
simulation; sim-to-real
```

---

## 5. Record 10 — `DP-2026-001-10-content-addressed-execution.pdf`

**Why this priority:** Provenance and audit of autonomous-system executions — active filing, and regulatory pressure is increasing it.

**Title** (copy-paste):

```
Content-addressed identity for a robot execution, so that what ran is recoverable by digest
```

**Abstract** (copy-paste):

```
After an incident, the question "what exactly was running" is usually unanswerable. Code
version, configuration, policy, model weights, inputs, and the effects performed are recorded in
different systems with different retention, and the combination is not identified by anything.
An execution is identified by the cryptographic digest of a structured value that names all of
its determinants together:
```

**Keywords** (copy-paste, semicolon-separated):

```
content addressing; execution identity; provenance; audit; reproducibility; effect log; receipt;
causal DAG; incident reconstruction; Merkle
```

---

## 6. Record 08 — `DP-2026-001-08-unified-occupancy-fusion.pdf`

**Why this priority:** Real but crowded art already; marginal value lower.

**Title** (copy-paste):

```
One occupancy representation fused from heterogeneous range sources through a single ingest interface
```

**Abstract** (copy-paste):

```
A mobile robot typically maintains one obstacle representation per sensor modality, fused late
and inconsistently, so that the planner's notion of free space differs from any single sensor's.
A single occupancy grid in a stated frame, with one ingest operation per source modality but a
single common representation: a rotating-lidar ring sweep and a depth image from a camera with
stated intrinsics and extrinsics both reduce to marked world points in the same grid. The grid
then supports, uniformly and independently of which sensor produced the evidence:
```

**Keywords** (copy-paste, semicolon-separated):

```
occupancy grid; sensor fusion; lidar; depth camera; A-star path planning; pure pursuit;
inflation; stuck detection; mobile robot navigation
```

---

## 7. Record 05 — `DP-2026-001-05-actuator-bom-torque-gate.pdf`

**Why this priority:** Design-automation / robot-sizing tooling. Moderate.

**Title** (copy-paste):

```
Actuator bill-of-materials derived from the kinematic description, with continuous-torque validation as a design gate
```

**Abstract** (copy-paste):

```
A robot's mechanical description (links, joints, limits, inertias) and its actuator selection
are conventionally maintained in different tools by different people. A design therefore passes
review with an actuator that cannot in fact hold the joint under continuous load. The kinematic
description is authored as structured data in which each joint may carry an actuator
specification as a field of the joint itself. From this single description two artifacts are
derived mechanically:
```

**Keywords** (copy-paste, semicolon-separated):

```
URDF; actuator sizing; bill of materials; continuous torque; robot design automation; inertia
validation; joint limits; design gate
```

---

## 8. Record 07 — `DP-2026-001-07-domain-randomization-as-data.pdf`

**Why this priority:** Moderate. The 'randomization as declarative data' framing is the distinctive part.

**Title** (copy-paste):

```
Domain randomization expressed as data, with reproducibility by construction
```

**Abstract** (copy-paste):

```
A policy trained in simulation transfers poorly unless the simulation is randomized.
Randomization implemented as code inside the environment is neither auditable nor reproducible:
the distribution that was actually sampled is not recoverable from the artifact, and a run
cannot be replayed exactly. The randomization is a declarative document, not code. It states,
per physical parameter, a range or distribution; per environment instance in a vectorized batch,
a sampled configuration is drawn; the draw uses an explicitly seeded, specified generator so
that the batch is a pure function of the seed and the document.
```

**Keywords** (copy-paste, semicolon-separated):

```
domain randomization; reinforcement learning; sim-to-real transfer; reproducibility; seeded
generator; vectorized environment; declarative configuration
```

---

## 9. Record 09 — `DP-2026-001-09-vehicle-parameterized-gnc.pdf`

**Why this priority:** Moderate. The distinctive part is vehicle physics as data under one unchanged controller.

**Title** (copy-paste):

```
One guidance-navigation-control step function parameterized by vehicle physics across media
```

**Abstract** (copy-paste):

```
Ground, marine, and aerial autonomy are built as separate stacks. The guidance and decision
logic is duplicated three times and diverges, although the difference between the vehicles is
confined to the physics of how a command becomes motion. A single guidance, navigation and
control step function — goal handling, path planning, path tracking, clearance checking, stuck
detection, arrival detection, and telemetry — parameterized by a vehicle model value supplying
only the medium-specific relations:
```

**Keywords** (copy-paste, semicolon-separated):

```
guidance navigation control; multi-domain autonomy; bicycle model; fixed wing; multirotor;
marine vehicle; heterogeneous fleet; vehicle model as data
```

---

## 10. Record 04 — `DP-2026-001-04-deadman-chord-estop.pdf`

**Why this priority:** Lowest marginal value — much of the deadman/e-stop art is old. Submit last, or not at all.

**Title** (copy-paste):

```
Deadman and chord-based emergency stop derived from a general-purpose input device
```

**Abstract** (copy-paste):

```
Teleoperation from a consumer input device is convenient and unsafe: the device has no certified
deadman, and a dropped or jammed controller continues to command motion. The mapping from device
state to actuation command is a pure function of the decoded device state, and it enforces two
independent conditions:
```

**Keywords** (copy-paste, semicolon-separated):

```
teleoperation; deadman switch; emergency stop; gamepad; chord input; heartbeat timeout; fail
safe; human-robot interface
```

---

## Two things to get right at submission time

- **Author field:** `Jun Kawasaki` — consistently across all ten, so the set is findable as a set.
- **Do not paraphrase the abstract into something shorter and vaguer.** The abstract is
  the field a keyword search hits. Vagueness there is the single easiest way to make a
  correct disclosure unfindable, which is the same as not publishing it.

