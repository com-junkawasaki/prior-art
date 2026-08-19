# Content-addressed identity for a robot execution, so that what ran is recoverable by digest

> **Defensive publication.** This document is published solely to place its
> subject matter into the public domain of technical knowledge as prior art.
> **No patent protection is sought by the author for any subject matter described
> here.**
>
> **Author / declarant:** Jun Kawasaki (`root@junkawasaki.com`)
> **Disclosure set:** DP-2026-001, part 10 of 10 · first published 2026-08-19 (UTC)
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

---

## 中文摘要 / Chinese abstract

本文件为**防御性公开**（defensive publication）。作者自愿公开以下技术方案，使其成为**现有技术**（专利法第二十二条所称"为公众所知的技术"），并声明**不为其中任何技术方案申请专利**。参考实现的源代码自 2026 年 8 月 19 日起在互联网上公开获取，采用 Apache-2.0 许可，仓库地址与提交哈希见 `https://github.com/com-junkawasaki/prior-art`。

**机器人执行的内容寻址身份**：以 `{程序，输入，初始状态，运行时身份，生效策略，副作用集合}` 规范序列化后的密码学摘要标识一次执行；程序、策略、能力授权、模型权重、场景描述与随机化文档各自内容寻址，故执行标识传递性地固定全部决定因素；所执行的副作用记为哈希链式有序日志，其摘要为执行记录的一部分；**收据**将执行标识绑定到结果、验证结论与审计账本条目；因果关系记为每主体的签名有向无环图与逻辑时钟，而非压平为单一全局序列；**身份**（摘要）与**位置**（字节存放处）及**命名**（可变指针）三者分离。执行标识仅在副作用集合为空或副作用日志完整时可作为记忆化键，否则它是收据而非缓存键。

11. 另有以下方案一并公开：作为纯数据的机器人中间件线格式编解码器（对齐感知的 CDR、标准消息、QoS、桥接协议，无套接字）；作为纯函数的传感器合成（惯性、接触、深度相机、旋转雷达）；同一受监管行为体安排下的晶圆搬运与厂房门禁自动化；含力曲线与安全区的协作机器人装配记录；现场总线编解码器；跨厂商机器人的统一记录与查询面。

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
