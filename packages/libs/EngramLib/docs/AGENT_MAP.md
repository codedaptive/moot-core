---
doc: AGENT_MAP
package: EngramLib
repo: moot-core
authored_commit: b2a5c30b794cf477e18022c55e2fea348614d337
authored_date: 2026-07-04
sources:
  - path: Sources/EngramLib/EngramLib.swift
    blob: b7f236bfaa193967daf5c5d6061ef4591207ed22
  - path: Sources/EngramLib/Match.swift
    blob: 5c61fd65888a795c703ab9f52022dffeae9c7595
---

# AGENT_MAP: EngramLib

PURPOSE: product-facing similarity/retrieval/aggregation API over 256-bit engrams (`Fingerprint256`). Thin wrapper that hides substrate kernel selection: distance, batch-distance, top-K nearest, radius filter, OR-union. Two files total; no internal multi-stage pipeline.

DEPS: imports SubstrateTypes (`Fingerprint256`, aliased `Engram`), SubstrateKernel (`SubstrateKernel` protocol, `PortableKernel.kernelForCurrentPlatform()`). Imported by (within SDK): VectorKit, CorpusKit (kits, moot-memory repo: kits compose libs, never the reverse). Rust port in `rust/` (`engram_lib` crate) mirrors the full API 1:1 against `substrate-types`/`substrate-kernel` Rust crates; conformance is behavioral parity checked by mirrored test suites (`rust/tests/engram_lib_tests.rs` vs `Tests/EngramLibTests/`), not a shared fixture file.

ENTRY POINTS (most callers need only these):
- EngramLib.swift:77 `EngramLib.distance(_ a: Engram, _ b: Engram) -> Int` | Hamming distance, 0...256
- EngramLib.swift:99 `EngramLib.findNearest(probe:in:k:) -> [Match]` | top-K nearest, distance-asc / index-asc tiebreak
- EngramLib.swift:216 `EngramLib.session() -> Session` | reusable kernel handle for hot loops

## Symbol Table

### Engram type: EngramLib.swift
- :43 `public typealias Engram = Fingerprint256` | substrate type, product-facing name; do not add a wrapping struct
- :45-53 `extension Engram { init(blocks b0:_ b1:_ b2:_ b3:) }` | convenience 4-block constructor
- :33 `internal let _engramLibCachedKernel: any SubstrateKernel` | resolved ONCE at module load via `PortableKernel.kernelForCurrentPlatform()`; every call path reads this, never re-resolves

### Static API: enum EngramLib (EngramLib.swift:71)
- :77 `distance(_:_:) -> Int` | single-pair Hamming distance
- :84 `distances(probe:candidates:) -> [Int]` | batched, same order/count as `candidates`; `[]` on empty input
- :99 `findNearest(probe:in:k:) -> [Match]` | `[]` if `k <= 0` or candidates empty; clamps to `min(k, candidates.count)`
- :112 `findNearest(probe:in:) -> Match?` | k=1 convenience; `nil` on empty candidates
- :124 `findWithin(probe:in:maxDistance:) -> [Match]` | `[]` if candidates empty OR `maxDistance < 0`; inclusive upper bound
- :149 `union(_ engrams: [Engram]) -> Engram` | bitwise OR-reduce; zero engram is the empty-input identity
- :154 `union(_ a:_ b:) -> Engram` | pairwise OR convenience, delegates to `Fingerprint256.union(_:)`
- :222 `private static func kernel() -> any SubstrateKernel` | single choke point back to the cached kernel

### Session: struct EngramLib.Session (EngramLib.swift:166), Sendable
- :169 `init()` | captures `_engramLibCachedKernel`
- :173/:177/:184/:193/:210 `distance` / `distances` / `findNearest` / `findWithin` / `union` | instance mirrors of the static functions above, byte-identical results, no new behavior
- :216 `EngramLib.session() -> Session` | factory, equivalent to `Session()`

### Result type: Match.swift
- :14 `public struct Match: Hashable, Sendable, Codable` | `index: Int` (position in caller's candidate array), `distance: Int` (Hamming distance from probe)
- :18 `init(index:distance:)` | public memberwise init
- :24-28 `extension Match: Comparable` | `<` compares `distance` first, `index` second; this IS the sort/tiebreak contract used by `findNearest`/`findWithin`

## INVARIANTS / GOTCHAS

- Kernel is resolved exactly once per process (`_engramLibCachedKernel`, EngramLib.swift:33), at module load, not per call. Do not reintroduce per-call kernel resolution: it was removed as a measured, needless dispatch cost (see file header comment, Phase 4.3 / decision 2026-05-28 §6.4.3).
- DO NOT REIMPLEMENT SUBSTRATE MATH (EngramLib.swift:14-24 banner comment). Every primitive this library exposes, Hamming distance, batch distance, top-K, OR-reduce, already exists in SubstrateTypes/SubstrateKernel/SubstrateML; EngramLib only forwards to it.
- Tie-break rule is fixed and load-bearing: distance ascending, then candidate index ascending (`Match.<`, Match.swift:25-27). `findNearest` and `findWithin` both depend on this for deterministic, reproducible output: same probe + same candidate list + same order = same result, always.
- All empty/degenerate-input guards return empty collections or `nil`, never throw and never crash: empty candidates → `[]`/`nil`; `k <= 0` → `[]`; `maxDistance < 0` → `[]`; empty `union` input → `Engram.zero` (the OR identity).
- `Session` and the static functions are result-identical by construction: both bottom out in the same cached kernel. `Session` exists only to save the small per-call cost of re-touching the static cache in a hot loop; it is not a different code path.
- `Engram` is a type alias, not a wrapper type. Treat the underlying representation (`Fingerprint256`, four `UInt64` blocks) as substrate-owned; construct via `Engram(blocks:_:_:_:)` or substrate constructors, not by touching blocks directly from product code.
- Whole library is stateless and thread-safe (file header, EngramLib.swift:7-10): every static method creates no persistent state; `Session` is `Sendable` and safe to share across tasks.
- Rust and Swift legs must stay behaviorally identical: same guard conditions, same tie-break order, same identity values. There is no shared byte-fixture gate here (contrast LatticeLib's conformance JSON): parity is enforced by keeping `rust/tests/engram_lib_tests.rs` and `Tests/EngramLibTests/` mirrored. Changing a guard or ordering rule in one leg without the other is a silent cross-platform divergence.
- Package has no pinned artifacts, no build-time-only code, and no privacy seams: unlike larger sibling libs (e.g. LatticeLib), there is nothing here gated by an environment variable or a resource bundle.
