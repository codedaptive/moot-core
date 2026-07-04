---
doc: DETAILS
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

# SubstrateTypes Details

This document walks through every source file in the package. Read
`OVERVIEW.md` first for the big picture. Files appear here in conceptual
order: the row shape and its lifecycle, the two coordinate systems a row
carries (lattice anchor and time), the fingerprint construction pipeline and
its algebra, integrity hashing, the audit trail, the population-statistics
matrices, and finally the recall wire vocabulary.

## NounType.swift

This file provides `NounType`, an eight-case enum naming the kind of thing a
row holds: a drawer, a tunnel, a knowledge-graph fact, a diary entry, a
proposal, an association, a learned reference, or an ambient sample.

The enum is backed by `UInt8` raw values (0 through 7) rather than a string,
because the noun type is stored in a compact wire format alongside the rest
of a row. The file's opening comment states the values are wire-stable and
must never be renumbered: a stored row's noun type is read back by its raw
number, so changing a number would silently reclassify every row already on
disk.

## RowState.swift

This file provides the three pure-data types that describe a row's
lifecycle: `RowState` (the ten states a row can be in), `RowVerb` (the twelve
mutations the row-state automaton accepts), and `RowStateError` (the typed
failure a rejected transition returns). The automaton that enforces which
transitions are legal is compute, not data, so it remains one layer up, in
SubstrateLib; this file only names the states and verbs the automaton
operates on.

The ten states use deliberately spaced raw values — 0 to 3, then a gap, then
16 to 19, another gap, then 32 and 33 — rather than a plain 0-to-9 count.
This spacing lets any consumer recover a state's lifecycle cluster with one
shift-and-mask operation instead of a lookup table. `RowState.cluster` reads
`(rawValue >> 4) & 0x3` to classify a state as cluster A (active or
becoming: active, pending, contested, accepted), cluster B (superseded or
historical: superseded, decayed, withdrawn, expired — retired but
revivable), or cluster C (terminal: rejected, tombstoned — retired and
final). `RowState.isActiveCluster` is the convenience most callers want: a
single true-or-false answer to "is this row currently believed."

`RowState.cluster(ofRawState:)` performs the same classification starting
from a raw `UInt8` rather than a typed `RowState`, for code that has only
read a bare byte out of a persisted bitmap and has not decoded it yet.
`RowState.activeClusterUpperBoundRaw` names the one boundary value (16) a
storage-layer predicate can compare against directly — for example, in a SQL
`WHERE` clause — when it cannot call a Swift function; the file is explicit
that any code holding a decoded `RowState` should prefer `cluster` over a
hand-rolled numeric comparison, because a future state added inside one of
the gaps would silently misclassify under a magic-number boundary but not
under the shift-and-mask.

`RowState.description` and `RowStateError.description` supply the exact
lowercase English strings ("active", "pending", and so on, and the compact
"active --reject-->" form) that downstream parsers depend on for parity with
the Rust port's `Display` implementation. Both are written as explicit
`switch` statements rather than the default `String(describing:)`, because
calling that default from inside a type's own `description` recurses forever.

## RowBitmaps.swift

This file provides `RowBitmaps`, the named layout over a row's three bitmap
columns, and `BitVector216`, a dense 216-bit view over that same data for
consumers that want to treat it as a flat grid.

A row carries three `Int64` columns — adjective, operational, and
provenance — and each column packs several independently named fields at
specific bit ranges. For some purposes (updating the population-statistics
matrices), it is easier to think of these three columns as one uniform grid
of thirty-six fields of six bits each, even though the real fields are not
laid out that uniformly. `RowBitmaps.field(_:)` performs this translation:
given a field index from 0 to 35, it locates the right column and bit
offset and returns that field's six-bit value. The function carries two
defensive casts the comment explains in detail: a shift-amount guard,
because Swift silently wraps an out-of-range shift instead of returning
zero, and a cast to `UInt64` before shifting, because right-shifting a
negative `Int64` sign-extends and would leak the sign bit into a field's
value.

`RowBitmaps.bit(field:bit:)` reads one bit out of a field's six-bit value.
`RowBitmaps.fieldValues()` returns all thirty-six (field, value) pairs in
order, which is what `MatrixO.applyRow` and `MatrixT.applyPair` consume to
update the co-occurrence and causality matrices. `RowBitmaps.bitVector()`
builds a `BitVector216` from a live row.

`BitVector216` has two constructors for two different sources of the same
216-bit shape. `init(rowBitmaps:)` builds it from a real row through
`RowBitmaps.field(_:)`; because real rows never set bits above position 71
of the abstract grid, this path is safe for updating `MatrixF` from row
capture and expunge events. `init(presenceBytes:)` builds it directly from a
raw 27-byte pattern that may set any of the 216 bits, including ones no real
`RowBitmaps` value can represent — the shape a conformance-test harness
vector uses. `bit(at:)` and `bit(field:bit:)` read one bit by absolute index
or by (field, bit) coordinate.

## Row.swift

This file provides `Row`, the substrate's central data shape, and `RowId`,
a type alias for `UUID` used wherever the substrate vocabulary calls for a
row identifier.

`RowId` is a plain alias rather than a wrapper type because a `UUID`'s
16-byte big-endian wire form is already byte-identical to the Rust port's
`RowId(u128)` newtype; no translation step is needed at the boundary, and
callers can use `UUID`'s existing API directly.

`Row` itself is a value type holding one row's complete current state: its
identifier, noun type, lifecycle state, three bitmap columns, fingerprint,
lattice anchor, and two optional fields — a lineage identifier, linking a
row to the row it was derived from, and the verbatim content. The file is
explicit that `Row` is pure data: no logic and no I/O. The full substrate
object that bundles many rows together with the audit log and the
statistics matrices, and the verb functions that mutate a row's state,
remain in SubstrateLib. `Row` only says what a row looks like at rest.

## LatticeAnchor.swift

This file provides `LatticeAnchor`, the sixteen-byte reference a row carries
to its position in the classification lattice: an eight-byte hash of a
lattice code plus an eight-byte hash of a concept identity (a Q-ID), or zero
for either half when absent.

A lattice anchor does not store the lattice code text itself — it stores a
deterministic hash of it. `LatticeAnchor.fnv1a64(_:)` is the private FNV-1a
64-bit hash used for this purpose; because it is the exact FNV-1a algorithm
(matching `FNV.hash64` elsewhere in the package), two callers hashing the
same code string always agree, and the result matches the Rust port bit for
bit.

`LatticeAnchor.udc(_:)` builds an anchor from a lattice code string alone,
with a null Q-ID pointer. `LatticeAnchor.udcQid(_:qid:)` builds an anchor
that carries both the lattice code and a specific concept identity; the file
explains why this second form matters: without it, every row about a broad
subject collapses onto one anchor value, losing the distinction between
different things classified under the same broad code. An empty Q-ID string
yields the same null pointer as `udc(_:)`, so the two constructors agree
when no concept identity is available. `LatticeAnchor.isNull` reports
whether both halves are zero — the anchor equivalent of "unclassified."

## HLC.swift

This file provides `HLC`, the Hybrid Logical Clock timestamp that orders
events across replicas of an estate without requiring synchronized wall
clocks, and `HLCGenerator`, the per-replica state machine that produces
HLC values.

An `HLC` is three fields compared in order: `physicalTime` (milliseconds
since the Unix epoch), `logicalCount` (a counter that advances when
physical time does not), and `nodeID` (a per-replica tiebreaker). `HLC.<`
compares these three fields lexicographically, which gives any two HLC
values an unambiguous order even when two replicas' clocks briefly agree or
run backward relative to each other. `HLC.zero` is the earliest possible
value, used as a sentinel. `HLC.advanced()` returns a copy with the logical
counter incremented, holding physical time and node fixed.

`HLCGenerator.send(now:)` produces the timestamp for a locally originated
event: if the wall clock has moved forward since the last call, it adopts
the new time and resets the logical counter to zero; otherwise it holds the
physical time and bumps the logical counter, guaranteeing every timestamp
this replica emits is strictly greater than the last. `HLCGenerator.receive(remote:now:)`
implements the HLC paper's merge rule for an event arriving from another
replica: it takes the maximum of the local clock, the remote clock, and the
current wall time, and sets the logical counter according to which of the
three inputs was tied for that maximum. This is the operation that lets two
replicas exchange audit events and converge on one shared sense of "what
happened before what," which the convergence proof in `GSetAuditLog.swift`
depends on.

`HLC.packed` and `init(packed:)` convert to and from a lossy 8-byte form
used by federation tier-contribution messages, trading precision (40 bits of
milliseconds, roughly 34 years of range) for a smaller wire size. `HLC.wireBytes`
and `init(wireBytes:)` are the lossless 16-byte round trip used elsewhere.
The file also carries a `nodeId`-spelled initializer overload, kept only so
older call sites written against that casing keep compiling unchanged.

## TimeRange.swift

This file provides `TimeRange`, a closed interval `[start, end]` of two HLC
values.

`TimeRange.init(start:end:)` rejects an interval whose end precedes its
start with a precondition, because an inverted range has no sensible
meaning for any caller. `TimeRange.contains(_:)` reports whether a given HLC
falls within the closed interval, inclusive of both endpoints — the
predicate a caller uses to test "did this event happen during this window."
The file notes that the higher-level primitive consuming these windows,
`MomentSummary`, stays in SubstrateLib; this type is only the interval
shape.

## AsOfCoordinate.swift

This file provides `AsOfCoordinate`, a two-case enum that says whether a
read should return the live state (`.present`) or the state as of a
specific HLC (`.asOf(hlc)`).

The file explains the bug this type prevents: if "present" were represented
by a sentinel HLC value such as zero, a caller could not distinguish "give
me now" from "give me the state at the dawn of time." Making the choice an
explicit enum discriminant removes that ambiguity by construction. The
custom `Codable` conformance encodes the case as an explicit `"kind"` field
(`"present"` or `"asOf"`) plus, for the second case, the HLC itself, rather
than relying on Swift's default enum encoding, so the wire form stays stable
and self-describing across the Swift and Rust legs.

## SnapshotId.swift

This file provides `SnapshotId`, a `UUID` wrapper identifying one
point-in-time snapshot of an estate.

Unlike `RowId`, which is a bare type alias, `SnapshotId` is a distinct
struct. The file states the reason: a snapshot identifier must not be
substitutable for a drawer, node, or estate identifier at the type level,
even though all of these are UUIDs underneath. `init(uuidString:)` and the
`uuidString` accessor round-trip through the standard UUID string form;
`Codable` conformance encodes and decodes that same string form, throwing
`SnapshotIdError.invalidUUID` on a malformed string rather than crashing.

## BlockMask.swift

This file provides `BlockMask`, an `OptionSet` selecting any subset of the
fingerprint's four 64-bit blocks.

Before this type existed, callers passed a `Set<Int>` to select blocks,
which allocated memory on every call — a measured hot-path cost in nearest-
neighbor search over large row collections. `BlockMask` replaces that with
four fixed bit flags (`.block0` through `.block3`), an `.all` convenience
covering every block, and an empty `.none`. Because it is a plain integer
under the hood, membership tests are branchless and allocation-free.
`BlockMask.blockCount` counts how many blocks are selected (0 through 4),
which `Hamming.similarity` uses to compute the correct maximum-distance
denominator for a partial-block comparison.

## Fingerprint256.swift

This file provides `Fingerprint256`, the substrate's 256-bit structural
fingerprint, along with the pure combinators that other files in this
package build on.

A fingerprint is four independent 64-bit blocks — `block0` through
`block3` — each a SimHash over a different aspect of a row: its bitmaps,
its lattice position, its lineage and timing, and its channel and source.
The file is careful to distinguish a fingerprint (the coordinate for
structural similarity) from a lattice anchor (the coordinate for topic
similarity); the two answer different questions and neither substitutes for
the other.

`Fingerprint256.bit(at:)` and `Fingerprint256.with(bit:set:)` give bit-level
read and copy-on-write access across the full 256-bit span, translating a
flat index into the right block automatically. `with(bit:set:)` returns a
new value rather than mutating in place, matching the package's general
preference for value semantics over in-place mutation. `Fingerprint256.union(_:)`
computes the bitwise OR of two fingerprints — set-union over the bits each
one has set. `Fingerprint256.fromBits(_:)` builds a fingerprint from a
256-element array of booleans, mainly useful for tests and for converting
from an external bit representation. `wireBytes` and `init(wireBytes:)`
convert to and from the canonical 32-byte little-endian wire form;
`toBytes()` and `fromBytes(_:)` are thin aliases kept for parity with call
sites that expect a throws-free, optional-returning variant.

The file's closing section defines a small combinator layer:
`zip4(_:_:)` applies a binary operation to each of the four block pairs
between two fingerprints in one call, `map4(_:)` applies a unary operation
to all four blocks of one fingerprint, `reduce4(_:_:)` folds a whole
sequence of fingerprints block-wise with a binary operation, and
`popcount()` counts set bits across all four blocks. These four functions
exist because the same "unroll across four blocks" pattern previously
appeared, hand-written, in seven or more places across the codebase — this
package's `ORReduce`, `BitwiseArithmetic`, and `Hamming` among them.
Expressing the unroll once here, and asking every other block-wise operation
to call it, means a future change to how blocks are combined only has one
call site to change, with no risk that one of the seven repeated copies
falls out of sync with the rest. `zip4Batch(_:_:_:)` and `map4Batch(_:_:)`
extend the same idea across an array of fingerprints, for callers vectorizing
across many rows rather than across one fingerprint's four blocks.

## HyperplaneFamily.swift

This file provides `Hyperplane`, one ±1-valued dividing plane used by
SimHash, and `HyperplaneFamily`, the fixed set of sixty-four such planes
that produce one fingerprint block.

A hyperplane of length *n* would normally be an array of *n* values, each
+1, −1, or 0. Because binary inputs make this arithmetic reduce to counting
set bits, `Hyperplane` instead stores two bitmasks — `positiveMask` and
`negativeMask` — with a 1 wherever the plane is +1 or −1 respectively, and 0
in both where the plane's value is 0 at that position (a "sparse
hyperplane"). `Hyperplane.sign(over:)` computes whether the plane's dot
product with a binary input vector is strictly positive, by comparing the
popcount of the input ANDed with each mask; a tied dot product resolves to
false, which the file notes is vanishingly rare in practice because the
per-block input has enough bits that an exact tie is unlikely.

`HyperplaneFamily.generate(seed:blockIndex:inputBitLength:density:)` builds
a family of sixty-four hyperplanes deterministically from a 32-byte seed
using a private, non-cryptographic pseudo-random generator
(`HyperplanePRNG`). Determinism here is load-bearing: the file states that
hyperplane seeds are fixed once, at estate creation, and never rotate,
because rotating them would change every fingerprint an estate has ever
computed and break cross-replica sync. The `density` parameter controls
what fraction of a plane's positions are active (nonzero) rather than zero;
at density 1.0 every position is active, and the code takes a direct path
around a floating-point computation that would otherwise round up past
`UInt64.max` and trap.

`HyperplaneFamily.canonicalHash()` computes a stable 64-bit fingerprint of
a whole family's content by folding every mask word through FNV-1a, used to
label a pairing arrangement in an audit trail without storing the full
family. `HyperplaneFamily.blockFamilies(baseSeed:density:)` is the single
routine that builds all four block families — for block 0 (192 bits) and
blocks 1 through 3 (64 bits each) — from one base seed, fixing two mistakes
an earlier, separately-derived generation path had: reusing one seed across
all four blocks (which collapsed them into one projection instead of four
independent ones) and assuming a uniform 64-bit width for every block
(block 0 is 192 bits). `diversifiedSeed(base:blockIndex:)` mixes a block
index into the base seed so each block gets an independent 32-byte seed, and
`expandSeed64(_:)` stretches a 64-bit seed back out to 32 bytes through four
rounds of a SplitMix64-style mixing function.

## FloatSimHashPlanes.swift

This file provides `FloatSimHashPlanes`, the materialized hyperplane set for
the floating-point-input variant of SimHash — the float analogue of
`HyperplaneFamily`, which serves the binary-input variant.

The file explains why this type lives in SubstrateTypes rather than in
either of the two packages that use it: `SubstrateKernel` applies these
planes (pure signed-sum-and-sign arithmetic, no randomness) and
`SubstrateML` generates them from a seed (using the SplitMix64 random
generator). Placing the shared data shape at the lowest layer lets both
higher packages depend on it without either depending on the other, which
would otherwise invert the SDK's layering. `FloatSimHashPlanes` packs
`256 * dim` sign bits — one per (hyperplane, coordinate) pair — into a flat
array of 64-bit words, row-major by hyperplane, matching the draw order the
generator produces them in. Because the planes are immutable once generated
for a given seed and dimensionality, a caller materializes one instance per
(seed, dimension) pair and reuses it across every projection under that seed.

## SimHash.swift

This file provides `SimHash`, the construction that turns an input vector
and a hyperplane family into one 64-bit fingerprint block, plus
`SimHashInput`, the set of helpers that assemble the four blocks' input
vectors from a row's actual fields.

`SimHash.block(over:family:)` is the core operation: for each of the
family's sixty-four hyperplanes, it sets one output bit if the hyperplane's
dot product with the input vector is positive. `SimHash.fingerprint(bitmapInput:latticeInput:lineageTemporalInput:channelSourceInput:families:)`
calls this once per block and assembles the four results into a complete
`Fingerprint256`. `SimHash.fingerprintBatch(...)` is the same computation
repeated across many rows, returning results in input order; the file notes
that this reference loop is what a hardware-accelerated backend elsewhere in
the SDK must reproduce bit for bit, even though that backend vectorizes the
popcount work across many rows at once for speed.
`SimHash.fingerprint(fromSubhashes:hyperplanes:)` is a convenience path for
callers that have already reduced each block's input down to one 64-bit
subhash rather than a full input vector.

`SimHashInput.bitmap(adjective:operational:provenance:)` concatenates a
row's three bitmap columns into block 0's 192-bit input.
`SimHashInput.lattice(udcPrefixHash:qidDirectHash:qidClosureHash:)` packs
three hash components into block 1's 64-bit input at fixed bit offsets.
`SimHashInput.lineageTemporal(...)` and `SimHashInput.channelSource(...)`
do the same for blocks 2 and 3. Each function's bit-offset layout is fixed
by the cookbook specification the file cites; a mismatched offset here would
silently produce fingerprints that no longer compare correctly against ones
built the standard way, so these helpers exist precisely to keep every
caller assembling the same layout.

## Hamming.swift

This file provides `Hamming`, the substrate's primary structural-similarity
measurement: how many bit positions differ between two fingerprints.

`Hamming.distance(_:_:blocks:)` computes this by XORing the two
fingerprints and counting the set bits in the result — `a.zip4(b, ^).popcount()`
on the common case of all four blocks, or a per-block loop when the caller
restricts the comparison to a `BlockMask` subset. Restricting to a subset
answers a more specific question, such as "how similar are these two rows
in lattice position alone, ignoring timing and provenance." `Hamming.similarity(_:_:blocks:)`
converts a distance into a `[0, 1]` score, where 1.0 means identical and 0.0
means maximally distant over the selected blocks. `HammingDistance` is a
type alias re-exporting `Hamming` under the name some call sites use.

## BitwiseArithmetic.swift

This file provides three fingerprint operations beyond OR-reduction —
intersection, symmetric difference, and a cohort prototype — plus
`FingerprintBuilder`, a small expression type for composing them
declaratively.

`BitwiseArithmetic.intersect(_:_:)` computes the bitwise AND of two
fingerprints: the bits both rows share. `BitwiseArithmetic.difference(_:_:)`
computes the bitwise XOR: the bits where the two disagree. Both delegate to
`Fingerprint256.zip4(_:_:)`. `BitwiseArithmetic.prototype(_:)` computes the
bit-for-bit majority vote across a whole cohort of fingerprints — the
fingerprint of a "typical" member — by folding the cohort into a
`CountVector256` and reading off its majority-vote view; the file notes this
replaced an earlier hand-written per-bit counting loop with an equivalent
call into the one canonical cohort-fold primitive.

`FingerprintBuilder` is an indirect enum with four cases — a literal
fingerprint, an intersection of two sub-expressions, a difference of two
sub-expressions, or the prototype of a fingerprint list — and one method,
`evaluate()`, that walks the tree and computes the result. This lets a
caller build up a compound fingerprint query as data (for example,
"the intersection of Bob's typical pattern and Amelia's typical pattern")
and evaluate it once, rather than nesting direct function calls.

## ORReduce.swift

This file provides `ORReduce`, the substrate's universal aggregation
primitive over collections of fingerprints.

`ORReduce.reduce(_:)` computes the bitwise OR across a whole sequence of
fingerprints, returning `Fingerprint256.zero` for an empty sequence (OR's
identity element); it delegates to `Fingerprint256.reduce4(_:|:)`.
`ORReduce.reduce(_:blocks:defaults:)` restricts the reduction to specific
blocks, filling any block outside the selection from a supplied default —
useful when a caller wants only the topic-block aggregate across a cohort,
say, without touching the other three blocks. The file explains why OR
reduction matters beyond simple aggregation: because it is commutative,
associative, and idempotent, it works as the join operator for a CRDT of
fingerprints — replicas can merge any number of contributions, in any
order, with any duplication, and converge on the same result. It is also
a privacy mechanism: once several fingerprints are OR-reduced together, a
reader can see which structural patterns appear somewhere in the group but
cannot recover which specific contributor set which bit.

## CountVector256.swift

This file provides `CountVector256`, the per-bit counting accumulator that
underlies cohort-level fingerprint statistics, including the majority-vote
prototype `BitwiseArithmetic.prototype(_:)` reads from it.

The file's opening comment makes the key design argument: a majority-vote
summary does not compose. If node A's majority is computed from its
children, and node B's majority from its children, the majority of "A's
majority and B's majority" is not generally the same as the true majority
over every leaf under both A and B combined. A count vector does compose:
counts add, member totals add, and the sum is exact regardless of the order
members are folded in. So the count vector, not the majority-vote
fingerprint, is the object worth storing at every level of a tree; the
majority-vote fingerprint is a read-time view computed from it whenever a
caller actually needs a Hamming-comparable engram.

`CountVector256.accumulate(_:)` folds one fingerprint into the vector: for
every set bit, its counter increases by one, and the member count increases
by one. `CountVector256.merge(_:)` and its operator form `+` combine two
count vectors by adding element-wise — the tree-fold step, safe to apply to
a node's children in any order because addition is commutative and
associative. `CountVector256.majorityVote()` reads off the fingerprint whose
bit `j` is set exactly when a strict majority of accumulated members had bit
`j` set (`2 * count > n`; an exact tie at half does not set the bit, a
convention the file calls out as identical across every kernel backend and
both language ports). `CountVector256.profile()` returns the 256
per-bit probabilities instead of a threshold, for callers that want the
Bernoulli parameter of each bit rather than a hard yes-or-no answer. The
static `CountVector256.fold(_:)` is the reference implementation of folding
a whole array of fingerprints at once, which a vectorized kernel-layer
backend is required to match exactly.

## FNV.swift

This file provides `FNV`, the Fowler–Noll–Vo 1a string hash family the
substrate uses everywhere a deterministic string hash is required: drawer
fingerprints, manifest-derived identifiers, and deterministic tokenization.

`FNV.hash64(_:)` and `FNV.hash32(_:)` are two independent FNV-1a hashes with
different offset bases and primes — the file is explicit that the 32-bit
result is not a truncation of the 64-bit one, but a wholly separate
computation over the same input bytes. `FNV.hash16(_:)` is different in
kind: it is the low 16 bits of `hash64(_:)`, not a from-scratch FNV-1a
variant (FNV-1a has no official 16-bit definition), used where the substrate
needs a compact prefix hash such as a lattice sub-hash.

## MerkleDomain.swift

This file provides `MerkleDomain`, the four one-byte tags prepended before
hashing to keep different kinds of nodes in the Merkle content-integrity
tree from ever colliding: `leaf` (a single drawer's content and vectors),
`interior` (a parent summarizing its children), `tombstone` (an expunged
payload), and `commitment` (a keyed HMAC-SHA256 commitment). Domain
separation means a leaf hash and an interior hash can never be mistaken for
each other even if their underlying bytes happened to coincide, because
each was computed with a different one-byte prefix. The file states these
four values are frozen and must match exactly between the Swift and Rust
legs, permanently.

## ContentHash.swift

This file provides `ContentHash`, the 32-byte SHA-256 digest over one leaf
payload — a drawer's content plus its vectors.

The type stores its 32 bytes privately, exposed only through the `bytes`
accessor, so the fixed size is enforced by construction rather than by
convention. `ContentHash.tombstone` is a named constant: the SHA-256 hash of
the bare tombstone domain tag byte, used as the sentinel content hash for an
expunged (hard-deleted) drawer. The file explains why this constant is a
literal byte array rather than a value computed at runtime: SubstrateTypes,
as the lowest layer, cannot import SubstrateKernel (which owns the SHA-256
implementation) without inverting the SDK's dependency direction, so the
precomputed literal stands in, checked against the real computation by a
bridge test in SubstrateKernel. `hexString` and `description` render the
bytes as lowercase hex; the custom `Codable` conformance encodes and decodes
that same hex string, validating its length and character set on decode
rather than trusting external input. The file is explicit that `ContentHash`
is not interchangeable with `MerkleRoot`, even though both are 32-byte
hashes: one is a payload digest, the other a subtree summary, and the type
system keeps them from being swapped by mistake.

## MerkleRoot.swift

This file provides `MerkleRoot`, the 32-byte hash summarizing an interior
node's children in the Merkle content-integrity tree — the counterpart to
`ContentHash`'s single-payload digest.

Its shape mirrors `ContentHash` closely: private byte storage, a `bytes`
accessor, hex rendering, and the same style of custom `Codable`
implementation. `MerkleRoot.empty` is the named constant for the hash of a
node with no live children — the SHA-256 of the bare interior domain tag —
again stored as a literal for the same layering reason `ContentHash.tombstone`
is. Keeping `MerkleRoot` and `ContentHash` as two distinct types, rather than
one hash type used for both purposes, means a function that expects a
subtree summary cannot accidentally be handed a single payload's digest; the
compiler catches the mistake instead of a runtime bug surfacing later.

## AuditEvent.swift

This file provides `AuditEvent`, a single recorded mutation of a row: the
before-and-after bitmap and lattice-anchor state, which verb performed it,
who performed it, and when.

`AuditEvent.eventID`, paired with `hlc`, gives every event a compound key
that makes replaying the same event twice — for example, after a sync
retry — a safe no-op rather than a duplicate. `beforeBitmaps` is optional
because a row's very first event, its capture, has no prior state to record.
`reason` is an optional human-readable explanation threaded from the call
site that performed the mutation; the file notes it is populated for
explicit actions such as an expunge but left `nil` for the great majority of
routine mutations. `AuditEvent.withReason(_:)` returns a copy of an event
with a caller-supplied reason attached, used because the structural
validator that first produces an event (`AuditGate.admit`, in SubstrateLib)
is a pure checker with no concept of "why," so the reason is layered on
afterward by the verb that called it.

## GSetAuditLog.swift

This file provides `GSetAuditLog`, the substrate's append-only audit log,
and its two supporting types, `AuditEntry` (one immutable log row) and
`AuditValue` (a typed field value inside an entry).

The log is the substrate's source of truth: a row's visible current state is
a projection computed by replaying its log entries in order, never stored
independently of the log. `AuditEntry.id` is a 32-byte SHA-256 content hash
computed over the entry's other fields, which gives the log a natural way to
deduplicate: two replicas that independently record the same logical
mutation compute the same id and, once merged, keep only one copy.
`AuditValue` is an enum covering the four kinds of value a field change can
carry — a bitmap, a string, a fingerprint, or an integer — with a hand-written
`Codable` conformance that encodes each case as a single-key JSON object
(`{"bitmap": 42}`) rather than Swift's default synthesized shape
(`{"bitmap": {"_0": 42}}`), so the wire format matches what the Rust port's
serde derive produces natively and both legs agree on one JSON shape.

`GSetAuditLog` itself stores its entries keyed by content hash internally,
for O(1) deduplication, but its `Codable` conformance serializes them as a
plain array sorted by id, because the conceptually correct wire
representation of a grow-only set is a set, not a hash map — the file
explains this distinction explicitly to justify writing custom `Codable`
rather than relying on the default. `GSetAuditLog.add(_:)` inserts one
entry, a no-op if the same id is already present. `GSetAuditLog.merge(_:)`
is the CRDT join: the union of two logs' entries, correct regardless of
which replica calls it or in what order two logs are merged.
`GSetAuditLog.orderedEntries` replays every entry in HLC order, the
sequence a projection applies to compute visible state.
`GSetAuditLog.entries(forRow:)` and `GSetAuditLog.entries(since:)` scope
that same ordered replay to one row or to everything after a cutoff,
serving the row-state automaton and the sync protocol respectively. The
file's closing comment sketches why this design converges: G-Set merge is
set union, set union is commutative, associative, and idempotent, and HLC
gives any two entries an unambiguous order to replay them in — so any two
replicas that have exchanged all of each other's entries compute identical
visible state.

## MatrixF.swift

This file provides `MatrixF`, the field-presence matrix: for every
(field, bit) pair, how many rows in the estate currently have that bit set.

`MatrixF` stores 216 `Int64` counts in one flat array, indexed by
`field * bitsPerField + bit`; `MatrixF.cellIndex(field:bit:)` computes that
index with bounds checking. Its layout constants (`fieldCount`,
`bitsPerField`, `cellCount`) are aliases onto the identical constants in
`RowBitmaps`, kept as aliases rather than a second independent definition so
the two types cannot silently drift out of agreement about the shape of the
216-cell grid they both describe. `MatrixF.applyRow(delta:bitVector:)`
updates every cell whose bit is set in a `BitVector216`, adding `delta` —
positive one on a row's capture, negative one on its expunge, or the pair of
calls needed to represent a mutation as a removal followed by an addition.
`MatrixF.totalCount` sums every cell as a sanity check.
`MatrixF.writeWire(into:)` and `MatrixF.readWire(_:)` serialize the matrix
to and from its canonical 1,728-byte little-endian wire form (216 cells of 8
bytes each).

## MatrixC.swift

This file provides `MatrixC`, the correlation matrix: the marginal
probability of each (field, bit) pair, derived from `MatrixF` rather than
updated independently.

`MatrixC` shares `MatrixF`'s 216-cell shape but stores `Float` values in
`[0, 1]` instead of raw counts. `MatrixC.derive(from:nRows:)` is its only
way to change: it divides each of `MatrixF`'s counts by the total row count,
routing the division through `Double` before narrowing back to `Float32`,
which the file notes is necessary for cross-language agreement — both the
Swift and Rust legs perform the identical cast-divide-cast sequence, and
IEEE-754 guarantees the same bit pattern results on both. When there are no
rows at all, every cell is zero rather than an undefined division result.
`MatrixC.writeWire(into:)` and `MatrixC.readWire(_:)` serialize to and from
an 864-byte wire form (216 cells of 4 bytes each, `Float32` bit patterns).

## MatrixO.swift

This file provides `MatrixO`, the co-occurrence matrix: how often each pair
of (field, value) settings appears together across rows, and
`CooccurrenceKey`, the packed key identifying one such pair.

Because most of the roughly 46,000 possible (field, value, field, value)
combinations never occur in a typical estate, `MatrixO` stores only nonzero
cells, as a list sorted by `CooccurrenceKey.packed` rather than as a dense
array or an unordered dictionary — sorted order makes both iteration and
serialization deterministic across languages, which an unordered dictionary
could not guarantee. `CooccurrenceKey.packed` folds all four components
into one `UInt32` for compact storage and for the ordering comparison.
`MatrixO.count(_:)` and `MatrixO.increment(_:by:)` use binary search over
the sorted entries to read or update one cell in logarithmic time, and
`increment` removes a cell entirely if its count returns to zero, keeping
the canonical form free of dead entries. `MatrixO.applyRow(delta:fieldValues:)`
updates the matrix for one row: it iterates every ordered pair of the row's
(field, value) settings, including a field paired with itself, and
increments each pair's cell by `delta`. The file notes explicitly that the
matrix is not required to be symmetric in storage even though co-occurrence
is conceptually symmetric — each unordered pair contributes to two distinct
cells, (i, j) and (j, i) — and that an implementation which stores only one
direction and infers the other is a valid optimization, but the reference
here stores both for clarity. `MatrixO.writeWire(into:)` and
`MatrixO.readWire(_:)` define the canonical wire form: an entry count
followed by each entry's packed key and count.

## MatrixT.swift

This file provides `MatrixT`, the temporal causality matrix: how often a row
with one (field, value) setting is followed, within a bounded time window,
by a row with another (field, value) setting — the substrate's tool for
distinguishing "these things happen together" from "one of these things
tends to precede the other."

`CausalityKey` extends `CooccurrenceKey`'s idea with a fifth component, a
lag bucket, and packs all five into a `UInt64`. `MatrixT.lagBucket(forMinutes:)`
converts a raw time difference in minutes into one of eight log-spaced
buckets — 1, 2, 4, 8, 16, 32, 64, and 128 minutes as lower bounds — returning
`nil` for a difference outside the supported 1-to-256-minute window, in
which case no update to the matrix occurs at all. The file states this
matrix is explicitly asymmetric: the cell for "(field i, value i) preceding
(field j, value j)" is tracked completely separately from the reverse
direction, unlike `MatrixO`'s conceptually symmetric co-occurrence. Storage,
lookup (`count(_:)`), update (`increment(_:by:)`), and wire serialization
follow the identical sorted-array, binary-search pattern `MatrixO` uses.
`MatrixT.applyPair(delta:rowAFieldValues:rowBFieldValues:lagMinutes:)` is the
row-pair version of `MatrixO.applyRow`: given that row A precedes row B by a
known number of minutes, it increments the cell for every combination of A's
field values against B's field values, once a valid lag bucket exists for
that separation.

## ThreeDBitTensor.swift

This file provides `ThreeDBitTensor`, the dense bit-sliced storage layout
that answers "which rows have field X set to value Y" quickly across up to
a million rows.

Rather than storing one 216-bit block per row (a row-major layout), the
tensor keeps six separate bit-slices, one per bit position 0 through 5 of a
field's six-bit value; each slice packs one bit per (row, field) pair. This
"bit-sliced" arrangement is what makes `scanFieldEquals(field:value:)` fast:
testing whether a whole batch of rows matches a target value reduces to
scanning six flat byte buffers rather than reading and unpacking a
six-bit value out of every row individually. `valueAt(row:field:)` and
`setValue(row:field:value:)` provide ordinary cell-level read and write,
each internally looping over the six bit-slices; `setValue` enforces the
six-bit width invariant with a precondition. `bitSet(row:field:bit:)` and
`setBit(row:field:bit:on:)` are the underlying single-bit primitives both
higher-level functions call. `scanFieldEquals(field:value:)` returns a byte
mask marking every row matching the target value; `enumerateMatches(_:)`
turns that mask into a plain array of matching row indices.
`reserveCapacity(_:)` grows the tensor to hold more rows, extending each
slice with zero bytes and leaving all existing data untouched — a no-op if
the requested size is not larger than the current one. The file notes the
current scan implementation loops row by row within each bit-slice pass
rather than operating on whole 64-bit words at once, which is a known
optimization opportunity, not yet taken, for a future vectorized version.

## RecallTypes.swift

This file provides the shared wire vocabulary that every recall primitive
and every federation query returns: `RecallScore`, `DistanceBreakdown`,
`RecallResult`, and `RowProjection`.

These four types live in SubstrateTypes, rather than in whichever library
happens to implement a particular recall strategy, because federation needs
them too — a federated query's response is shaped exactly like a local
recall result — and because defining them once here means no two libraries
can quietly redefine the same shape and drift apart. `RecallScore` pairs one
`RowId` with one `Float32` score; the file is explicit that the score's
meaning is specific to whichever recall primitive produced it (cosine
similarity for vector recall, inverted Hamming distance for fingerprint
recall, BM25 for text recall), so any code that combines scores from
different primitives must normalize first. `DistanceBreakdown` reports how
much each of four components — lattice, fingerprint, temporal, bitmap —
contributed to a match, each normalized to `[0, 1]`, both to explain "why
this matched" to a caller and to weight Reciprocal Rank Fusion when
combining several recall strategies' results. `RecallResult` bundles a
ranked list of scores with an optional breakdown, an optional confidence
interval, and the name of the primitive that produced it, so a composition
step downstream knows which combination rule to apply. `RowProjection` is
the minimal slice of a row that a recall primitive actually needs to rank
candidates: its identifier, capture time, fingerprint, lattice anchor,
bitmaps, and lifecycle state, deliberately omitting the row's verbatim
content and any metadata beyond the bitmaps. The file explains this
omission is intentional — ranking operates on structure, and the heavier
verbatim content is fetched separately, only for the rows that survive
ranking.

## Rust Port and Conformance

The `rust/` directory mirrors this package one module per Swift file:
`rust/src/row.rs` for `Row.swift`, `rust/src/fingerprint256.rs` for
`Fingerprint256.swift`, and so on through all thirty files, with
`rust/src/lib.rs` re-exporting the primary types at the crate root the same
way this package's types are used directly by Swift callers. There is no
separate conformance-fixture directory analogous to LatticeLib's
`rust/tests/fixtures/`; because SubstrateTypes ships no pinned data
artifacts, agreement between the two legs is instead verified by shared
test vectors embedded directly in each module's own test block (for
example, the illustrative Hamming and Fingerprint256 test vectors quoted in
their Swift source comments), checked against the equivalent Rust unit
tests. When a function in this package changes — most sensitively the
fingerprint combinators, the Hamming and SimHash arithmetic, and the FNV
hash — the corresponding Rust module must change identically, because every
one of these functions is a cross-platform agreement contract, not an
implementation detail local to one language.
