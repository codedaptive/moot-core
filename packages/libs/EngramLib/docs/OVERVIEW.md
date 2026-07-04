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

EngramLib compares engrams. An engram is a 256-bit fingerprint: a short,
fixed-size code computed from a piece of content. Similar content produces
similar fingerprints, so comparing two engrams is fast even though the
content behind them may be long. EngramLib answers three kinds of question
about a set of engrams: how far apart are two of them, which ones are
nearest to a given engram, and how do many of them combine into one summary
engram.

MOOTx01 is an on-device AI memory system. It stores what an AI observes over
time and helps the AI recall it later. Every stored memory can carry an
engram that summarizes its structure. When the system needs to find
memories that resemble a new one, it does not compare the memories
directly. It compares their engrams instead, using the distance and
nearest-neighbor functions this library provides.

## The Problem It Solves

Comparing bit patterns sounds simple, but doing it correctly and quickly
across an entire memory store is not. The building blocks already exist in
a lower-level package called the substrate: routines for measuring the
distance between two 256-bit codes, for scanning many codes to find the
closest ones, and for combining codes with a bitwise OR. The substrate also
picks the fastest way to run these routines on the current device, choosing
among a plain reference implementation and hardware-accelerated versions
for different chip families.

Product code should not have to know about any of that. A consumer that
wants to find the ten memories most similar to a probe engram should not
have to pick a chip-specific routine, manage a kernel object, or reason
about which hardware path is active. EngramLib is a thin, stable layer over
the substrate that hides those choices. It exposes a small set of static
functions with plain names — `distance`, `findNearest`, `findWithin`,
`union` — and it always picks the best available implementation for the
device it runs on.

## How It Works

Every public function in EngramLib forwards its work to a kernel, an
object that knows how to run the underlying bit operations on the current
device. EngramLib selects a kernel once, when the module first loads, and
reuses it for every call. Consumers never see the kernel or choose one
themselves.

`EngramLib.distance(_:_:)` reports the Hamming distance between two
engrams. Hamming distance is the number of bit positions where two
equal-length codes differ; a smaller number means the two engrams — and
likely the content they summarize — are more alike. `EngramLib.distances`
computes that same distance from one probe engram to many candidates in a
single call, which is faster than calling `distance` in a loop because the
kernel can batch the work.

`EngramLib.findNearest` and `EngramLib.findWithin` build on that
distance measurement to answer retrieval questions. `findNearest` returns
the closest matches to a probe, ranked by distance, for use when a caller
wants a fixed number of results — the ten memories most like this one, for
example. `findWithin` instead returns every candidate inside a distance
radius, for use when a caller wants everything similar enough, however many
that turns out to be. Both break ties the same way: when two candidates
sit at equal distance, the one that appeared earlier in the input list
comes first. This tie-break rule matters because it makes the result
reproducible. The same probe against the same candidate list always
produces the same ranking, on every run.

`EngramLib.union` combines many engrams into one, bit by bit, keeping a
bit set in the result wherever any input had it set. This is useful for
building a single summary engram for a group of memories — a cohort, a
conversation, a topic cluster — that carries the structural features of
every member.

Most calls create a fresh, momentary link to the shared kernel and are
done. For code that runs the same kind of comparison many thousands of
times in a loop, EngramLib also offers a `Session`: a small object that
holds the kernel once and exposes the same functions as instance methods.
A session produces identical results to the static functions; it exists
only to save the small cost of re-resolving the kernel on every call.

## How the Pieces Fit

The package is deliberately small: one file defines the public API, and one
file defines its result type. `EngramLib.swift` contains the `EngramLib`
enum with its static functions, the `Session` type for reuse in hot loops,
and the `Engram` type alias itself. `Match.swift` contains `Match`, the
small value returned by every retrieval function, holding the position of
a candidate in its input list and its distance from the probe.

`Engram` is a public alias for `Fingerprint256`, a type owned by the
substrate's `SubstrateTypes` package. EngramLib does not define its own
fingerprint representation; it borrows the substrate's and gives it a
product-facing name, so that a future change to the underlying
representation does not force every consumer to rename their variables.
The kernel that does the actual bit math comes from `SubstrateKernel`,
the substrate's other half. Because the whole library is two short files
wrapping a single external kernel call, there is no internal topology
worth diagramming; every function's story is "validate the input, then
call the kernel."

## What Ships in the Package

The package ships the two Swift source files and their test suites, plus a
Rust port in `rust/` that mirrors the Swift API function for function:
`EngramLib::distance`, `EngramLib::find_nearest`, `EngramLib::union`, and
the rest. Both legs depend on their language's version of the substrate
packages and forward to the same kernel logic, so a Swift caller and a Rust
caller comparing the same two engrams get the same distance.
