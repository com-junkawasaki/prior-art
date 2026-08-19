# How to verify this bundle

Nothing here asks you to trust the author. Every claim below is checkable with
`openssl`, `git`, and `curl`.

## 1. The files are the ones that were timestamped

```sh
sha256sum DISCLOSURE-001-robotics.md evidence-manifest.txt
# compare against SHA256SUMS in this directory
```

## 2. The RFC 3161 tokens are valid over those files

```sh
openssl ts -verify -data evidence-manifest.txt \
  -in evidence-manifest.txt.tsr \
  -CAfile tsa/freetsa-cacert.pem -untrusted tsa/freetsa-tsa.crt
# => Verification: OK

openssl ts -reply -in evidence-manifest.txt.tsr -text | grep -E 'Time stamp|Hash|Serial'
```

Same for `DISCLOSURE-001-robotics.md.tsr`.

**Read the "Honest limits" section of the parent README before relying on these.** FreeTSA
is not an accredited authority. The token proves the digest existed before the stated instant
*if* you accept FreeTSA's key; it is verifiable, not accredited.

## 3. The OpenTimestamps attestations are anchored in Bitcoin

```sh
pip install opentimestamps-client
ots verify evidence-manifest.txt.ots            # needs the data file alongside
ots info    evidence-manifest.txt.ots
```

A freshly created `.ots` is a *pending* attestation against the calendar servers; it becomes a
Bitcoin-anchored proof once the calendars include it in a block, typically within a few hours.
Run `ots upgrade <file>.ots` to fetch the completed proof, then `ots verify` reports the Bitcoin
block height and time. Chinese internet courts have accepted blockchain-anchored evidence
(最高人民法院 online-litigation rules); this is that form of evidence.

## 4. The repositories really were at those commits

For any row of `evidence-manifest.txt`:

```sh
REPO=robotics
SHA=$(awk -F'\t' -v r="$REPO" '$1==r{print $4}' evidence-manifest.txt)
git clone --no-checkout https://github.com/kotoba-lang/$REPO /tmp/$REPO
git -C /tmp/$REPO cat-file -t $SHA          # => commit
git -C /tmp/$REPO log --format=%T -1 $SHA   # => must equal column 5 (tree_sha1)
```

The tree SHA-1 commits every byte of every file in the repository at that commit, recursively.
Matching it means the content you are reading is the content that was timestamped.

## 5. The archival requests were lodged

```sh
jq -r '.[] | "\(.repo)\t\(.status)\t\(.id)"' software-heritage-save-requests.json
curl -s https://archive.softwareheritage.org/api/1/origin/save/2438360/ | jq .
```

Requests recorded as `HTTP429` were refused by an anonymous rate limit at submission time and
were re-lodged later; check the live Software Heritage API for the current archival status of
any origin:

```sh
curl -s 'https://archive.softwareheritage.org/api/1/origin/https://github.com/kotoba-lang/robotics/visits/' | jq '.[0]'
```

## 6. What none of this proves

- It does not prove the *content* is novel, useful, or correct. It proves it was public on a date.
- It does not prove the repositories were public *before* the measurement date, only that they
  were public at it and that the commits carry earlier self-asserted dates. The dates in
  `repo_created_utc` and `commit_date_utc` come from GitHub and from git respectively and are
  **not** independently attested by the tokens in this bundle.
- It says nothing about whether this subject matter infringes anyone else's rights.
