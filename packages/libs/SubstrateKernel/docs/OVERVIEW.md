---
doc: OVERVIEW
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

# SubstrateKernel Overview

## What This Library Does

SubstrateKernel computes the small set of bit-level and hash
operations that MOOTx01 runs on every memory it stores. It compares
fingerprints, extracts values from packed bitmaps, and computes
cryptographic hashes. A fingerprint is a short fixed-size code
computed from a piece of content; similar content produces similar
fingerprints, so the system compares things quickly without reading
them in full. This package works with a specific fingerprint shape,
`Fingerprint256`, a 256-bit code held as four 64-bit blocks.

Comparing two fingerprints means counting how many bit positions
differ between them. That count is called Hamming distance: the
number of positions where two equal-length codes differ. A smaller
Hamming distance means the two fingerprints, and the content behind
them, are more similar. Finding the memories most similar to a query
means finding the fingerprints with the smallest Hamming distance to
a probe fingerprint. This is the single most frequent operation in
MOOTx01's recall path, so SubstrateKernel gives it several
implementations, each tuned for a different kind of hardware.

SubstrateKernel is layer 2 of a four-package substrate split. Layer 1,
SubstrateTypes, defines pure data types with no logic, including
`Fingerprint256` itself. Layer 2, this package, computes over those
types. Layer 3, SubstrateML, builds learning and graph algorithms on
top of layer 2. Layer 4, SubstrateLib, orchestrates the whole
substrate for its callers. A package that composes libraries into a
larger subsystem is called a kit; kits depend on libraries like this
one, never the reverse. Four kits currently depend on SubstrateKernel
directly for their hot paths: LocusKit, CorpusKit, GeniusLocusKit, and
EngramLib.

## The Problem It Solves

Two devices must agree on how similar two fingerprints are. MOOTx01
estates — one user's complete memory store — can federate, meaning
separate devices share and compare memories. If one device's Hamming
distance for a pair of fingerprints disagreed with another device's
distance for the same pair, shared recall would produce different
answers on different hardware. Every operation in this package must
therefore produce the same output for the same input, on every
platform, every time. This package calls that guarantee its
conformance contract, and it treats one implementation, `ScalarKernel`,
as the oracle every other implementation must match bit for bit.

Meeting that contract while also running fast is harder than meeting
it alone. A scan over a million fingerprints, at 32 bytes each, moves
32 megabytes through memory for a single query. That volume of
straight-line bit manipulation is bandwidth-bound: its cost is
dominated by how fast data moves through the processor, not by how
complex the arithmetic is. Different processors move bandwidth-bound
data at different speeds depending on which instructions they use, so
one fixed implementation cannot be fastest everywhere. SubstrateKernel
resolves this by defining one interface, several interchangeable
implementations behind it, and a single reference implementation that
every one of them is checked against.

A second, smaller problem is duplication. Bit-field extraction from a
packed bitmap, and content hashing for the audit log, are needed by
many packages upstream of SubstrateKernel. Left unmanaged, each
package would write its own version, and a change to the bit layout
or the hash algorithm would require updating every copy. SubstrateKernel
centralizes both so a single implementation serves every caller.

## How It Works

The `SubstrateKernel` protocol declares the operations every backend
must supply: Hamming distance between two fingerprints, top-K nearest
neighbor search, bitwise OR-reduction across a group of fingerprints,
SimHash projection (folding a set of numbers into a fingerprint), and
a handful of batched variants of each. `ScalarKernel` implements the
protocol with a plain loop over each fingerprint's four 64-bit blocks.
It is always available, on every platform, and it is the oracle: any
other implementation that disagrees with it on any input is, by
definition, wrong.

Three more implementations specialize the same protocol for particular
hardware. `SimdKernel` uses Swift's portable `simd` module, which the
compiler turns into ARM NEON instructions on Apple Silicon. `NeonKernel`
frames the same computation a different way, at the level of
individual bytes rather than 64-bit words, to test whether that shape
compiles to tighter machine code. `MetalKernel` dispatches the batched
Hamming distance computation to the GPU through Apple's Metal
framework, which pays a fixed per-call setup cost but scales well past
roughly one hundred thousand candidates. A caller selects an
implementation through `PortableKernel`, either automatically for the
current platform or explicitly by name for testing.

Two more file pairs round out the package. `BitField` extracts and
writes fixed-width fields inside a 64-bit packed bitmap — the encoding
MOOTx01 uses to pack several small values into one machine word.
`SHA256` computes a standard cryptographic hash, used both to give
each audit-log entry a unique, content-derived identifier and, through
`HKDF`, to derive keys for the estate's cryptographic grants. A last
file, `FloatVecOps`, defines the canonical floating-point vector
operations — length, normalization, dot product, cosine similarity —
that any faster backend for those operations must also match bit for
bit. A final file, `HammingNN`, offers a simpler, general-purpose top-K
search over any sequence of candidates, independent of the
`PortableKernel` backends, for callers that do not need backend
selection.

## How the Pieces Fit

Figure 1 shows the library's topology — its major parts and how data
moves between them.

![Figure 1. Topology of SubstrateKernel](topology.svg)

*Figure 1. Topology of SubstrateKernel. A caller asks `PortableKernel`
for a kernel; the dispatcher hands back one of four interchangeable
implementations of the `SubstrateKernel` protocol, all conformance-
gated against the `ScalarKernel` oracle. `HammingNN`, `BitField`,
`SHA256`, `HKDF`, and `FloatVecOps` are independent primitives that do
not go through the dispatcher.*

`PortableKernel.kernelForCurrentPlatform()` picks `SimdKernel` on
64-bit ARM and falls back to `ScalarKernel` everywhere else; it never
selects `MetalKernel`; a caller wanting the GPU path must ask for it
by name with `PortableKernel.kernel(of: .metal)`. `NeonKernel` is
likewise available only by explicit request, since it exists to be
measured against `SimdKernel`, not to replace it automatically.
Whichever implementation runs, its output for a given input is
required to match `ScalarKernel`'s output for that same input; a
conformance test suite enforces this for every backend, on both the
Swift and Rust legs.

## What Ships in the Package

The package ships nine Swift source files and no pinned data
artifacts — unlike some sibling libraries, SubstrateKernel's behavior
depends only on its algorithms, not on any versioned reference data.
It also ships a Rust port in `rust/`, mirroring every file except the
two that have no meaningful Rust equivalent: `NeonKernel`, because
Rust's portable-SIMD path already covers the same ground on the
relevant targets, and `MetalKernel`, because Metal is an Apple-only
framework with no Linux or Windows counterpart. The Rust leg adds one
backend the Swift leg does not have — an AVX-512 implementation for
x86-64 processors — but keeps it dark: built, tested, and reachable
only by explicit request, never chosen automatically, until a future
performance study proves it belongs in the default path.
