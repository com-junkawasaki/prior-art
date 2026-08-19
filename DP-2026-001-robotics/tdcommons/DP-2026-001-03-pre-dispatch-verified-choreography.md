# Multi-agent choreography verified in full before any vehicle is dispatched

> **Defensive publication.** This document is published solely to place its
> subject matter into the public domain of technical knowledge as prior art.
> **No patent protection is sought by the author for any subject matter described
> here.**
>
> **Author / declarant:** Jun Kawasaki (`root@junkawasaki.com`)
> **Disclosure set:** DP-2026-001, part 3 of 10 · first published 2026-08-19 (UTC)
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

---

## 中文摘要 / Chinese abstract

本文件为**防御性公开**（defensive publication）。作者自愿公开以下技术方案，使其成为**现有技术**（专利法第二十二条所称"为公众所知的技术"），并声明**不为其中任何技术方案申请专利**。参考实现的源代码自 2026 年 8 月 19 日起在互联网上公开获取，采用 Apache-2.0 许可，仓库地址与提交哈希见 `https://github.com/com-junkawasaki/prior-art`。

**在任何飞行器起飞之前对整场多智能体编队进行完整验证**：整场演出预先计算为数据，验证器对整条时间线采样并检验成对**间距**（含足迹半径）、**速度**、**加速度**与**电子围栏**约束，垂直与水平分量分别检验，返回**违规集合**而非单一布尔值；只有全部通过后才下发为各机的位置设定点消息，并保留全体中止（abort-all）原语。变型包括区间算术或平方和证明代替采样、扫掠体保守求交、单机失效与失联的余度约束、异构机群的分类足迹与限值。

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
