# ProofStamp via Bitcoin / OpenTimestamps

Work in progress.

This repository is being prepared for a ProofStamp product that creates portable Bitcoin-backed timestamp evidence using the OpenTimestamps protocol.

The intended boundary is:

**ProofStamp evidence format and UX → OpenTimestamps proof → Bitcoin**

ProofStamp should not become a proprietary timestamp authority. The target design keeps source files local, requires no account or wallet for normal use, and preserves standard OpenTimestamps interoperability so a completed proof can be verified outside ProofStamp.

## Architecture work first

The current branch plan deliberately separates architecture review from implementation.

Read these documents before adding OpenTimestamps code:

- [OpenTimestamps v0 implementation plan](docs/opentimestamps-v0-plan.md)
- [ProofStamp Manifest v1 draft](docs/proofstamp-manifest-v1.md)
- [OpenTimestamps / Bitcoin threat model](docs/opentimestamps-threat-model.md)
- [Current inherited client architecture](docs/architecture.md)

The architecture PR does **not** add Bitcoin stamping yet.

## Current repository baseline

The current application code is inherited from **ProofStamp via Email** and provides a useful local-first baseline:

- local browser SHA-256 hashing;
- local file previews;
- no file upload in the core flow;
- portable plain-text ProofStamp receipts;
- local verification;
- an independent Rust/RustCrypto SHA-256 verifier compiled to WebAssembly;
- restrictive browser security headers and CSP;
- Node and Playwright test coverage.

Some code, filenames, copy, and documentation are still email-specific. They should be removed or renamed only after the OpenTimestamps protocol path and manifest format are proven interoperable.

## Target v0 properties

- One source file in the first product flow.
- File remains on the user's device.
- SHA-256 calculated locally through two implementation paths before stamping.
- ProofStamp Manifest v1 binds the file fingerprint and declared context.
- Manifest commitment is timestamped through standard OpenTimestamps calendars.
- Pending proof is preserved in the primary portable ProofStamp receipt.
- Standard `.ots` data remains exportable.
- Later upgrade adds the Bitcoin attestation.
- Verification distinguishes file match, OTS validity, Bitcoin anchoring, and confirmation state.
- No wallet, seed phrase, token, gas payment, ProofStamp account, or ProofStamp proof database is required for the core design.

## Important claim boundary

A valid ProofStamp can support a claim that a specific committed digital state existed no later than its verified Bitcoin anchoring block.

It does not by itself prove:

- truth of the contents;
- authorship;
- location;
- original creation time;
- whether editing occurred before stamping.

Bitcoin block time must not be presented as an exact trusted file-creation clock.

## Development plan

Implementation is intentionally phased:

1. Repository and architecture preparation.
2. OpenTimestamps interoperability spike and fixture corpus.
3. ProofStamp Manifest v1 canonicalization and golden vectors.
4. Browser create/stamp flow with dual local hashing.
5. Pending-proof upgrade and Bitcoin verification.
6. Parser/network hardening, fuzzing, CSP tests, and failure injection.
7. Remove inherited email-specific code and prepare an experimental release.

See [docs/opentimestamps-v0-plan.md](docs/opentimestamps-v0-plan.md) for exit criteria and release gates.

## Security posture

Treat every imported ProofStamp or `.ots` proof as attacker-controlled binary data.

The planned implementation must use explicit parser limits and must never allow proof-provided URLs to expand the browser's production network allowlist.

Multiple OpenTimestamps calendars are used for availability and resilience. They are not independent timestamp authorities and their responses must not be described as consensus.

Browser verification against public Bitcoin data providers is a convenience path. The strongest independent verification path remains standard OpenTimestamps tooling against independently validated Bitcoin chain data, such as a locally controlled Bitcoin Core node.

See [docs/opentimestamps-threat-model.md](docs/opentimestamps-threat-model.md).

## Run the current baseline locally

The current baseline uses Playwright for development and browser tests.

```bash
npm install
npx playwright install chromium webkit
npm run check
npx serve public
```

Open the local URL shown by `serve`.

## Build

```bash
npm run build
```

The current build runs the Node tests, builds the locked Rust verifier, checks the local verification path, and generates the static `dist/` output.

## Repository transition rule

Do not add a production OpenTimestamps dependency or relax the current network policy merely to make a demo work.

The first implementation PR should be an interoperability spike with test fixtures. The product flow comes after standard `.ots` read/write compatibility is demonstrated.

## License and trademarks

Repository source code is licensed under the [MIT License](LICENSE) unless a file or incorporated dependency states otherwise.

The ProofStamp name and branding are not licensed under MIT; see [TRADEMARKS.md](TRADEMARKS.md).

Any OpenTimestamps library introduced later must receive an explicit dependency and license review before production use.
