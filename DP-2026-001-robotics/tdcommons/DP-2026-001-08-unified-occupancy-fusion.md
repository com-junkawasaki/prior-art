# One occupancy representation fused from heterogeneous range sources through a single ingest interface

> **Defensive publication.** This document is published solely to place its
> subject matter into the public domain of technical knowledge as prior art.
> **No patent protection is sought by the author for any subject matter described
> here.**
>
> **Author / declarant:** Jun Kawasaki (`root@junkawasaki.com`)
> **Disclosure set:** DP-2026-001, part 8 of 10 · first published 2026-08-19 (UTC)
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

---

## 中文摘要 / Chinese abstract

本文件为**防御性公开**（defensive publication）。作者自愿公开以下技术方案，使其成为**现有技术**（专利法第二十二条所称"为公众所知的技术"），并声明**不为其中任何技术方案申请专利**。参考实现的源代码自 2026 年 8 月 19 日起在互联网上公开获取，采用 Apache-2.0 许可，仓库地址与提交哈希见 `https://github.com/com-junkawasaki/prior-art`。

**异构测距源经单一接入接口融合为同一占据表示**：旋转激光雷达环扫与带内参外参的深度图像都归约为同一栅格中的世界点；栅格统一支持按足迹半径膨胀、代价图转换、视线通畅性、最近自由格搜索，以及**分别**由雷达证据与相机证据回答的前向净空查询，使模态间的分歧可见而非被平均掉。规划仅消费栅格（A\* 加路径简化与平滑），跟踪仅消费路径（纯追踪加基于 Menger 曲率的限速与横向误差项）；**卡滞检测器**在指令未产生实测进展时转入既定恢复行为。

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
