---
doc: AGENT_MAP
package: SubstrateKernel
repo: moot-core
authored_commit: b2a5c30b794cf477e18022c55e2fea348614d337
authored_date: 2026-07-04
sources:
  - path: Sources/SubstrateKernel/BitField.swift
    blob: 755ce095e0db4d4d4f5f01cc3568cb8403e5952f
  - path: Sources/SubstrateKernel/FloatVecOps.swift
    blob: b1c4593a99fe078b19ad0058a2e76e9949abd524
  - path: Sources/SubstrateKernel/HammingNN.swift
    blob: c810cb5ada7b0c1cdc7789791552478b19bdad07
  - path: Sources/SubstrateKernel/HKDF.swift
    blob: 53cd1dbef1ea0b9a8c932cdcfc303b50eb49adba
  - path: Sources/SubstrateKernel/PortableKernel-Metal.swift
    blob: 4c9837448547cb44fde52a08b8e639876d23f150
  - path: Sources/SubstrateKernel/PortableKernel-NEON.swift
    blob: 996063a101bcd76d2ea965b58072eac2141adf98
  - path: Sources/SubstrateKernel/PortableKernel-SIMD.swift
    blob: 88caebd2a66c6541f9dff3aff7555dabc557ee5f
  - path: Sources/SubstrateKernel/PortableKernel.swift
    blob: 5279dadeb909d4d8980f2fc3a3100caf7d9066d3
  - path: Sources/SubstrateKernel/SHA256.swift
    blob: 143a3530a8dee15446fa5711022638e0d61219b2
---

# AGENT_MAP: SubstrateKernel

PURPOSE: layer 2 of the four-package substrate split (SubstrateTypes → SubstrateKernel → SubstrateML → SubstrateLib). Bandwidth-bound bit ops over `Fingerprint256`: Hamming distance/top-K, OR-reduce, SimHash, count-fold, popcount; plus BitField (packed-bitmap field extract/write), SHA-256 content hashing, HKDF-SHA256 key derivation, and canonical scalar float-vector ops. No pinned data artifacts: behavior is pure algorithm, not versioned reference data.

DEPS: imports SubstrateTypes (Fingerprint256, HyperplaneFamily, CountVector256, FloatSimHashPlanes, SimHash, Hamming, BlockMask), IntellectusLib (Intellectus.report telemetry: zero-dep leaf), Foundation, `simd` (Apple + aarch64 Linux), Metal (Apple only, `#if canImport(Metal)`). Imported by: SubstrateML (depends on SubstrateTypes + SubstrateKernel per dependency graph); hot-path consumers LocusKit, CorpusKit, GeniusLocusKit, EngramLib; SubstrateLib's AuditGate consumes only BitField + SHA256 (not the kernel dispatch layer). Rust port in rust/ mirrors kernel.rs/bit_field.rs/hamming_nn.rs/sha256.rs/hkdf.rs/float_vec_ops.rs function-for-function; kernel_simd.rs (nightly-gated `simd-nightly` feature) covers NEON's ground so there is no neon.rs; kernel_avx512.rs (x86_64-only, DARK path, P11-VAL-015) has no Swift counterpart; Metal has no Rust counterpart (explicit platform waiver, no Linux/Windows GPU path).

ENTRY POINTS (most callers need only these):
- PortableKernel.swift:313 `PortableKernel.kernelForCurrentPlatform() -> SubstrateKernel`: auto-selects SimdKernel (arm64) or ScalarKernel (else); never selects Metal
- PortableKernel.swift:368 `PortableKernel.kernel(of: KernelKind) -> SubstrateKernel`: explicit backend selection (tests, conformance harness, opt-in GPU/NEON)
- BitField.swift:62/86 `BitField.extractField/writeField(...)`: packed-bitmap field access, the F18 centralization surface
- SHA256.swift:30 `SHA256.hash(_ bytes:) -> [UInt8]`: FIPS 180-4 content hash
- HKDF.swift:54 `GrantHKDF.deriveKey(...)`: RFC 5869 HKDF-SHA256 grant scope-key derivation

## Symbol Table

### Protocol + reference kernel: PortableKernel.swift
- :49 `protocol SubstrateKernel: Sendable`: every backend conforms; all methods must be bit-identical across conformers
- :54 `kind: KernelKind`: runtime introspection; default `.scalar` (extension :137), every concrete kernel overrides
- :57 `popcount64(_:) -> Int`: 64-bit population count
- :60 `hammingDistance256(_:_:) -> Int`: bit-differences between two Fingerprint256
- :64 `orReduce256(_:) -> Fingerprint256`: bitwise OR fold; identity = zero
- :69 `hammingTopK(probe:candidates:k:) -> [(index:Int,distance:Int)]`: K nearest by Hamming distance, ties → lower index
- :76 `simhashCompute(subhashes:families:) -> Fingerprint256`: 4-subhash → Fingerprint256 projection
- :91/:97/:102 `hammingDistanceBatch` / `simhashBlockBatch` / `orReduceBatch`: batched variants; protocol-extension defaults are naive loops (:139/:144/:157), always correct, backends may override for speed
- :112/:116 `countFold256` / `countFoldBatch`: per-bit-position set counts across a cohort; default (:166) = `CountVector256.fold`; OR-reduce is this fold's saturating-at-one degenerate case
- :127 `floatSimHashProject(vector:planes:) -> Fingerprint256`: float analogue of simhashCompute; default impl (:183) is the SCALAR ORACLE for this op: fixed left-to-right accumulation order (FP `+` non-associative, order IS the contract); precondition on `planes.dim == vector.count`
- :222 `struct ScalarKernel: SubstrateKernel`: THE ORACLE; every other backend's output must match this bit-for-bit for the same input
- :233 `hammingDistance256`: `a.zip4(b, ^).popcount()` (SubstrateTypes combinator)
- :239 `orReduce256`: `Fingerprint256.reduce4(fingerprints, |)`
- :245 `hammingTopK`: bounded max-heap O(N log k); tie-break lower-index-wins is the cross-backend contract
- :287 `enum KernelKind`: scalar|simd|neon|metal|avx512|avx2 (avx512/avx2 always fall to scalar in THIS Swift build: no Swift AVX implementation exists)
- :296 `enum PortableKernel`: dispatcher namespace
- :313 `kernelForCurrentPlatform()`: arm64→SimdKernel else ScalarKernel; COMPILE-TIME choice, no runtime CPU-feature probing (contrast Rust's Avx512 CPUID guard, which DOES probe at runtime because that backend is unsafe without it); emits `substrate.kernel.backend_selected` telemetry, autoclosure skipped (no clock read) when monitoring off
- :355 `currentArchTag`: static string ("arm64"/"x86_64"/"other"), must track the `#if arch()` predicate above it
- :368 `kernel(of:)`: explicit selector; `.metal` falls back to ScalarKernel if `MetalKernel()` init fails (no GPU); `.avx512`/`.avx2` always → ScalarKernel
- :406 `assertEqual(lhs:rhs:probe:candidates:k:)`: hammingTopK-based bit-identity check; conformance-suite building block
- :430 `ScalarKernelScored` / :440 `ScalarKernelMaxHeap` (internal): (distance, index) max-heap shared plumbing; root = worst retained

### SIMD backend: PortableKernel-SIMD.swift
- :39 `struct SimdKernel: SubstrateKernel`: selected by kernelForCurrentPlatform() on arm64; `import simd` → NEON codegen
- :65 `hammingDistance256`: SIMD4<UInt64> XOR + per-lane nonzeroBitCount, summed
- :114 `hammingTopK`: sorted-ladder maintenance (NOT a heap); identical sorted output + tie-break to ScalarKernel
- :214 `simhashBlockBatch`: uses :233 `PackedFamily` (SoA repack of 64 hyperplanes into 16 groups of 4) built ONCE per batch, amortized across inputs via :275 `simhashBlockSIMD`
- :376 `countFold256`: bit-sliced vertical binary counter across SIMD4<UInt64> lanes; O(fingerprints × ~2 ops) vs scalar's bit-by-bit walk; conformance-gated against `CountVector256.fold`

### NEON backend: PortableKernel-NEON.swift
- :38 `struct NeonKernel: SubstrateKernel`: NOT auto-selected; reachable only via `PortableKernel.kernel(of: .neon)`; exists to be BENCHMARKED against SimdKernel, not to replace it
- :63 `toBytes(_:) -> SIMD32<UInt8>` (static): reframes Fingerprint256 at byte level via single 256-bit load (not byte-by-byte copy)
- :87 `popcountBytes(_:)` (static): 32 scalar nonzeroBitCount calls; Swift does NOT auto-vectorize this into one `cnt.16b` (measured gap, see file header)
- :104/:113/:128 `hammingDistance256` / `hammingDistanceBatch` / `hammingTopK`: byte-level XOR + popcount + horizontal sum; hammingTopK uses same ladder pattern as SimdKernel
- :170/:182/:186 `orReduce256` / `orReduceBatch` / `simhashCompute`: NOT re-framed at byte level; identical to SimdKernel's implementation (byte reframing judged not to help these ops)

### Metal backend: PortableKernel-Metal.swift (Apple-only, `#if canImport(Metal)`)
- :67 `MetalBufferPool` (fileprivate, `@unchecked Sendable`): persistent GPU buffers sized for `maxN`; caller-enforced per-call synchronization, harness is single-threaded
- :110 `struct MetalKernel: SubstrateKernel`: NOT auto-selected; reachable only via `PortableKernel.kernel(of: .metal)`
- :128 `defaultMaxN = 100_000`: buffer-pool sizing default (~3.6 MB), tuned to dreaming-daemon batch-index scale
- :134 `init?(maxN:)`: returns nil on ANY setup failure (no GPU, shader compile fail, pipeline fail); PortableKernel.kernel(of:) treats nil as "fall back to ScalarKernel", not an error
- :172 `popcount64` / :182 `hammingDistance256`: inherited scalar semantics; per-pair GPU dispatch not worth its ~10–30µs overhead
- :203 `hammingDistanceBatch`: :210 dispatches :221 `dispatchWithPool` (N ≤ maxN, reuse buffers) or :283 `dispatchWithFreshBuffers` (N > maxN, allocate per-call)
- :418 `scalarFallback` (private): CPU fallback on ANY Metal failure (buffer alloc, command-buffer creation, GPU execution error): never crashes or returns a wrong/partial result
- :364 `hammingTopK`: GPU batched distance + CPU heap-of-K selection (GPU-side top-K selection judged not worth it for small K)
- :440 `shaderSource` (private static): embedded Metal shader source (`hamming_distance_kernel`); must be kept byte-for-byte in sync with the canonical `.metal` file or the conformance gate's CRC check fails

### Standalone top-K reference: HammingNN.swift
- :32 `struct HammingNNHit: Hashable, Sendable`: (rowID: UUID, distance: Int)
- :51 `enum HammingNN`
- :53 `topK<S>(anchor:candidates:k:blocks:) -> [HammingNNHit]`: NOT part of the SubstrateKernel/PortableKernel dispatch; operates over any `(UUID, Fingerprint256)` sequence, not a dense array; supports partial-block comparison via `blocks: BlockMask` (e.g. topic-only similarity); requires `k > 0`
- tie-break: rowID.uuidString ascending (DIFFERENT key from PortableKernel backends' index-ascending: do not conflate the two top-K contracts)
- :94/:101/:114 `heapLarger` / `heapifyUp` / `heapifyDown` (private): bounded max-heap, same shape as ScalarKernelMaxHeap but keyed on UUID string not array index

### Packed-bitmap fields: BitField.swift
- :45 `enum BitField`: F18 atomic-centralization surface; kits must NOT open-code bit-shift/mask logic
- :62 `extractField(_:shift:width:) -> Int64`: precondition: 0≤shift, 1≤width≤64, shift+width≤64
- :86 `writeField(_:into:shift:width:) -> Int64`: same preconditions; value bits outside `width` are SILENTLY truncated (documented contract, not a bug)
- :135 `maskedEquals(_:mask:expected:) -> Bool`: one-step `(bitmap & mask) == expected`; caller invariant: expected must already be masked/aligned
- :153/:164 `extractFlag` / `writeFlag`: single-bit case; precondition 0≤bit<64
- :181 `popcount(_:) -> Int`: via `UInt64(bitPattern:).nonzeroBitCount`
- :188 `hammingDistance(_:_:) -> Int`: `popcount(a ^ b)`, single-Int64 version (contrast Fingerprint256's 256-bit version elsewhere)
- :200 `xorFold<S>(_:) -> Int64`: running XOR reduce; empty → 0

### Content hashing: SHA256.swift
- :27 `enum SHA256`: FIPS 180-4, dependency-free
- :30 `hash(_ bytes: [UInt8]) -> [UInt8]`: 32-byte digest; ported VERBATIM from GeniusLocusKit's prior in-kit copy so centralization preserves every existing content ID byte-for-byte
- :119 `rotr` (private): right-rotate helper

### Key derivation: HKDF.swift
- :37 `enum GrantHKDF`: RFC 5869 HKDF-SHA256, built entirely over this package's own SHA256 (no CryptoKit) for cross-platform byte-identical key derivation
- :54 `deriveKey(inputKeyMaterial:salt:info:outputByteCount:) -> [UInt8]`: public entry; `outputByteCount` ≤ 8160 (32×255)
- :69 `extract(salt:ikm:)`: RFC 5869 §2.2, `hmac(key: salt, data: ikm)`
- :76 `expand(prk:info:length:)`: RFC 5869 §2.3, chained HMAC with counter byte
- :101 `hmac(key:data:) -> [UInt8]`: public (not just internal) because SubstrateLib's KeyedCommitment API reuses this exact construction; RFC 2104, keys > 64 bytes pre-hashed

### Scalar float-vector oracle: FloatVecOps.swift
- :62 `enum FloatVecOps`: canonical IEEE-754 scalar reference; every faster backend (Accelerate/BLAS/etc.) MUST match bit-for-bit
- :72 `l2Norm(_:) -> Float`: sqrt(sum(x*x)); empty → 0.0
- :99 `l2Normalize(_:) -> [Float]`: zero-vector PASSTHROUGH (not NaN, not a panic): the "no information" signal that projects to Engram.zero via FloatSimHash
- :121 `dot(_:_:) -> Float`: precondition a.count == b.count
- :147 `cosine(_:_:) -> Float`: DEFINED ONLY for pre-normalized unit vectors (caller must call l2Normalize first); debug-only assert checks norm ≈ 1.0 within 1e-5, compiled away in release

## INVARIANTS / GOTCHAS

- CONFORMANCE IS THE CONTRACT. Every `SubstrateKernel` backend (Scalar/Simd/Neon/Metal, both legs) must produce bit-identical output to `ScalarKernel` for the same input. A divergence is a bug in the backend, never grounds to change the scalar oracle.
- `FloatVecOps` is a second, independent oracle for decimal-vector math (l2Norm/l2Normalize/dot/cosine); same bit-identical-to-scalar rule applies to any faster override elsewhere in the substrate.
- TWO DIFFERENT top-K contracts coexist and must not be conflated: `PortableKernel` backends' `hammingTopK` (dense array input, ties → lower ARRAY INDEX) vs. `HammingNN.topK` (candidate-sequence input, ties → lower `rowID.uuidString`, supports partial-block comparison via `BlockMask`).
- `kernelForCurrentPlatform()` is a COMPILE-TIME choice (`#if arch(arm64)`), not a runtime CPU-feature probe. It never selects `.metal`; Metal requires explicit `PortableKernel.kernel(of: .metal)`. `.avx512`/`.avx2` have no Swift implementation and always resolve to `ScalarKernel` in this build.
- Rust's AVX-512 backend (`kernel_avx512.rs`) is the opposite of the Swift dispatcher: it DOES check `is_x86_feature_detected!("avx512f")` && `("avx512vpopcntdq")` at runtime before every unsafe intrinsic call, because skipping that check is undefined behavior (SIGILL) on CPUs lacking those features. This guard was present from the initial commit and was hardened/test-covered in the 2026-06-28 security review (`tests/avx512_hamming_conformance.rs::cpuid_guard_prevents_intrinsics_without_feature`). `for_current_platform()` never auto-selects it regardless (dark path, P11-VAL-015 gates enable).
- `MetalKernel.init?` returning `nil` is a NORMAL, expected outcome (no GPU / headless CI / virtualization), not an error to surface: callers fall back to scalar.
- `MetalKernel.hammingDistanceBatch` falls back to `scalarFallback` on ANY Metal failure (buffer alloc, command-buffer/encoder creation, GPU execution error): never crashes, never returns a partial/wrong result.
- `MetalKernel.defaultMaxN` = 100,000: batches at or under this reuse the persistent `MetalBufferPool`; batches over it pay full per-call buffer allocation via `dispatchWithFreshBuffers`.
- `MetalKernel.shaderSource` is a hand-maintained string copy of the canonical `.metal` shader file; the conformance gate's CRC check is the only thing that catches drift between them: keep them in sync manually.
- `BitField.writeField`'s value truncation (bits of `value` outside `width` are silently dropped) is an intentional contract, not a bug: matches packed-row semantics where the field's width defines its value space.
- `SHA256.hash` was ported VERBATIM from GeniusLocusKit specifically to preserve existing audit-log content IDs byte-for-byte; do not "clean up" the implementation without checking downstream ID stability.
- `HKDF`/`GrantHKDF` uses ONLY the in-repo `SHA256`, never CryptoKit: required for the Swift↔Rust byte-identical key-derivation guarantee (PAR-4-GL1).
- `GrantHKDF.hmac` is public (not internal) because `SubstrateLib`'s KeyedCommitment API depends on reusing this exact HMAC-SHA256 construction; do not narrow its access without checking that caller.
- No pinned/versioned data artifacts ship in this package (contrast e.g. LatticeLib's Resources/ lexicon+frame+signatures): everything here is pure algorithm; conformance is enforced by test fixtures, not artifact versioning.
- `AuditGate` (in `SubstrateLib`, NOT in this package) consumes only `BitField` + `SHA256` from here: it does not touch the kernel-dispatch layer at all.
- No NEON or Metal Rust files exist by design: `kernel_simd.rs` (nightly `simd-nightly` feature) covers NEON's aarch64 ground on the Rust side; Metal has no cross-platform equivalent and is an explicit, documented platform waiver.
- Telemetry (`Intellectus.report` / `substrate.kernel.backend_selected`) is off by default; when off, the reporting autoclosure is never evaluated and no clock (`Date()`) is read: selection has zero overhead in the default configuration.
