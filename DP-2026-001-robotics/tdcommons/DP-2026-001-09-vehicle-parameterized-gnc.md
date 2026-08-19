# One guidance-navigation-control step function parameterized by vehicle physics across media

> **Defensive publication.** This document is published solely to place its
> subject matter into the public domain of technical knowledge as prior art.
> **No patent protection is sought by the author for any subject matter described
> here.**
>
> **Author / declarant:** Jun Kawasaki (`root@junkawasaki.com`)
> **Disclosure set:** DP-2026-001, part 9 of 10 · first published 2026-08-19 (UTC)
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

---

## 中文摘要 / Chinese abstract

本文件为**防御性公开**（defensive publication）。作者自愿公开以下技术方案，使其成为**现有技术**（专利法第二十二条所称"为公众所知的技术"），并声明**不为其中任何技术方案申请专利**。参考实现的源代码自 2026 年 8 月 19 日起在互联网上公开获取，采用 Apache-2.0 许可，仓库地址与提交哈希见 `https://github.com/com-junkawasaki/prior-art`。

**一个制导-导航-控制步进函数由载具物理参数化，跨越介质**：目标处理、规划、跟踪、净空检查、卡滞检测、到达判定与遥测为同一份代码，仅以载具模型（地面自行车模型、船舶水动力、固定翼失速与坡度限转弯、多旋翼推重比与倾角限加速度）作为数据。因此异构机群可由一份实现指挥、验证与审计，新增载具类别只需提供模型。

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
