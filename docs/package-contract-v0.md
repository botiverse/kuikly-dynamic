# Offline JS package contract (draft v0)

Status: proposal, not a working loader or a security certification. This contract separates package acquisition and verification from the Kuikly JS runtime. Local files and ordinary HTTPS must work without a proprietary release service.

## Archive

A package contains `manifest.json`, `signature.json`, a source-JavaScript entry (`app.js` for the first prototype), and optional assets. It is not an APK patch or a WebView rewrite. Runtime adapters can be extended separately; the initial target is Kuikly native rendering.

Required manifest fields:

- `schemaVersion`, `packId`, immutable `version`, and `entry`;
- `runtimeId`, `kuiklyBuildId`, `bridgeAbi`, host compatibility constraints, and platforms;
- requested capabilities;
- a list of files with normalized relative path, byte length and SHA-256.

The first Android prototype uses `quickjs@<exact-version>` (placeholder MUST be resolved before execution), source JS rather than engine bytecode, and provisional `kuikly-bridge/1`. The runtime owner defines method tables and the precise compiler/core build identity. These values remain provisional until an actual demo validates them. Unknown compatibility identifiers fail closed.

A detached signature covers the exact immutable UTF-8 manifest bytes; verify before parsing, reject duplicate JSON keys, and verify every file against the authenticated inventory. `signature.json` contains an allowlisted algorithm, key ID and signature. Algorithm selection is not frozen until supported implementations are verified across target platforms. Packages cannot introduce their own trusted root. Download URL authorization and hashes do not replace publisher authentication.

## Preparation

Download to an application-private staging directory with compressed-size, expanded-size, file-count and time budgets. Reject absolute paths, traversal, links, duplicate normalized paths and platform case collisions. Reject files absent from the inventory. Never execute staged or partly verified content. Promote the fully verified directory atomically on the same filesystem and persist activation metadata crash-safely.

`VerifiedPackage` is an opaque handle minted by the verifier, not a publicly constructible path wrapper. It binds authenticated manifest bytes, content identity and an immutable directory. Prevent replacement between verification and execution; acquire a lease before reading and retain it until session disposal.

## Activation and recovery

States: Downloading → Verified → Candidate → Active → Healthy. Write a pending-start record before execution. Mark healthy only after the runtime reports the first completed layout, not VM creation or the first bridge call. A failed/unconfirmed candidate falls back to the last verified healthy version or the bundled baseline after restart.

Running sessions retain their package version. Updates apply only to new sessions. Garbage collection cannot delete a leased, fallback or pending package. Enforce a bounded storage budget and reject updates when safe space cannot be reserved.

Network selection must not silently downgrade the accepted release sequence. Local recovery to a known-good slot is distinct from a remotely authorized rollback. Do not permit irreversible shared-data migrations in v0. Offline clients cannot promise immediate server revocation.

## Runtime capabilities

The runtime accepts only a `VerifiedPackage`, never an arbitrary URL. Native calls are mediated by a capability broker bound to package identity and session generation. No arbitrary reflection, shell, native library loading, filesystem paths or raw user credentials. Account/server/package data are isolated. Cancel callbacks, timers and requests when the session ends; old callbacks cannot target a replacement session.

A JS engine alone is not a permission sandbox. CPU/memory limits, interruptibility and native-call cancellation require runtime-specific tests. Distribution policies need separate platform review; JS execution alone is not proof of compliance.

## Acceptance

First prove a bundled/local demo: native view creation, an event, timer, asynchronous bridge call and disposal. Then test identical bytes via HTTPS, offline cold start, v1→v2 and recovery to v1. Negative cases: invalid signature/hash/ABI, partial downloads, archive attacks, low disk, process termination at each activation boundary, infinite loops, unauthorized calls and stale callbacks. Record host/package/runtime/bridge identities and failure stages. No latency or memory claims before measurement.
