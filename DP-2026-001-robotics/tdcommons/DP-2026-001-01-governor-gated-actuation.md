# Actuation gated by an independent governor, with the proposal and the permission held by separate parties

> **Defensive publication.** This document is published solely to place its
> subject matter into the public domain of technical knowledge as prior art.
> **No patent protection is sought by the author for any subject matter described
> here.**
>
> **Author / declarant:** Jun Kawasaki (`root@junkawasaki.com`)
> **Disclosure set:** DP-2026-001, part 1 of 10 · first published 2026-08-19 (UTC)
> **Enabling reference implementation:** public source code under
> `https://github.com/kotoba-lang/`, licensed Apache-2.0, fixed by commit and tree
> SHA-1 in the evidence bundle at
> `https://github.com/com-junkawasaki/prior-art` — that bundle carries RFC 3161
> time-stamp tokens and OpenTimestamps attestations over the fixing manifest.

## Note on the "Variants and generalizations" section below

The variants are stated deliberately and at length. They are disclosed so that a
later filing directed to an obvious variation of the mechanism is met by an
express **written** disclosure of that variation, and not merely by an argument
that the variation would have been obvious. Where the text says "any", the word
is intended in its ordinary broad sense and the enumeration following it is
illustrative, not exhaustive.

Working source code implementing the mechanism is publicly available at the
commits fixed in the evidence bundle. A person of ordinary skill in robotics
software can read, build and run that code; this document is therefore an
enabling disclosure and not a mere statement of a result.

---


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

---

## 中文摘要 / Chinese abstract

本文件为**防御性公开**（defensive publication）。作者自愿公开以下技术方案，使其成为**现有技术**（专利法第二十二条所称"为公众所知的技术"），并声明**不为其中任何技术方案申请专利**。参考实现的源代码自 2026 年 8 月 19 日起在互联网上公开获取，采用 Apache-2.0 许可，仓库地址与提交哈希见 `https://github.com/com-junkawasaki/prior-art`。

**由独立监管器（governor）把关的机器人执行动作**：行为体（可为神经网络策略、大语言模型、规划器或人类遥操作者）只能**提议**动作，不具有到达执行机构的通路；一个独立编写、可独立审查的监管器依据动作的**安全等级**准许或拒绝该提议。任务（mission）为一次有界操作，具有明确的步数上限与边界（地理围栏、工作空间、时间），内部不循环。安全停止（safety-stop）与**遥测证明**（将传感观测绑定到任务并写入只可追加的审计账本）均为一等数据记录。变型包括：安全等级可为有序枚举、数值风险分、格结构或多轴向量；监管器可为程序、规则集、经形式化验证的模块、硬件联锁、独立进程或独立机器、或由两个以上独立监管器组成的法定人数；准许可为逐动作、按批、分阶段、可撤销、有时限或有配额，并在委派时**递减**权限。

上文"变型与推广"一节系**有意穷举列举**，以使针对本机制之显而易见变化的在后申请面对的是明确的书面公开，而非仅仅是"显而易见"的主张。

---

## Statement of dedication

The author does not seek and will not seek patent protection for any subject
matter disclosed in this document. This document and the referenced source code
are published to establish the subject matter as prior art available to the
public as of the publication date stated above. The source code remains licensed
under the Apache License 2.0, whose Section 3 grants an express patent licence
from each contributor to every recipient.

Nothing in this document is an admission that any third party holds, or does not
hold, rights in any subject matter described. Where a referenced repository is a
clean-room, API-surface-compatible implementation of a third-party product, the
disclosure is of the author's own implementation only.
