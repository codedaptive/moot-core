---
doc: OVERVIEW
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

# EngramLib Overview

## What This Library Does

EngramLib compares engrams. An engram is a fingerprint. It is a short,
fixed-size code computed from a piece of content. Similar content produces
similar fingerprints. So comparing two engrams is fast, even when the
content behind them is long. EngramLib answers three questions about a set
of engrams. How far apart are two of them? Which ones sit nearest to a
given engram? How do many of them combine into one summary engram?

MOOTx01 is an on-device AI memory system. It stores what an AI observes
over time. It helps the AI recall that memory later. Every stored memory
can carry an engram that sums up its structure. When the system looks for
memories that resemble a new one, it does not compare the memories
directly. It compares their engrams instead. It uses the distance and
nearest-neighbor functions this library provides.

## The Problem It Solves

Comparing bit patterns sounds simple. Doing it well and fast, across a
whole memory store, is not simple. The building blocks already exist in a
lower-level package called the substrate. The substrate holds routines
that measure the distance between two 256-bit codes. It holds routines
that scan many codes to find the closest ones. It holds routines that
combine codes with a bitwise OR. The substrate also picks the fastest way
to run these routines on the current device. It chooses among a plain
reference version and hardware-tuned versions for different chip
families.

Product code should not need to know any of that. Picture a consumer that
wants the ten memories most like a probe engram. That consumer should not
have to pick a chip-specific routine. It should not have to manage a
kernel object. It should not have to reason about which hardware path is
active. EngramLib is a thin, stable layer over the substrate that hides
those choices. It exposes a small set of static functions with plain
names: `distance`, `findNearest`, `findWithin`, `union`. It always picks
the best version for the device it runs on.

## How It Works

Every public function in EngramLib passes its work to a kernel. A kernel
is an object that knows how to run bit operations on the current device.
EngramLib picks a kernel once, when the module first loads. It reuses
that kernel for every call after that. Consumers never see the kernel.
They never pick one themselves.

`EngramLib.distance(_:_:)` reports the Hamming distance between two
engrams. Hamming distance counts the bit positions where two equal-length
codes differ. A smaller number means the two engrams are more alike. The
content they sum up likely is more alike too. `EngramLib.distances`
computes that same distance from one probe engram to many candidates in a
single call. This is faster than calling `distance` in a loop, because the
kernel can batch the work.

`EngramLib.findNearest` and `EngramLib.findWithin` build on that distance
measure to answer retrieval questions. `findNearest` returns the closest
matches to a probe, ranked by distance. Use it when a caller wants a fixed
number of results, such as the ten memories most like this one. `findWithin`
instead returns every candidate inside a distance radius. Use it when a
caller wants everything similar enough, however many results that turns
out to be. Both functions break ties the same way. When two candidates sit
at equal distance, the one that appeared earlier in the input list comes
first. This tie-break rule matters, because it makes the result
reproducible. The same probe against the same candidate list always
produces the same ranking, on every run.

`EngramLib.union` combines many engrams into one, bit by bit. It keeps a
bit set in the result wherever any input had that bit set. This helps
build one summary engram for a group of memories, such as a cohort, a
conversation, or a topic cluster. That summary engram carries the
structural traits of every member.

Most calls create a brief link to the shared kernel and finish. Some code
runs the same kind of comparison many thousands of times in a loop. For
that code, EngramLib also offers a `Session`. A session is a small object
that holds the kernel once. It exposes the same functions as instance
methods. A session returns the same results as the static functions. It
exists only to skip the small cost of finding the kernel again on every
call.

## How the Pieces Fit

The package stays small on purpose. One file defines the public API. One
file defines its result type. `EngramLib.swift` holds the `EngramLib`
enum with its static functions, the `Session` type for reuse in hot
loops, and the `Engram` type alias itself. `Match.swift` holds `Match`.
`Match` is the small value every retrieval function returns. It carries
the position of a candidate in its input list and its distance from the
probe.

`Engram` is a public alias for `Fingerprint256`, a type owned by the
substrate's `SubstrateTypes` package. EngramLib does not define its own
fingerprint shape. It borrows the substrate's shape and gives it a
product-facing name. A future change to that shape then does not force
every consumer to rename their variables. The kernel that does the actual
bit math comes from `SubstrateKernel`, the substrate's other half. The
whole library is two short files that wrap a single external kernel call.
So there is no internal layout worth a diagram. Every function tells the
same story: check the input, then call the kernel.

## What Ships in the Package

The package ships the two Swift source files and their test suites. It
also ships a Rust port in `rust/` that mirrors the Swift API function for
function: `EngramLib::distance`, `EngramLib::find_nearest`,
`EngramLib::union`, and the rest. Both legs depend on their own
language's version of the substrate packages. Both forward to the same
kernel logic. So a Swift caller and a Rust caller, comparing the same two
engrams, get the same distance.
