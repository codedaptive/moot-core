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
public API surface, and the result type it returns.

## EngramLib.swift

This file holds the entire public surface of the library. It defines the
`Engram` type alias, the `EngramLib` enum with its static comparison and
retrieval functions, and the `Session` type. Callers that run many
comparisons in a loop use `Session`.

### How It Works

The file opens with a type alias: `public typealias Engram =
Fingerprint256`. `Fingerprint256` is a 256-bit code defined in the
substrate's `SubstrateTypes` package. It packs four 64-bit blocks into one
fixed-size fingerprint. EngramLib does not define its own bit shape. It
reuses the substrate's shape, under a product-facing name. This choice
matters for one reason. If a future substrate version widens the shape,
product code that only ever wrote `Engram` never has to change. An
extension adds `Engram.init(blocks:_:_:_:)`. This convenience initializer
takes the four blocks as separate arguments. Callers can write
`Engram(blocks: 0xDEAD, 0xBEEF, 0xCAFE, 0xBABE)` instead of naming the
underlying block parameters.

Every comparison in the file calls a kernel in the end. A kernel is an
object, supplied by the substrate's `SubstrateKernel` package, that knows
how to run bit operations well on the current device. The file resolves
this kernel exactly once, at module load time. It stores that kernel in
the private constant `_engramLibCachedKernel`. Every static function then
reads that same cached value, instead of asking the substrate to choose
again. The file's own comment explains why. The kernel choice does not
change while the program runs. Resolving it on every call was pure
overhead, paid on every single check.

### Function Coverage

`EngramLib.distance(_:_:)` returns the Hamming distance between two
engrams. Hamming distance counts the bit positions where two equal-length
codes differ. Identical engrams return zero. Engrams that are exact bit
inverses of each other return 256, the full width of the fingerprint.
This function matters because every other comparison in the file builds
from it.

`EngramLib.distances(probe:candidates:)` computes the distance from one
probe engram to every candidate in a list. It returns an array in the
same order as the input. This matters because computing distances one at
a time, in a loop, would call into the kernel once per candidate. This
function hands the whole batch to the kernel in one call. The kernel then
processes it more efficiently. It returns an empty array right away when
the candidate list is empty, so no work goes to waste.

`EngramLib.findNearest(probe:in:k:)` returns the `k` candidates closest
to the probe. It sorts them by distance, from nearest to farthest. Ties
break by the candidate's position in the input list. This is the
library's core retrieval function. It answers which memories look most
like this one, bounded to a fixed count. It returns an empty array when
`k` is zero or negative, or when the candidate list is empty. Neither
case counts as an error. When `k` exceeds the candidate count, the
function returns every candidate rather than failing.

`EngramLib.findNearest(probe:in:)` is the two-argument overload. It is a
convenience for the common case of wanting only the single closest match.
It calls the three-argument version with `k` fixed to one. It returns the
first result, or `nil` when the candidate list is empty. This overload
matters because finding the closest one is common enough to earn its own
name. Callers should not have to unwrap a one-element array by hand.

`EngramLib.findWithin(probe:in:maxDistance:)` returns every candidate
whose distance from the probe sits at or under `maxDistance`. It sorts
results the same way as `findNearest`. This matters because some callers
do not know in advance how many results they want. They want everything
within a similarity radius, however many candidates that turns out to
include. The function builds the full distance array first, then filters
it. It sorts only the matches that passed the filter. So the sort never
touches a candidate it is about to drop. The function also guards against
a negative `maxDistance`. A negative value cannot match any real
distance. Without the guard, it would silently match nothing, which
would confuse a caller.

`EngramLib.union(_:)` runs a bitwise OR across a list of engrams. The
result carries a one-bit at every position where at least one input
engram had one. This matters because a memory system often wants one
summary engram for a whole group, such as a cohort of related memories,
or the union of a topic's structural traits. An empty input returns the
zero engram. That is the correct identity value for OR, since combining
it with anything leaves that thing unchanged.

`EngramLib.union(_:_:)` is the two-argument overload. It ORs exactly two
engrams together. It exists as a light convenience for the common
pairwise case. A caller merging two engrams does not have to build a
two-element array just to call the list version.

`EngramLib.Session` is a struct that holds one kernel reference. It
exposes `distance`, `distances`, `findNearest`, `findWithin`, and `union`
as instance methods. Each mirrors the behavior of its static counterpart.
It matters for one reason: reuse. The static functions already share one
cached kernel inside, so a `Session` unlocks no new capability. It exists
so that call sites processing thousands of comparisons in a tight loop
can hold one reference. That reference skips the small, repeated cost of
reaching back into the static cache on every call. A session is
`Sendable`. So the same session can be shared safely across concurrent
tasks. `EngramLib.session()` is a static factory that returns a new
`Session`. It is equivalent to calling `Session()` directly. It exists so
callers do not need to know the type's name to build one.

The private function `kernel()` is the single choke point every static
function calls through to reach the cached kernel. Routing every call
through one small function keeps that indirection in one place. This
matters should the caching strategy ever need to change.

## Match.swift

This file provides `Match`, the value every retrieval function in
`EngramLib.swift` returns.

### How It Works

A `Match` is a pair of two integers. `index` is the position of the
matched candidate in the caller's original input array. `distance` is the
Hamming distance from the probe to that candidate. The type carries no
reference to the engram itself, only its position and its distance,
because the caller already holds the original candidate array. The caller
can look the engram up by index if needed. Keeping the type this small
makes it cheap to build in bulk during a retrieval scan.

`Match` conforms to `Hashable`, `Sendable`, and `Codable`. It can be
stored in sets. It can be passed safely across concurrent code. It can be
serialized for storage or logging, without any extra work.

### Function Coverage

`Match.init(index:distance:)` is the memberwise initializer. It is public
so that code outside the package can build a `Match` directly. This
covers test suites, or a caller assembling results from its own logic,
rather than only receiving a `Match` from an `EngramLib` function.

The `Comparable` conformance sits in a small extension. It defines the
order that every retrieval function in `EngramLib.swift` relies on.
Distance ascends first. For two matches at equal distance, index ascends
second.

This order is what makes `findNearest` and `findWithin` deterministic.
Given the same probe and the same candidate list, the result always
sorts the same way. The tie-break rule never depends on anything but the
candidate's fixed position in the input.

## Rust Port and Conformance

The `rust/` directory holds a second implementation of the same API, in
two files. `lib.rs` defines the `EngramLib` struct with its associated
functions (`distance`, `find_nearest`, `find_within`, `union`, and the
rest), plus the `Session` type. `matchx.rs` defines the `Match` struct
with its `Ord` implementation. The function names differ only in the
ways Swift and Rust naming conventions differ. `findNearest` in Swift is
`find_nearest` in Rust. `findNearest(probe:in:)` becomes
`find_nearest_one`, since Rust has no argument-label overloading. The
behavior matches the Swift version function for function, including
every empty-input guard and every tie-break rule. Both legs depend on
their own language's version of `SubstrateTypes` and `SubstrateKernel`.
Both route their bit math through the same kernel abstraction. So a
Swift caller and a Rust caller, comparing the same two engrams, compute
the same distance. `rust/tests/engram_lib_tests.rs` mirrors the Swift
test suite in `Tests/EngramLibTests/`. A change to one leg's behavior
requires the same change to the other leg, and to its tests.
