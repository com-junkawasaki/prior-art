# prior-art

**Published site: <https://com-junkawasaki.github.io/prior-art/>** — one indexable page per
record, with Dublin Core and Google Scholar `citation_*` metadata, a bilingual index, and a
sitemap. The site exists because a GitHub blob view of a `.md` file is close to invisible to
the indexes that matter: Scholar indexes on `citation_*`, and Scholar is what feeds Google
Patents' non-patent-literature corpus — the corpus a patent examiner actually searches.

**Defensive publications.** Each directory is one dated disclosure, published so that
the subject matter is prior art available to the public from the stated date. No patent
protection is sought for any subject matter disclosed here.

| ID | Subject | Date (UTC) | Repos fixed |
|---|---|---|---|
| [DP-2026-001](DP-2026-001-robotics/) | Governed robotics — capability-gated actuation, verified multi-agent choreography, deterministic portable dynamics | 2026-08-19 | 31 |

DP-2026-001 is additionally split into **ten individually-indexable records** under
[`DP-2026-001-robotics/tdcommons/`](DP-2026-001-robotics/tdcommons/), each self-contained,
each with an English body and a Chinese abstract, as PDF and Markdown. See
[`SUBMISSION-SHEET.md`](DP-2026-001-robotics/tdcommons/SUBMISSION-SHEET.md).
The split exists because a prior-art search hits a record's **title and abstract** — ten
focused records are found where one omnibus document is not.

## What each disclosure contains

- `DISCLOSURE-*.md` — the disclosure itself: problem, mechanism, and a deliberately broad
  enumeration of variants, per numbered section. English body, with a full Chinese abstract
  (中文摘要) and a Japanese summary (日本語要旨).
- `evidence-manifest.txt` — every referenced repository, fixed by its remote default-branch
  commit SHA-1 and tree SHA-1 as measured through the GitHub API. A git commit SHA-1 commits
  the entire tree by Merkle construction, so this one small file fixes the complete content
  of all listed repositories.
- `*.tsr` / `*.tsq` — [RFC 3161](https://www.rfc-editor.org/rfc/rfc3161.html) time-stamp
  tokens over those files.
- `*.ots` — [OpenTimestamps](https://opentimestamps.org/) attestations, anchored in the
  Bitcoin blockchain.
- `software-heritage-save-requests.json` — archival requests lodged with
  [Software Heritage](https://archive.softwareheritage.org/).
- `VERIFY.md` — how to check all of the above yourself.

## Honest limits of this evidence

**The RFC 3161 tokens here are from FreeTSA, which is not an accredited timestamp authority
in any jurisdiction.** They are structurally valid and independently verifiable, and they are
better than a git commit date, which is self-asserted and trivially forgeable. They are *not*
equivalent to a token from Japan's 総務大臣認定タイムスタンプ, an eIDAS qualified timestamp,
or China's 联合信任时间戳服务中心 (UniTrust). Where a filing or a proceeding requires an
accredited timestamp, one must be obtained separately over the same digests — the digests in
this bundle are what an accredited token would cover, so a later accredited token strengthens
this bundle without replacing anything in it.

`kotoba-lang/org-ietf-rfc3161` makes exactly this distinction in code: `:verified` (the token
is structurally sound and its signature checks) and `:trusted` (someone vouched for the signing
certificate) are separate fields, and with no trust predicate supplied the answer is
`:unknown`, never `true`.

## 中文说明

本仓库收录**防御性公开**（defensive publication）文件。公开的唯一目的是使其中的技术方案
自公开之日起成为**现有技术**（中国专利法第二十二条所称"为公众所知的技术"），从而阻止他人
就相同或显而易见变形的技术方案取得在后专利。**作者声明不为其中任何技术方案申请专利。**

每份公开文件均按"课题 — 机构 — **变型与推广**"三段撰写，其中变型系有意穷举列举，以使针对
上述机制之显而易见变化的在后申请面对的是明确的**书面公开**，而非仅仅是"显而易见"的主张。

证据部分包含：以提交哈希（commit / tree SHA-1）固定的全部参考实现仓库清单、
[RFC 3161](https://www.rfc-editor.org/rfc/rfc3161.html) 时间戳令牌、以及锚定于比特币区块链的
[OpenTimestamps](https://opentimestamps.org/) 存证。**请注意：本仓库所附 RFC 3161 令牌来自
FreeTSA，并非任何法域的认定时间戳服务机构** —— 其可被独立验证，但不等同于中国联合信任时间戳
服务中心、日本总务大臣认定时间戳或 eIDAS 合格时间戳。如需具备上述效力的时间戳，可就本仓库
已固定的同一摘要另行申请，届时为增强而非替换。

`DP-2026-001-robotics/VERIFY.md` 载明任何人自行验证上述全部内容的方法。

## Licence

The disclosure documents and evidence bundles in this repository are dedicated to the public
domain under [CC0 1.0](LICENSE). The source code they reference is separately licensed under
Apache-2.0 in its own repositories.
