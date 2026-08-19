# Domain randomization expressed as data, with reproducibility by construction

> **Defensive publication.** This document is published solely to place its
> subject matter into the public domain of technical knowledge as prior art.
> **No patent protection is sought by the author for any subject matter described
> here.**
>
> **Author / declarant:** Jun Kawasaki (`root@junkawasaki.com`)
> **Disclosure set:** DP-2026-001, part 7 of 10 · first published 2026-08-19 (UTC)
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

---

## 中文摘要 / Chinese abstract

本文件为**防御性公开**（defensive publication）。作者自愿公开以下技术方案，使其成为**现有技术**（专利法第二十二条所称"为公众所知的技术"），并声明**不为其中任何技术方案申请专利**。参考实现的源代码自 2026 年 8 月 19 日起在互联网上公开获取，采用 Apache-2.0 许可，仓库地址与提交哈希见 `https://github.com/com-junkawasaki/prior-art`。

**域随机化表示为数据而非代码，从而构造性可复现**：逐物理参数的范围或分布写在声明式文档中，以明确算法与明确种子的生成器为每个环境实例抽样，使整批环境为（文档，种子）的纯函数，训练运行可精确重放，随机化的变更是文档的差异而非程序的差异。变型包括计数器型生成器由（种子，环境序号，参数序号）导出独立流，以及将上述内容的摘要绑定进训练收据。

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
