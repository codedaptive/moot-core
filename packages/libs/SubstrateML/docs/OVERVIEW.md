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
    blob: ca87cf2e57a5469b950ab46f4951ca3a05c1c864
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

SubstrateML is the learning layer of the MOOTx01 substrate. MOOTx01 is
an on-device AI memory system. It stores what an AI observes over
time. It helps the AI recall that later. The substrate is the math
base under that store. It holds the types, the bit operations, and the
algorithms that every higher layer needs.

The substrate splits into layered packages. `SubstrateTypes` holds
pure data types. One example is the 256-bit row fingerprint.
`SubstrateKernel` holds hot-path bit operations. These run on every
capture. SubstrateML is layer three. It holds the cold-path
algorithms: math that learns structure from stored memories. It does
not store memories itself.

A memory in this system is a row. Each row has one fingerprint, one
classification anchor, and a set of bitmap fields. SubstrateML never
touches storage. Every function here takes plain values in and
returns plain values out.

Most of these algorithms run during dreaming. Dreaming is the
system's idle-time maintenance cycle. A background daemon runs it
while the device sits quiet. The daemon clusters memories. It decays
old evidence. It summarizes clusters and re-scores the memory estate.

An estate is one user's complete memory store. The algorithms here
find the estate's themes, its communities, and its habits. They find
its rules and its condensable clusters. They also forget on a
schedule, so old evidence fades over time.

## The Problem It Solves

An on-device memory system must learn without a cloud. Cloud machine
learning changes without notice. It needs a network. It sees private
data. SubstrateML avoids all three problems. It ships small, exact,
well-bounded reference algorithms. Every one of them runs entirely on
the device.

The substrate must also learn the same way everywhere. MOOTx01
estates can federate: separate devices share and compare results. Two
devices that factor the same matrix must get the same answer. Two
devices that mine the same rules must also agree. Otherwise shared
recall falls apart.

SubstrateML holds one agreement property across two implementations.
A Swift leg serves Apple platforms. A Rust leg, in `rust/`, serves
everything else. Both legs share one canonical pseudo-random number
generator, SplitMix64. Both use the same pinned seeds. Both use the
same tie-breaking rules and the same arithmetic order. The result is
bit-identical output on both legs. Conformance fixtures gate every
change. A fixture is a recorded input and output pair that both legs
must reproduce exactly.

Two library-wide rules protect that promise. First, SubstrateML never
reads a clock. Every timestamp comes from the caller instead. Second,
no algorithm here holds hidden state. Each one is a pure function or
a plain value type. Each one is safe to run from any thread.

## How It Works

The package holds thirty-eight source files. They form seven working
groups.

**Fingerprint and distance math.** A fingerprint is a short fixed-size
code computed from content. Similar content gives similar
fingerprints. `FloatSimHash` projects float embedding vectors from
external models into the substrate's 256-bit fingerprint form.
`CompositeDistance` blends classification distance and fingerprint
distance into one score. Recall ranks candidates by that score.
`LatticeDistance`, `PartialStateRecall`, and `ShingleSimilarity` supply
specialized distances for narrower needs. `MomentSummary` and
`TemporalCompression` reduce many row fingerprints into one signature
per time window. That lets the system compare "what was going on
during this hour" as a single lookup.

**Ingestion shaping.** `FeatureExtractors` turns raw ambient sensor
samples into fingerprinted rows. The samples come from health,
location, calendar, screen time, and telemetry sources.
`AuditLogFold` replays a row's append-only change log. That replay
reconstructs the row's state at any point in time. `RowAttributeView`
reshapes that same log into flat attribute lists. The pattern miners
consume those lists directly.

**Learning and decay.** `MatrixDecay` applies exponential half-life
decay to every statistics matrix. This is the system's forgetting
mechanism. `ActionOutcomeMatrix` tracks which actions succeed.
`BradleyTerry` learns a per-row ranking strength from recall feedback.
`LLMCalibrationCurve` tracks how honest a model's confidence claims
are. `Sampling` provides deterministic Normal, Gamma, and Beta
samplers. Thompson-sampling decisions draw on these samplers.

**Graph analytics.** The estate graph connects rows by association.
`CommunityDetection` runs Louvain clustering to find its communities.
`EigenvalueCentrality` scores each row's authority for keystone
recall. `RandomWalks` wanders the graph for exploratory recall.
`NMFAlternatingLeastSquares` factors the matrices into latent themes.
`AnomalyDetection` flags unusual values. `JacobiSVD` and `FFT` supply
deterministic linear algebra and rhythm analysis. Both sit beneath
semantic embeddings and periodicity detection. These five graph
algorithms emit telemetry signals when monitoring is enabled. The
signal names live in `VizGraphSignals`.

**Pattern mining.** `AssociationRuleMining` and `AprioriMining` find
rules of the form "when A is set, B tends to be set."
`FormalConceptAnalysis` and `ConceptImplications` find exact groupings
and implications that always hold. `TemporalCausalityFold` mines
statistics of the form "X changed, then Y changed some minutes later."
`InformationTheory` supplies the entropy and divergence math behind
these miners.

**Distillation.** Distillation compresses a cluster of related
memories into one condensed factoid. `DistillationScorer` decides
whether a cluster is coherent enough to compress.
`DeltaFeatureExtractor` and `TypedDecayWeighting` handle trends and
staleness in the underlying features. `DistillationPipeline` runs the
whole five-stage algorithm. It emits the factoid plus its
fingerprint.

**Federation and privacy.** `PairingHandshake` lets two estates derive
a shared fingerprint basis. It needs no extra network round trip.
`TierContributionFingerprint` packs an estate's shareable summary into
a fixed sixty-four-byte wire format. `TierAscendingQuery` and
`DPORReduction` add differential-privacy noise and k-anonymity. Both
protections keep any single estate's contribution hidden from
aggregate answers.

## How the Pieces Fit

Figure 1 shows the library's topology. It shows the major parts and
how data moves between them.

![Figure 1. Topology of SubstrateML](topology.svg)

*Figure 1. Topology of SubstrateML. Rows, audit events, and sensor
samples enter on the left. The shaping layer feeds the four algorithm
families. Dashed regions mark the substrate packages below and the
telemetry seam. All outputs return to the calling kits, never to
storage.*

SubstrateML has no single facade. Each algorithm family is its own
entry point. The consuming kits call the piece each one needs. These
kits are LocusKit, CognitionKit, GeniusLocusKit, NeuronKit, and the
dreaming daemon. A kit is a larger package that composes libraries
into a subsystem. Kits depend on libs. Libs never depend back on
kits.

The package depends downward only. It depends on `SubstrateTypes` for
shared types. It depends on `SubstrateKernel` for dispatched bit
kernels. It depends on `IntellectusLib`, the zero-dependency telemetry
leaf. Monitoring is off by default. When it is off, every telemetry
emit costs one atomic boolean load and nothing more.

## What Ships in the Package

The package ships the thirty-eight Swift sources. It ships a matching
test suite. It also ships the Rust port in `rust/`, crate
`substrate-ml`, with one module per Swift file plus shared conformance
tests. There are no bundled data artifacts. Every input comes from the
caller.

Determinism comes from pinned constants instead: seeds, thresholds,
half-lives, and bucket tables. These live in the sources. Conformance
vectors lock them in place. The same input therefore produces the
same result on every platform, every time.
