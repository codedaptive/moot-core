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
`OVERVIEW.md` first for the big picture. Files appear here in one
order. First comes the shared protocol and its reference
implementation. Next come the three specialized hardware backends.
Then comes the standalone top-K primitive. Last come three independent
utilities: bit fields, hashing, and float-vector math.

## PortableKernel.swift

This file provides the `SubstrateKernel` protocol. It also provides the
`ScalarKernel` reference implementation. It provides the
`PortableKernel` dispatcher, which hands callers a concrete kernel.

The protocol declares every operation a kernel must supply. `popcount64`
counts the one-bits in a 64-bit word. `hammingDistance256` counts
differing bits between two fingerprints. `orReduce256` combines a group
of fingerprints with a bitwise OR. `hammingTopK` finds the K candidates
nearest a probe fingerprint. `simhashCompute` folds a set of 64-bit
numbers into a fingerprint. `countFold256` tallies, per bit position,
how many fingerprints in a group have that bit set. `floatSimHashProject`
does the same folding, but starts from a vector of decimal numbers
instead of integers. A protocol extension supplies default batched
variants of each operation. These defaults are simple loops over the
single-item version. They are correct, but not optimized. A backend
with a faster batched strategy overrides the default. Every backend is
correct even before it does.

`ScalarKernel` implements every required method as a plain loop. It
uses no special hardware instructions. It is deliberately the simplest
possible correct implementation, because it is the oracle. A backend
counts as correct only when its output matches `ScalarKernel`'s output,
for the same input. Two of its methods matter beyond their obvious
job. `hammingTopK` keeps a bounded max-heap of the K best candidates
seen so far. It evicts the current worst kept candidate whenever a
better one arrives. This keeps the cost proportional to N times the
log of K, rather than N times the log of N. Ties between equal
distances break toward the candidate with the lower index. Every other
backend must match that exact tie-break.
`floatSimHashProject`'s default body lives in the protocol extension.
Every backend that does not override it inherits this body, and that
body is itself the reference for the operation. For each of two
hundred fifty-six hyperplanes, it accumulates a signed sum over the
input vector in a fixed left-to-right order. Floating-point addition
is not associative, so the order is part of the contract. It is not
just an implementation detail.

`PortableKernel.kernelForCurrentPlatform()` is the entry point most
callers use. On 64-bit ARM, known as `arm64`, it returns `SimdKernel`.
This covers both Apple Silicon and ARM64 Linux. Everywhere else it
returns `ScalarKernel`. The choice is fixed at compile time for a
given platform. There is no runtime hardware probing here. This
contrasts with the Rust port's AVX-512 backend, which checks the
processor's feature flags at runtime, because that backend is unsafe
to run without them. Every call reports which backend it chose,
through IntellectusLib's telemetry system, but only when telemetry is
enabled. When telemetry is off, the reporting closure never runs, and
no clock is read. So the selection has no cost in normal operation.
`PortableKernel.kernel(of:)` is the explicit-selection counterpart.
Tests use it. So does any caller that wants a specific backend
regardless of platform. Requesting `.metal` falls back to
`ScalarKernel` if no GPU is available. Requesting `.avx512` or `.avx2`
always falls back to `ScalarKernel` in this Swift build, since neither
has a Swift implementation. `PortableKernel.assertEqual` runs
`hammingTopK` on two kernels with the same inputs. It reports whether
their results match. This is the building block the conformance test
suite uses to check every backend against the oracle.

## PortableKernel-SIMD.swift

This file provides `SimdKernel`. It is the backend
`kernelForCurrentPlatform` selects on 64-bit ARM.

`SimdKernel` uses Swift's `simd` module. This module represents four
64-bit numbers as one `SIMD4<UInt64>` value. It lets ordinary operators
like XOR and OR act on all four at once. The Swift compiler turns
those operators into single ARM NEON instructions, rather than four
separate ones. `hammingDistance256` XORs two fingerprints as
`SIMD4<UInt64>` values. It then counts the one-bits in each of the four
resulting 64-bit lanes, and adds the four counts together.
`hammingTopK` uses a different strategy from `ScalarKernel`'s heap. It
keeps a small sorted array of the K best candidates found so far,
called a ladder, and shifts it to make room for a new entry. The
file's comments note that this is faster than a heap for the small K
values this package expects. It produces the identical sorted output
and tie-break.

`simhashBlockBatch` computes one 64-bit SimHash block for many input
vectors, against the same hyperplane family. A hyperplane family is a
fixed set of 64 reference directions. Each direction decides, for one
bit of the output, which side of that direction the input vector falls
on. Rather than compute that decision one hyperplane at a time for
every input, `PackedFamily` rearranges the family's 64 hyperplanes once,
into groups of four. This lets the per-input inner loop process four
hyperplanes together, with the same `SIMD4<UInt64>` trick used for
Hamming distance. Building this packed layout costs a little extra
time up front. That cost is worthwhile, because it is paid once, and
the savings apply to every input in the batch.

`countFold256` computes, for a group of fingerprints, how many of them
have each of the 256 bit positions set. It does not count each
fingerprint's contribution one position at a time. Instead, `SimdKernel`
keeps a growing binary counter for each bit position. It holds this
counter across `SIMD4<UInt64>` lanes, and folds each new fingerprint in
with two bitwise operations. The counter's next bit is the counter XOR the
fingerprint. The amount to carry into the following bit is the counter
AND the fingerprint. This produces the same result as adding one for
every set bit. It does the addition four fingerprint-blocks at a time,
instead of bit by bit.

## PortableKernel-NEON.swift

This file provides `NeonKernel`. It is an alternate ARM implementation,
kept side by side with `SimdKernel`, to test a different way of
expressing the same computation.

`SimdKernel` counts bits four 64-bit words at a time. `NeonKernel`
instead treats a fingerprint as 32 individual bytes. It counts bits one
byte at a time, then adds all 32 counts together. The file's comments
record the reasoning. Processors often have an instruction that counts
bits across a whole row of bytes in one step. The hope was that
framing the computation at the byte level would let the compiler use
that instruction better than the word-level framing does. Whether it
actually helps is a question the package answers by measurement, not
assumption. A companion benchmark compares the two. `NeonKernel` exists
so that measurement has something concrete to measure. It is never
chosen automatically. Callers reach it only through
`PortableKernel.kernel(of: .neon)`.

Every method mirrors its `SimdKernel` counterpart at the byte level,
instead of the word level. `hammingDistance256` and
`hammingDistanceBatch` XOR two byte-vectors, and sum per-byte bit
counts. `hammingTopK` uses the same ladder-based top-K selection as
`SimdKernel`. `orReduce256`, `orReduceBatch`, and `simhashCompute` are
not re-framed at the byte level. The file's comments judge that the
byte-level trick that helps Hamming distance does not help those
operations. So they use the same implementation `SimdKernel` uses.

## PortableKernel-Metal.swift

This file provides `MetalKernel`. It runs the batched Hamming distance
computation on the graphics processor, through Apple's Metal framework.
It is available only on Apple platforms.

Dispatching work to the GPU has a fixed cost per call. Setting up
buffers and handing off the computation takes tens of microseconds,
and that cost does not shrink as the batch grows. That fixed cost
makes GPU dispatch a poor choice for comparing one pair of
fingerprints. So `MetalKernel` does not override `hammingDistance256`.
It inherits the scalar version for single pairs. It only specializes
the batched form, `hammingDistanceBatch`, where the GPU's throughput
advantage outweighs its setup cost. `MetalBufferPool` removes a second
fixed cost. Rather than allocate a fresh block of GPU-visible memory on
every call, the kernel allocates its buffers once. It sizes them for
up to `defaultMaxN`, or one hundred thousand candidates, and reuses
them on every call within that size. A batch larger than that falls
back to `dispatchWithFreshBuffers`, which allocates fresh buffers for
that call only.

`init?(maxN:)` builds the one-time Metal state. It asks the system for
its default GPU. It compiles the compute shader embedded in
`shaderSource`. It builds the buffer pool. Any step can fail: there may
be no GPU, as in a virtualized environment, or the shader may fail to
compile. Either failure makes the initializer return `nil`. That is a
normal outcome, not an error condition. `PortableKernel` falls back to
`ScalarKernel` whenever `MetalKernel`'s initializer returns `nil`.
`hammingDistanceBatch` writes the probe and candidate fingerprints into
GPU-visible memory. It dispatches one GPU thread per candidate. It
waits for the GPU to finish, then reads the resulting distances back.
If any Metal call along that path fails, `scalarFallback` computes the
same batch on the CPU instead. So a transient GPU problem degrades
performance, rather than producing a wrong or missing answer.
`hammingTopK` computes distances through the GPU path above. It then
selects the K smallest with the same CPU-side max-heap `ScalarKernel`
uses. GPU-side selection of a small K, from a large result set, is not
worth its complexity.

## HammingNN.swift

This file provides `HammingNN`. It is a standalone top-K search over
fingerprints that does not go through the `PortableKernel` protocol at
all.

`PortableKernel`'s backends operate on a dense array of candidates, and
select the fastest hardware path. `HammingNN.topK` instead operates
over any sequence of `(rowID, fingerprint)` pairs. It is meant as a
simple, general-purpose reference. A caller can use it directly,
without choosing a backend. It also supports comparing only a subset
of a fingerprint's four blocks, through its `blocks` parameter. This is
useful when a caller wants similarity on just one part of a
fingerprint, such as the topic-related portion, instead of the whole
fingerprint. Internally it
keeps the same kind of bounded max-heap `ScalarKernel` uses. It breaks
ties the same way in spirit, meaning deterministically, though by the
row's UUID string rather than by array index. This function's
candidates do not come with array positions of their own. `topK`
requires `k` to be positive. It returns the results sorted by distance,
then by UUID string, so that two runs over the same data always
produce the same order.

## BitField.swift

This file provides `BitField`. It is a set of pure functions for
reading and writing fixed-width fields packed inside a 64-bit integer.

MOOTx01 packs several small values, such as flags, small counters, and
short codes, into single 64-bit words, called bitmaps. Doing so keeps
those words compact and fast to compare. Every package that reads or
writes one of those packed fields is expected to call through this
file. It should not write its own bit-shifting logic. That way, a
change to a field's layout only has to be made once.
`extractField(_:shift:width:)` reads a field of `width` bits, starting
at bit position `shift`. `writeField(_:into:shift:width:)` writes a new
value into that same position, while leaving every other bit
untouched. `extractFlag` and `writeFlag` are the single-bit special
case of the same idea. `maskedEquals` tests whether a bitmap's masked
bits equal an expected value in one step. This is useful when a caller
only wants a yes-or-no answer, and does not need the field's actual
value. `popcount`, `hammingDistance`, and `xorFold` round out the file
with the same bit-counting and bit-comparison operations the kernel
layer uses. Here they work over plain 64-bit integers, instead of
256-bit fingerprints. This suits callers with a single packed word,
rather than a whole fingerprint.

## SHA256.swift

This file provides `SHA256`. It is a complete implementation of the
SHA-256 cryptographic hash algorithm, with no dependency on any system
library.

MOOTx01's audit log identifies each entry by a hash of its exact
contents. That way, two replicas that receive the same entry compute
the same identifier. Each replica can then recognize the entry as the
same, without comparing the whole thing. This only works if every
device computes the hash the same way. That is why this file
implements the standard itself. It does not rely on a platform library
that could change behavior between operating system versions. The
implementation carries
over unchanged from an existing in-kit copy. Centralizing it here does
not change any identifier a caller had already computed and stored.
`SHA256.hash(_:)` is the entire public surface. It takes a list of
bytes and returns the 32-byte digest, following the standard's padding
and compression steps exactly.

## HKDF.swift

This file provides `GrantHKDF`. It is an implementation of
HKDF-SHA256, a standard method for deriving new cryptographic keys
from existing key material. It builds entirely on top of this
package's own `SHA256`.

MOOTx01's grant system needs to derive a key specific to one sharing
grant. It derives this key from an estate's underlying cryptographic
identity, without exposing that identity directly. This derivation
builds on the in-repo
`SHA256`, rather than on a system cryptography library. That way, a
key derived on one platform is guaranteed to match the same derivation
run on another platform. This guarantee matters when two devices must
independently arrive at the same derived key.
`GrantHKDF.deriveKey(inputKeyMaterial:salt:info:outputByteCount:)` is
the public entry point. It runs the standard's two steps, extract and
expand, in sequence. `extract` and `expand` implement those two steps
directly from the specification. `hmac(key:data:)` implements the
keyed-hash construction both steps depend on. It is itself public,
because `SubstrateLib`'s commitment-signing feature needs the same
HMAC-SHA256 construction over its own data. This avoids a second
implementation existing anywhere in the substrate.

## FloatVecOps.swift

This file provides `FloatVecOps`. It holds the canonical scalar
implementations of four vector operations: length, normalization, dot
product, and cosine similarity. All four operate over ordinary lists
of decimal numbers.

Just as `ScalarKernel` is the oracle for fingerprint operations,
`FloatVecOps` is the oracle for decimal-vector operations. Any faster
implementation elsewhere in the substrate must match these functions
bit for bit. This includes an implementation that uses a platform math
library. The match must be exact, not merely approximate. `l2Norm(_:)`
computes a vector's length. It sums
the square of each element, then takes the square root.
`l2Normalize(_:)` rescales a vector to length one, which is the form
most similarity comparisons expect. A vector of all zeros has no
meaningful direction to rescale to. So the function returns it
unchanged, rather than dividing by zero. That unchanged zero vector
itself carries meaning. Elsewhere in the substrate, it signals "no
information available." `dot(_:_:)` sums the products of corresponding
elements from two equal-length vectors. `cosine(_:_:)` measures the
angle between two vectors. It only gives a meaningful answer when both
inputs are already unit-length. Normalizing on every call would waste
work, when a caller normalizes once and compares many times. So the
function requires the caller to normalize first instead. In debug
builds only, it also checks that the caller did so.

## Rust Port and Conformance

The `rust/` directory holds the second leg of the library. `kernel.rs`
mirrors `PortableKernel.swift`'s protocol, `ScalarKernel`, and
dispatcher. Five files mirror their same-named Swift files, function
for function: `bit_field.rs`, `hamming_nn.rs`, `sha256.rs`, `hkdf.rs`,
and `float_vec_ops.rs`. Two Swift backends have no Rust counterpart.
`NeonKernel` does not need one. Rust's own portable-SIMD path,
`kernel_simd.rs`, already covers the same aarch64 targets. This path is
gated behind the nightly-only `simd-nightly` Cargo feature.
`MetalKernel` cannot have one, since Metal is an Apple-only framework.
The Rust port's documentation records this fact as a platform waiver,
not a gap.

The Rust leg also ships one backend the Swift leg lacks entirely:
`kernel_avx512.rs`, an AVX-512 implementation for x86-64 processors. It
compiles, and is reachable through explicit selection, but
`PortableKernel::for_current_platform()` never chooses it. It stays a
dark path, until a future performance study proves it belongs in the
default path on real AVX-512 hardware. An unguarded call into its
processor-specific instructions would crash on hardware that lacks
them. So every one of its entry points checks the processor's actual
feature flags at runtime, before making that call. This guard was
verified in a 2026-06-28 security review. A dedicated cross-platform
test covers it.

Both legs share one conformance obligation. Whichever backend a caller
selects, on either leg, its output must match the scalar reference bit
for bit. The test suites for both legs enforce this rule on every
change.
