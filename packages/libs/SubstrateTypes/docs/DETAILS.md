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
    blob: 46508e5af2a855b29ec45b4e02635477e3460544
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
`OVERVIEW.md` first for the big picture. Files appear here in one order.
First comes the row shape and its lifecycle. Next come the two coordinate
systems a row carries: the lattice anchor and time. Then comes the
fingerprint construction pipeline and its algebra. After that comes
integrity hashing, the audit trail, and the population-statistics
matrices. The recall wire vocabulary closes the document.

## NounType.swift

This file provides `NounType`. It is an eight-case enum. It names the kind
of thing a row holds. A row can be a drawer, a tunnel, or a
knowledge-graph fact. It can be a diary entry, a proposal, or an
association. It can also be a learned reference or an ambient sample.

The enum uses `UInt8` raw values, zero through seven, instead of a string.
A row's noun type is stored in a compact wire format alongside the rest of
the row. That format needs a small fixed-size number, not a string. The
file's opening comment states a rule. These values are wire-stable. They
must never be renumbered. A stored row's noun type is read back by its raw
number. Changing a number would silently reclassify every row already on
disk.

## RowState.swift

This file provides three pure-data types. They describe a row's
lifecycle. `RowState` names the ten states a row can be in. `RowVerb`
names the twelve mutations the row-state automaton accepts.
`RowStateError` is the typed failure a rejected transition returns. The
automaton that enforces which transitions are legal is compute, not data.
It stays one layer up, in SubstrateLib. This file only names the states
and verbs the automaton works with.

The ten states use deliberately spaced raw values. They run zero to
three, then a gap, then sixteen to nineteen, another gap, then thirty-two
and thirty-three. This is not a plain zero-to-nine count. The spacing lets
any consumer recover a state's lifecycle cluster with one shift-and-mask
operation. No lookup table is needed. `RowState.cluster` reads
`(rawValue >> 4) & 0x3`. This classifies a state into one of three
clusters. Cluster A is active or becoming: active, pending, contested,
accepted. Cluster B is superseded or historical: superseded, decayed,
withdrawn, expired. These are retired but revivable. Cluster C is
terminal: rejected, tombstoned. These are retired and final.
`RowState.isActiveCluster` is the convenience most callers want. It gives
a single true-or-false answer to "is this row currently believed."

`RowState.cluster(ofRawState:)` performs the same classification from a
raw `UInt8`. It works even when code has only read a bare byte out of a
persisted bitmap and has not decoded it yet.
`RowState.activeClusterUpperBoundRaw` names one boundary value: sixteen.
A storage-layer predicate can compare against this directly. One example
is a SQL `WHERE` clause, when it cannot call a Swift function. The file is
explicit about one thing. Code holding a decoded `RowState` should prefer
`cluster` over a hand-rolled numeric comparison. A future state added
inside one of the gaps would silently misclassify under a magic-number
boundary. It would not misclassify under the shift-and-mask.

`RowState.description` and `RowStateError.description` supply exact
lowercase English strings. Examples are "active", "pending", and the
compact "active --reject-->" form. Downstream parsers depend on these
strings for parity with the Rust port's `Display` implementation. Both are
written as explicit `switch` statements. They do not use the default
`String(describing:)`. Calling that default from inside a type's own
`description` recurses forever.

## RowBitmaps.swift

This file provides `RowBitmaps`. It is the named layout over a row's
three bitmap columns. The file also provides `BitVector216`, a dense
216-bit view over that same data, for consumers that want to treat it as
a flat grid.

A row carries three `Int64` columns: adjective, operational, and
provenance. Each column packs several independently named fields at
specific bit ranges. For some purposes, such as updating the
population-statistics matrices, it helps to think of these three columns
as one uniform grid. That grid holds thirty-six fields of six bits each.
The real fields are not laid out quite that uniformly, though.
`RowBitmaps.field(_:)` performs this translation. Given a field index
from zero to thirty-five, it locates the right column and bit offset. It
returns that field's six-bit value. The function carries two defensive
casts, and the source comment explains both in detail. One is a
shift-amount guard: Swift silently wraps an out-of-range shift instead of
returning zero. The other is a cast to `UInt64` before shifting:
right-shifting a negative `Int64` sign-extends. That would leak the sign
bit into a field's value.

`RowBitmaps.bit(field:bit:)` reads one bit out of a field's six-bit value.
`RowBitmaps.fieldValues()` returns all thirty-six (field, value) pairs in
order. `MatrixO.applyRow` and `MatrixT.applyPair` consume this list to
update the co-occurrence and causality matrices. `RowBitmaps.bitVector()`
builds a `BitVector216` from a live row.

`BitVector216` has two constructors for two different sources of the same
216-bit shape. `init(rowBitmaps:)` builds it from a real row, through
`RowBitmaps.field(_:)`. Real rows never set bits above position
seventy-one of the abstract grid. This path is therefore safe for
updating `MatrixF` from row capture and expunge events.
`init(presenceBytes:)` builds it directly from a raw 27-byte pattern. That
pattern may set any of the 216 bits, including ones no real `RowBitmaps`
value can represent. A conformance-test harness vector uses this second
shape. `bit(at:)` and `bit(field:bit:)` read one bit, by absolute index
or by (field, bit) coordinate.

## Row.swift

This file provides `Row`, the substrate's central data shape. It also
provides `RowId`, a type alias for `UUID`. Every place the substrate
vocabulary calls for a row identifier uses `RowId`.

`RowId` is a plain alias, not a wrapper type. A `UUID`'s 16-byte
big-endian wire form is already byte-identical to the Rust port's
`RowId(u128)` newtype. No translation step is needed at the boundary.
Callers can use `UUID`'s existing API directly.

`Row` itself is a value type. It holds one row's complete current state:
its identifier, noun type, lifecycle state, three bitmap columns,
fingerprint, and lattice anchor. It also holds two optional fields. One is
a lineage identifier, linking a row to the row it was derived from. The
other is the verbatim content. The file is explicit that `Row` is pure
data. It has no logic and no input or output. The full substrate object
that bundles many rows together, with the audit log and the statistics
matrices, remains in SubstrateLib. So do the verb functions that mutate a
row's state. `Row` only says what a row looks like at rest.

## LatticeAnchor.swift

This file provides `LatticeAnchor`. It is the sixteen-byte reference a
row carries to its position in the classification lattice. It packs an
eight-byte hash of a lattice code plus an eight-byte hash of a concept
identity, called a Q-ID. Either half can be zero when absent.

A lattice anchor does not store the lattice code text itself. It stores
a deterministic hash of it. `LatticeAnchor.fnv1a64(_:)` is the private
FNV-1a 64-bit hash used for this purpose. It is the exact FNV-1a
algorithm, matching `FNV.hash64` elsewhere in the package. Two callers
hashing the same code string always agree. The result also matches the
Rust port bit for bit.

`LatticeAnchor.udc(_:)` builds an anchor from a lattice code string
alone, with a null Q-ID pointer. `LatticeAnchor.udcQid(_:qid:)` builds an
anchor that carries both the lattice code and a specific concept
identity. The file explains why this second form matters. Without it,
every row about a broad subject collapses onto one anchor value. That
loses the distinction between different things classified under the same
broad code. An empty Q-ID string yields the same null pointer as
`udc(_:)`. The two constructors therefore agree when no concept identity
is available. `LatticeAnchor.isNull` reports whether both halves are
zero. This is the anchor's way of saying "unclassified."

## HLC.swift

This file provides `HLC`, the Hybrid Logical Clock timestamp. It orders
events across replicas of an estate without needing synchronized wall
clocks. The file also provides `HLCGenerator`, the per-replica state
machine that produces HLC values.

An `HLC` has three fields, compared in order. `physicalTime` is
milliseconds since the Unix epoch. `logicalCount` is a counter that
advances when physical time does not. `nodeID` is a per-replica
tiebreaker. `HLC.<` compares these three fields in that order. This gives
any two HLC values an unambiguous order. That holds even when two
replicas' clocks briefly agree, or run backward relative to each other.
`HLC.zero` is the earliest possible value, used as a sentinel.
`HLC.advanced()` returns a copy with the logical counter incremented. It
holds physical time and node fixed.

`HLCGenerator.send(now:)` produces the timestamp for a locally originated
event. If the wall clock has moved forward since the last call, it adopts
the new time and resets the logical counter to zero. Otherwise it holds
the physical time and bumps the logical counter. Either way, every
timestamp this replica emits is strictly greater than the last.
`HLCGenerator.receive(remote:now:)` implements the HLC paper's merge rule
for an event arriving from another replica. It takes the maximum of the
local clock, the remote clock, and the current wall time. It sets the
logical counter according to which input was tied for that maximum. This
is the operation that lets two replicas exchange audit events. They
converge on one shared sense of "what happened before what." The
convergence proof in `GSetAuditLog.swift` depends on this operation.

`HLC.packed` and `init(packed:)` convert to and from a lossy 8-byte form.
Federation tier-contribution messages use this form. It trades precision,
forty bits of milliseconds, for a smaller wire size. That is roughly
thirty-four years of range. `HLC.wireBytes` and `init(wireBytes:)` are
the lossless 16-byte round trip used elsewhere. The file also carries a
`nodeId`-spelled initializer overload. It exists only so older call sites
written against that casing keep compiling unchanged.

## TimeRange.swift

This file provides `TimeRange`, a closed interval `[start, end]` of two
HLC values.

`TimeRange.init(start:end:)` rejects an interval whose end precedes its
start, with a precondition. An inverted range has no sensible meaning for
any caller. `TimeRange.contains(_:)` reports whether a given HLC falls
within the closed interval. Both endpoints are included. This is the
predicate a caller uses to test "did this event happen during this
window." The file notes that the higher-level primitive consuming these
windows, `MomentSummary`, stays in SubstrateLib. This type is only the
interval shape.

## AsOfCoordinate.swift

This file provides `AsOfCoordinate`, a two-case enum. It says whether a
read should return the live state or the state as of a specific HLC. The
first case is `.present`. The second case is `.asOf(hlc)`.

The file explains the bug this type prevents. Suppose "present" were
represented by a sentinel HLC value such as zero. A caller could not then
distinguish "give me now" from "give me the state at the dawn of time."
Making the choice an explicit enum discriminant removes that ambiguity by
construction. The custom `Codable` conformance encodes the case as an
explicit `"kind"` field, either `"present"` or `"asOf"`. For the second
case, it also encodes the HLC itself. This is not Swift's default enum
encoding. The wire form stays stable and self-describing across the Swift
and Rust legs.

## SnapshotId.swift

This file provides `SnapshotId`, a `UUID` wrapper. It identifies one
point-in-time snapshot of an estate.

Unlike `RowId`, which is a bare type alias, `SnapshotId` is a distinct
struct. The file states the reason. A snapshot identifier must not be
substitutable for a drawer, node, or estate identifier at the type level.
This holds even though all of these are UUIDs underneath.
`init(uuidString:)` and the `uuidString` accessor round-trip through the
standard UUID string form. `Codable` conformance encodes and decodes that
same string form. It throws `SnapshotIdError.invalidUUID` on a malformed
string, rather than crashing.

## BlockMask.swift

This file provides `BlockMask`, an `OptionSet`. It selects any subset of
the fingerprint's four 64-bit blocks.

Before this type existed, callers passed a `Set<Int>` to select blocks.
That allocated memory on every call, a measured hot-path cost in
nearest-neighbor search over large row collections. `BlockMask` replaces
that with four fixed bit flags, `.block0` through `.block3`. It adds an
`.all` convenience covering every block, and an empty `.none`. It is a
plain integer under the hood. So membership tests are branchless and need
no allocation. `BlockMask.blockCount` counts how many blocks are
selected, zero through four. `Hamming.similarity` uses this count to
compute the correct maximum-distance denominator for a partial-block
comparison.

## Fingerprint256.swift

This file provides `Fingerprint256`, the substrate's 256-bit structural
fingerprint. It also provides the pure combinators that other files in
this package build on.

A fingerprint is four independent 64-bit blocks: `block0` through
`block3`. Each is a SimHash over a different aspect of a row. One covers
its bitmaps. One covers its lattice position. One covers its lineage and
timing. One covers its channel and source. The file is careful to
distinguish a fingerprint from a lattice anchor. A fingerprint is the
coordinate for structural similarity. A lattice anchor is the coordinate
for topic similarity. The two answer different questions. Neither
substitutes for the other.

`Fingerprint256.bit(at:)` and `Fingerprint256.with(bit:set:)` give
bit-level read and copy-on-write access across the full 256-bit span.
Both translate a flat index into the right block automatically.
`with(bit:set:)` returns a new value rather than mutating in place. This
matches the package's general preference for value semantics over
in-place mutation. `Fingerprint256.union(_:)` computes the bitwise OR of
two fingerprints: set-union over the bits each one has set.
`Fingerprint256.fromBits(_:)` builds a fingerprint from a 256-element
array of booleans. This is mainly useful for tests, and for converting
from an external bit representation. `wireBytes` and `init(wireBytes:)`
convert to and from the canonical 32-byte little-endian wire form.
`toBytes()` and `fromBytes(_:)` are thin aliases, kept for parity with
call sites that expect a throws-free, optional-returning variant.

The file's closing section defines a small combinator layer. `zip4(_:_:)`
applies a binary operation to each of the four block pairs between two
fingerprints, in one call. `map4(_:)` applies a unary operation to all
four blocks of one fingerprint. `reduce4(_:_:)` folds a whole sequence of
fingerprints block-wise, with a binary operation. `popcount()` counts set
bits across all four blocks. These four functions exist for a reason.
The same "unroll across four blocks" pattern used to appear, hand-written,
in seven or more places across the codebase. This package's `ORReduce`,
`BitwiseArithmetic`, and `Hamming` were among them. Expressing the unroll
once here means a future change has only one call site to update. There is
no risk that one of the seven repeated copies falls out of sync with the
rest. `zip4Batch(_:_:_:)` and `map4Batch(_:_:)` extend the same idea across
an array of fingerprints. These serve callers vectorizing across many
rows, rather than across one fingerprint's four blocks.

## HyperplaneFamily.swift

This file provides `Hyperplane`, one plus-or-minus-one-valued dividing
plane used by SimHash. It also provides `HyperplaneFamily`, the fixed set
of sixty-four such planes that produce one fingerprint block.

A hyperplane of length *n* would normally be an array of *n* values. Each
value would be plus one, minus one, or zero. Binary inputs make this
arithmetic reduce to counting set bits. So `Hyperplane` instead stores two
bitmasks, `positiveMask` and `negativeMask`. Each has a one wherever the
plane is plus one or minus one, and a zero in both where the plane's
value is zero at that position. This is called a sparse hyperplane.
`Hyperplane.sign(over:)` computes whether the plane's dot product with a
binary input vector is strictly positive. It does this by comparing the
popcount of the input, ANDed with each mask. A tied dot product resolves
to false. The file notes this is vanishingly rare in practice, because
the per-block input has enough bits that an exact tie is unlikely.

`HyperplaneFamily.generate(seed:blockIndex:inputBitLength:density:)`
builds a family of sixty-four hyperplanes deterministically. It starts
from a 32-byte seed, using a private, non-cryptographic pseudo-random
generator called `HyperplanePRNG`. Determinism here is load-bearing. The
file states that hyperplane seeds are fixed once, at estate creation, and
never rotate. Rotating them would change every fingerprint an estate has
ever computed. It would break cross-replica sync. The `density` parameter
controls what fraction of a plane's positions are active, rather than
zero. At density 1.0, every position is active. The code then takes a
direct path around a floating-point computation. That computation would
otherwise round up past `UInt64.max` and trap.

`HyperplaneFamily.canonicalHash()` computes a stable 64-bit fingerprint
of a whole family's content. It folds every mask word through FNV-1a.
This labels a pairing arrangement in an audit trail without storing the
full family. `HyperplaneFamily.blockFamilies(baseSeed:density:)` is the
single routine that builds all four block families from one base seed.
Block 0 needs 192 bits; blocks 1 through 3 need 64 bits each. This
routine fixes two mistakes an earlier, separately derived generation path
had made. One mistake was reusing one seed across all four blocks, which
collapsed them into one projection instead of four independent ones. The
other was assuming a uniform 64-bit width for every block, when block 0
is 192 bits. `diversifiedSeed(base:blockIndex:)` mixes a block index into
the base seed. Each block then gets an independent 32-byte seed.
`expandSeed64(_:)` stretches a 64-bit seed back out to 32 bytes. It does
this through four rounds of a SplitMix64-style mixing function.

## FloatSimHashPlanes.swift

This file provides `FloatSimHashPlanes`, the materialized hyperplane set
for the floating-point-input variant of SimHash. It is the float
analogue of `HyperplaneFamily`, which serves the binary-input variant.

The file explains why this type lives in SubstrateTypes, rather than in
either of the two packages that use it. `SubstrateKernel` applies these
planes, through pure signed-sum-and-sign arithmetic with no randomness.
`SubstrateML` generates them from a seed, using the SplitMix64 random
generator. Placing the shared data shape at the lowest layer lets both
higher packages depend on it without either depending on the other. That
would otherwise invert the SDK's layering. `FloatSimHashPlanes` packs
`256 * dim` sign bits, one per hyperplane-coordinate pair, into a flat
array of 64-bit words. The words are ordered row-major by hyperplane,
matching the draw order the generator produces them in. The planes are
immutable once generated for a given seed and dimensionality. So a caller
materializes one instance per seed-dimension pair. That instance is
reused across every projection under that seed.

## SimHash.swift

This file provides `SimHash`, the construction that turns an input
vector and a hyperplane family into one 64-bit fingerprint block. It also
provides `SimHashInput`, the set of helpers that assemble the four
blocks' input vectors from a row's actual fields.

`SimHash.block(over:family:)` is the core operation. For each of the
family's sixty-four hyperplanes, it sets one output bit. It sets that bit
if the hyperplane's dot product with the input vector is positive.
`SimHash.fingerprint(bitmapInput:latticeInput:lineageTemporalInput:channelSourceInput:families:)`
calls this once per block. It assembles the four results into a complete
`Fingerprint256`. `SimHash.fingerprintBatch(...)` is the same computation,
repeated across many rows, returning results in input order. The file
notes that this reference loop is what a hardware-accelerated backend
elsewhere in the SDK must reproduce bit for bit. That backend vectorizes
the popcount work across many rows at once, for speed, but the result
must match exactly.
`SimHash.fingerprint(fromSubhashes:hyperplanes:)` is a convenience path.
It serves callers that have already reduced each block's input down to
one 64-bit subhash, rather than a full input vector.

`SimHashInput.bitmap(adjective:operational:provenance:)` concatenates a
row's three bitmap columns into block 0's 192-bit input.
`SimHashInput.lattice(udcPrefixHash:qidDirectHash:qidClosureHash:)` packs
three hash components into block 1's 64-bit input, at fixed bit offsets.
`SimHashInput.lineageTemporal(...)` and `SimHashInput.channelSource(...)`
do the same for blocks 2 and 3. Each function's bit-offset layout is
fixed by the cookbook specification the file cites. A mismatched offset
here would silently produce fingerprints that no longer compare
correctly against ones built the standard way. These helpers exist
precisely so every caller assembles the same layout.

## Hamming.swift

This file provides `Hamming`, the substrate's primary
structural-similarity measurement. It counts how many bit positions
differ between two fingerprints.

`Hamming.distance(_:_:blocks:)` computes this by XORing the two
fingerprints and counting the set bits in the result. That is
`a.zip4(b, ^).popcount()` on the common case of all four blocks. A
per-block loop handles the case where the caller restricts the comparison
to a `BlockMask` subset. Restricting to a subset answers a more specific
question, such as "how similar are these two rows in lattice position
alone, ignoring timing and provenance." `Hamming.similarity(_:_:blocks:)`
converts a distance into a `[0, 1]` score. A score of 1.0 means
identical. A score of 0.0 means maximally distant over the selected
blocks. `HammingDistance` is a type alias, re-exporting `Hamming` under
the name some call sites use.

## BitwiseArithmetic.swift

This file provides three fingerprint operations beyond OR-reduction.
They are intersection, symmetric difference, and a cohort prototype. It
also provides `FingerprintBuilder`, a small expression type for composing
them declaratively.

`BitwiseArithmetic.intersect(_:_:)` computes the bitwise AND of two
fingerprints: the bits both rows share. `BitwiseArithmetic.difference(_:_:)`
computes the bitwise XOR: the bits where the two disagree. Both delegate
to `Fingerprint256.zip4(_:_:)`. `BitwiseArithmetic.prototype(_:)` computes
the bit-for-bit majority vote across a whole cohort of fingerprints. This
is the fingerprint of a "typical" member. It works by folding the cohort
into a `CountVector256` and reading off its majority-vote view. The file
notes this replaced an earlier hand-written per-bit counting loop. That
loop is now an equivalent call into the one canonical cohort-fold
primitive.

`FingerprintBuilder` is an indirect enum with four cases. One is a
literal fingerprint. One is an intersection of two sub-expressions. One
is a difference of two sub-expressions. One is the prototype of a
fingerprint list. It has one method, `evaluate()`, that walks the tree
and computes the result. This lets a caller build up a compound
fingerprint query as data. One example is "the intersection of Bob's
typical pattern and Amelia's typical pattern." The caller can evaluate it
once, rather than nesting direct function calls.

## ORReduce.swift

This file provides `ORReduce`, the substrate's universal aggregation
primitive over collections of fingerprints.

`ORReduce.reduce(_:)` computes the bitwise OR across a whole sequence of
fingerprints. It returns `Fingerprint256.zero` for an empty sequence,
OR's identity element. It delegates to `Fingerprint256.reduce4(_:|:)`.
`ORReduce.reduce(_:blocks:defaults:)` restricts the reduction to specific
blocks. It fills any block outside the selection from a supplied
default. This helps when a caller wants only the topic-block aggregate
across a cohort, without touching the other three blocks. The file
explains why OR reduction matters beyond simple aggregation. It is
commutative, associative, and idempotent. So it works as the join
operator for a CRDT of fingerprints. Replicas can merge any number of
contributions. They can merge in any order, with any duplication. They
still converge on the same result. It is also a privacy mechanism. Once several fingerprints
are OR-reduced together, a reader can see which structural patterns
appear somewhere in the group. The reader cannot recover which specific
contributor set which bit.

## CountVector256.swift

This file provides `CountVector256`, the per-bit counting accumulator.
It underlies cohort-level fingerprint statistics, including the
majority-vote prototype that `BitwiseArithmetic.prototype(_:)` reads from
it.

The file's opening comment makes the key design argument. A
majority-vote summary does not compose. Suppose node A's majority is
computed from its children, and node B's majority from its children. The
majority of "A's majority and B's majority" is not generally the same as
the true majority over every leaf under both A and B combined. A count
vector does compose. Counts add. Member totals add. The sum is exact
regardless of the order members are folded in. So the count vector, not
the majority-vote fingerprint, is the object worth storing at every level
of a tree. The majority-vote fingerprint is a read-time view, computed
from it whenever a caller actually needs a Hamming-comparable engram.

`CountVector256.accumulate(_:)` folds one fingerprint into the vector.
For every set bit, its counter increases by one, and the member count
increases by one. `CountVector256.merge(_:)` and its operator form `+`
combine two count vectors by adding element-wise. This is the tree-fold
step. It is safe to apply to a node's children in any order, because
addition is commutative and associative. `CountVector256.majorityVote()`
reads off the fingerprint whose bit `j` is set exactly when a strict
majority of accumulated members had bit `j` set. The rule is
`2 * count > n`. An exact tie at half does not set the bit. The file
calls this convention identical across every kernel backend and both
language ports. `CountVector256.profile()` returns the 256 per-bit
probabilities instead of a threshold. This suits callers that want the
Bernoulli parameter of each bit, rather than a hard yes-or-no answer. The
static `CountVector256.fold(_:)` is the reference implementation of
folding a whole array of fingerprints at once. A vectorized kernel-layer
backend is required to match it exactly.

## FNV.swift

This file provides `FNV`, the Fowler-Noll-Vo 1a string hash family. The
substrate uses it everywhere a deterministic string hash is required:
drawer fingerprints, manifest-derived identifiers, and deterministic
tokenization.

`FNV.hash64(_:)` and `FNV.hash32(_:)` are two independent FNV-1a hashes.
They use different offset bases and primes. The file is explicit that
the 32-bit result is not a truncation of the 64-bit one. It is a wholly
separate computation over the same input bytes. `FNV.hash16(_:)` is
different in kind. It is the low sixteen bits of `hash64(_:)`, not a
from-scratch FNV-1a variant, because FNV-1a has no official 16-bit
definition. The substrate uses it where it needs a compact prefix hash,
such as a lattice sub-hash.

## MerkleDomain.swift

This file provides `MerkleDomain`, four one-byte tags. They are
prepended before hashing, to keep different kinds of nodes in the Merkle
content-integrity tree from ever colliding. `leaf` tags a single
drawer's content and vectors. `interior` tags a parent summarizing its
children. `tombstone` tags an expunged payload. `commitment` tags a keyed
HMAC-SHA256 commitment. Domain separation means a leaf hash and an
interior hash can never be mistaken for each other. This holds even if
their underlying bytes happened to coincide, because each was computed
with a different one-byte prefix. The file states these four values are
frozen. They must match exactly between the Swift and Rust legs,
permanently.

## ContentHash.swift

This file provides `ContentHash`, the 32-byte SHA-256 digest over one
leaf payload: a drawer's content plus its vectors.

The type stores its 32 bytes privately, exposed only through the `bytes`
accessor. The fixed size is enforced by construction, not by convention.
`ContentHash.tombstone` is a named constant. It is the SHA-256 hash of
the bare tombstone domain tag byte, used as the sentinel content hash for
an expunged, hard-deleted drawer. The file explains why this constant is
a literal byte array, rather than a value computed at runtime.
SubstrateTypes is the lowest layer. It cannot import SubstrateKernel,
which owns the SHA-256 implementation. Importing it would invert the
SDK's dependency direction. So the precomputed literal stands in instead. A bridge test in
SubstrateKernel checks it against the real computation. `hexString` and
`description` render the bytes as lowercase hex. The custom `Codable`
conformance encodes and decodes that same hex string. It validates the
string's length and character set on decode, rather than trusting
external input. The file is explicit that `ContentHash` is not
interchangeable with `MerkleRoot`, even though both are 32-byte hashes.
One is a payload digest. The other is a subtree summary. The type system
keeps them from being swapped by mistake.

## MerkleRoot.swift

This file provides `MerkleRoot`, the 32-byte hash summarizing an
interior node's children in the Merkle content-integrity tree. It is the
counterpart to `ContentHash`'s single-payload digest.

Its shape mirrors `ContentHash` closely. It uses private byte storage. It
has a `bytes` accessor, hex rendering, and the same `Codable` style.
`MerkleRoot.empty` is the named constant for the hash of
a node with no live children. It is the SHA-256 of the bare interior
domain tag, again stored as a literal, for the same layering reason
`ContentHash.tombstone` is. `MerkleRoot` and `ContentHash` stay two
distinct types, rather than one hash type used for both purposes. That
way, a function that expects a subtree summary cannot accidentally be
handed a single payload's digest. The compiler catches the mistake,
instead of a runtime bug surfacing later.

## AuditEvent.swift

This file provides `AuditEvent`, a single recorded mutation of a row. It
holds the before-and-after bitmap and lattice-anchor state. It records
which verb performed the mutation. It records who performed it, and
when.

`AuditEvent.eventID`, paired with `hlc`, gives every event a compound
key. That key makes replaying the same event twice, after a sync retry
for example, a safe no-op rather than a duplicate. `beforeBitmaps` is optional. A row's very first event is its capture.
That first event has no prior state to record. `reason` is an optional
human-readable explanation. It is threaded from the call site that
performed the mutation. The file notes it is populated for explicit
actions, such as an expunge, but left `nil` for the great majority of
routine mutations. `AuditEvent.withReason(_:)` returns a copy of an event
with a caller-supplied reason attached. This exists because the
structural validator that first produces an event is a pure checker.
That validator, `AuditGate.admit` in SubstrateLib, has no idea why a
mutation happened. The reason is layered on afterward, by the verb that
called it.

## GSetAuditLog.swift

This file provides `GSetAuditLog`, the substrate's append-only audit
log. It also provides two supporting types. `AuditEntry` is one immutable
log row. `AuditValue` is a typed field value inside an entry.

The log is the substrate's source of truth. A row's visible current
state is a projection, computed by replaying its log entries in order.
It is never stored independently of the log. `AuditEntry.id` is a
32-byte SHA-256 content hash, computed over the entry's other fields.
This gives the log a natural way to deduplicate. Two replicas that
independently record the same logical mutation compute the same id.
Once merged, only one copy remains. `AuditValue` is an enum covering four
kinds of value a field change can carry: a bitmap, a string, a
fingerprint, or an integer. It has a hand-written `Codable` conformance.
That conformance encodes each case as a single-key JSON object, such as
`{"bitmap": 42}`. It does not use Swift's default synthesized shape,
`{"bitmap": {"_0": 42}}`. This way the wire format matches what the Rust
port's serde derive produces natively. Both legs agree on one JSON shape.

`GSetAuditLog` itself stores its entries keyed by content hash
internally, for constant-time deduplication. Its `Codable` conformance
serializes them as a plain array sorted by id instead. The conceptually
correct wire representation of a grow-only set is a set, not a hash map.
The file explains this distinction explicitly, to justify writing custom
`Codable` rather than relying on the default. `GSetAuditLog.add(_:)`
inserts one entry. It is a no-op if the same id is already present.
`GSetAuditLog.merge(_:)` is the CRDT join: the union of two logs'
entries. This is correct regardless of which replica calls it, or in
what order two logs are merged. `GSetAuditLog.orderedEntries` replays
every entry in HLC order. That is the sequence a projection applies to
compute visible state. `GSetAuditLog.entries(forRow:)` and
`GSetAuditLog.entries(since:)` scope that same ordered replay. One scopes
to one row; the other scopes to everything after a cutoff. They serve
the row-state automaton and the sync protocol respectively. The file's
closing comment sketches why this design converges. G-Set merge is set
union. Set union is commutative, associative, and idempotent. HLC gives
any two entries an unambiguous order to replay them in. So any two
replicas that have exchanged all of each other's entries compute
identical visible state.

## MatrixF.swift

This file provides `MatrixF`, the field-presence matrix. For every
(field, bit) pair, it counts how many rows in the estate currently have
that bit set.

`MatrixF` stores 216 `Int64` counts in one flat array. It indexes them
by `field * bitsPerField + bit`. `MatrixF.cellIndex(field:bit:)` computes
that index, with bounds checking. Its layout constants are `fieldCount`,
`bitsPerField`, and `cellCount`. These are aliases onto the identical
constants in `RowBitmaps`. They are kept as aliases, rather than a second
independent definition, so the two types cannot silently drift out of
agreement about the shape of the 216-cell grid they both describe.
`MatrixF.applyRow(delta:bitVector:)` updates every cell whose bit is set
in a `BitVector216`, adding `delta`. That delta is positive one on a
row's capture, negative one on its expunge, or the pair of calls needed
to represent a mutation as a removal followed by an addition.
`MatrixF.totalCount` sums every cell, as a sanity check.
`MatrixF.writeWire(into:)` and `MatrixF.readWire(_:)` serialize the
matrix to and from its canonical 1,728-byte little-endian wire form:
216 cells of eight bytes each.

## MatrixC.swift

This file provides `MatrixC`, the correlation matrix. It holds the
marginal probability of each (field, bit) pair, derived from `MatrixF`
rather than updated independently.

`MatrixC` shares `MatrixF`'s 216-cell shape. It stores `Float` values in
`[0, 1]` instead of raw counts. `MatrixC.derive(from:nRows:)` is its only
way to change. It divides each of `MatrixF`'s counts by the total row
count. The division routes through `Double` before narrowing back to
`Float32`. The file notes this is necessary for cross-language
agreement. Both the Swift and Rust legs perform the identical
cast-divide-cast sequence. IEEE-754 guarantees the same bit pattern
results on both. When there are no rows at all, every cell is zero,
rather than an undefined division result. `MatrixC.writeWire(into:)` and
`MatrixC.readWire(_:)` serialize to and from an 864-byte wire form:
216 cells of four bytes each, as `Float32` bit patterns.

## MatrixO.swift

This file provides `MatrixO`, the co-occurrence matrix. It counts how
often each pair of (field, value) settings appears together across rows.
It also provides `CooccurrenceKey`, the packed key identifying one such
pair.

Each combination pairs one field-value setting against another. Most of
the roughly forty-six thousand possible combinations never occur in a
typical estate. So `MatrixO` stores
only nonzero cells, as a list sorted by `CooccurrenceKey.packed`, rather
than as a dense array or an unordered dictionary. Sorted order makes both
iteration and serialization deterministic across languages. An unordered
dictionary could not guarantee that. `CooccurrenceKey.packed` folds all
four components into one `UInt32`, for compact storage and for the
ordering comparison. `MatrixO.count(_:)` and `MatrixO.increment(_:by:)`
use binary search over the sorted entries. This reads or updates one
cell in logarithmic time. `increment` removes a cell entirely if its
count returns to zero, keeping the canonical form free of dead entries.
`MatrixO.applyRow(delta:fieldValues:)` updates the matrix for one row. It
iterates every ordered pair of the row's field-value settings. This
includes a field paired with itself. It increments each pair's cell by
`delta`. The file notes the matrix is not required to be symmetric in
storage, even though co-occurrence is conceptually symmetric. Each
unordered pair contributes to two distinct cells, (i, j) and (j, i). An
implementation that stores only one direction and infers the other is a
valid optimization. The reference here stores both, for clarity.
`MatrixO.writeWire(into:)` and `MatrixO.readWire(_:)` define the
canonical wire form: an entry count, followed by each entry's packed key
and count.

## MatrixT.swift

This file provides `MatrixT`, the temporal causality matrix. It counts
how often a row with one field-value setting is followed by a row with
another field-value setting. This must happen within a bounded time
window. It is the substrate's tool for telling apart two ideas. One idea
is "these things happen together." The other is "one of these things
tends to precede the other."

`CausalityKey` extends `CooccurrenceKey`'s idea with a fifth component, a
lag bucket. It packs all five into a `UInt64`.
`MatrixT.lagBucket(forMinutes:)` converts a raw time difference in
minutes into one of eight log-spaced buckets. The lower bounds are one,
two, four, eight, sixteen, thirty-two, sixty-four, and one hundred
twenty-eight minutes. It returns `nil` for a difference outside the
supported one-to-two-hundred-fifty-six-minute window. In that case, no
update to the matrix occurs at all. The file states this matrix is
explicitly asymmetric. The cell for "(field i, value i) preceding
(field j, value j)" is tracked completely separately from the reverse
direction. This is unlike `MatrixO`'s conceptually symmetric
co-occurrence. Storage works the same way `MatrixO`'s storage does.
Lookup through `count(_:)` and update through `increment(_:by:)` follow
the same sorted-array, binary-search pattern too. So does wire
serialization.
`MatrixT.applyPair(delta:rowAFieldValues:rowBFieldValues:lagMinutes:)` is
the row-pair version of `MatrixO.applyRow`. Given that row A precedes row
B by a known number of minutes, it increments the cell for every
combination of A's field values against B's field values. This happens
once a valid lag bucket exists for that separation.

## ThreeDBitTensor.swift

This file provides `ThreeDBitTensor`, the dense bit-sliced storage
layout. It answers "which rows have field X set to value Y" quickly,
across up to a million rows.

Rather than storing one 216-bit block per row, a row-major layout, the
tensor keeps six separate bit-slices. There is one slice per bit position
zero through five of a field's six-bit value. Each slice packs one bit
per (row, field) pair. This bit-sliced arrangement is what makes
`scanFieldEquals(field:value:)` fast. Testing whether a whole batch of
rows matches a target value reduces to scanning six flat byte buffers.
That beats reading and unpacking a six-bit value out of every row
individually. `valueAt(row:field:)` and `setValue(row:field:value:)`
provide ordinary cell-level read and write. Each internally loops over
the six bit-slices. `setValue` enforces the six-bit width invariant with
a precondition. `bitSet(row:field:bit:)` and `setBit(row:field:bit:on:)`
are the underlying single-bit primitives both higher-level functions
call. `scanFieldEquals(field:value:)` returns a byte mask, marking every
row matching the target value. `enumerateMatches(_:)` turns that mask
into a plain array of matching row indices. `reserveCapacity(_:)` grows
the tensor to hold more rows. It extends each slice with zero bytes and
leaves all existing data untouched. This is a no-op if the requested
size is not larger than the current one. The file notes the current scan
implementation loops row by row within each bit-slice pass, rather than
operating on whole 64-bit words at once. This is a known optimization
opportunity, not yet taken, for a future vectorized version.

## RecallTypes.swift

This file provides the shared wire vocabulary that every recall
primitive and every federation query returns: `RecallScore`,
`DistanceBreakdown`, `RecallResult`, and `RowProjection`.

These four types live in SubstrateTypes, rather than in whichever
library happens to implement a particular recall strategy. Federation
needs them too. A federated query's response is shaped exactly like a
local recall result. Defining them once here also means no two libraries
can quietly redefine the same shape and drift apart. `RecallScore` pairs
one `RowId` with one `Float32` score. The file is explicit that the
score's meaning is specific to whichever recall primitive produced it.
Vector recall uses cosine similarity. Fingerprint recall uses inverted
Hamming distance. Text recall uses BM25. Any code that combines scores
from different primitives must normalize first. `DistanceBreakdown`
reports how much each of four components contributed to a match: lattice,
fingerprint, temporal, and bitmap. Each is normalized to `[0, 1]`. This
serves two purposes: it explains "why this matched" to a caller, and it
weights Reciprocal Rank Fusion when combining several recall strategies'
results. `RecallResult` bundles a ranked list of scores with an optional
breakdown, an optional confidence interval, and the name of the primitive
that produced it. A composition step downstream then knows which
combination rule to apply. `RowProjection` is the minimal slice of a row
that a recall primitive actually needs to rank candidates. It carries the
row's identifier, capture time, fingerprint, lattice anchor, bitmaps, and
lifecycle state. It deliberately omits the row's verbatim content and any
metadata beyond the bitmaps. The file explains this omission is
intentional. Ranking operates on structure. The heavier verbatim content
is fetched separately, only for the rows that survive ranking.

## Rust Port and Conformance

The `rust/` directory mirrors this package, one module per Swift file.
`rust/src/row.rs` matches `Row.swift`. `rust/src/fingerprint256.rs`
matches `Fingerprint256.swift`. The pattern continues through all thirty
files. `rust/src/lib.rs` re-exports the primary types at the crate root,
the same way this package's types are used directly by Swift callers.
There is no separate conformance-fixture directory, unlike LatticeLib's
`rust/tests/fixtures/`. SubstrateTypes ships no pinned data artifacts. So
agreement between the two legs is instead verified by shared test
vectors, embedded directly in each module's own test block. The
illustrative Hamming and Fingerprint256 test vectors, quoted in their
Swift source comments, are one example. These are checked against the
equivalent Rust unit tests. When a function in this package changes, the
corresponding Rust module must change identically. This matters most for
the fingerprint combinators, the Hamming and SimHash arithmetic, and the
FNV hash. Every one of these functions is a cross-platform agreement
contract, not an implementation detail local to one language.
