---
doc: AGENT_MAP
package: IntellectusLib
repo: moot-core
authored_commit: b2a5c30b794cf477e18022c55e2fea348614d337
authored_date: 2026-07-04
sources:
  - path: Sources/IntellectusLib/Intellectus.swift
    blob: 47d1dbe60d0ed880eaf657986ef9371f66423fda
  - path: Sources/IntellectusLib/RecentWindowSink.swift
    blob: 275f059bb1c0b5ebea68c7d32d2e6bb5ad5ed034
  - path: Sources/IntellectusLib/StatSample.swift
    blob: 5a63d181680c35a1b79279f83651f8cb62dcd9b7
  - path: Sources/IntellectusLib/StatsSink.swift
    blob: 60dc64a00370e491273c2328409a9d56fc46463b
---

# AGENT_MAP — IntellectusLib

PURPOSE: substrate self-report telemetry faculty. Zero-dependency LEAF/floor library. Caller → `Intellectus.report(sample)` → lock-free enabled gate → (if on) installed `StatsSink.receive(_:)`. Off-path = one atomic load + branch, payload never built.

DEPS: imports Foundation (NSLock; StatSample/RecentWindowSink/Intellectus), Synchronization (Atomic<Bool>; Intellectus.swift only). Zero repo dependencies — this IS the floor of the dependency tree (SubstrateTypes/SubstrateKernel/SubstrateLib/SubstrateML are meant to depend on this in a later mission per Package.swift header; not yet wired as of this commit). Imported by: not enumerated in Package.swift/product graph as of this commit; source-comment call sites (forward references, not verified imports) — `AriaResident.installManagerTelemetry` / `AriaResident.runResidentDaemon` (StatsSink.swift, Intellectus.swift), `LocusKit.EstateVerbs`, `NeuronKit.EstateDreamingSink`, `ObserverSink.PersistenceStatsSink` (StatSample.swift). Rust port in `rust/` mirrors every file 1:1 (sample.rs/sink.rs/window.rs/holder.rs/global.rs/lib.rs); NO shared conformance fixtures — StatSample is opaque caller-supplied data, not a computed/derived value, so there is nothing for the two legs to agree on byte-for-byte.

ENTRY POINTS (most callers need only these):
- Intellectus.swift:238 `Intellectus.report(_ make: @autoclosure () -> StatSample)` — the only call substrate hot-path code makes; off when disabled, no payload build
- Intellectus.swift:185 `Intellectus.install(sink: any StatsSink)` — host wires a real sink, once, at startup
- Intellectus.swift:205 `Intellectus.setEnabled(_ enabled: Bool)` — host/daemon flips the gate

## Symbol Table

### Sample type — StatSample.swift
- :32 `enum StatSample: Sendable` — the telemetry datum; value type, two cases
- :43 `case metric(name:value:tags:ts:)` — named Double measurement + String:String tags
- :70 `case event(kind:nounType:rowID:estate:ts:)` — substrate lifecycle transition; nounType is Int (caller casts NounType→Int to keep this crate dependency-free)
- :86 `enum EventKind: String, Sendable, Hashable, CaseIterable` — `.capture` (:87) | `.think` (:88); rawValue is the wire string
- :96 `StatSample.ts: Double` (extension) — timestamp regardless of case

### Receiver contract — StatsSink.swift
- :36 `protocol StatsSink: Sendable` — one requirement
- :47 `receive(_ sample: StatSample)` — must be non-blocking/inexpensive; not self-enforced, caller (Intellectus) owns the enabled gate, sink does not re-check it
- :60 `struct NoOpSink: StatsSink` — default installed sink; discards, O(1), no allocation
- :63 `NoOpSink.shared` — shared singleton instance
- :69 `NoOpSink.receive(_:)` — intentional no-op

### Bundled sink — RecentWindowSink.swift
- :63 `final class RecentWindowSink: StatsSink, @unchecked Sendable` — bounded ring buffer + optional forward decorator
- :69 `capacity: Int` — fixed at init, clamped to min 1
- :72 `forward: (any StatsSink)?` (private) — decorator target; nil = window-only
- :76 `lock = NSLock()` (private) — guards ring/head/filled/_totalReceived only
- :80-86 `ring`/`head`/`filled`/`_totalReceived` (private) — backing storage; _totalReceived counts every receive() ever, ignoring eviction
- :97 `init(capacity:forward:)` — capacity clamped via `max(1, capacity)`
- :114 `receive(_ sample:)` — O(1) ring write under lock (oldest slot overwritten when full); forward call happens AFTER lock release
- :133 `snapshot() -> [StatSample]` — point-in-time copy, oldest-first; safe to iterate lock-free
- :148 `count: Int` — current retained count, 0...capacity
- :157 `totalReceived: Int` — monotonic lifetime receive count; 0 = "nothing has ever crossed the gate," distinct from count==0

### Global facade + holder — Intellectus.swift
- :44 `final class _IntellectusHolder: @unchecked Sendable` (internal, not public) — all mutable state lives here; `Intellectus` enum is a thin wrapper over the singleton
- :56 `_enabled: Atomic<Bool>` (private) — lock-free gate, default false
- :62 `_sinkLock = NSLock()` (private) — guards `_sink` only; acquired only on the on-path (enabled) and during install()
- :67 `_sink: any StatsSink` (private, var) — starts as `NoOpSink.shared`
- :77 `install(sink:)` — replace sink under lock; rare/write-path call
- :91 `setEnabled(_:)` — `.releasing` store
- :100 `isEnabled: Bool` — `.acquiring` load
- :118 `report(_ make: @autoclosure () -> StatSample)` — OFF-path: single atomic load(.acquiring) + branch, autoclosure never forced. ON-path: snapshot sink under lock, release lock, THEN evaluate autoclosure, THEN `sink.receive(...)` — payload build and sink call both happen outside the lock
- :141 `let _intellectus = _IntellectusHolder()` — module-private global singleton; the only instance the public API touches
- :172 `enum Intellectus` — public, caseless namespace; all members static
- :185 `Intellectus.install(sink:)` — forwards to `_intellectus.install`
- :205 `Intellectus.setEnabled(_:)` — forwards to `_intellectus.setEnabled`; driven in production by `AriaResident.runResidentDaemon`'s poll loop reading the stats-store monitoring flag (per doc comment; that call site is outside this package)
- :210 `Intellectus.isEnabled: Bool` — forwards to `_intellectus.isEnabled`
- :238 `Intellectus.report(_:)` — public entry; re-declares `@autoclosure` so short-circuit holds end-to-end from call site to atomic check

## INVARIANTS / GOTCHAS

- OFF-PATH CONTRACT: when `Intellectus.isEnabled` is false, the `@autoclosure` argument to `report(_:)` is NEVER evaluated — no allocation, no lock, no sink call. This is enforced by ordering the `guard _enabled.load(...)` check BEFORE any reference to `make`. Do not restructure `report(_:)` in a way that forces the closure before the gate check.
- Atomic ordering pairing is load-bearing, not decorative: `setEnabled` stores with `.releasing`, `isEnabled`/the gate load with `.acquiring`. This is what guarantees a `setEnabled(true)` on one thread makes state written before it (e.g., a just-completed `install(sink:)`) visible to `report(_:)` on another thread. Do not change either ordering independently — they are a matched pair, mirrored by Rust's `Ordering::Release`/`Ordering::Acquire` in holder.rs. (Rust's own inline comment on `_enabled` load calls it `Relaxed`-sufficient in one place but the code actually uses `Ordering::Acquire`/`Ordering::Release` — treat the Swift acquire/release pairing as the authoritative contract.)
- Sink lock discipline: `_sinkLock` / `RecentWindowSink.lock` are held ONLY for the O(1) state mutation (snapshot/copy the sink reference, or write one ring slot). The actual `sink.receive(...)` / forward call ALWAYS runs after the lock is released. This is SPEC I-7 in the source comments. Breaking this (calling receive under the lock) risks deadlock if a sink's own `receive` re-enters this library or takes a lock a caller also holds.
- `RecentWindowSink` bound is contractual (SPEC I-8 in source comments): memory is O(capacity) regardless of how many samples are ever received. Do not add unbounded accumulation (e.g., a growing log) to this type.
- `RecentWindowSink.totalReceived` counts EVERY direct `receive(_:)` call, including calls made outside the `Intellectus` gate (e.g., in tests that call `window.receive(...)` directly). It is not scoped to gated traffic only.
- `NoOpSink` is the sink installed before any `install(sink:)` call. Because the default `isEnabled` is false, `NoOpSink.receive` is not expected to run in default configuration — but it must remain safe to call, because a host could call `setEnabled(true)` before `install(sink:)`.
- Timestamps are ALWAYS caller-supplied (`Double` epoch seconds). No file in this package reads a clock. Do not add a `Date()`/clock read inside `IntellectusLib` — determinism for replay/testing depends on this.
- Zero repo dependencies is a structural constraint, not a preference: Package.swift's header comment explicitly forbids importing SubstrateTypes/SubstrateKernel or any other repo target from this package, because this package is meant to become the dependency floor those targets build on. Adding a repo import here would create the exact layering cycle the package exists to avoid.
- `EventKind` is a two-case, `CaseIterable`, `String`-backed enum (`.capture`/`.think`). The test suite pins `allCases.count == 2` — adding a case is a breaking, deliberate surface change, not a silent addition.
- Platform floor is macOS 26 / iOS 26 specifically because `Synchronization.Atomic` needs it per project policy; do not lower the floor without re-deriving the atomic gate's performance argument.
- Rust parity is structural (file-for-file, function-for-function) but NOT byte-fixture-gated like LatticeLib: there is no `rust/tests/fixtures/*.json` here, because `StatSample` values are opaque caller data, not values this library derives from an input. Conformance is judged by matching test suite sections (see rust/tests/intellectus_lib_tests.rs vs. Tests/IntellectusLibTests/IntellectusLibTests.swift §1–§8), not by fixture replay.
