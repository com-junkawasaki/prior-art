# Defensive Publication DP-2026-001 — Governed Robotics: Capability-Gated Actuation, Verified Choreography, and Deterministic Portable Dynamics

**Declarant / Author:** Jun Kawasaki (`root@junkawasaki.com`)
**Date of publication:** 2026-08-19 (UTC)
**Publication venue:** public GitHub organization `kotoba-lang`; this document; Software Heritage archive; see `evidence-manifest.txt` in this bundle
**Purpose:** This is a **defensive publication**. It is published solely to place the subject matter into the public domain of technical knowledge as prior art. No patent protection is sought by the author for any subject matter described here.
**Licence of all referenced source code:** Apache License 2.0
**Enabling reference implementations:** the 31 public repositories fixed by commit SHA-1 in `evidence-manifest.txt`, all under `https://github.com/kotoba-lang/`

---

## 0. Reader's note on what this document is for

Each numbered section below states a **technical problem**, a **mechanism** that solves it, and — deliberately and at length — a list of **variants and generalizations** of that mechanism. The variants are not incidental. They are stated so that a later filing directed to an obvious variation of the mechanism is met by an express written disclosure of that variation, not merely by an argument that it would have been obvious.

Working source code implementing each mechanism is publicly available at the commits fixed in `evidence-manifest.txt`. A person of ordinary skill in robotics software can read, build, and run that code; this document is therefore an enabling disclosure and not a mere statement of a result.

Where a section says "any", the word is intended in its ordinary broad sense and the enumeration that follows it is illustrative, not exhaustive.

---

## 1. Actuation gated by an independent governor, with the proposal and the permission held by separate parties

### Problem

A robot controlled or advised by a learned model (a neural policy, a large language model, a planner) can emit an actuation command that is physically unsafe. Filtering the model's output inside the model, or inside the same program that generates it, does not bound the failure: the component that decides what to do is the component that decides whether it is allowed.

### Mechanism

Two separate software components with separate authority:

1. An **actor** which may be arbitrary, learned, non-deterministic, or untrusted. It **proposes** an action. It has no path to actuation.
2. A **governor** which is small, deterministic, independently authored, and independently reviewable. It **admits or refuses** the proposal. Only its admission reaches actuation.

The proposal is a data record, not a function call, containing at minimum: an action identifier, the mission it belongs to, an action kind, a **safety classification** drawn from an ordered set (for example `:none`, `:low`, `:medium`, `:high`, `:safety-critical`), and parameters.

The governor is given an explicit **permitted set** of safety classifications and returns a decision record (`{:gate/decision :permit}` / `:refuse` with a reason). An action whose classification is outside the permitted set is refused. An action whose classification exceeds a threshold additionally requires a recorded human sign-off before it can be admitted.

A **mission** bounds the operation: one mission is one bounded operation with an explicit step ceiling and explicit boundaries (geographic, workspace, time). A mission does not loop internally; repetition is a durable outer loop that starts a new mission, so that every repetition re-enters admission.

A **safety-stop** record is a first-class value with a cause (`:e-stop`, `:watchdog`, `:governor-refusal`, `:operator`) and a source, and terminates the mission.

A **telemetry proof** binds a sensor observation (lidar return, force reading, contact, camera frame digest) to the mission and to a timestamp, and is appended to an append-only audit ledger, so that the sensing which justified an admission is recoverable after the fact.

The governing library performs **policy, not control**: it does not drive motors. It produces the records a governor needs in order to refuse unsafe actuation before the command reaches hardware.

### Variants and generalizations

- The safety classification may be: an ordered enumeration; a numeric risk score with thresholds; a lattice or partial order; a vector of independent axes (energy, reach, human proximity, irreversibility) combined by any monotone rule; a learned classifier whose output is *itself* subject to a conservative default when it fails to classify.
- The governor may be: a program; a table; a decision tree; a set of first-order or Datalog rules; a formally verified module; a hardware interlock; a separate microcontroller; a separate process, container, address space, or physical machine; a quorum of two or more independent governors requiring unanimity or a stated majority; a governor that itself must be admitted by a higher governor.
- The actor may be: a neural network policy; a large language model; a classical planner; a human teleoperator; a scripted sequence; another robot; a fleet scheduler; any composition of these.
- Admission may be: synchronous per action; batched per plan; staged (plan admitted, then each step re-admitted); revocable mid-execution; time-limited by a deadline or lease; quantity-limited by a budget of admitted actions; attenuated on delegation, so a sub-actor receives strictly fewer permissions than its parent.
- Refusal may be: a hard stop; a substitution with a stated safe default; a request for human sign-off; a degradation to a lower safety class; an escalation to a supervisor actor; a transition to a loiter, hold, or park behaviour.
- The audit record may be: an append-only file; a database table; a signed log; a hash-linked chain; a Merkle tree; a content-addressed object store; a distributed ledger; a quorum-certified reference. The record may be a full copy of the observation or a cryptographic digest of it with the observation retained separately.
- The bounded-operation unit may be called a mission, task, job, episode, work order, or transaction, and may be bounded by any of: step count, wall-clock deadline, energy budget, geofence, joint-space region, Cartesian workspace region, allowed object set, allowed human-proximity envelope.
- Human sign-off may be: a signature; a two-person rule; a hardware key; a biometric; a recorded verbal confirmation; an out-of-band confirmation on a separate channel.
- The whole arrangement applies unchanged to: mobile robots, manipulators, aerial vehicles, underwater vehicles, surface vessels, spacecraft, agricultural machinery, construction machinery, medical devices, laboratory automation, industrial process plant, and software agents whose effects are non-physical (payments, messages, deployments).

### Reference implementation

`kotoba-lang/robotics` (`kotoba.robotics`: `mission`, `action`, `requires-sign-off?`, `safety-stop`, `telemetry-proof`, `gate`), `kotoba-lang/governor`.

---

## 2. Actuation as a typed capability, such that an ungranted actuation cannot be expressed

### Problem

Section 1 places a check between the actor and the hardware. That check can be bypassed if the actor's language gives it *ambient authority* — the ability to reach a device, socket, file, or driver directly, without asking. Every practical safety mechanism built on filtering an output stream can be defeated by a component that writes to the device by another route.

### Mechanism

The actor's program is written in a language, or a restricted subset of a language, in which the ability to cause an externally observable effect **is a value that must be received**, not an operation that is always available.

Specifically:

- The program declares the set of effects it may perform. The declaration is part of the compiled artifact.
- The compiler **infers** the effect set from the program body and **refuses to compile** if the program performs an effect it did not declare, and equally if it declares an effect it does not perform.
- At load time the host supplies a set of **grants**. If the grants do not match the artifact's required effect set exactly, **instantiation fails** — the program never begins executing, rather than failing at the moment it first attempts the effect.
- The program has no general-purpose escape: no arbitrary module loading, no `eval`, no reflection, no foreign-function interface, no ambient global mutable state, no direct access to devices, sockets, filesystems, or credentials. There is exactly one way for an effect to occur, and it is through a granted capability.
- Each effect is additionally bounded by quantitative limits enforced by the host: an execution-step or "fuel" budget, a memory ceiling, a queue depth, a message-size ceiling. Exhaustion is a defined, reported outcome, not undefined behaviour.
- Errors are returned as values (a result type carrying either the success value or a typed error), not thrown, so that no control-flow path silently skips a check.
- Policy is **deny-by-default**: absence of a grant is refusal, never permission.
- Capabilities are **attenuable on delegation**: a component may pass on a strictly narrower capability (narrower resource, shorter deadline, smaller budget) but never a broader one.

Applied to robotics, the effects are precisely the actuation and sensing surfaces: set joint effort, publish a velocity command, energize a tool, read a sensor. A policy that was granted "read the lidar" and "publish a velocity command bounded to 0.4 m/s" **cannot express** energizing a gripper, because the operation is not merely forbidden — it is not reachable from the program text.

The capability is identified by a **name in a scoped semantic model** (kind, resource, holder) rather than by a numeric wire index; numeric indices, if any, are an internal transport detail of the host and do not appear in the program's source.

### Variants and generalizations

- The restricted language may be: a purpose-designed language; a subset of an existing language enforced by a verifier; a bytecode with a typed import table; WebAssembly with an explicitly declared import set; a sandboxed interpreter; a proof-carrying binary.
- The effect set may be: declared and checked; wholly inferred; inferred with declaration required only at public API and package boundaries; declared as a maximum ceiling with the inferred set required to be a subset.
- The grant may be: a plain token; a signed certificate; an object capability reference; a UCAN, CACAO, macaroon, biscuit, or comparable attenuable credential; a hardware-backed key; a lease with an expiry; a quorum-issued certificate. The binding of grant to artifact may be by exact set equality, by subset, or by a stated compatibility relation, provided that absence of a grant is refusal.
- The failure-closed point may be: compile time; link time; instantiation time; first use. Instantiation-time refusal is preferred because it removes the partially executed state, but each is a variant.
- Quantitative bounds may be: instruction counts; wall-clock deadlines; energy budgets; actuation-magnitude ceilings (velocity, force, torque, jerk, temperature); count of actuations; distance travelled; volume of workspace swept.
- The same arrangement covers non-robotic effects: network access, filesystem access, payment, message publication, model inference, code deployment.
- The host may be: a server process; a browser; an embedded runtime; a real-time executive; a microkernel; bare metal. The mechanism does not depend on an operating system being present.
- The capability namespace may be: flat names; hierarchical paths; URIs; content-addressed identifiers of the capability's own semantic definition.

### Reference implementation

`kotoba-lang/kotoba-lang` (`lang/capability-semantics.edn`, `lang/surface-status.edn`), `kotoba-lang/amu` (capability kits, effect inference, admission), `kotoba-lang/kototama` (host linking), `kotoba-lang/capability-*` (the individual capability contracts, including `capability-kami-engine`, `capability-motion-read`, `capability-topic-publish`, `capability-topic-subscribe`, `capability-irq-subscribe`, `capability-mmio-map`, `capability-dma-map`).

---

## 3. Multi-agent choreography verified in full before any vehicle is dispatched

### Problem

A coordinated multi-vehicle motion — a drone light show, a warehouse fleet, a formation of agricultural machines — is conventionally made safe by reactive collision avoidance during flight. Reactive avoidance cannot give an advance guarantee, and it fails exactly when the agents are densest.

### Mechanism

The entire choreography is **precomputed as data** and **verified as a whole before dispatch**, so that dispatch is conditional on a completed proof rather than on runtime reaction.

- A show is a set of performers, each with a sequence of timestamped waypoints, plus a footprint radius per performer.
- Positions between waypoints are obtained by a stated interpolation rule, giving a total function from time to the position of every performer.
- The verifier **samples the whole timeline** and evaluates a set of constraint predicates over the sampled configuration:
  - **separation**: pairwise distance minus the sum of footprint radii must exceed a minimum for all sampled times;
  - **speed**: the implied speed on each segment must not exceed a per-performer limit;
  - **acceleration**: the implied acceleration between consecutive segments must not exceed a limit;
  - **geofence**: every sampled position must lie inside a permitted volume;
  - segment feasibility is additionally checked separately in the vertical and horizontal components, because the achievable rates differ between them for most airframes.
- The verifier returns the **set of violations**, each naming the performers, the time, and the quantity by which the constraint was exceeded — not a single boolean. A show with any violation is not dispatchable.
- Only after the show verifies is it lowered to per-vehicle setpoint messages on the flight-control transport, and an `abort-all` primitive is retained that terminates every performer.

### Variants and generalizations

- Interpolation may be: linear; polynomial (cubic, quintic); spline; minimum-jerk; Bézier; a dynamics-consistent trajectory obtained by an optimizer; a recorded trajectory.
- Sampling may be: fixed-rate; adaptive; refined near predicted minima; replaced by an interval-arithmetic or sum-of-squares certificate that bounds the constraint over a continuum rather than at samples; replaced by conservative swept-volume intersection.
- Constraints may include, in addition to the above: minimum and maximum altitude; airspace or corridor membership; time-of-day windows; battery-energy feasibility per performer; radio-link-margin feasibility; downwash interaction; noise limits; light-output limits; wind envelope; loss-of-one-vehicle contingency (the show must remain separated when any single performer holds position or descends); loss-of-communication contingency; audience-exclusion-zone clearance.
- Performers may be: multirotors; fixed-wing aircraft; ground vehicles; surface vessels; underwater vehicles; manipulator arms sharing a workspace; gantries; stage machinery; mobile robots in a warehouse; a mixed heterogeneous set with per-class footprints and limits.
- The verified artifact may be: a file; a signed bundle; a content-addressed object whose digest is recorded in an audit ledger, so that the show that flew is provably the show that verified.
- Verification may be: a precondition of dispatch; re-run continuously during execution against measured positions with divergence beyond a bound triggering abort; re-run on any amendment, with amendment forbidden while running.
- The same arrangement applies to any precomputed multi-agent plan where an advance guarantee is preferred to runtime reaction, including robotic surgery instrument choreography, coordinated crane lifts, and multi-arm assembly cells.

### Reference implementation

`kotoba-lang/swarm-choreo` (`waypoint`, `performer`, `show`, `interpolate-position`, `positions-at`, `min-separation`, `separation-violations`, `speed-violations`, `accel-violations`, `geofence-violations`, `validate`, `abort-all`, `->pose-stamped`, `setpoint-publish-op`), on `kotoba-lang/org-ros`.

---

## 4. Deadman and chord-based emergency stop derived from a general-purpose input device

### Problem

Teleoperation from a consumer input device is convenient and unsafe: the device has no certified deadman, and a dropped or jammed controller continues to command motion.

### Mechanism

The mapping from device state to actuation command is a **pure function of the decoded device state**, and it enforces two independent conditions:

1. A **deadman**: a designated control must be *continuously held* for any nonzero motion command to be produced. Absence of the hold produces a zero command, not the previous command. Loss of the input report is therefore indistinguishable from release, and fails to stop.
2. An **emergency-stop chord**: a designated simultaneous combination of controls, chosen so that it cannot be produced by an ordinary grip or by a single stuck control, produces a safety-stop record which terminates the mission and is not overridable by subsequent motion input.

The decoded state is converted to a bounded velocity command (a linear and angular velocity pair) with per-axis clamping, and then to a transport-level publish operation. The proposal path passes through the governor of Section 1, so the teleoperation command is admitted on the same terms as an autonomous one.

### Variants and generalizations

- The device may be: a game controller; a joystick; a keyboard; a touchscreen; a phone; a haptic master device; a wearable; a gesture sensor; a voice channel; a brain-computer interface; a networked web client.
- The deadman may be: a held button; a held trigger beyond a threshold; a grip sensor; a required periodic re-press; a required continuous stream of heartbeat messages at a minimum rate, with staleness treated as release; a dead-man's handle in hardware.
- The chord may be: a simultaneous button combination; a sequence within a time window; a distinctive gesture; a spoken phrase; a physical guarded switch. In each case the requirement is that ordinary operation and single-fault stuck inputs cannot produce it.
- The command may be: a body-frame velocity; a joint velocity; a Cartesian pose delta; an end-effector wrench; a discrete action; a waypoint.
- The transport may be: a robotics middleware topic; a serial link; a CAN bus; a fieldbus; a websocket; a radio link.
- The mapping may additionally impose: rate limiting; slew limiting; scaling by measured proximity to obstacles or humans; scaling by remaining battery; scaling by link latency, with latency beyond a bound treated as release.

### Reference implementation

`kotoba-lang/teleop` (`deadman-held?`, `estop-chord-held?`, `->twist`, `->move-action`, `step`, `teleop-mission`, `teleop-action`, `cmd-vel-publish-op`), `kotoba-lang/com-sony-dualsense`.

---

## 5. Actuator bill-of-materials derived from the kinematic description, with continuous-torque validation as a design gate

### Problem

A robot's mechanical description (links, joints, limits, inertias) and its actuator selection are conventionally maintained in different tools by different people. A design therefore passes review with an actuator that cannot in fact hold the joint under continuous load.

### Mechanism

The kinematic description is authored as **structured data** in which each joint may carry an **actuator specification** as a field of the joint itself. From this single description two artifacts are derived mechanically:

1. A **chain actuator list / bill of materials** — one entry per joint, obtained by walking the joint graph, with a stated default where a joint omits its specification, and with **named realization variants** that override selected entries so that alternative build configurations of the same kinematic design are expressed as overrides rather than as forked descriptions.
2. A **continuous-torque validation**: for each joint, the torque demanded under the stated condition is compared against the specified actuator's continuous rating, and the design is reported as failing at the specific joint if the demand exceeds the rating.

The description is additionally validated structurally: the joint graph must be a well-formed tree with every parent and child link resolving and no cycle; joint limits (lower, upper, effort, velocity) must be present and ordered; inertia tensors must be physically admissible (symmetric, positive-definite, satisfying the triangle inequalities on principal moments).

Because the actuator data lives inside the kinematic description, a change to a joint's placement, limit, or inertia and the re-derivation of the bill of materials cannot fall out of step.

### Variants and generalizations

- The description format may be: URDF or another XML robot description; SDF; MJCF; USD; a JSON or EDN document; a database; a CAD assembly's derived data; a spreadsheet.
- The derived artifact may be: an actuator list; a full bill of materials including gearboxes, bearings, belts, encoders, brakes, cabling, and connectors; a cost roll-up; a mass and inertia roll-up used back in the dynamics; a lead-time and supply-risk roll-up; a procurement order.
- The validated quantity may be: continuous torque; peak torque; RMS torque over a duty cycle; thermal rise over a duty cycle; velocity; power; backdrivability; brake holding torque; gearbox rated input speed; bearing life; belt tension; encoder resolution against required repeatability; structural deflection; first natural frequency.
- The demand may be obtained from: a static gravity-load computation; inverse dynamics over a stated reference trajectory; a worst case over a set of trajectories; a measured duty cycle recorded from a real robot; a stochastic envelope.
- Variant selection may be: named overrides; a parameter sweep; an optimizer choosing actuators to minimize cost or mass subject to the validation passing; a catalogue search against vendor data.
- The validation may be: a report; a gate that blocks a design review; a continuous-integration check that fails a commit; a signed certificate attached to the released design.
- The same arrangement applies to any machine described kinematically: manipulators, mobile bases, gantries, exoskeletons, prosthetics, vehicle suspensions, animatronics, machine tools.

### Reference implementation

`kotoba-lang/org-ros-urdf` (`from-edn`, `chain-actuators-from-edn`, `default-bom-from-edn`, `bom-from-edn` with `:arm/realization :variants`, `validate-torque`), `kotoba-lang/kami-articulated` (`parse-urdf`, `link-index`, `joint-index`, `joint-graph-indexed-valid`, `inertia-tensor-valid`, `limit-*`).

---

## 6. One deterministic, dependency-free dynamics kernel serving simulation, training, and control

### Problem

Robotics practice maintains at least three implementations of the same physics: one inside the simulator, one inside the training environment, and one inside the controller's model. They disagree, and the disagreement is discovered on the real robot. The simulators are additionally platform-bound: they cannot run in a browser, in a serverless worker, or inside a verifier.

### Mechanism

A single **pure, deterministic, zero-dependency** articulated-body dynamics kernel, written in a portable source language and compiled to every target that needs it (server runtime, browser via WebAssembly, native ahead-of-time binary, and a reference interpreter used as an oracle), such that the same code answers all three roles.

The kernel implements, on spatial-vector (Plücker) algebra:

- rigid-body and articulated-body state, with joint transforms and motion subspaces per joint type;
- forward kinematics to world poses, and the **geometric Jacobian** per link;
- the **recursive Newton-Euler** algorithm for inverse dynamics and bias forces, and the **composite rigid-body** algorithm for the mass matrix, with a factorization-based linear solve;
- **forward dynamics** and gravity-compensation torques;
- contact detection by **GJK** distance and **EPA** penetration on convex shapes, oriented-bounding-box separating-axis tests with manifold generation, sphere–plane and conservative-advancement time-of-impact;
- contact resolution by impulses with an effective-mass computation, friction via a tangent basis, warm starting, and Baumgarte stabilization; joint-limit resolution by the same impulse machinery;
- **inverse kinematics** by damped-least-squares on position and on full pose, with an SO(3) orientation error;
- an **LQR** controller obtained by linearizing about an operating point and solving the discrete algebraic Riccati equation;
- trajectory generation: cubic, quintic, minimum-jerk, and waypoint-sequenced;
- a **vectorized batch** stepping interface with per-environment configuration, for training.

Because the kernel is pure and has no dependencies, its determinism is testable by identity: the same inputs produce byte-identical outputs on every target, and a **parity harness** compares targets against the reference interpreter. Because it has no I/O, it can be embedded inside the governor of Section 1 — a governor can *predict* the consequence of a proposed action using the same physics the simulator used, before admitting it.

### Variants and generalizations

- The portable source language may be any language with multiple compilation targets; the targets may include native machine code, WebAssembly, JavaScript, a bytecode, or an interpreted reference form. The claim is the *single-kernel, multi-target, parity-checked* arrangement, not a particular language.
- The dynamics formulation may be: spatial-vector recursive algorithms as above; maximal-coordinate with constraints; reduced-coordinate Lagrangian; articulated-body algorithm; featherstone variants; a differentiable formulation carrying gradients.
- Contact may be resolved by: sequential impulses; projected Gauss-Seidel; a linear- or nonlinear-complementarity-problem solver; compliant/penalty contact; position-based dynamics; an implicit time-stepping scheme.
- The controller may be: LQR; PD with gravity compensation; computed torque; model-predictive control; a learned policy; a hybrid where the learned policy proposes and a model-based term bounds.
- Determinism may be secured by: fixed-point arithmetic; a specified floating-point evaluation order with no fast-math reassociation; a specified reduction order in batch operations; a recorded seed for every stochastic element; byte-comparison against a reference target.
- The three roles unified may be any subset or superset of: simulation, training-environment stepping, controller internal model, governor's predictive check, digital twin, hardware-in-the-loop rig, post-hoc accident reconstruction, and a certification artifact.
- The kernel may be shipped as: a library; a WebAssembly module with a declared import set (Section 2); a native shared object; a service. Where shipped as a capability-gated module, it has no ambient authority and its outputs are values, so it cannot actuate by itself.

### Reference implementation

`kotoba-lang/com-nvidia-isaac-sim` (spatial algebra `plucker`/`crm`/`crf`/`spatial-inertia`; `mass-matrix`, `forward-dynamics`, `inverse-dynamics`, `gravity-torque`, `geometric-jacobian`, `point-jacobian`; `gjk-distance`, `epa-penetration`, `obb-sat`, `obb-manifold`, `conservative-advancement-toi`; `resolve-contacts`, `resolve-static-contact-friction`, `warm-start-static-contact`, `resolve-limits`; `position-ik`, `pose-ik`, `solve-dls`; `solve-dare`, `compute-gain`; `cubic-polynomial-trajectory`, `quintic-polynomial-trajectory`, `min-jerk`, `waypoint-trajectory`; `step-vectorized`, `step-vectorized-per-env`, `set-pd-drive`), `kotoba-lang/physics`, `kotoba-lang/physics-2d`, `kotoba-lang/kami-engine`.

---

## 7. Domain randomization expressed as data, with reproducibility by construction

### Problem

A policy trained in simulation transfers poorly unless the simulation is randomized. Randomization implemented as code inside the environment is neither auditable nor reproducible: the distribution that was actually sampled is not recoverable from the artifact, and a run cannot be replayed exactly.

### Mechanism

The randomization is a **declarative document**, not code. It states, per physical parameter, a range or distribution; per environment instance in a vectorized batch, a sampled configuration is drawn; the draw uses an **explicitly seeded, specified generator** so that the batch is a pure function of the seed and the document.

Consequently: the exact configuration of every environment instance is recorded and re-derivable; a training run is replayable from (document, seed); and a change to the randomization is a diff of a document rather than a diff of a program.

The batch exposes per-environment configuration set and clear operations, per-environment state read and write, and a stepping interface returning per-environment observation, reward, termination, and truncation.

### Variants and generalizations

- The document may be: EDN, JSON, YAML, TOML, a protocol-buffer message, a database row set, a content-addressed object.
- The randomized quantities may include: masses; inertias; friction coefficients; restitution; joint damping and stiffness; actuator gains, latency, backlash, and saturation; sensor noise, bias, drift, dropout, and latency; geometry scale; initial state; goal placement; obstacle placement and count; lighting, texture, and camera intrinsics and extrinsics; wind, current, and terrain; communication delay and packet loss.
- The distribution may be: uniform; normal; truncated normal; log-uniform; categorical; a mixture; an empirical distribution from measured hardware; an adversarial or curriculum-scheduled distribution whose schedule is itself part of the document.
- The generator may be any specified pseudorandom generator with recorded seed and recorded algorithm; counter-based generators additionally allow the per-environment stream to be derived from (seed, environment index, parameter index) so that instances are independent and individually reproducible without sequential draw order.
- The recorded artifact may be: the seed and document only; the seed, document, and generator identity; the fully expanded per-environment configuration; a digest of any of these bound into a training receipt together with the code version, so that a trained policy names the distribution it was trained on.
- The same arrangement applies to randomized testing of any system, not only robot learning: fuzzing, fault injection, load generation, and Monte-Carlo safety assessment.

### Reference implementation

`kotoba-lang/com-nvidia-isaac-lab` (`load-scene-edn`, `range-new`, `range-fixed`, `dr-around`, `dr-identity`, `dr-sample`, `dr-sample-n`, `lcg-new`, `lcg-step`, `next-uniform`, `randomize-physics`, `vectorized-cartpole-env-new`, `vectorized-reach-env-new`, `set-per-env-configs!`, `per-env-configs`, `step-result`), `kotoba-lang/org-farama-gymnasium`.

---

## 8. One occupancy representation fused from heterogeneous range sources through a single ingest interface

### Problem

A mobile robot typically maintains one obstacle representation per sensor modality, fused late and inconsistently, so that the planner's notion of free space differs from any single sensor's.

### Mechanism

A single **occupancy grid** in a stated frame, with **one ingest operation per source modality but a single common representation**: a rotating-lidar ring sweep and a depth image from a camera with stated intrinsics and extrinsics both reduce to marked world points in the same grid. The grid then supports, uniformly and independently of which sensor produced the evidence:

- **inflation** by the robot's footprint radius, so the planner may treat the robot as a point;
- conversion to a **cost grid**;
- **line-of-sight clearance** tests between cells;
- **nearest free cell** search, for recovering from a goal or start inside an obstacle;
- **forward clearance** queries, answered separately from the lidar-derived and camera-derived evidence so that a disagreement between modalities is visible rather than averaged away.

Planning consumes only the grid: an A\* search with an admissible heuristic, followed by path **simplification** (removing intermediate points whose removal leaves the path collision-free) and **smoothing**. Tracking consumes only the resulting path: a **pure-pursuit** steering law with a lookahead, plus a curvature-limited speed law derived from the discrete (Menger) curvature of the path, and a cross-track-error term.

A **stuck detector** observes that commanded progress is not producing measured progress and transitions to a defined recovery — an open-side steer or a loiter — rather than continuing to command a motion that is not working.

### Variants and generalizations

- The occupancy representation may be: a 2-D grid; a 2.5-D elevation map; a 3-D voxel grid; an octree; a signed-distance field; a set of convex free-space regions; a topological graph; a learned latent map. The invariant is that heterogeneous sources reduce to one representation before planning.
- Sources may include: rotating and solid-state lidar; stereo, structured-light, and time-of-flight depth cameras; monocular depth estimation; radar; sonar; ultrasonic rings; bumpers and tactile skins; wheel-slip detection; prior maps; other robots' shared observations; human-supplied annotations.
- Fusion may be: binary marking; log-odds occupancy with per-sensor inverse models; Bayesian update; Dempster-Shafer; per-modality layers preserved with an explicit combination rule (maximum-conservative, weighted, or learned); temporal decay for dynamic obstacles.
- Planning may be: A\*; Dijkstra; weighted, any-angle, or lazy variants; D\* and incremental replanning; RRT and RRT\*; probabilistic roadmaps; potential fields; lattice search with motion primitives; trajectory optimization; a learned planner. Post-processing may be: shortcutting; corner smoothing; spline fitting; optimization subject to clearance.
- Tracking may be: pure pursuit; Stanley; model-predictive control; feedback linearization; a learned controller. Speed limiting may be by: path curvature; clearance; visibility distance; braking distance to the nearest obstacle; measured traction; comfort limits.
- Recovery may be: rotate-in-place; back-up-and-retry; open-side steer; loiter or hold; request human assistance; replan with inflated cost; mark the cell as untraversable and replan.
- Disagreement between modalities may be: reported; resolved conservatively (the most pessimistic clearance wins); used to trigger a slow-down; used to trigger a sensor-fault diagnosis.

### Reference implementation

`kotoba-lang/kami-autodrive` (`ingest-lidar`, `ingest-camera-depth`, `inflated`, `to-cost-grid`, `line-clear?`, `nearest-free`, `forward-clearance`, `forward-clearance-camera`, `astar-grid`, `simplify`, `smooth`, `plan`, `pure-pursuit`, `steer`, `menger-curvature`, `curvature-speed-limit`, `cross-track-error`, `register-stuck`, `open-side-steer`, `loiter-step`), `kotoba-lang/kami-sensor-sim` (`lidar-new`, `lidar-intrinsics-vlp16`, `ring-sweep`, `camera-intrinsics-from-hfov`, `render-points-to-depth-image`, `imu-new`, `contact-sensor-new`), `kotoba-lang/kami-autodrive-scene`.

---

## 9. One guidance-navigation-control step function parameterized by vehicle physics across media

### Problem

Ground, marine, and aerial autonomy are built as separate stacks. The guidance and decision logic is duplicated three times and diverges, although the difference between the vehicles is confined to the physics of how a command becomes motion.

### Mechanism

A **single** guidance, navigation and control step function — goal handling, path planning, path tracking, clearance checking, stuck detection, arrival detection, and telemetry — parameterized by a **vehicle model value** supplying only the medium-specific relations:

- a ground model (bicycle kinematics with steering and wheelbase);
- a marine model (hull hydrodynamic resistance and thrust);
- a fixed-wing model (stall speed, bank-limited turn radius, air-density lapse with altitude);
- a multirotor model (thrust-to-weight, tilt-limited acceleration).

The step function is the same code in every case; the vehicle model is data. A fleet may therefore be heterogeneous and still be commanded, verified, and audited by one implementation, and a new vehicle class is added by supplying a model rather than by writing a stack.

The same function supports fleet-level operation: multiple agents each with pose and goal over a shared static scene, with an "ahead and closing" predicate used for mutual yielding, a minimum-separation query over the fleet, and an all-arrived predicate.

### Variants and generalizations

- Vehicle classes may further include: differential-drive; omnidirectional and mecanum; tracked; Ackermann with trailer; articulated and multi-body machines; legged; underwater vehicles with buoyancy and added mass; airships; spacecraft with reaction control; hybrid amphibious vehicles.
- The parameterization may be: a record of coefficients; a function value; a table; an identified model fitted from logged data; a learned dynamics model; a differentiable model.
- The shared logic may extend to: mission sequencing; energy-aware routing; contingency and return-to-home; formation keeping; traffic rules and right-of-way; collision-avoidance protocol; degraded-mode behaviour.
- Heterogeneous fleets may be coordinated by: pairwise yielding predicates; priority ordering; auction or market assignment; centralized scheduling; the precomputed verified choreography of Section 3.
- The arrangement applies to simulation and to real vehicles with the same code, and, with Section 6, to a governor predicting a proposed motion's consequence before admitting it.

### Reference implementation

`kotoba-lang/kami-autodrive` (`bicycle-model`, `ship-hydro`, `stall-speed`, `fixed-wing`, `multirotor`, `isa-density`, `limits`, `step`, `step-multimodal`, `fleet-agent`, `new-fleet`, `ahead-and-closing?`, `min-separation`, `all-arrived?`, `autopilot-config`, `new-autopilot`, `set-goal`, `telemetry`).

---

## 10. Content-addressed identity for a robot execution, so that what ran is recoverable by digest

### Problem

After an incident, the question "what exactly was running" is usually unanswerable. Code version, configuration, policy, model weights, inputs, and the effects performed are recorded in different systems with different retention, and the combination is not identified by anything.

### Mechanism

An **execution** is identified by the cryptographic digest of a structured value that names all of its determinants together:

`{program, input, initial state, runtime identity, effective policy, effect set} → content identifier`

The identifier is a digest over a canonical serialization, so it is computed independently by anyone holding the parts, and any difference in any determinant yields a different identifier. Around it:

- **Program, policy, capability grants, model weights, scene description, and randomization document are each content-addressed**, so the execution identifier transitively fixes all of them.
- The **effects performed** are logged as an ordered, hash-linked sequence, and the log's digest is part of the execution record. An execution whose effect log is complete is **replayable**; one whose log is incomplete is marked as such rather than presented as reproducible.
- A **receipt** binds the execution identifier to the outcome, the verification results, and the audit ledger entries of Section 1, including the telemetry proofs that justified each admission.
- Causality between executions is recorded as a **directed acyclic graph per principal**, with signed edges and a logical clock, rather than being flattened into a single global sequence. Agreement between principals is required only where a decision genuinely needs it.
- The identifier is **identity**, distinct from **location** (where the bytes are stored) and from **naming** (a mutable pointer to the current version). Conflating them is what makes a stored artifact unverifiable after it moves.
- An execution identifier is usable as a memoization key **only** when the effect set is empty or the effect log is complete, because otherwise the cached result does not observe that the world changed. Otherwise it is a receipt, not a cache key.

### Variants and generalizations

- The digest may be any collision-resistant hash; the identifier may be a bare digest, a multihash, a CID, a URN, or a digest with an algorithm label. Algorithm agility requires that the label be inside the identifier.
- The canonical serialization may be: a deterministic CBOR or JSON canonicalization; a defined binary encoding; a sorted key-value form. The requirement is that two parties independently produce identical bytes from the same value.
- Storage may be: a filesystem; an object store; a content-addressed store; a peer-to-peer network; a version-control repository; a distributed ledger. Availability is a separate property from identity and requires its own replication count and periodic verification.
- The effect log may be: append-only file; hash chain; Merkle tree with inclusion proofs; a quorum-certified log; a signed ledger. Timestamping of log entries may use a trusted timestamp authority, a blockchain anchor, or both.
- The determinants may be extended to include: sensor calibration; firmware versions; hardware serial numbers; ambient conditions; operator identity; the human sign-offs of Section 1.
- The arrangement applies to: robot missions; training runs; simulation runs; build and deployment; medical device operation; vehicle autonomous-mode engagement; any regulated operation for which "what exactly ran" must be answerable years later.

### Reference implementation

The canonical intermediate representations (state, transaction, capability, causal link, effect, execution) specified in this workspace's architecture records and implemented across `kotoba-lang/kototama`, `kotoba-lang/amu`, `kotoba-lang/kotobase-storage`, `kotoba-lang/io-ipld`, `kotoba-lang/io-ipld-car`, `kotoba-lang/kotoba-kir`; audit-ledger binding in `kotoba-lang/robotics` and `kotoba-lang/capability-ledger-append`; timestamping in `kotoba-lang/org-ietf-rfc3161` and `kotoba-lang/org-ietf-ers`.

---

## 11. Additional disclosed subject matter, in brief

The following are disclosed on the same terms; each is implemented in the referenced public repositories.

- **A robotics middleware wire codec as pure data.** An alignment-aware Common Data Representation writer and reader, standard message shapes, quality-of-service profiles, and a JSON-shaped bridge protocol, implemented as pure functions with no sockets, so that middleware messages can be constructed, verified, recorded, and replayed in environments where the middleware cannot be installed — including inside a browser, a verifier, or a governor. Variants: any middleware and any serialization; any transport supplied by the caller; the codec used as a conformance oracle against a reference implementation. — `kotoba-lang/org-ros`.
- **Sensor synthesis as pure functions over an analytic scene.** Inertial, contact, depth-camera, and rotating-lidar readings produced from primitive intersections (plane, sphere, box) with stated sensor intrinsics, giving deterministic, dependency-free synthetic sensing usable as an oracle for a perception pipeline and as the sensing side of the governor's predictive check. — `kotoba-lang/kami-sensor-sim`.
- **Fab and process automation under the same governed-actor arrangement.** Wafer handling, plant access, and factory cell control expressed as proposals admitted by an independent governor with the audit binding of Section 1. — `kotoba-lang/com-semicon-robotics`, `kotoba-lang/kami-app-hygaccess-plant`, `kotoba-lang/kami-app-giemon-factory`, `kotoba-lang/kami-app-sarutahiko-factory`, `kotoba-lang/giemon`, `kotoba-lang/kami-app-giemon`.
- **Collaborative-robot assembly records with force profiles and safety zones as first-class governed data**, including end-effector and joint records, assembly tasks, safety zones, and force profiles. — `kotoba-lang/com-cobot-assembly`.
- **A fieldbus request/response codec as pure data**, on the same terms as the middleware codec above. — `kotoba-lang/com-modbus-tcp`.
- **Vendor-facade layers with a uniform record and query surface** over heterogeneous robot vendors, so that a fleet spanning vendors is queried and audited through one representation. — `kotoba-lang/com-abb-robotics`, `kotoba-lang/com-universal-robots`, `kotoba-lang/com-ros-robotics`, `kotoba-lang/com-ros2-nav`, `kotoba-lang/com-bluefin-robotics`, `kotoba-lang/com-liquid-robotics`.

---

## 中文摘要 / Chinese abstract

本文件为**防御性公开**（defensive publication）。作者自愿将以下技术方案公开，使其成为**现有技术**（专利法第二十二条所称"为公众所知的技术"），并声明**不为其中任何技术方案申请专利**。全部参考实现的源代码自公开之日起可在互联网上公开获取，采用 Apache-2.0 许可，具体仓库地址与提交哈希见本包中的 `evidence-manifest.txt`。

公开的技术方案包括：

1. **由独立监管器（governor）把关的机器人执行动作**：行为体（可为神经网络策略、大语言模型、规划器或人类遥操作者）只能**提议**动作，不具有到达执行机构的通路；一个独立编写、可独立审查的监管器依据动作的**安全等级**准许或拒绝该提议。任务（mission）为一次有界操作，具有明确的步数上限与边界（地理围栏、工作空间、时间），内部不循环。安全停止（safety-stop）与**遥测证明**（将传感观测绑定到任务并写入只可追加的审计账本）均为一等数据记录。变型包括：安全等级可为有序枚举、数值风险分、格结构或多轴向量；监管器可为程序、规则集、经形式化验证的模块、硬件联锁、独立进程或独立机器、或由两个以上独立监管器组成的法定人数；准许可为逐动作、按批、分阶段、可撤销、有时限或有配额，并在委派时**递减**权限。

2. **将执行机构操作表示为带类型的能力（capability），使未被授予的操作无法被表达**：程序所声明的副作用集合由编译器从程序体**推断**，声明与推断不一致即拒绝编译；装载时若宿主提供的授权与制品所需能力集合不一致，则**实例化失败**，程序根本不会开始执行。语言中不存在任意模块装载、`eval`、反射、外部函数接口、环境全局可变状态，也不存在对设备、套接字、文件系统或凭据的直接访问；错误以值（result 类型）返回而非抛出；策略为**默认拒绝**；宿主强制执行燃料（步数）、内存、队列深度与消息大小上限。因此一个仅被授予"读取激光雷达"和"以不超过 0.4 m/s 发布速度指令"的策略，**无法表达**给夹持器通电这一操作。

3. **在任何飞行器起飞之前对整场多智能体编队进行完整验证**：整场演出预先计算为数据，验证器对整条时间线采样并检验成对**间距**（含足迹半径）、**速度**、**加速度**与**电子围栏**约束，垂直与水平分量分别检验，返回**违规集合**而非单一布尔值；只有全部通过后才下发为各机的位置设定点消息，并保留全体中止（abort-all）原语。变型包括区间算术或平方和证明代替采样、扫掠体保守求交、单机失效与失联的余度约束、异构机群的分类足迹与限值。

4. **由通用输入设备导出的持续握持（deadman）与组合键紧急停止**：由设备状态到指令为纯函数；指定控件必须**持续握持**才产生非零运动指令，输入报文丢失与松手不可区分且导向停止；指定的**组合键**（普通握持与单点卡滞均无法产生）生成安全停止记录并终止任务。变型包括心跳消息速率不足即视为松手、按时延与人体接近度缩放指令。

5. **由运动学描述导出执行器物料清单，并以连续转矩校验作为设计闸门**：执行器规格作为关节自身的字段，机械地导出链式物料清单（含默认值与**具名实现变型**覆盖）与逐关节连续转矩校验；同时校验关节图为良构树、关节限值齐备有序、惯性张量物理可行（对称、正定、满足主惯量三角不等式）。变型包括 RMS 转矩、热升、制动保持力、轴承寿命、结构变形与一阶固有频率等被校验量，以及由优化器在校验通过的约束下选型。

6. **一个确定性、无依赖的动力学内核同时服务于仿真、训练与控制**：基于空间向量（Plücker）代数实现正运动学、几何雅可比、递归牛顿-欧拉逆动力学、复合刚体质量矩阵、正动力学与重力补偿；GJK 距离与 EPA 穿透、有向包围盒分离轴与流形生成、保守推进碰撞时刻；冲量法接触求解含有效质量、切向摩擦、热启动与 Baumgarte 稳定化，关节限位复用同一机制；阻尼最小二乘位置与位姿逆运动学；线性化后解离散代数 Riccati 方程得到 LQR；三次、五次、最小抖动与航路点轨迹生成；带逐环境配置的**向量化批处理**步进。该内核为纯函数且无依赖，可编译到原生、WebAssembly 与参考解释器等多目标并以**一致性校验**逐字节比对，因此可嵌入监管器中，在准许某动作之前用与仿真相同的物理**预测其后果**。

7. **域随机化表示为数据而非代码，从而构造性可复现**：逐物理参数的范围或分布写在声明式文档中，以明确算法与明确种子的生成器为每个环境实例抽样，使整批环境为（文档，种子）的纯函数，训练运行可精确重放，随机化的变更是文档的差异而非程序的差异。变型包括计数器型生成器由（种子，环境序号，参数序号）导出独立流，以及将上述内容的摘要绑定进训练收据。

8. **异构测距源经单一接入接口融合为同一占据表示**：旋转激光雷达环扫与带内参外参的深度图像都归约为同一栅格中的世界点；栅格统一支持按足迹半径膨胀、代价图转换、视线通畅性、最近自由格搜索，以及**分别**由雷达证据与相机证据回答的前向净空查询，使模态间的分歧可见而非被平均掉。规划仅消费栅格（A\* 加路径简化与平滑），跟踪仅消费路径（纯追踪加基于 Menger 曲率的限速与横向误差项）；**卡滞检测器**在指令未产生实测进展时转入既定恢复行为。

9. **一个制导-导航-控制步进函数由载具物理参数化，跨越介质**：目标处理、规划、跟踪、净空检查、卡滞检测、到达判定与遥测为同一份代码，仅以载具模型（地面自行车模型、船舶水动力、固定翼失速与坡度限转弯、多旋翼推重比与倾角限加速度）作为数据。因此异构机群可由一份实现指挥、验证与审计，新增载具类别只需提供模型。

10. **机器人执行的内容寻址身份**：以 `{程序，输入，初始状态，运行时身份，生效策略，副作用集合}` 规范序列化后的密码学摘要标识一次执行；程序、策略、能力授权、模型权重、场景描述与随机化文档各自内容寻址，故执行标识传递性地固定全部决定因素；所执行的副作用记为哈希链式有序日志，其摘要为执行记录的一部分；**收据**将执行标识绑定到结果、验证结论与审计账本条目；因果关系记为每主体的签名有向无环图与逻辑时钟，而非压平为单一全局序列；**身份**（摘要）与**位置**（字节存放处）及**命名**（可变指针）三者分离。执行标识仅在副作用集合为空或副作用日志完整时可作为记忆化键，否则它是收据而非缓存键。

11. 另有以下方案一并公开：作为纯数据的机器人中间件线格式编解码器（对齐感知的 CDR、标准消息、QoS、桥接协议，无套接字）；作为纯函数的传感器合成（惯性、接触、深度相机、旋转雷达）；同一受监管行为体安排下的晶圆搬运与厂房门禁自动化；含力曲线与安全区的协作机器人装配记录；现场总线编解码器；跨厂商机器人的统一记录与查询面。

以上各节均以"问题—机制—变型"三段书写，其中**变型系有意穷举列举**，以使针对上述机制之显而易见变化的在后申请面对的是明确的书面公开，而非仅仅是"显而易见"的主张。

---

## 日本語要旨

本書は**防御的公開（defensive publication）**である。以下の技術思想を公知にして先行技術とすることのみを目的として公開し、著者はここに記載のいかなる技術思想についても特許を取得しない。参考実装のソースコードは公開日以降インターネット上で公開されており、ライセンスは Apache-2.0、リポジトリと commit ハッシュは本バンドルの `evidence-manifest.txt` に固定してある。

公開する技術思想は、(1) 独立したガバナが admit する robot 動作 — actor は提案しかできず、アクチュエータへの経路を持たない、(2) アクチュエーションを型付き capability として表し、付与されていない動作を**そもそも記述できなくする**方式（宣言と推論の不一致でコンパイル拒否、grant 不一致で instantiate 自体が失敗、deny-by-default、fuel/memory/queue 上限）、(3) 1 機も飛ばす前に全編隊を検証する事前計算コレオグラフィ（間隔・速度・加速度・ジオフェンスを全時間軸で、違反集合を返す）、(4) 汎用入力デバイスから導く deadman と chord 型 e-stop、(5) 運動学記述からアクチュエータ BOM を導出し連続トルク検証を設計ゲートにする方式、(6) シミュレーション・学習・制御を 1 本で兼ねる決定論的・無依存の動力学カーネル（空間ベクトル代数、RNEA/CRBA、GJK/EPA、DLS-IK、LQR、軌道生成、ベクトル化バッチ、多ターゲット parity）、(7) domain randomization をコードでなくデータで表し構成的に再現可能にする方式、(8) 異種測距源を単一 ingest 面から同一 occupancy 表現へ融合し、モダリティ間の不一致を平均せず可視化する方式、(9) 載具物理をデータとして受け取る単一の GNC step 関数（地上・船舶・固定翼・マルチロータ）、(10) 実行の content-addressed identity（`{program, input, state, runtime, policy, effects}` の digest、effect log、receipt、identity と location と naming の分離）、および (11) ミドルウェア wire codec・センサ合成・fab 自動化・協働ロボット組立記録・fieldbus codec・ベンダ facade。

各節は「課題 — 機構 — **変形例**」の形で書かれており、変形例の列挙は意図的に広い。後願が上記機構の自明な変形に向けられた場合に、それが単なる「自明」の主張ではなく**明示の書面開示**に当たるようにするためである。

---

## Statement of dedication

The author does not seek and will not seek patent protection for any subject matter disclosed in this document. This document and the referenced source code are published to establish the subject matter as prior art available to the public as of the publication date stated above. The source code remains licensed under the Apache License 2.0, whose Section 3 grants an express patent licence from each contributor to every recipient.

Nothing in this document is an admission that any third party holds, or does not hold, rights in any subject matter described. Sections describing clean-room, API-surface-compatible implementations of third-party products are disclosures of the author's own implementation only.
