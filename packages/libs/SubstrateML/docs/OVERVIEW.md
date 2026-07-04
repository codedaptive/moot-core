---
doc: OVERVIEW
package: SubstrateML
repo: moot-core
authored_commit: b2a5c30b794cf477e18022c55e2fea348614d337
authored_date: 2026-07-04
sources:
  - path: Sources/SubstrateML/ActionOutcomeMatrix.swift
    blob: 612ee840126c72dd0d07505147ba2991ac3bb0b9
  - path: Sources/SubstrateML/AnomalyDetection.swift
    blob: ff7d227353ebfc69cf01b5c468b2612d307b8632
  - path: Sources/SubstrateML/AprioriMining.swift
    blob: 189a4408f3c6e600a538a6cfc560d24688cbd2af
  - path: Sources/SubstrateML/AssociationRuleMining.swift
    blob: 9f20faa808883c065412face0063c7682eabbfd6
  - path: Sources/SubstrateML/AuditLogFold.swift
    blob: 7423db7764c628cf5fd2aab0e9bb2d98d8c687bd
  - path: Sources/SubstrateML/BradleyTerry.swift
    blob: 63d335ee50ccee803f0508eb6f3135a29b6545db
  - path: Sources/SubstrateML/CommunityDetection.swift
    blob: bfba8e0255baeb66d4c79ccfaf34cd11e21a5ebd
  - path: Sources/SubstrateML/CompositeDistance.swift
    blob: a5f94c631d89c92d45b60e9da43be7fe6bdd19a2
  - path: Sources/SubstrateML/ConceptImplications.swift
    blob: 1ac6ccbb70665532f135d2813f260cbbdad21716
  - path: Sources/SubstrateML/DeltaFeatureExtractor.swift
    blob: 1ca396968e5e565096dbebb059b45b8fc3e5719f
  - path: Sources/SubstrateML/DistillationPipeline.swift
    blob: 82ce8bfaaa2e7a92077510ef239a6997a585c888
  - path: Sources/SubstrateML/DistillationScorer.swift
    blob: 2cdc6dfe121080d432859d17487f2c8d15fd2910
  - path: Sources/SubstrateML/DPORReduction.swift
    blob: 3d3c79e8ce7694b8308bb1c970e86a24de0446b0
  - path: Sources/SubstrateML/EigenvalueCentrality.swift
    blob: f427430c34add289231520ba19ae4781316cbd13
  - path: Sources/SubstrateML/FeatureExtractors.swift
    blob: 7e6d388a588ff7bec04be631948387a086a253ed
  - path: Sources/SubstrateML/FFT.swift
    blob: fdfaceeb902428149ead96107ed69c308bca0fff
  - path: Sources/SubstrateML/FloatSimHash.swift
    blob: 39cd2c4c36492c7cbaf55d6d22a2f32e273cfbae
  - path: Sources/SubstrateML/FormalConceptAnalysis.swift
    blob: 9cdf9613b30bf2e3a9a85088489c0e2b683a8a77
  - path: Sources/SubstrateML/InformationTheory.swift
    blob: f7ba2a27904ceb19eef0fb5b06d1c5aad8b3894b
  - path: Sources/SubstrateML/JacobiSVD.swift
    blob: fcd924ff0a8409f224b56e7229f59659a6b5f51e
  - path: Sources/SubstrateML/LatticeDistance.swift
    blob: a7b96a376dfba0c815e4dc91962da1ccaeb2ac1d
  - path: Sources/SubstrateML/LLMCalibrationCurve.swift
    blob: 4e01882a65570ccce72e2789c23e95c46b6399a7
  - path: Sources/SubstrateML/MatrixDecay.swift
    blob: b9595064a54b79f8ddee343385af4b8ebdc7f3e5
  - path: Sources/SubstrateML/MomentSummary.swift
    blob: e8c07841c8c3440deb0375500ebcd35a6f116bb4
  - path: Sources/SubstrateML/NMFAlternatingLeastSquares.swift
    blob: d64aadc8db165cc3b9f51b4fb1c644a6d6a4ca3c
  - path: Sources/SubstrateML/NMFDoubleFrobeniusSquared.swift
    blob: 0a725d72b70120828cb5911de594b0278ca450e5
  - path: Sources/SubstrateML/PairingHandshake.swift
    blob: cb608882478ec57a6e6ba1f7728646ff3ea610f1
  - path: Sources/SubstrateML/PartialStateRecall.swift
    blob: ad7b42bc25f9c4f283d75593d2c42cd546ccf074
  - path: Sources/SubstrateML/RandomWalks.swift
    blob: 99f40b1685a876d81f84ca38b4857e5c6ba4e1dd
  - path: Sources/SubstrateML/RowAttributeView.swift
    blob: f63cf544a87adde435bede527531d1a9414dad03
  - path: Sources/SubstrateML/Sampling.swift
    blob: 84fcdcade559229003688a7fc151f5c0a6dac6e6
  - path: Sources/SubstrateML/ShingleSimilarity.swift
    blob: bf9dbaa5b0d5a5ee4ec5dfe42a5d65c85bdd0326
  - path: Sources/SubstrateML/TemporalCausalityFold.swift
    blob: d99d7358893eeae0a442e0bd4e52f8a9b1cd7f5e
  - path: Sources/SubstrateML/TemporalCompression.swift
    blob: f34dde022faadd40058e35b62da12c8985763e59
  - path: Sources/SubstrateML/TierAscendingQuery.swift
    blob: 8681cbdae9af7facfe7792392dcee15c8f8fe0aa
  - path: Sources/SubstrateML/TierContributionFingerprint.swift
    blob: bb56b7a82532f8c324168bd81edff6bf68fb33e9
  - path: Sources/SubstrateML/TypedDecayWeighting.swift
    blob: 82412f392848c378b5b28221bbddade9f03bf1bf
  - path: Sources/SubstrateML/VizGraphSignals.swift
    blob: 4f90110f2a19cb92f3d45baf3bc4d82ac9c3dc65
---

# SubstrateML Overview

## What This Library Does

SubstrateML is the learning layer of the MOOTx01 substrate. MOOTx01 is an
on-device AI memory system. It stores what an AI observes over time and
helps the AI recall it later. The substrate is the mathematical foundation
under that store: the types, the bit operations, and the algorithms that
every higher layer builds on.

The substrate is split into layered packages. `SubstrateTypes` holds pure
data types, such as the 256-bit row fingerprint. `SubstrateKernel` holds
hot-path bit operations that run on every capture. SubstrateML — this
package, layer 3 — holds the cold-path algorithms: the math that learns
structure from stored memories rather than storing them. A memory in this
system is a row: one record with a fingerprint, a classification anchor,
and a set of bitmap fields. SubstrateML never touches storage. Every
function here takes plain values in and returns plain values out.

Most of these algorithms run during dreaming. Dreaming is the system's
idle-time maintenance cycle: a background daemon that clusters, decays,
summarizes, and re-scores the memory estate while the device is otherwise
quiet. An estate is one user's complete memory store. The algorithms here
find the estate's themes, its communities, its habits, its rules, and its
condensable clusters — and they forget on schedule, so old evidence fades.

## The Problem It Solves

An on-device memory system must learn without a cloud. Cloud machine
learning changes without notice, needs a network, and sees private data.
SubstrateML instead ships small, exact, well-bounded reference algorithms
that run entirely on the device.

It must also learn identically everywhere. MOOTx01 estates can federate,
which means separate devices share and compare results. Two devices that
factor the same matrix or mine the same rules must get the same answer, or
shared recall falls apart. SubstrateML therefore holds one agreement
property across two implementations: a Swift leg for Apple platforms and a
Rust leg (in `rust/`) for everything else. Both legs use the same canonical
pseudo-random number generator (SplitMix64), the same pinned seeds, the
same tie-breaking rules, and the same arithmetic order, so results are
bit-identical. Conformance fixtures — recorded input and output pairs both
legs must reproduce exactly — gate every change.

Two library-wide rules protect that promise. First, SubstrateML never reads
a clock; every timestamp is passed in by the caller. Second, no algorithm
here holds hidden state; everything is a pure function or a plain value
type, safe to run from any thread.

## How It Works

The thirty-eight source files form seven working groups.

**Fingerprint and distance math.** A fingerprint is a short fixed-size code
computed from content; similar content gives similar fingerprints.
`FloatSimHash` projects float embedding vectors from external models into
the substrate's 256-bit fingerprint form. `CompositeDistance` blends
classification distance and fingerprint distance into the one score recall
ranks by. `LatticeDistance`, `PartialStateRecall`, and `ShingleSimilarity`
supply specialized distances. `MomentSummary` and `TemporalCompression`
OR-reduce many row fingerprints into one signature per time window, so the
system can ask "what was going on during this hour" as a single comparison.

**Ingestion shaping.** `FeatureExtractors` turns raw ambient sensor samples
(health, location, calendar, screen time, telemetry) into fingerprinted
rows. `AuditLogFold` replays a row's append-only change log to reconstruct
its state at any point in time. `RowAttributeView` reshapes that same log
into flat attribute lists that the pattern miners consume.

**Learning and decay.** `MatrixDecay` applies exponential half-life decay
to every statistics matrix — the system's forgetting mechanism.
`ActionOutcomeMatrix` tracks which actions succeed. `BradleyTerry` learns
per-row ranking strength from recall feedback. `LLMCalibrationCurve` tracks
how honest an LLM's confidence claims are. `Sampling` provides the
deterministic Normal, Gamma, and Beta samplers under Thompson-sampling
decisions.

**Graph analytics.** The estate graph connects rows by association.
`CommunityDetection` (Louvain) finds its clusters. `EigenvalueCentrality`
scores each row's authority for keystone recall. `RandomWalks` wanders the
graph for exploratory recall. `NMFAlternatingLeastSquares` factors the
matrices into latent themes. `AnomalyDetection` flags unusual values.
`JacobiSVD` and `FFT` supply the deterministic linear algebra and rhythm
analysis beneath semantic embeddings and periodicity detection. These five
graph algorithms emit telemetry signals, named in `VizGraphSignals`, when
monitoring is enabled.

**Pattern mining.** `AssociationRuleMining` and `AprioriMining` find
"when A is set, B tends to be set" rules. `FormalConceptAnalysis` and
`ConceptImplications` find exact groupings and always-true implications.
`TemporalCausalityFold` mines "X changed, then Y changed N minutes later"
statistics. `InformationTheory` supplies the entropy and divergence math.

**Distillation.** Distillation compresses a cluster of related memories
into one condensed factoid. `DistillationScorer` decides whether a cluster
is coherent enough. `DeltaFeatureExtractor` and `TypedDecayWeighting`
handle trends and staleness. `DistillationPipeline` runs the whole
five-stage algorithm and emits the factoid plus its fingerprint.

**Federation and privacy.** `PairingHandshake` lets two estates derive a
shared fingerprint basis with no extra network round trip.
`TierContributionFingerprint` packs an estate's shareable summary into a
fixed 64-byte wire format. `TierAscendingQuery` and `DPORReduction` add
differential-privacy noise and k-anonymity so aggregate answers never
expose any single estate's contribution.

## How the Pieces Fit

Figure 1 shows the library's topology — its major parts and how data moves
between them.

![Figure 1. Topology of SubstrateML](topology.svg)

*Figure 1. Topology of SubstrateML. Rows, audit events, and sensor samples
enter on the left. The shaping layer feeds the four algorithm families.
Dashed regions mark the substrate packages below and the telemetry seam;
all outputs return to the calling kits, never to storage.*

SubstrateML has no single facade. Each algorithm family is its own entry
point, and the consuming kits — LocusKit, CognitionKit, GeniusLocusKit,
NeuronKit, and the dreaming daemon — call the piece they need. A kit is a
larger package that composes libraries into a subsystem; kits depend on
libs, never the reverse. The package depends downward only: on
`SubstrateTypes` for shared types, on `SubstrateKernel` for dispatched bit
kernels, and on `IntellectusLib`, the zero-dependency telemetry leaf. When
monitoring is off (the default), every telemetry emit costs one atomic
boolean load and nothing more.

## What Ships in the Package

The package ships the thirty-eight Swift sources, a matching test suite,
and the Rust port in `rust/` (crate `substrate-ml`, one module per Swift
file plus shared conformance tests). There are no bundled data artifacts;
every input is caller-supplied. Determinism is carried instead by pinned
constants — seeds, thresholds, half-lives, and bucket tables recorded in
the sources and locked by conformance vectors — so the same input produces
the same result on every platform, every time.
