# Package manager boundary (draft v0)

This is a language-neutral interface proposal, not an implemented SDK. The runtime and signing primitives are intentionally independent of transport.

```text
PackageSource.resolve(HostCapabilities, Channel) -> Descriptor | NoUpdate
PackageSource.open(Descriptor) -> BoundedByteStream
PackageStore.prepare(Descriptor, ByteStream, TrustStore) -> VerifiedPackage | Failure
PackageStore.acquire(PackageIdentity) -> PackageLease | Failure
PackageStore.stageActivation(PackageLease) -> PendingStartToken
Runtime.create(PackageLease, CapabilityBroker, PendingStartToken) -> Session | Failure
Session.onBridgeHandshake()                       // not yet healthy
Session.onFirstLayout() -> PackageStore.markHealthy(PendingStartToken)
Session.dispose()                                // cancel callbacks, then release lease
PackageStore.recoverUnconfirmedStart() -> HealthyPackage | BundledBaseline
```

`Descriptor` contains immutable content identity and location; it is not trusted until signature verification. `HostCapabilities` includes exact runtime, compiler/core identity, bridge ABI, host version, platform and allowed capabilities. Trust policy is supplied by the host, never by the downloaded package.

Failures are typed: transport, size budget, unsafe archive, invalid signature, content mismatch, incompatible host/runtime/bridge, permission denial, disk capacity, activation failure and runtime failure. Preserve the healthy slot for every failed preparation. Do not fall back from an invalid signature to unsigned loading.

Local-file, HTTPS and optional release-service adapters implement the same source contract. v0 downloads full packages. APK installation and APK differential updates are outside this API.

The provisional Android entry is a single Kotlin/JS source file `app.js`. `kuikly-bridge/1` is specified in [bridge ABI v0](../docs/bridge-abi-v0.md); manager code must not duplicate or infer its numeric method tables. A first native-view bridge call establishes handshake; completed first layout establishes readiness.

See [package contract](../docs/package-contract-v0.md) for validation and recovery requirements.
