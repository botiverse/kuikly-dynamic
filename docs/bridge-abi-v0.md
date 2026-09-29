# Bridge ABI v0 (`kuikly-bridge/1`)

Status: draft for the first Android prototype. Frozen once the prototype renders a page end to end; any later change bumps the ABI version.

Source of truth: upstream Tencent-TDS/KuiklyUI 2.28.0 — `core/src/commonMain/.../manager/BridgeManager.kt` (`KotlinMethod`, `NativeMethod`), `core/src/jsMain/.../ImportFunctions.kt`, `core/src/jsMain/.../nvi/NativeBridge.kt`.

## Direction: host → page bundle

The bundle is a single Kotlin/JS (IR) file that defines a global function:

```
callKotlinMethod(methodId, arg0, arg1, arg2, arg3, arg4, arg5)
```

| methodId | name |
|---|---|
| 1 | createInstance |
| 2 | updateInstance |
| 3 | destroyInstance |
| 4 | fireCallback |
| 5 | fireViewEvent |
| 6 | layoutView |

## Direction: page bundle → host

The host injects these globals before evaluating the bundle:

```
callNative(methodId, arg0, arg1, arg2, arg3, arg4, arg5) -> any
nativeLog(message)
```

| methodId | name |
|---|---|
| 1 | createRenderView |
| 2 | removeRenderView |
| 3 | insertSubRenderView |
| 4 | setViewProp |
| 5 | setRenderViewFrame |
| 6 | calculateRenderViewSize |
| 7 | callViewMethod |
| 8 | callModuleMethod |
| 9 | createShadow |
| 10 | removeShadow |
| 11 | setShadowProp |
| 12 | setShadowForView |
| 13 | setTimeout |
| 14 | callShadowMethod |
| 15 | fireFatalException |
| 16 | syncFlushUI |
| 17 | callTDFModuleMethod |

Optional: the bundle may export `registerCallNative(pagerId, callback)`. Once any page registers a callback this way, the bundle routes every native call through the per-page callbacks instead of the global `callNative` (upstream behaviour).

## Identity fields used by the package manifest

- `bridgeAbi`: `kuikly-bridge/1`. A host must refuse to run a bundle whose `bridgeAbi` it does not implement exactly.
- `runtimeId`: engine plus exact version, e.g. `quickjs@<version>` (Android prototype), later `jsc@system` (iOS) and `jsvm@<api-level>` (HarmonyOS). v0 accepts source JS only, no engine bytecode.
- `kuiklyBuildId`: Kuikly core version and Kotlin compiler version the bundle was built with, e.g. `kuikly-core 2.28.0 / kotlin 2.1.21`.
- `entry`: path of the single bundle file inside the package, `app.js` in v0.

## Lifecycle

1. `Runtime.create(packageLease, capabilityBroker, pendingStartToken)` (see [manager boundary](../manager/README.md)) creates a fresh JS context, injects `callNative` / `nativeLog`, and evaluates `entry`.
2. The host calls `callKotlinMethod(1 /* createInstance */, pagerId, pageName, pageData, ...)`.
3. Bridge handshake succeeds on the first `callNative(1 /* createRenderView */)`.
4. `ready()` fires after the first layout pass completes, which is what lets the manager mark the package healthy; any failure before that is reported with its stage (VM creation, bundle evaluation, handshake, first frame).
5. `dispose()` calls `callKotlinMethod(3 /* destroyInstance */, pagerId)` and tears the context down; a disposed context is never reused.

Open points to settle in the prototype: argument marshalling for each engine (strings, numbers, JSON objects, callbacks), where `setTimeout` is scheduled, and how module calls are authorised by the capability broker.
