---
doc: OVERVIEW
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

# SubstrateTypes Overview

## What This Library Does

SubstrateTypes defines the smallest shared unit of information that MOOTx01
stores: a row. A row is one recorded observation — a diary entry, a fact
pulled from a knowledge graph, a proposal, a sample of ambient context — held
in one common shape so every other library can read it and write it the same
way. SubstrateTypes is where that shape, and the handful of coordinate
systems attached to it, are defined once for the whole SDK.

Two coordinate systems ride on every row. A fingerprint is a 256-bit code
that captures structural similarity: rows that look alike in shape produce
fingerprints that share many bits. A lattice anchor is a compact reference to
a lattice code, the classification code that LatticeLib's FDC engine assigns;
rows about the same subject carry related anchors. A third value, the Hybrid
Logical Clock (HLC), timestamps every change to a row so that two devices
holding copies of the same estate can agree on the order of events without
trusting either device's wall clock.

Around these three coordinate systems sit a small set of pure, deterministic
functions: Hamming distance and bitwise arithmetic over fingerprints,
SimHash construction from hyperplane families, FNV string hashing, and
OR-reduction for combining many fingerprints into one. These functions are
"canonical reference compute" — not heavy algorithms, but the exact
arithmetic that every consumer of the shapes must reproduce identically, on
every platform, forever.

## The Problem It Solves

MOOTx01's higher libraries — the ones that rank memories, detect drift, or
sync an estate across a phone and a laptop — all need to talk about the same
row shape. Without one shared definition, each library would invent its own
version of a fingerprint or a timestamp, and the definitions would drift
apart. A comparison between two rows would then depend on which library
produced them, which breaks every guarantee the system makes about
comparing memories.

SubstrateTypes solves this by being the single, dependency-free home for
these shapes. It ships as the lowest of four packages that together make up
the substrate: SubstrateTypes (this package, pure shape and zero compute
beyond canonical reference arithmetic), SubstrateKernel (dispatch to
hardware-accelerated backends), SubstrateML (learned models), and
SubstrateLib (the orchestration layer, which still owns the row-state
automaton and the verb mechanics that operate the shapes). SubstrateTypes
depends on none of the other three. A library that only needs to describe a
row's shape — for example, ConvergenceKit, which serializes rows for
CloudKit — depends on SubstrateTypes alone and pulls in no compute it does
not need.

A second problem the package solves is cross-platform agreement. MOOTx01
estates can federate, so a Swift-based Apple client and a Rust-based
service must compute the identical Hamming distance, the identical
fingerprint, and the identical hash for the same input. The package ships a
Rust port in `rust/` that mirrors every type and function here, so both legs
of the SDK share one arithmetic contract.

## How It Works

Every row (`Row`) carries a fixed set of fields: an identifier, a noun type
(what kind of thing it is — a diary entry, a proposal, and so on), a
lifecycle state, three bitmap columns of adjective, operational, and
provenance flags, a fingerprint, a lattice anchor, and optional lineage and
content references. `RowState` and `RowBitmaps` describe the lifecycle and
the bitmap layout in more detail; both are pure data, so the automaton that
enforces which state transitions are legal lives one layer up, in
SubstrateLib.

The fingerprint is built by SimHash, a locality-sensitive hash: unlike a
cryptographic hash, where a one-bit change in the input scrambles the whole
output, SimHash produces outputs where similar inputs share many bits. Each
of the fingerprint's four 64-bit blocks is computed by comparing an input
vector against sixty-four random hyperplanes — flat dividing surfaces fixed
at estate creation — recorded in a `HyperplaneFamily`. Once built,
fingerprints are compared and combined by a small algebra: `Hamming.distance`
counts differing bits, `BitwiseArithmetic` computes intersection and
symmetric difference, `ORReduce` folds many fingerprints into one that keeps
their shared structure while losing which fingerprint contributed which bit,
and `CountVector256` accumulates per-bit counts across a whole cohort so a
majority-vote fingerprint, or any other statistic, can be read off later.

Every change to a row is recorded, never overwritten. `AuditEvent` is one
recorded mutation; `GSetAuditLog` is a grow-only set of such events, a
conflict-free replicated data type (CRDT). Two replicas of an estate can
exchange their audit logs in any order, merge by taking the union of
entries, and land on the same result, because set union does not care about
order. The `HLC` timestamp on every event gives the log a total order to
replay, and `TimeRange` and `AsOfCoordinate` let a caller ask what a row
looked like at a past point in that order.

A separate family of types tracks population-level statistics across an
entire estate rather than one row at a time. `MatrixF` counts how often each
bitmap bit is set across all rows; `MatrixC` derives the marginal probability
of each bit from `MatrixF`; `MatrixO` counts how often pairs of bitmap values
co-occur; `MatrixT` counts how often one bitmap value precedes another within
a time window, which is the substrate's tool for telling correlation apart
from likely causation. `ThreeDBitTensor` is the dense, bit-sliced storage
layout that answers "which rows have field X set to value Y" quickly across
a million rows.

A last family of types protects data integrity and supports recall.
`ContentHash` and `MerkleRoot` are two distinct fixed-size hashes —
one per leaf payload, one per subtree of children — kept as separate types so
the compiler rejects any code that confuses them; `MerkleDomain` supplies the
one-byte tags that keep leaf, interior, and tombstone hashes from ever
colliding. `RecallTypes` defines the wire vocabulary — `RecallScore`,
`RecallResult`, `DistanceBreakdown`, `RowProjection` — that every recall
primitive and every federation query returns, so composing results from
different recall strategies never requires an ad hoc translation step.

## How the Pieces Fit

Figure 1 shows how the major types connect: a row's own fields, the
fingerprint construction pipeline, the fingerprint algebra, the
population-statistics matrices, the audit trail, and the integrity and
recall vocabularies that read from a row without holding a reference to it.

![Figure 1. Topology of SubstrateTypes](topology.svg)

*Figure 1. Topology of SubstrateTypes. A row carries a fingerprint and a
lattice anchor. Hyperplane families and SimHash build the fingerprint; the
fingerprint algebra (Hamming distance, bitwise arithmetic, OR-reduction,
count-vectors) operates on it afterward. The row's bitmaps feed the
population-statistics matrices. Every mutation is recorded as an audit event
under Hybrid-Logical-Clock ordering in the grow-only audit log. Integrity
hashes and the recall wire vocabulary both read from row data without
belonging to the row shape itself.*

Nothing in this package reaches upward. The row-state automaton, the verb
implementations that call it, and the full substrate object that bundles
rows with an audit log and the statistics matrices all remain in
SubstrateLib. SubstrateTypes supplies the vocabulary; SubstrateLib supplies
the behavior.

## What Ships in the Package

The package ships thirty Swift source files under `Sources/SubstrateTypes/`
and a mirrored Rust crate under `rust/src/`, one Rust module per Swift file
with matching names. There are no pinned data artifacts in this package —
unlike a library such as LatticeLib, SubstrateTypes carries no bundled
reference tables, because every value here is either supplied by a caller
(a hyperplane seed, a row's own fields) or computed from caller-supplied
input by a pure function (a Hamming distance, a SimHash block, an FNV hash).
Reproducibility rests on the arithmetic itself being pinned, verified by
conformance tests that check the Swift and Rust legs agree bit for bit.
