---
doc: DETAILS
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

# SubstrateKernel Details

This document walks through every source file in the package. Read
`OVERVIEW.md` first for the big picture. Files appear here in this
order: the shared protocol and its reference implementation, the three
specialized hardware backends, the standalone top-K primitive, and
finally the three independent utilities — bit fields, hashing, and
float-vector math.

## PortableKernel.swift

This file provides the `SubstrateKernel` protocol, the `ScalarKernel`
reference implementation, and the `PortableKernel` dispatcher that
hands callers a concrete kernel.

The protocol declares every operation a kernel must supply:
`popcount64` (count the 1-bits in a 64-bit word), `hammingDistance256`
(count differing bits between two fingerprints), `orReduce256`
(bitwise-OR a group of fingerprints into one), `hammingTopK` (find the
K candidates nearest a probe fingerprint), `simhashCompute` (fold a
set of 64-bit numbers into a fingerprint), `countFold256` (tally, per
bit position, how many fingerprints in a group have that bit set), and
`floatSimHashProject` (the same folding operation starting from a
vector of decimal numbers instead of integers). A protocol extension
supplies default, correct-but-unoptimized implementations of the
batched variants of each operation, built as simple loops over the
single-item version; a backend that has a faster batched strategy
overrides the default, but every backend is correct even before it
does.

`ScalarKernel` implements every required method as a straightforward
loop with no special hardware instructions. It is deliberately the
simplest possible correct implementation, because it is the oracle: a
backend is defined as correct when its output for a given input
matches `ScalarKernel`'s output for that same input. Two of its
methods matter beyond their obvious job. `hammingTopK` maintains a
bounded max-heap of the K best candidates seen so far, evicting the
current worst kept candidate whenever a better one arrives, which
keeps the cost proportional to N times the log of K rather than N
times the log of N; ties between equal distances are broken toward the
candidate with the lower index, and every other backend must match
that exact tie-break. `floatSimHashProject`'s default body (defined in
the protocol extension, inherited by every backend that does not
override it) is itself the reference for that operation: for each of
256 hyperplanes it accumulates a signed sum over the input vector in a
fixed left-to-right order, because floating-point addition is not
associative and the order is therefore part of the contract, not an
implementation detail.

`PortableKernel.kernelForCurrentPlatform()` is the entry point most
callers use. On 64-bit ARM (`arm64`, which covers Apple Silicon and
ARM64 Linux) it returns `SimdKernel`; everywhere else it returns
`ScalarKernel`. The choice is fixed at compile time for a given
platform — there is no runtime hardware probing here, unlike the
Rust port's AVX-512 backend, which checks the processor's feature
flags at runtime because that backend is unsafe to run without them.
Every call reports which backend it chose through IntellectusLib's
telemetry system, but only when telemetry is enabled; when it is off,
the reporting closure is never evaluated and no clock is read, so the
selection has no cost in normal operation. `PortableKernel.kernel(of:)`
is the explicit-selection counterpart used by tests and by any caller
that wants a specific backend regardless of platform; requesting
`.metal` falls back to `ScalarKernel` if no GPU is available, and
requesting `.avx512` or `.avx2` always falls back to `ScalarKernel` in
this Swift build, since neither has a Swift implementation.
`PortableKernel.assertEqual` runs `hammingTopK` on two kernels with the
same inputs and reports whether their results match — the building
block the conformance test suite uses to check every backend against
the oracle.

## PortableKernel-SIMD.swift

This file provides `SimdKernel`, the backend `kernelForCurrentPlatform`
selects on 64-bit ARM.

`SimdKernel` uses Swift's `simd` module, which represents four 64-bit
numbers as one `SIMD4<UInt64>` value and lets ordinary operators like
XOR and OR act on all four at once. The Swift compiler turns those
operators into single ARM NEON instructions rather than four separate
ones. `hammingDistance256` XORs two fingerprints as `SIMD4<UInt64>`
values, then counts the 1-bits in each of the four resulting 64-bit
lanes and adds the four counts together. `hammingTopK` uses a
different strategy from `ScalarKernel`'s heap: a small sorted array of
the K best candidates found so far ("ladder"), shifted to make room
for a new entry, which the file's comments note is faster than a heap
for the small K values this package expects, but produces the
identical sorted output and tie-break.

`simhashBlockBatch` computes one 64-bit SimHash block for many input
vectors against the same hyperplane family — a fixed set of 64
reference directions used to decide, for each bit of the output, which
side of that direction the input vector falls on. Rather than compute
that decision one hyperplane at a time for every input, `PackedFamily`
rearranges the family's 64 hyperplanes once, into groups of four, so
the per-input inner loop can process four hyperplanes together with
the same `SIMD4<UInt64>` trick used for Hamming distance. Building this
packed layout costs a little extra time up front, which is worthwhile
because that cost is paid once and the savings apply to every input in
the batch.

`countFold256` computes, for a group of fingerprints, how many of them
have each of the 256 bit positions set. Rather than count each
fingerprint's contribution to each bit position one at a time,
`SimdKernel` keeps a growing binary counter for each bit position, held
across `SIMD4<UInt64>` lanes, and folds each new fingerprint in with
two bitwise operations: the counter's next bit is the counter XOR the
fingerprint, and the amount to carry into the following bit is the
counter AND the fingerprint. This produces the same result as adding
one for every set bit, but does the addition four fingerprint-blocks
at a time instead of bit by bit.

## PortableKernel-NEON.swift

This file provides `NeonKernel`, an alternate ARM implementation kept
side by side with `SimdKernel` to test a different way of expressing
the same computation.

`SimdKernel` counts bits four 64-bit words at a time. `NeonKernel`
instead treats a fingerprint as 32 individual bytes and counts bits one
byte at a time before adding all 32 counts together. The file's
comments record the reasoning: processors often have an instruction
that counts bits across a whole row of bytes in one step, and the
hope was that framing the computation at the byte level would let the
compiler use that instruction more effectively than the word-level
framing does. Whether it actually helps is a question the package
answers by measurement, not assumption — a companion benchmark
compares the two — and `NeonKernel` exists so that measurement has
something concrete to measure. It is never chosen automatically;
callers reach it only through `PortableKernel.kernel(of: .neon)`.

Every method mirrors its `SimdKernel` counterpart at the byte level
instead of the word level: `hammingDistance256` and
`hammingDistanceBatch` XOR two byte-vectors and sum per-byte bit
counts, and `hammingTopK` uses the same ladder-based top-K selection
as `SimdKernel`. `orReduce256`, `orReduceBatch`, and `simhashCompute`
are not re-framed at the byte level, because the file's comments judge
that the byte-level trick that helps Hamming distance does not help
those operations, so they use the same implementation `SimdKernel`
uses.

## PortableKernel-Metal.swift

This file provides `MetalKernel`, which runs the batched Hamming
distance computation on the graphics processor through Apple's Metal
framework, and which is available only on Apple platforms.

Dispatching work to the GPU has a fixed cost per call — tens of
microseconds to set up buffers and hand off the computation — that
does not shrink as the batch grows. That fixed cost makes GPU dispatch
a poor choice for comparing one pair of fingerprints, so `MetalKernel`
does not override `hammingDistance256`; it inherits the scalar version
for single pairs and only specializes the batched form,
`hammingDistanceBatch`, where the GPU's throughput advantage outweighs
its setup cost. `MetalBufferPool` removes a second fixed cost: rather
than allocate a fresh block of GPU-visible memory on every call, the
kernel allocates its buffers once, sized for up to `defaultMaxN`
(100,000) candidates, and reuses them on every call within that size;
a batch larger than that falls back to `dispatchWithFreshBuffers`,
which allocates for that call only.

`init?(maxN:)` builds the one-time Metal state: it asks the system for
its default GPU, compiles the compute shader embedded in
`shaderSource`, and builds the buffer pool. Any step failing —
because there is no GPU (a virtualized environment, for instance) or
the shader fails to compile — makes the initializer return `nil`,
which is a normal outcome, not an error condition: `PortableKernel`
falls back to `ScalarKernel` whenever `MetalKernel`'s initializer
returns `nil`. `hammingDistanceBatch` writes the probe and candidate
fingerprints into GPU-visible memory, dispatches one GPU thread per
candidate, waits for the GPU to finish, and reads the resulting
distances back; if any Metal call along that path fails, `scalarFallback`
computes the same batch on the CPU instead, so a transient GPU problem
degrades performance rather than producing a wrong or missing answer.
`hammingTopK` computes distances through the GPU path above, then
selects the K smallest with the same CPU-side max-heap `ScalarKernel`
uses, because GPU-side selection of a small K from a large result set
is not worth its complexity.

## HammingNN.swift

This file provides `HammingNN`, a standalone top-K search over
fingerprints that does not go through the `PortableKernel` protocol at
all.

Where `PortableKernel`'s backends operate on a dense array of
candidates and select the fastest hardware path, `HammingNN.topK`
operates over any sequence of `(rowID, fingerprint)` pairs and is meant
as a simple, general-purpose reference a caller can use directly
without choosing a backend. It also supports comparing only a subset
of a fingerprint's four blocks through its `blocks` parameter — useful
when a caller wants similarity on, say, only the topic-related portion
of a fingerprint rather than the whole thing. Internally it maintains
the same kind of bounded max-heap `ScalarKernel` uses, and it breaks
ties the same way in spirit — deterministically — though by the row's
UUID string rather than by array index, since this function's
candidates do not come with array positions of their own. `topK`
requires `k` to be positive and returns the results sorted by distance,
then by UUID string, so that two runs over the same data always
produce the same order.

## BitField.swift

This file provides `BitField`, a set of pure functions for reading and
writing fixed-width fields packed inside a 64-bit integer.

MOOTx01 packs several small values — flags, small counters, short
codes — into single 64-bit "bitmap" words, because doing so keeps
those words compact and fast to compare. Every package that needs to
read or write one of those packed fields is expected to call through
this file rather than write its own bit-shifting logic, so that a
change to how a field is laid out only has to be made once.
`extractField(_:shift:width:)` reads a field of `width` bits starting
at bit position `shift`; `writeField(_:into:shift:width:)` writes a new
value into that same position while leaving every other bit
untouched. `extractFlag` and `writeFlag` are the single-bit special
case of the same idea. `maskedEquals` tests whether a bitmap's masked
bits equal an expected value in one step, which is useful when a
caller only wants a yes-or-no answer and does not need the field's
actual value. `popcount`, `hammingDistance`, and `xorFold` round out
the file with the same bit-counting and bit-comparison operations the
kernel layer uses, but defined here over plain 64-bit integers instead
of 256-bit fingerprints, for callers working with a single packed
word rather than a whole fingerprint.

## SHA256.swift

This file provides `SHA256`, a complete implementation of the
SHA-256 cryptographic hash algorithm with no dependency on any
system library.

MOOTx01's audit log identifies each entry by a hash of its exact
contents, so that two replicas that received the same entry compute
the same identifier and can recognize it as the same entry without
comparing the whole thing. That only works if every device computes
the hash the same way, which is why this file implements the standard
itself instead of relying on a platform library that could change
behavior between operating system versions. The implementation was
carried over unchanged from an existing in-kit copy specifically so
that centralizing it here would not change any identifier a caller
had already computed and stored. `SHA256.hash(_:)` is the entire
public surface: it takes a list of bytes and returns the 32-byte
digest, following the standard's padding and compression steps
exactly.

## HKDF.swift

This file provides `GrantHKDF`, an implementation of HKDF-SHA256 (a
standard method for deriving new cryptographic keys from existing key
material), built entirely on top of this package's own `SHA256`.

MOOTx01's grant system needs to derive a key specific to one sharing
grant from an estate's underlying cryptographic identity, without
exposing that identity directly. Building this derivation on the
in-repo `SHA256`, rather than on a system cryptography library, means
a key derived on one platform is guaranteed to be identical to the
same derivation run on another platform, which is essential when two
devices must independently arrive at the same derived key.
`GrantHKDF.deriveKey(inputKeyMaterial:salt:info:outputByteCount:)` is
the public entry point; it runs the standard's two steps, extract and
expand, in sequence. `extract` and `expand` implement those two steps
directly from the specification. `hmac(key:data:)` implements the
keyed-hash construction both steps depend on and is itself public,
because SubstrateLib's commitment-signing feature needs the same
HMAC-SHA256 construction over its own data and this avoids a second
implementation existing anywhere in the substrate.

## FloatVecOps.swift

This file provides `FloatVecOps`, the canonical scalar implementations
of four vector operations: length, normalization, dot product, and
cosine similarity, all over ordinary lists of decimal numbers.

Just as `ScalarKernel` is the oracle for fingerprint operations,
`FloatVecOps` is the oracle for decimal-vector operations: any faster
implementation elsewhere in the substrate — using a platform math
library, for instance — is required to match these functions bit for
bit, not merely approximately. `l2Norm(_:)` computes a vector's length
by summing the square of each element and taking the square root.
`l2Normalize(_:)` rescales a vector to length one, which is the form
most similarity comparisons expect; a vector of all zeros has no
meaningful direction to rescale to, so the function returns it
unchanged rather than dividing by zero, and that unchanged zero vector
is itself a meaningful signal elsewhere in the substrate for "no
information available." `dot(_:_:)` sums the products of corresponding
elements from two equal-length vectors. `cosine(_:_:)` measures the
angle between two vectors, but only gives a meaningful answer when
both inputs are already unit-length; rather than normalize them itself
on every call, which would be wasted work when a caller normalizes
once and compares many times, it requires the caller to normalize
first and, in debug builds only, checks that the caller did so.

## Rust Port and Conformance

The `rust/` directory contains the second leg of the library: `kernel.rs`
mirrors `PortableKernel.swift`'s protocol, `ScalarKernel`, and
dispatcher; `bit_field.rs`, `hamming_nn.rs`, `sha256.rs`, `hkdf.rs`, and
`float_vec_ops.rs` mirror their same-named Swift files function for
function. Two Swift backends have no Rust counterpart. `NeonKernel`
does not need one, because Rust's own portable-SIMD path
(`kernel_simd.rs`, gated behind the nightly-only `simd-nightly` Cargo
feature) already covers the same aarch64 targets. `MetalKernel` cannot
have one: Metal is an Apple-only framework, and the Rust port's
documentation records this explicitly as a platform waiver rather than
a gap. The Rust leg also ships one backend the Swift leg lacks
entirely: `kernel_avx512.rs`, an AVX-512 implementation for x86-64
processors. It compiles and is reachable through explicit selection,
but `PortableKernel::for_current_platform()` never chooses it — it
stays a dark path until a future performance study proves it belongs
in the default path on real AVX-512 hardware. Because an unguarded call
into its processor-specific instructions would crash on hardware that
lacks them, every one of its entry points checks the processor's
actual feature flags at runtime before making that call, a guard
verified in a 2026-06-28 security review and covered by a dedicated
cross-platform test. Both legs share one conformance obligation:
whichever backend a caller selects, on either leg, its output must
match the scalar reference bit for bit, and the test suites for both
legs enforce this on every change.
