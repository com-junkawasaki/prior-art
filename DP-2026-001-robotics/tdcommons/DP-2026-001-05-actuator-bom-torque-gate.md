# Actuator bill-of-materials derived from the kinematic description, with continuous-torque validation as a design gate

> **Defensive publication.** This document is published solely to place its
> subject matter into the public domain of technical knowledge as prior art.
> **No patent protection is sought by the author for any subject matter described
> here.**
>
> **Author / declarant:** Jun Kawasaki (`root@junkawasaki.com`)
> **Disclosure set:** DP-2026-001, part 5 of 10 · first published 2026-08-19 (UTC)
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

---

## 中文摘要 / Chinese abstract

本文件为**防御性公开**（defensive publication）。作者自愿公开以下技术方案，使其成为**现有技术**（专利法第二十二条所称"为公众所知的技术"），并声明**不为其中任何技术方案申请专利**。参考实现的源代码自 2026 年 8 月 19 日起在互联网上公开获取，采用 Apache-2.0 许可，仓库地址与提交哈希见 `https://github.com/com-junkawasaki/prior-art`。

**由运动学描述导出执行器物料清单，并以连续转矩校验作为设计闸门**：执行器规格作为关节自身的字段，机械地导出链式物料清单（含默认值与**具名实现变型**覆盖）与逐关节连续转矩校验；同时校验关节图为良构树、关节限值齐备有序、惯性张量物理可行（对称、正定、满足主惯量三角不等式）。变型包括 RMS 转矩、热升、制动保持力、轴承寿命、结构变形与一阶固有频率等被校验量，以及由优化器在校验通过的约束下选型。

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
