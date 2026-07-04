---
doc: AGENT_MAP
package: SubstrateTypes
repo: moot-core
authored_commit: b2a5c30b794cf477e18022c55e2fea348614d337
authored_date: 2026-07-04
sources:
  - path: Sources/SubstrateTypes/AsOfCoordinate.swift
    blob: b4433864e36b5803533266c97b153a836ee04adb
  - path: Sources/SubstrateTypes/AuditEvent.swift
    blob: fbfabeb4eaba4bf43b6e45d8c2b772d994a12b7b
  - path: Sources/SubstrateTypes/BitwiseArithmetic.swift
    blob: a6270b0f7395c17915c54fdc119e854ca5a44077
  - path: Sources/SubstrateTypes/BlockMask.swift
    blob: 904c2179be96d5c20472f0f356947e82e9d8cf37
  - path: Sources/SubstrateTypes/ContentHash.swift
    blob: 835744792dc191e824e676d8715c96111b4945ff
  - path: Sources/SubstrateTypes/CountVector256.swift
    blob: edabb34cb02005135d745ffa6da5541835159927
  - path: Sources/SubstrateTypes/Fingerprint256.swift
    blob: 465607fe4164bb0dd94009dbbb20d6e766b973a8
  - path: Sources/SubstrateTypes/FloatSimHashPlanes.swift
    blob: b5506e7d12fd153b721f4ca7a52a65a6667ea2f6
  - path: Sources/SubstrateTypes/FNV.swift
    blob: f3d8874705ad66f513e49ccbf0c714980a1172e8
  - path: Sources/SubstrateTypes/GSetAuditLog.swift
    blob: 1a4452e4c8a147d4fac299a3319b0cb4e70cb795
  - path: Sources/SubstrateTypes/Hamming.swift
    blob: dc899742f77434910d51bb3cd830cc9603f56f0a
  - path: Sources/SubstrateTypes/HLC.swift
    blob: a74442251bd542e8f403029ee475c2ac9d7b91b0
  - path: Sources/SubstrateTypes/HyperplaneFamily.swift
    blob: f30c4640629116b1952df48790ed6f5ae67e9025
  - path: Sources/SubstrateTypes/LatticeAnchor.swift
    blob: 5f3ba24efa5a85109d20f3b61e6acb17e21cb0a5
  - path: Sources/SubstrateTypes/MatrixC.swift
    blob: a8612ea7b09e6d4d93a4acbeedbe2b3b91776a74
  - path: Sources/SubstrateTypes/MatrixF.swift
    blob: ed6e38a78c654562b04074878bf86686da0172c3
  - path: Sources/SubstrateTypes/MatrixO.swift
    blob: c10fa0b14d8abd172503d0375dfab9302da32025
  - path: Sources/SubstrateTypes/MatrixT.swift
    blob: cd31ca818be6446f8f1cd98c6b912c640a78c613
  - path: Sources/SubstrateTypes/MerkleDomain.swift
    blob: 4e7d1888a04b2e9979e62c58b54ba698712e56d5
  - path: Sources/SubstrateTypes/MerkleRoot.swift
    blob: 1979925f4646cdbed8785a43e2f2a14a7f9c0cac
  - path: Sources/SubstrateTypes/NounType.swift
    blob: 940d6c51329a66849e572a95ff8ae51648784f2f
  - path: Sources/SubstrateTypes/ORReduce.swift
    blob: 3f9f6e6ed30fb233f48457f95047f274bd271745
  - path: Sources/SubstrateTypes/RecallTypes.swift
    blob: 7f01133df8495156fab87f1f4e441fb7867b3434
  - path: Sources/SubstrateTypes/Row.swift
    blob: d7cf86f774dbccbbdf9daf1f3a5ed9be29fb24e8
  - path: Sources/SubstrateTypes/RowBitmaps.swift
    blob: b879191a7076807c6fcd20e54cf806a639e63388
  - path: Sources/SubstrateTypes/RowState.swift
    blob: 3f8c772be1fe1e3b44b90a5ea8681d4ce4326db7
  - path: Sources/SubstrateTypes/SimHash.swift
    blob: e32a8b685c3db29408ed9e7fb21da4a7da10c467
  - path: Sources/SubstrateTypes/SnapshotId.swift
    blob: b476bad15abecba2dacea225e0a7db7f445be777
  - path: Sources/SubstrateTypes/ThreeDBitTensor.swift
    blob: 47b53642dda804afe8fbc0d863dc4ccd11d3ec55
  - path: Sources/SubstrateTypes/TimeRange.swift
    blob: e1b7fbed94b6505b596ac5701cfd7c693e7d2425
---

# AGENT_MAP — SubstrateTypes

PURPOSE: layer 1 of the four-package substrate split. Pure data shapes (Row,
Fingerprint256, LatticeAnchor, HLC, RowState/RowBitmaps, AuditEvent) plus
canonical reference compute logically inseparable from those shapes
(Hamming, SimHash, FNV, ORReduce, BitwiseArithmetic, CountVector256). Zero
transcendentals, zero I/O, zero pinned data artifacts. Consumers needing
only substrate-shape (e.g. ConvergenceKit) depend on this package alone.

DEPS: imports Foundation only (no intra-SDK imports — this is the dependency
floor). Imported by: SubstrateKernel, SubstrateML, SubstrateLib, EngramLib,
IntellectusLib, and the root app package (Package.swift). SubstrateLib
RETAINS the row-state automaton, the verb mechanics, and the full
multi-row substrate object — those are compute, not shape, and stay one
layer up. Rust port in `rust/src/` mirrors every module 1:1 (see DEPS note
in Rust Port and Conformance, DETAILS.md); no fixture directory — agreement
is verified by matched unit-test vectors in each module, not a shared
fixtures/ dir.

ENTRY POINTS (most callers need only these):
- Row.swift:23 `struct Row` — the substrate's central data shape
- Fingerprint256.swift:43 `struct Fingerprint256` — 256-bit structural coordinate
- LatticeAnchor.swift:12 `struct LatticeAnchor` — topic coordinate (hash of a lattice code + Q-ID)
- HLC.swift:37 `struct HLC` — ordering coordinate across replicas
- Hamming.swift:37 `Hamming.distance(_:_:blocks:)` — structural-similarity primitive
- GSetAuditLog.swift:134 `struct GSetAuditLog` — CRDT audit trail

## Symbol Table

### Row shape and lifecycle
- Row.swift:19 `typealias RowId = UUID` — wire-identical to Rust `RowId(u128)`
- Row.swift:23 `struct Row: Sendable` — id, nounType, state, 3 bitmap columns, fingerprint, latticeAnchor, lineageId?, content?
- NounType.swift:12 `enum NounType: UInt8` — drawer=0/tunnel=1/kgFact=2/diaryEntry=3/proposal=4/association=5/learnedReference=6/ambientSample=7; wire-stable, never renumber
- RowState.swift:35 `enum RowState: UInt8` — 10 scale-gapped states (active=0..accepted=3, superseded=16..expired=19, rejected=32/tombstoned=33)
- RowState.swift:63 `enum RowStateCluster: UInt8` — a=active/becoming, b=superseded/historical, c=terminal
- RowState.swift:98 `RowState.cluster: RowStateCluster` — `(rawValue >> 4) & 0x3`; canonical active/retired partition, never re-derive by hand
- RowState.swift:90 `RowState.activeClusterUpperBoundRaw = 16` — the one boundary constant for storage-layer predicates that can't call `cluster`
- RowState.swift:115 `RowState.cluster(ofRawState:)` — classify from a raw byte, nil if undefined
- RowState.swift:122 `enum RowVerb: String` — 12 verbs the (SubstrateLib) automaton accepts
- RowState.swift:137 `enum RowStateError` — `.illegalTransition(RowState, RowVerb)` / `.violatesInvariant(String)`
- RowBitmaps.swift:37 `struct RowBitmaps: Sendable, Hashable, Codable` — adjective/operational/provenance Int64 columns
- RowBitmaps.swift:65 `RowBitmaps.field(_:) -> UInt8` — 36-field/6-bit uniform-grid view over the 3 named Int64 columns; UInt64 cast before shift (avoids Int64 arithmetic-shift sign bleed)
- RowBitmaps.swift:110 `RowBitmaps.fieldValues() -> [(field,value)]` — feeds MatrixO.applyRow / MatrixT.applyPair
- RowBitmaps.swift:151 `struct BitVector216: Sendable, Hashable` — dense 216-bit view; feeds MatrixF
- RowBitmaps.swift:167 `BitVector216.init(rowBitmaps:)` — real-row path, bits 60-71 always zero
- RowBitmaps.swift:191 `BitVector216.init(presenceBytes:)` — raw 27-byte harness/wire path, can set bits 60-71

### Lattice anchor (topic coordinate)
- LatticeAnchor.swift:12 `struct LatticeAnchor: Hashable, Sendable` — udcCode: UInt64, qidPointer: UInt64 (0 = null)
- LatticeAnchor.swift:21 `isNull: Bool` — both halves zero
- LatticeAnchor.swift:41 `LatticeAnchor.udc(_:)` — FNV-1a64 hash of a UDC string, null Q-ID
- LatticeAnchor.swift:50 `LatticeAnchor.udcQid(_:qid:)` — hashes both; empty qidString ⇒ same as `udc(_:)`

### Time coordinates
- HLC.swift:37 `struct HLC: Hashable, Sendable, Codable, Comparable` — (physicalTime, logicalCount, nodeID), lexicographic order
- HLC.swift:58 `HLC.zero` — sentinel earliest value
- HLC.swift:77 `HLC.advanced()` — bump logicalCount, hold physical/node
- HLC.swift:99 `HLC.packed: UInt64` / :108 `init(packed:)` — lossy 8-byte federation form (40-bit ms ≈ 34yr range)
- HLC.swift:206 `HLC.wireBytes` / :224 `init(wireBytes:)` — lossless 16-byte form
- HLC.swift:125 `struct HLCGenerator: Sendable` — per-replica HLC state machine (Kulkarni et al. 2014)
- HLC.swift:139 `HLCGenerator.send(now:)` — local-event timestamp; strictly monotonic per replica
- HLC.swift:156 `HLCGenerator.receive(remote:now:)` — merge rule on message receipt; underlies GSetAuditLog convergence
- TimeRange.swift:11 `struct TimeRange: Sendable, Equatable` — closed [start,end] HLC interval; init preconditions end >= start
- TimeRange.swift:24 `contains(_:) -> Bool` — inclusive both ends
- AsOfCoordinate.swift:18 `enum AsOfCoordinate: Hashable, Sendable, Codable` — `.present` | `.asOf(HLC)`; enum discriminant prevents zero-HLC ⇔ "now" ambiguity
- SnapshotId.swift:15 `struct SnapshotId: Hashable, Sendable, Codable` — UUID wrapper, type-distinct from RowId/drawer/estate ids

### Fingerprint construction
- Fingerprint256.swift:43 `struct Fingerprint256: Hashable, Sendable, Codable` — block0..block3: UInt64 (256 bits); block0=Bitmap-LSH, block1=Lattice-LSH, block2=Lineage+Temporal, block3=Channel+Source
- Fingerprint256.swift:59 `Fingerprint256.zero` — OR-identity / null-block value
- Fingerprint256.swift:88 `with(bit:set:) -> Fingerprint256` — value-semantic bit set/clear (replaces old mutating setBit)
- Fingerprint256.swift:120 `union(_:) -> Fingerprint256` — bitwise OR
- Fingerprint256.swift:153 `Fingerprint256.fromBits(_:)` — from [Bool] of 256
- Fingerprint256.swift:166/178 `wireBytes` / `init(wireBytes:)` — 32-byte LE canonical form; `toBytes()`/`fromBytes(_:)` are throws-free aliases
- Fingerprint256.swift:252 `zip4(_:_:)` — pairwise per-block binary op; the ONE unroll every block-wise op in this package (and Hamming/ORReduce/BitwiseArithmetic) must route through
- Fingerprint256.swift:270 `Fingerprint256.reduce4(_:_:)` — block-wise fold over a fingerprint sequence
- Fingerprint256.swift:281 `map4(_:)` — per-block unary op
- Fingerprint256.swift:296 `popcount() -> Int` — sum of 4 block popcounts
- Fingerprint256.swift:313/331 `zip4Batch` / `map4Batch` — vectorize across row arrays, not across one fingerprint's blocks
- HyperplaneFamily.swift:30 `struct Hyperplane: Sendable, Codable, Equatable` — ±1 plane as (positiveMask, negativeMask) bit-pair; 0/0 = sparse zero entry
- HyperplaneFamily.swift:53 `sign(over:) -> Bool` — popcount(v&pos) > popcount(v&neg); tie ⇒ false
- HyperplaneFamily.swift:68 `struct HyperplaneFamily: Sendable, Codable, Equatable` — exactly 64 planes, fixed blockIndex 0..3 + inputBitLength (192 for block0, 64 for 1-3)
- HyperplaneFamily.swift:95 `HyperplaneFamily.generate(seed:blockIndex:inputBitLength:density:)` — deterministic from 32-byte seed; CONSTITUTIONAL: seeds fixed at estate creation, never rotate
- HyperplaneFamily.swift:147 `canonicalHash() -> UInt64` — FNV-1a over canonical wire form, for pairing-audit labels
- HyperplaneFamily.swift:247 `HyperplaneFamily.blockFamilies(baseSeed:density:)` — canonical 4-family generator; fixes prior bug of reusing 1 seed across blocks + assuming uniform 64-bit width
- HyperplaneFamily.swift:261 `diversifiedSeed(base:blockIndex:)` / :273 `expandSeed64(_:)` — per-block seed derivation (SplitMix64-style)
- FloatSimHashPlanes.swift:30 `struct FloatSimHashPlanes: Sendable, Equatable` — materialized ±1 planes for the FLOAT-input SimHash path; lives here so SubstrateKernel (applies) and SubstrateML (generates) both reach it without a layering inversion
- SimHash.swift:37 `SimHash.block(over:family:) -> UInt64` — one 64-bit block: bit k = sign(<v, family.planes[k]>)
- SimHash.swift:52 `SimHash.fingerprint(bitmapInput:latticeInput:lineageTemporalInput:channelSourceInput:families:)` — assembles all 4 blocks; families order MUST be H_0..H_3
- SimHash.swift:74 `SimHash.fingerprintBatch(...)` — reference loop; hardware backends must match bit-for-bit
- SimHash.swift:102 `SimHash.fingerprint(fromSubhashes:hyperplanes:)` — 4-subhash convenience path
- SimHashInput (SimHash.swift:123) — :128 `bitmap(...)`, :140 `lattice(...)`, :156 `lineageTemporal(...)`, :178 `channelSource(...)` — fixed bit-offset input-vector assemblers per cookbook §3.2-3.5; offset mismatch ⇒ silently incompatible fingerprints

### Fingerprint algebra
- Hamming.swift:21 `typealias HammingDistance = Hamming`
- Hamming.swift:37 `Hamming.distance(_:_:blocks:) -> Int` — XOR+popcount; `.all` fast path is `zip4(^).popcount()`
- Hamming.swift:54 `Hamming.similarity(_:_:blocks:) -> Double` — 1 - distance/max, max = 64*blockCount
- BlockMask.swift:26 `struct BlockMask: OptionSet<UInt8>` — .block0/.block1/.block2/.block3, .all, .none; replaced a Set<Int>-per-call allocation hot-path
- BlockMask.swift:42 `blockCount: Int` — popcount of rawValue
- BitwiseArithmetic.swift:31 `intersect(_:_:)` — AND (zip4 &)
- BitwiseArithmetic.swift:45 `difference(_:_:)` — XOR (zip4 ^)
- BitwiseArithmetic.swift:63 `prototype(_:) -> Fingerprint256` — cohort majority vote via CountVector256.fold(...).majorityVote()
- BitwiseArithmetic.swift:77 `indirect enum FingerprintBuilder` — literal/intersect/difference/prototypeOf; `.evaluate()` walks the tree
- ORReduce.swift:31 `ORReduce.reduce(_:) -> Fingerprint256` — OR-fold, zero for empty (identity); delegates to `Fingerprint256.reduce4(_:|:)`
- ORReduce.swift:46 `ORReduce.reduce(_:blocks:defaults:)` — block-subset OR, unselected blocks pass through `defaults`
- CountVector256.swift:41 `struct CountVector256: Sendable, Equatable, Codable` — counts:[UInt32] (256), n:UInt32; wrapping arithmetic (`&+=`)
- CountVector256.swift:52 `CountVector256.zero` — fold/merge identity
- CountVector256.swift:75 `accumulate(_:)` — fold one fingerprint (leaf step)
- CountVector256.swift:94/103 `merge(_:)` / `+` — commutative/associative tree-fold step; order-independent
- CountVector256.swift:120 `majorityVote() -> Fingerprint256` — bit set iff `2*count > n` (strict; exact tie does NOT set); n==0 ⇒ zero
- CountVector256.swift:134 `profile() -> [Float]` — per-bit Bernoulli parameter, not stored, recomputed on demand
- CountVector256.swift:148 `CountVector256.fold(_:)` — reference whole-array accumulate; kernel-layer vectorized backends gate against this
- FNV.swift:25 `FNV.hash64(_:) -> UInt64` — offset 0xCBF29CE484222325, prime 0x100000001B3
- FNV.swift:40 `FNV.hash32(_:) -> UInt32` — INDEPENDENT hash family, not a truncation of hash64
- FNV.swift:57 `FNV.hash16(_:) -> UInt16` — low-16 fold of hash64 (NOT a from-scratch FNV variant)

### Integrity hashing
- MerkleDomain.swift:20 `enum MerkleDomain` — leaf=0x00, interior=0x01, tombstone=0x02, commitment=0x03; conformance-frozen forever
- ContentHash.swift:20 `struct ContentHash: Hashable, Sendable, Codable` — 32-byte SHA-256 leaf-payload digest; private storage, `bytes`/`hexString` accessors
- ContentHash.swift:45 `ContentHash.tombstone` — literal bytes (SHA256 of MerkleDomain.tombstone); literal because layer-1 can't import SubstrateKernel's SHA256
- MerkleRoot.swift:16 `struct MerkleRoot: Hashable, Sendable, Codable` — 32-byte subtree-summary hash; NOT interchangeable with ContentHash (type-distinct on purpose)
- MerkleRoot.swift:41 `MerkleRoot.empty` — literal bytes (SHA256 of MerkleDomain.interior), same layering reason

### Audit trail (CRDT)
- AuditEvent.swift:15 `struct AuditEvent: Sendable` — eventID+hlc = idempotence key; before/afterBitmaps, before/afterLatticeAnchor, actor, reason?
- AuditEvent.swift:63 `withReason(_:) -> AuditEvent` — attach a reason post hoc (AuditGate.admit itself is reason-less)
- GSetAuditLog.swift:35 `struct AuditEntry: Hashable, Sendable, Codable` — id = 32-byte content hash (SHA-256 over wire fields); dedupe key
- GSetAuditLog.swift:62 `enum AuditVerb: String` — 9 cookbook verbs + migrate/dreamCompact
- GSetAuditLog.swift:95 `enum AuditValue: Hashable, Sendable, Codable` — bitmap/string/fingerprint/integer; HAND-WRITTEN Codable ⇒ externally-tagged `{"bitmap":42}` shape matching Rust serde, NOT Swift's default `{"bitmap":{"_0":42}}`
- GSetAuditLog.swift:134 `struct GSetAuditLog: Sendable, Codable` — internal store keyed by id (O(1) dedupe); WIRE FORM is a sorted array (`{"entries":[...]}`), not the hash map
- GSetAuditLog.swift:180 `add(_:)` — idempotent insert
- GSetAuditLog.swift:187 `merge(_:)` — CRDT join = set union; commutative/associative/idempotent
- GSetAuditLog.swift:198 `orderedEntries` — full HLC-order replay (drives projection)
- GSetAuditLog.swift:204 `entries(forRow:)` — HLC-order, one row (drives row-state automaton in SubstrateLib)
- GSetAuditLog.swift:212 `entries(since:)` — HLC-order, exclusive cutoff (sync delta)

### Population-statistics matrices
- MatrixF.swift:24 `struct MatrixF: Sendable, Equatable` — 216-cell Int64 field-presence counts; NO decay
- MatrixF.swift:31 `fieldCount`/`bitsPerField`/`cellCount` — ALIASES onto RowBitmaps' constants (must match by definition)
- MatrixF.swift:76 `applyRow(delta:bitVector:)` — +1 capture / -1 expunge / two calls for mutate
- MatrixC.swift:26 `struct MatrixC: Sendable, Equatable` — 216-cell Float32 marginal probabilities; derived only, NO decay
- MatrixC.swift:67 `MatrixC.derive(from:nRows:)` — Int64→Double→Float32 cast order is the cross-language bit-identity contract; nRows==0 ⇒ all zero
- MatrixO.swift:38 `struct CooccurrenceKey: Hashable, Comparable, Sendable` — packed UInt32 (fieldI|valueI|fieldJ|valueJ, 8 bits each)
- MatrixO.swift:70 `struct MatrixO: Sendable, Equatable` — sorted sparse cell list (NOT a Dictionary — deterministic iteration/serialization); decay half-life 365 days (applied elsewhere, by MatrixDecay)
- MatrixO.swift:165 `applyRow(delta:fieldValues:)` — ALL ordered pairs incl. i==j; NOT required symmetric in storage (unordered pair ⇒ 2 cells)
- MatrixT.swift:43 `struct CausalityKey: Hashable, Comparable, Sendable` — packed UInt64, adds lagBucket (0..7) to CooccurrenceKey's shape
- MatrixT.swift:84 `struct MatrixT: Sendable, Equatable` — ASYMMETRIC by design (source→target ≠ target→source); decay half-life 90 days
- MatrixT.swift:89 `bucketEdgesMinutes = [1,2,4,8,16,32,64,128]` — log-spaced lag buckets
- MatrixT.swift:99 `MatrixT.lagBucket(forMinutes:)` — nil outside [1,256); applyPair is then a no-op
- ThreeDBitTensor.swift:36 `struct ThreeDBitTensor: Sendable` — 6 bit-slices (one per bit position), row-major within each slice; hot-path layout, ~27MiB at 1M rows
- ThreeDBitTensor.swift:71 `setValue(row:field:value:)` — precondition value < 64 (6-bit width, I-6)
- ThreeDBitTensor.swift:107 `scanFieldEquals(field:value:) -> [UInt8]` — O(rowCount/8) per bit-slice, NOT yet word-vectorized
- ThreeDBitTensor.swift:140 `reserveCapacity(_:)` — grow-only, no-op if not larger, preserves existing data

### Recall wire vocabulary
- RecallTypes.swift:55 `struct RecallScore: Equatable, Sendable` — (RowId, Float32); score meaning is PER-PRIMITIVE, normalize before combining
- RecallTypes.swift:69 `struct DistanceBreakdown: Equatable, Sendable` — lattice/fingerprint/temporal/bitmap contributions, each [0,1]
- RecallTypes.swift:89 `struct RecallResult: Sendable` — rows + breakdown + confidenceInterval? + primitiveName (drives RRF/MMR composition)
- RecallTypes.swift:112 `struct RowProjection: Sendable` — rowId/captureHLC/fingerprint/lattice/bitmaps/rowState; DELIBERATELY omits verbatim content + rung-2 metadata

## INVARIANTS / GOTCHAS

- LAYERING FLOOR: this package imports Foundation only. Never add an
  intra-SDK import here — that is what would invert the four-package split
  (SubstrateTypes → SubstrateKernel → SubstrateML, with SubstrateLib
  orchestrating all three). `ContentHash.tombstone` and `MerkleRoot.empty`
  are byte literals, NOT computed SHA-256 calls, for exactly this reason;
  a SubstrateKernel bridge test verifies each literal against a live hash.
- `Fingerprint256.zip4(_:_:)` is the ONE place the four-block unroll is
  expressed. `Hamming.distance`, `ORReduce.reduce`, and
  `BitwiseArithmetic.intersect/difference` all delegate to it. Do not
  reintroduce a hand-unrolled block0/block1/block2/block3 loop anywhere else.
- `RowBitmaps.field(_:)` casts to `UInt64` before shifting. Do not shift the
  raw `Int64` — arithmetic (sign-extending) right shift on a negative Int64
  bleeds the sign bit into the field value. Also guards `shift >= 64`
  explicitly; Swift masks shift amounts by 63 and would otherwise compute
  the wrong result for fields 11 (shift=66) silently.
- `RowState`'s ten raw values are scale-gapped (0-3 / 16-19 / 32-33) on
  purpose so `cluster` is `(raw >> 4) & 0x3`. Never classify active-vs-
  retired by a hand-rolled numeric boundary except via the one named
  constant `activeClusterUpperBoundRaw`, which exists only for storage
  predicates that cannot call `cluster(ofRawState:)`.
- `HyperplaneFamily` seeds are CONSTITUTIONAL: fixed at estate creation,
  never rotated. Rotating changes every fingerprint the estate has ever
  produced and breaks CRDT convergence across replicas. `blockFamilies`
  is the only sanctioned way to derive all four block families from one
  base seed — do not reuse one un-diversified seed across blocks.
- `CountVector256.majorityVote()` uses STRICT `2*count > n`; an exact tie
  does not set the bit. This convention is identical across every kernel
  backend and both language ports — do not change the inequality direction.
- `GSetAuditLog`'s wire format is a sorted array (`{"entries":[...]}`), not
  its internal `[id: AuditEntry]` dictionary. `AuditValue`'s Codable is
  hand-written to match Rust serde's externally-tagged shape. Do not let
  either fall back to Swift's default synthesized Codable.
- `MatrixF`/`MatrixC` have NO decay; `MatrixO` decays at 365-day half-life;
  `MatrixT` decays at 90-day half-life. Decay application itself lives
  outside this package (MatrixDecay); these types only hold and update raw
  counts/derived marginals.
- `MatrixC.derive(from:nRows:)` casts Int64 → Double → Float32, in that
  order, on both language ports. This is the cross-language bit-identity
  contract for the correlation matrix; do not shortcut through Float32
  division directly.
- `MatrixT` is asymmetric by design: `(i,j)` and `(j,i)` are independent
  cells. `MatrixO` is conceptually symmetric but stores both directions
  from `applyRow`'s ordered-pair iteration (including the diagonal, i==j).
- `RowId` is a bare `typealias` for `UUID` (wire-identical to Rust's
  `RowId(u128)`); `SnapshotId` is a real wrapper struct on purpose, so it is
  NOT interchangeable with a bare UUID at the type level. Do not "simplify"
  `SnapshotId` into another typealias.
- `ContentHash` and `MerkleRoot` are both 32-byte hashes but are distinct,
  non-interchangeable types (payload digest vs. subtree summary). Same
  pattern for `RecallScore.score` — meaning is per-primitive, never compare
  raw scores across primitives without normalizing first.
- No pinned data artifacts ship in this package (contrast with LatticeLib).
  Every value is either caller-supplied (a seed, a row's fields) or a pure
  function of caller-supplied input. Reproducibility rests entirely on the
  arithmetic being pinned and mirrored in `rust/src/`, checked by matched
  unit-test vectors per module, not a shared fixtures directory.
- `Row`, `AuditEvent`, `RowState`/`RowVerb`/`RowStateError`, `NounType`,
  and `LatticeAnchor` are pure data only. The row-state automaton
  (transition table + validation), the verb implementations, and the
  full multi-row substrate object all remain in SubstrateLib. Do not
  add behavior to these types here.
