---
doc: DETAILS
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

# EngramLib Details

This document walks through every source file in the package. Read
`OVERVIEW.md` first for the big picture. The package has two files: the
public API surface and the result type it returns.

## EngramLib.swift

This file provides the entire public surface of the library: the `Engram`
type alias, the `EngramLib` enum with its static comparison and retrieval
functions, and the `Session` type for callers that run many comparisons in
a loop.

### How It Works

The file opens with a type alias, `public typealias Engram = Fingerprint256`.
`Fingerprint256` is a 256-bit code defined in the substrate's
`SubstrateTypes` package: four 64-bit blocks that together form one
fixed-size fingerprint. EngramLib does not define its own bit
representation. It reuses the substrate's, under a product-facing name.
This indirection matters for one reason: if a future version of the
substrate widens the representation, product code that only ever wrote
`Engram` never has to change. An extension adds `Engram.init(blocks:_:_:_:)`,
a convenience initializer that takes the four blocks as separate arguments,
so callers can write `Engram(blocks: 0xDEAD, 0xBEEF, 0xCAFE, 0xBABE)`
instead of naming the underlying block parameters.

Every comparison in the file ultimately calls a kernel: an object, supplied
by the substrate's `SubstrateKernel` package, that knows how to run bit
operations efficiently on the current device. The file resolves this
kernel exactly once, at module load time, into the private constant
`_engramLibCachedKernel`. Every static function then reads that same
cached value instead of asking the substrate to choose again. The file's
own comment explains why: the kernel choice does not change while the
program runs, so resolving it on every call was pure overhead, paid on
every single similarity check.

### Function Coverage

`EngramLib.distance(_:_:)` returns the Hamming distance between two
engrams. Hamming distance counts the bit positions where two equal-length
codes differ. Identical engrams return 0; engrams that are exact bit
inverses of each other return 256, the width of the fingerprint. This
function matters because it is the single building block every other
comparison in the file is built from.

`EngramLib.distances(probe:candidates:)` computes the distance from one
probe engram to every member of a candidate list, returning an array in
the same order as the input. It matters because computing distances one
at a time in a loop would call into the kernel once per candidate; this
function hands the whole batch to the kernel in one call, so the kernel
can process it more efficiently. It returns an empty array immediately
when the candidate list is empty, avoiding any wasted work.

`EngramLib.findNearest(probe:in:k:)` returns the `k` candidates closest to
the probe, sorted by distance from nearest to farthest, with ties broken
by the candidate's position in the input list. This is the library's core
retrieval function: it answers "which memories are most like this one,"
bounded to a fixed count. It returns an empty array when `k` is zero or
negative, or when the candidate list is empty, rather than treating either
case as an error. When `k` is larger than the candidate count, it returns
every candidate rather than failing.

`EngramLib.findNearest(probe:in:)`, the two-argument overload, is a
convenience for the common case of wanting only the single closest match.
It calls the three-argument version with `k` fixed to 1 and returns the
first result, or `nil` when the candidate list is empty. It matters
because "find the closest one" is common enough to deserve its own name,
rather than making every caller unwrap a one-element array.

`EngramLib.findWithin(probe:in:maxDistance:)` returns every candidate
whose distance from the probe is at most `maxDistance`, sorted the same
way as `findNearest`. It matters because some callers do not know in
advance how many results they want; they want everything within a
similarity radius, however many candidates that turns out to include. It
computes the full distance array first, filters it, and sorts only the
matches that passed the filter, so the sort never touches a candidate it
is about to discard. It also guards against a negative `maxDistance`,
which cannot correspond to any real distance and would otherwise silently
match nothing in a confusing way.

`EngramLib.union(_:)` performs a bitwise OR across a list of engrams: the
result has a 1-bit at every position where at least one input engram had
one. It matters because a memory system often wants one summary engram
for a whole group — a cohort of related memories, or the union of a
topic's structural features — rather than comparing every group member
individually. An empty input returns the zero engram, which is the
correct identity value for OR: combining it with anything leaves that
thing unchanged.

`EngramLib.union(_:_:)`, the two-argument overload, ORs exactly two
engrams together. It exists as a lightweight convenience for the common
pairwise case, so a caller merging two engrams does not have to build a
two-element array just to call the list version.

`EngramLib.Session` is a struct that holds one kernel reference and
exposes `distance`, `distances`, `findNearest`, `findWithin`, and `union`
as instance methods with the same behavior as their static counterparts.
It matters for one reason: reuse. The static functions already share one
cached kernel internally, so a `Session` does not unlock any new
capability; it exists so that call sites processing thousands of
comparisons in a tight loop can hold one reference and avoid the small,
repeated cost of reaching back into the static cache on every call. A
session is `Sendable`, so the same session can be shared safely across
concurrent tasks. `EngramLib.session()` is a static factory that returns a
new `Session`; it is equivalent to calling `Session()` directly and exists
purely so callers do not need to know the type's name to construct one.

The private function `kernel()` is the single choke point every static
function in the enum calls through to reach the cached kernel. Routing
every call through one small function, rather than reading the module
constant directly everywhere, keeps the indirection in one place should
the caching strategy ever need to change.

## Match.swift

This file provides `Match`, the value every retrieval function in
`EngramLib.swift` returns.

### How It Works

A `Match` is a pair of two integers: `index`, the position of the matched
candidate in the caller's original input array, and `distance`, the
Hamming distance from the probe to that candidate. The type carries no
reference to the engram itself, only its position and its distance,
because the caller already has the original candidate array and can look
the engram up by index if needed. Keeping the type this small makes it
cheap to create in bulk during a retrieval scan.

`Match` conforms to `Hashable`, `Sendable`, and `Codable`, so it can be
stored in sets, passed safely across concurrent code, and serialized for
persistence or logging without any extra work.

### Function Coverage

`Match.init(index:distance:)` is the memberwise initializer. It is public
so that code outside the package — test suites, or a caller assembling
results from its own logic — can construct a `Match` directly rather than
only receiving one from an `EngramLib` function.

The `Comparable` conformance, added in a small extension, defines the
ordering that every retrieval function in `EngramLib.swift` relies on:
distance ascending first, and for two matches at equal distance, index
ascending second. This ordering is what makes `findNearest` and
`findWithin` deterministic. Given the same probe and the same candidate
list, the result is always sorted the same way, because the tie-break
rule never depends on anything but the candidate's fixed position in the
input.

## Rust Port and Conformance

The `rust/` directory contains a second implementation of the same API,
in two files: `lib.rs` defines the `EngramLib` struct with its associated
functions (`distance`, `find_nearest`, `find_within`, `union`, and the
rest) plus the `Session` type, and `matchx.rs` defines the `Match` struct
with its `Ord` implementation. The function names differ only in the ways
that Swift and Rust naming conventions differ — `findNearest` in Swift is
`find_nearest` in Rust, `findNearest(probe:in:)` becomes
`find_nearest_one` because Rust has no argument-label overloading — but
the behavior, including every empty-input guard and every tie-break rule,
matches the Swift implementation function for function. Both legs depend
on their language's version of `SubstrateTypes` and `SubstrateKernel` and
route their bit math through the same kernel abstraction, so a Swift
caller and a Rust caller comparing the same two engrams compute the same
distance. `rust/tests/engram_lib_tests.rs` mirrors the Swift test suite
in `Tests/EngramLibTests/`; when you change the behavior of one leg,
update the other and its tests together.
