# prior-art

**Defensive publications.** Each directory is one dated disclosure, published so that
the subject matter is prior art available to the public from the stated date. No patent
protection is sought for any subject matter disclosed here.

| ID | Subject | Date (UTC) | Repos fixed |
|---|---|---|---|
| [DP-2026-001](DP-2026-001-robotics/) | Governed robotics — capability-gated actuation, verified multi-agent choreography, deterministic portable dynamics | 2026-08-19 | 31 |

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

## Licence

The disclosure documents and evidence bundles in this repository are dedicated to the public
domain under [CC0 1.0](LICENSE). The source code they reference is separately licensed under
Apache-2.0 in its own repositories.
