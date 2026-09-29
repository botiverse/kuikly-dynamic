# kuikly-dynamic

Open-source dynamic (JS) execution for [Kuikly](https://github.com/Tencent-TDS/KuiklyUI): ship Kuikly pages, including Kuikly Compose pages, as signed JS bundles, update them without an app-store release, and keep rendering through Kuikly's native renderers on Android, iOS and HarmonyOS.

Upstream Kuikly already separates the page runtime (Kotlin, compilable to Kotlin/JS) from the native renderers, which talk over a single `callNative` / `callKotlin` bridge, and exposes an execute-mode extension point on every platform. What is missing from the open-source release is the JS execution mode itself and the bundle delivery around it. This project fills that gap:

- **Runtime**: a JS execute mode per platform (Android, iOS via JavaScriptCore, HarmonyOS) that loads a Kotlin/JS page bundle and bridges it to the existing renderers.
- **Bundle format**: manifest, host-capability version, signature, atomic install and rollback; transport-agnostic (local file, HTTPS, or any release backend).
- **Tooling**: build a Kuikly page module into a bundle, sign it, and verify it.

Status: design and prototype phase. Nothing here is stable yet.

## License

Apache License 2.0. Kuikly itself is distributed under its own license terms; see the upstream repository.
