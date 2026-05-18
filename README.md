# masstech-notices-evidence

Append-only public chain-of-custody record for public meeting notices posted
by the **Massachusetts Technology Collaborative** (MassTech), a body politic
created by G.L. c. 40J subject to the Massachusetts Open Meeting Law,
G.L. c. 30A §§ 18–25.

This repository is the public, externally-witnessed record of *what MassTech
posted, and when*. It exists because MassTech has — across multiple
documented incidents — silently revised meeting notices after first posting
and, in at least one instance, inside the 48-hour pre-meeting window
protected by G.L. c. 30A § 20(b).

Each commit records one observation cycle. No commit is ever rewritten or
amended. The repository is force-push-disabled. The git history is itself
part of the evidentiary record.

## What is captured

For every observation, the watcher records:

| Field | Source | What it proves |
|---|---|---|
| Raw HTML of the public-notices index | `masstech.org/news-and-updates/public-notices` (and year-archive pages) | Verbatim content of the listing page at observation time |
| Raw PDF of each notice attachment | URLs referenced from the index | Verbatim binary of the agenda document at observation time |
| SHA-256 of every captured artifact | computed locally | Tamper-evidence: any later modification produces a different hash |
| `captured_at_utc` (ISO 8601) | system clock at fetch time | Local timestamp of observation |
| OpenTimestamps proof (`.ots`) | `opentimestamps-client` | Anchors the hash to the Bitcoin blockchain — defeats backdating of either this repo or any artifact |
| RFC 3161 token (`.tsr`) | free public TSA (Sectigo / DigiCert) | Third-party Certificate Authority signed timestamp, immediately verifiable |
| Wayback Machine snapshot URL | `web.archive.org/save/<url>` | Independent third-party archive of the source URL at fetch time |
| GitHub commit SHA | this repository's history | Publicly-witnessed timestamp recorded by GitHub's server clock |

## Repository layout

```
artifacts/<meeting_key>/<observed_at>/<sha256>.<html|pdf>
manifests/<YYYY>/<MM>/<DD>/<YYYYMMDD>.ndjson
state.json
```

- `artifacts/` — every captured page and PDF, content-addressed by SHA-256.
- `manifests/` — newline-delimited JSON; one line per observation. Append-only.
- `state.json` — the current `(notice_key → current_hash)` map. Mutable. Not evidence.

## How to independently verify

1. Pick any artifact path (e.g. `artifacts/2026-05-20_board/2026-05-19T03-22-11Z/<sha256>.pdf`).
2. `sha256sum` the file; confirm it matches the filename and the manifest line.
3. Verify the OpenTimestamps proof against the Bitcoin blockchain:
   `ots verify artifacts/.../<sha256>.pdf.ots`
4. Verify the RFC 3161 token against the relevant TSA's certificate chain:
   `openssl ts -verify -data <file> -in <file>.tsr -CAfile <tsa-chain.pem>`
5. Open the Wayback Machine snapshot URL recorded in the manifest line.
6. Check the GitHub commit that introduced the artifact — its commit
   timestamp and SHA are server-recorded by GitHub.

Any one of those proofs is meaningful on its own. To dispute a captured
observation in good faith, an opposing party would need to argue that the
SHA-256 collided, that Bitcoin's proof-of-work was forged, that a public
Certificate Authority backdated a token, that the Internet Archive falsified
a capture, and that GitHub backdated its commit log — all at once.

## What is *not* in this repository

- The watcher source code (which is in
  [`coffeethencode/sor-database`](https://github.com/coffeethencode/sor-database)
  under `masstech/`).
- Any credential, deploy key, or API token. The watcher pushes to this
  repository over an ed25519 deploy key restricted to push-only access on
  this single repository.

## Bug reports and verification problems

Open an issue. If you cannot independently verify a captured observation
using the steps above, that is itself something we want to know about.

## License

MIT. The contents — page captures of public records produced by a
Massachusetts public body — are public records.
