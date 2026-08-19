# Deadman and chord-based emergency stop derived from a general-purpose input device

> **Defensive publication.** This document is published solely to place its
> subject matter into the public domain of technical knowledge as prior art.
> **No patent protection is sought by the author for any subject matter described
> here.**
>
> **Author / declarant:** Jun Kawasaki (`root@junkawasaki.com`)
> **Disclosure set:** DP-2026-001, part 4 of 10 · first published 2026-08-19 (UTC)
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

---

## 中文摘要 / Chinese abstract

本文件为**防御性公开**（defensive publication）。作者自愿公开以下技术方案，使其成为**现有技术**（专利法第二十二条所称"为公众所知的技术"），并声明**不为其中任何技术方案申请专利**。参考实现的源代码自 2026 年 8 月 19 日起在互联网上公开获取，采用 Apache-2.0 许可，仓库地址与提交哈希见 `https://github.com/com-junkawasaki/prior-art`。

**由通用输入设备导出的持续握持（deadman）与组合键紧急停止**：由设备状态到指令为纯函数；指定控件必须**持续握持**才产生非零运动指令，输入报文丢失与松手不可区分且导向停止；指定的**组合键**（普通握持与单点卡滞均无法产生）生成安全停止记录并终止任务。变型包括心跳消息速率不足即视为松手、按时延与人体接近度缩放指令。

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
