---
doc: DETAILS
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

# SubstrateML Details

This document walks through each source file in the package. Read
`OVERVIEW.md` first for the big picture. The files appear in
working-group order. First come the telemetry names. Then come the
fingerprint and distance math. Then the ingestion shapers. Then the
learning and decay machinery. Then the graph analytics. Then the
pattern miners. Then the distillation stages. Last come the
federation and privacy layer.

## VizGraphSignals.swift

This file gives the canonical metric names for the package's
telemetry layer. It defines five string constants. The five
graph-analytic algorithms use them as signal names, one per call,
when each reports completion.

Telemetry is optional. It feeds the Topology visualization, a live
picture of the memory graph. Each algorithm emits exactly one metric
per invocation through `IntellectusLib`. It emits only when
monitoring is enabled. When monitoring is off, the default, the emit
costs one atomic boolean load and a branch. No clock read happens. No
allocation happens. Keeping the names in one file prevents a typo. A
typo could otherwise produce an orphaned metric that no dashboard
ever finds.

The five constants are `communityAssignment`, `centralityScore`,
`nmfFactor`, `anomalyFlag`, and `edgeDecayedWeight`.
`communityAssignment` is `"community.assignment"`. Its value is the
community count. `centralityScore` is `"centrality.score"`. Its
value is always 1.0, a completion marker. `nmfFactor` is
`"nmf.factor"`. Its value is the final reconstruction error.
`anomalyFlag` is `"anomaly.flag"`. Its value is the absolute z-score.
`edgeDecayedWeight` is `"edge.decayed_weight"`. Its value is the
applied decay factor. The file also records the caller contract.
Callers supply the estate tag and the timestamp, because SubstrateML
never reads a clock.

## FloatSimHash.swift

This file gives `FloatSimHash`. It projects a dense float
embedding vector into the substrate's canonical fingerprint type,
`Fingerprint256`. A typical vector comes from an external model.
Examples include a 384-dimension MiniLM vector or a 768-dimension
BERT vector.

The substrate compares memories by Hamming distance. Hamming
distance counts the differing bit positions between two
equal-length codes. External embeddings are floats, not bits. They
cannot join that comparison directly. `FloatSimHash` bridges the gap
with signed random-hyperplane projection, the classic SimHash
construction. The method generates 256 pseudo-random direction
vectors, each plus one or minus one, matching the input dimension.
It takes the sign of each hyperplane's dot product with the input.
It packs the 256 sign bits into four 64-bit blocks. Vectors that
point the same way land on the same side of most hyperplanes. Cosine
similarity in float space is thus approximately preserved as
Hamming closeness in bit space.

`project(vector:seed:)` is the entry point. The hyperplanes come
from a SplitMix64 generator seeded by the caller. They are drawn in
a pinned order: 256 planes outer, coordinates inner. They follow a
pinned sign convention, where a low bit of one means plus one. The
same seed always yields the same planes. This matters because stored
fingerprints must stay comparable to freshly computed ones. Different
embedding providers use different seeds. Each provider's fingerprints
thus live in their own namespace. The dot-product work itself is
dispatched to a `SubstrateKernel` backend, selected once per process.
A future SIMD kernel can drop in without touching this file.
`planes(seed:dim:)` materializes the hyperplane set directly. Some
callers, such as the kernel-equivalence conformance test, need it in
that raw form.

## CompositeDistance.swift

This file gives `CompositeDistance.distance`, the substrate's
primary retrieval metric. It returns one number between zero and
one. That number says how far apart two memory rows sit.

The score blends two normalized parts:
`alphaLattice * latticeDistance + alphaFingerprint * (hamming /
256)`. The lattice part shows topical distance in the
classification hierarchy. The fingerprint part is Hamming distance
divided by the fixed 256-bit width. Both weights default to 0.5.
Those defaults are only a starting point. The weights are learned
per user over time, through Bradley-Terry feedback (see
`BradleyTerry.swift`).

One edge case is engineered on purpose. Two fingerprints are only
comparable when they come from the same hyperplane family. That
family match is the `compatibleSeedScope` argument. It always holds
inside one estate. Across estates it requires a completed pairing
handshake. When the scopes are not compatible, the function keeps
only the lattice term. It does not rescale that term. The resulting
smaller distance is the documented, intended behavior. Inputs are
checked with preconditions rather than clamped. An out-of-range value
thus fails loudly at the source.

## LatticeDistance.swift

This file gives the two conceptual distance axes that feed the
composite metric. One axis is tree distance in the UDC
classification hierarchy. The other is graph distance in the
Wikidata concept graph.

`LatticeAnchorStr` pairs a UDC code string with an optional Wikidata
Q-ID, where zero means none. `UDCTreeDistance.distance(_:_:)`
compares two code strings by their longest common prefix. The more
leading characters two codes share, the closer their subjects sit in
the hierarchy. The raw formula can reach two, so the result is
clamped to the zero-to-one range the composite metric needs.
`WikidataGraphDistance` runs a breadth-first search over a
caller-injected `WikidataAdjacencyProvider`. That provider supplies
the "is-a" and "part-of" edges. The search is bounded at depth four.
It normalizes the path length with `1 - exp(-length / 3)`.
Unreachable concepts score the maximum, 1.0. The null Q-ID, which is
zero, also scores 1.0. The graph is directed as given. Symmetry holds
only if the provider supplies reverse edges.

`LatticeDistance.distance(_:_:provider:alphaUDC:alphaQID:)` blends
the two axes. Each weight defaults to 0.5. A legacy overload also
accepts the older integer-hashed anchor type. Hashing destroys prefix
structure. That overload can only answer zero for a match or one
for a difference. It survives purely for compatibility with earlier
call sites.

## PartialStateRecall.swift

This file gives a recall primitive for rows that match in one way
and differ in another. A row fingerprint has four 64-bit blocks.
Different blocks encode different aspects of a memory.
`PartialStateRecall` scores a candidate row against an anchor. It
checks one chosen block subset for a match. It checks another subset
for a difference. A real example pairs the same topic with a
different behavioral state.

`score(rowFingerprint:anchor:matchBlocks:differBlocks:)` finds a
restricted Hamming distance over each block subset. The match score
is one minus the normalized distance on the match blocks. The differ
score is the normalized distance on the differ blocks. The final
score is their product. A candidate must satisfy both constraints at
once. A row that matches the anchor everywhere scores zero, since it
fails to differ. A row unrelated on the match blocks also scores
zero. Block identifiers must come from the set `{0, 1, 2, 3}`. A
precondition enforces this rule. An out-of-range block would
silently corrupt the denominator and produce a subtly wrong score
instead of a crash. `topK(anchor:rows:...)` scores each row in a
linear scan. It returns the best k rows. This is the reference form
of the `recall_partial_match` query. `hammingBlocks(_:_:blocks:)`
exposes the block-restricted Hamming count on its own.

## ShingleSimilarity.swift

This file gives the substrate's one character-shingle Jaccard
similarity. It shows how alike two strings are. The measure is
the overlap of their three-character substring sets.

Two consuming kits each needed this exact function for recall
de-duplication. Neither kit may depend on the other. The shared math
thus lives here, in a layer below both, following the "one
implementation per substrate atomic" rule. `shingles(_:)` lowercases
the input. It slides a three-character window across the string. It
collects each substring into a set. Strings shorter than the window
collapse to one whole-string shingle. The empty string yields the
empty set. `similarity(_:_:)` returns the intersection size divided
by the union size. The case of both strings empty is defined as
0.0, a plain rule rather than an undefined case. No stemming
happens. No tokenization happens. No locale-sensitive transform
happens either. This restraint keeps Swift and Rust byte-identical
on the content the recall path actually compares. The window size,
three, is pinned. It is long enough to discriminate near-duplicates.
It is short enough to survive paraphrase.

## MomentSummary.swift

This file gives the moment summary. It is one 256-bit fingerprint
that stands for everything active during a time window. The system
builds it by OR-reducing the fingerprints of the matching rows.

OR-reduction sets a bit in the output whenever any input has that
bit set. This choice gives the summary useful algebra for free. The
result is order-independent. It is idempotent. It is monotonic: more
rows can only add bits, never clear them. An empty match yields the
all-zero fingerprint. Recall can then find similar past moments by
Hamming search over cached window summaries.

What counts as "active during" a window is left open on purpose. The
cookbook defines three legitimate meanings. Both `summarize`
overloads thus take the predicate as a caller-supplied closure.
One overload works on full `Row` values. The other works on
`RowLite`, a 40-byte fingerprint-plus-timestamp pair. The `RowLite`
path runs four to five times faster on large windows, purely
because its loop touches less memory. Both paths are verified
byte-identical against a conformance vector, CRC `0x6762440b`.
`capturedDuring(_:_:)` is the common convenience predicate.
`orReduce(_:)` exposes the raw reduction for callers that already
hold a fingerprint list.

## TemporalCompression.swift

This file gives hierarchical time roll-up. Hour summaries roll
into days. Days roll into weeks. Weeks roll onward through months and
quarters into years. Each level is a `TemporalWindow`: a summary
fingerprint, a row count, and start and end timestamps.

The summary operator is OR-reduction, so rolling up is associative
and commutative. The result does not depend on the order windows
arrive in. `compress(rows:startHLC:endHLC:level:)` builds a base
window from raw row fingerprints. `rollup(windows:to:)` merges finer
windows into one coarser window. It sums row counts with
overflow-safe addition. It takes the outermost time bounds.
`cascadeRollup(hourWindows:upTo:)` automates the whole ladder. It
buckets the current level's windows by which coarser slot they fall
into. That bucketing is integer division of epoch seconds by the
coarser level's approximate duration. It then rolls each bucket, one
level at a time.

The `WindowLevel` durations are approximate, on purpose. A month
counts as thirty days. A year counts as 365. These numbers serve only
as bucket divisors, never as calendar math. Storage and scheduling
belong to the caller, the dreaming daemon. This file is pure
computation. It never reads a clock.

## FeatureExtractors.swift

This file gives the ingestion boundary for ambient signals. Five
encoder types turn raw OS sensor samples into `AmbientSampleRow`
values. Each value is a fingerprinted, classification-anchored row
ready for storage and recall.

Each extractor covers one stream. `HealthKitExtractor` covers
biometrics. `CoreLocationExtractor` covers position.
`EventKitExtractor` covers calendar events. `ScreenTimeExtractor`
covers app usage. `SystemTelemetryExtractor` covers system load. The
OS-specific capture code lives in the host apps. Only the pure
encoding lives here. Each `extract` call finds four 64-bit
subhashes from the sample's salient fields. It feeds them, with a
hyperplane family, into the substrate's SimHash. That step produces
the 256-bit fingerprint. Numeric fields are quantized to integers
before hashing. Coordinates, for example, are multiplied by one
million first. This step means floating-point representation can
never change a fingerprint. Attendee lists are sorted before hashing.
The same meeting thus fingerprints identically, regardless of
listing order. Each stream carries a fixed UDC lattice code as its
coarse topical anchor. Health uses `"613.71"`. Location uses
`"914"`.

Only HealthKit rows carry a `payload`. It is a small, key-sorted,
deterministic binary encoding for human-readable recall detail. The
other streams emit no payload, by design. The fingerprint already
carries all recall-relevant signal. Storing less raw data also means
less to protect. The FNV-1a hash constants serve as the file's
general 64-bit mixers.

## AuditLogFold.swift

This file gives `AuditLogFold`. It reconstructs what a memory row
looked like at any point in time, by replaying its audit log.

The audit log is a grow-only set, called a G-Set in CRDT terms. Its
events are only ever added, never edited. Current state is the fold,
the sequential replay, of a row's events sorted by their HLC
timestamps. State as of time T is that same fold, truncated at T.
The fold depends only on the set of events, never on their arrival
order. Two devices that have exchanged the same events thus
converge to the same state. That property is what makes sync
correct without coordination.

`projectCurrentState(rowId:nounType:events:)` folds a full history.
`projectStateAt(...)` folds up to a cutoff.
`projectAll(events:asOf:...)` groups a whole log by row and folds
each one. The result type, `ProjectedRowState`, carries the row's
bitmaps, the lattice anchor, the tombstone flag, and the last-event
time. One rule is sticky, on purpose. Once the tombstone marker,
state raw value 33, appears, the row stays tombstoned. It stays
tombstoned even if a later, illegal event tries to revive it. The
fingerprint is intentionally absent from the projection. Callers
recompute it from the bitmaps and anchor when needed.

## RowAttributeView.swift

This file gives the bridge from the audit log to the pattern
miners. That bridge is `RowAttributeView`: one row's field writes,
flattened into sorted field-and-value byte pairs. This is the exact
shape Apriori and Formal Concept Analysis consume.

`from(auditEntries:)` runs a four-step pipeline. First, it builds a
vocabulary of all distinct field paths in the batch. That vocabulary
is sorted alphabetically and capped at 64 entries, because the
miners' item type stores the field index in six bits. Second, it
groups entries by row. It keeps only the latest write per field, by
HLC order. That rule is the same last-write-wins rule the audit fold
uses, so the two subsystems never disagree. Third, it expands each
surviving value. A bitmap value becomes one attribute per set bit. An
integer contributes its low byte. A null contributes nothing. Rows
left with no attributes are dropped. Attributes are sorted within a
view. Views are sorted by row identity. The output is thus
deterministic.

One subtlety is documented in the open. The vocabulary is ephemeral
to a single call. Two separate calls can assign different field
indexes. Callers who need a stable vocabulary must merge their entry
lists before calling. The input types `RowAuditEntry` and
`RowAuditValue` are defined here, in a lower layer than the kit that
owns the real audit types. That placement keeps the dependency arrow
pointing downward.

## MatrixDecay.swift

This file gives the substrate's forgetting mechanism. It applies
exponential half-life decay to each matrix in the matrix tier.
Those matrices are the accumulated statistics tables: field
presence, correlation, co-activation, and action outcomes.

The rule is constitutional. Decay may only shrink values. New
evidence enters through separate write operations. It never enters
through this file. `DecayingMatrix` is a plain row-major matrix. It
carries its own half-life and its last-decay timestamp.
`apply(to:nowSeconds:estate:ts:)` finds the elapsed time. It
derives one shared factor, `exp(-elapsed * ln2 / halfLife)`. It
multiplies each cell by that factor. A non-positive elapsed time is
a documented no-op, since decay never runs backward. The function
still emits its telemetry metric with factor 1.0 in that case. A
watcher can then use the emitted factor to tell two states apart.
Either the daemon ran and found nothing to decay. Or the daemon
never ran at all. Using one scalar factor per pass, rather than
per-cell clocks, is what makes decay commute with addition. The file
states
that property formally.

`decayFactor(elapsedSeconds:halfLifeSeconds:)` is the pure
projection helper. `decayAndAdd(...)` is the decay-then-add step the
online write path uses. `DecayHalfLives` records the per-matrix
schedule from the cookbook. Field presence gets 90 days. Correlation
gets 180. Co-activation gets 60. Temporal causality gets 30. Action
outcomes get 365. Calibration gets 730. Two details protect
determinism and safety. `ln2` is a local constant, chosen to match
Rust's value bit-for-bit. The elapsed-seconds cast for the telemetry
tag saturates instead of trapping on absurdly large values. Three
`applyExponentialDecay` overloads for the typed matrices are
deliberate no-op adapters at this reference level. The production
per-matrix routine lives in the consuming kit.

## ActionOutcomeMatrix.swift

This file gives the action-outcome matrix. It is the substrate's
reinforcement record of which actions succeed. Each cell is keyed by
a six-bit action kind and a six-bit outcome category. Each cell
accumulates a success count, a total count, and the last-update HLC.

`ActionOutcomeKey` enforces the six-bit ranges with preconditions.
It packs both bytes into one `UInt16` ordering key. `observe(...)`
increments a cell with overflow-safe arithmetic.
`successRate(...)` and `observationCount(...)` read cells back. The
most interesting method is `topActions(forOutcome:k:minObservations:)`.
It ranks actions by the Wilson lower bound, rather than by the raw
success rate. The Wilson bound is the bottom of a 95 percent
confidence interval around the rate. A cell with two observations
and a perfect record should score below a cell with two hundred
observations and a 95 percent record. The Wilson bound encodes
exactly that caution. The
method returns the rate, the bound, and the count together. Callers
can then see the value the ranking actually used. Ties break by
total count first, then by ascending action. Output stays
deterministic. Decay for this matrix runs on a 365-day half-life.
The dreaming daemon applies it lazily through `MatrixDecay`, not
here.

## BradleyTerry.swift

This file gives an online Bradley-Terry estimator. It learns a
per-row strength score from pairwise preference feedback. That
strength then reorders future recall candidates.

Each time recall surfaces a candidate set, the user or agent picks
one. That pick becomes an observation: the winner beat the losers.
The Bradley-Terry model says the probability that row i
beats row j equals `w_i / (w_i + w_j)`. The implementation works in
log-space, where `theta = log w`. The win probability then becomes a
sigmoid of the theta difference. Gradient updates never need to
guard positivity. `observe(_:)` performs one stochastic gradient step
per observation. It applies L2 regularization, pulling theta toward
zero. This pull keeps early sparse data from overfitting. For
multi-loser observations, the winner's theta updates sequentially
against each loser. Each update recomputes against the running
value. This is a pinned order, and the Rust port mirrors it
exactly.

`observeBatch(_:)` applies a day's worth of observations in order.
`strength(of:)` and `probability(_:beats:)` read the model. There is
no randomness anywhere. The same observation sequence always
produces bit-identical theta. A serialized theta dictionary
warm-restarts the estimator exactly. The defaults are a learning
rate of 0.05 and an L2 of 0.001. These values are tuned for the
ten-to-one-hundred observations per day a personal estate produces.
Strength is a
ranking signal only. It never changes what is stored. It never
filters what exists.

## LLMCalibrationCurve.swift

This file gives a twenty-bucket calibration histogram. It tracks
how well a model's stated confidence matches reality.

The cognition tier sometimes resolves a probabilistic claim. The
model might say it is 80 percent sure, and the claim later turns out
true or false. `observe(claimedConfidence:actualOutcome:)` clamps the
confidence into the range zero to just under one. It maps that value
to one of twenty buckets, each 0.05 wide. It bumps that bucket's
predicted and hit counters. The counters use wrapping arithmetic on
purpose. This is a long-lived accumulator, so overflow should wrap
rather than crash.

`actualRate(in:)` reads one bucket's observed hit rate.
`expectedCalibrationError()` shows the weighted average gap
between each bucket's observed rate and its midpoint, the perfectly
calibrated diagonal. `brierScore()` does the same with squared gaps.
`decay(factor:)` scales each counter by a fraction, so stale
calibration evidence fades smoothly. The dreaming daemon drives this
decay on a 730-day half-life schedule. The result lets recall
annotate a model's claimed confidence. It shows how much that
confidence has historically been worth.

## Sampling.swift

This file gives three deterministic distribution samplers:
Normal, Gamma, and Beta. They sit under Thompson sampling. Thompson
sampling is a decision strategy. It picks among options by drawing
from a belief distribution over each option's payoff. The dreaming
daemon uses it to choose maintenance strategies. The policy lives in
the consuming kit. The sampling math is centralized here, so each
consumer shares one implementation.

All three samplers take the caller's SplitMix64 generator by
`inout` reference. The random stream thus advances visibly and
reproducibly. `sampleNormal(rng:)` uses the Box-Muller transform. It
keeps only the cosine branch, on purpose, consuming exactly two
uniform draws per call. Caching the sine branch would desynchronize
the stream between ports. `sampleGamma(shape:rng:)` implements
Marsaglia-Tsang rejection sampling for a shape of at least one, with
the standard squeeze constant 0.0331. It reduces smaller shapes
through the Ahrens-Dieter identity. That reduction's extra uniform
draw happens before the recursive call. This order is pinned for
cross-port stream identity. `sampleBeta(alpha:beta:rng:)` derives
Beta from the ratio of two Gamma draws, with alpha drawn first.
Given the same seed and inputs, both legs consume the same words in
the same order. Both return the same values, gated by a shared
conformance vector.

## CommunityDetection.swift

This file gives Louvain community detection over the estate
graph. It labels each node with a community, so related memories
cluster together. Auto-rooming, keystone recall, and the daily
dreaming refresh all consume the resulting partition.

Louvain works in two phases. Phase one repeatedly tries moving each
node into a neighboring community. It keeps whichever move most
improves modularity, a score that rewards putting well-connected
nodes together relative to chance. Phase two condenses each
community into a supernode. It then repeats phase one on the smaller
graph. `detect(...)` runs phase one only. `detectFull(...)` runs the
full multi-level algorithm and is the normal entry point.

Two design choices deserve explanation. The first is determinism.
Candidate communities are evaluated in ascending label order, so
score ties always resolve to the lowest label. This avoids depending
on hash-map iteration order. A `canonicalize(_:)` step then renumbers
labels by first appearance, so Swift and Rust emit the same
partitions. The second choice is the resolution parameter. Plain
Louvain can get pair-locked on graphs that have strongly bonded pairs
plus weak star edges. It becomes unable to escape a local optimum.
The Reichardt-Bornholdt gamma scales only the degree-penalty term. A
small gamma, 0.05, forces pairs to merge into hub communities. The
default is 1.0. It reproduces classical Louvain exactly, since IEEE
arithmetic guarantees `1.0 * x == x`. The refactor thus cannot
perturb legacy results. The implementation is intentionally the
compact O(N·E)-per-pass reference, rather than the fastest possible
variant. One `community.assignment` telemetry signal fires per call,
when monitoring is on. A degenerate all-zero-weight graph returns
identity labels and emits nothing.

## EigenvalueCentrality.swift

This file gives eigenvalue centrality. It is a per-row authority
score, computed by power iteration over the estate graph. The score
is cached as each row's keystone score for `recall_keystone`.

Power iteration repeatedly multiplies a score vector by the graph's
adjacency, then renormalizes it. The vector converges to the
principal eigenvector. A node scores highly there when high-scoring
nodes connect to it. Direction matters, and it is a locked design
decision. `compute(...)` multiplies by the transpose. It
accumulates, at node j, the influence of nodes that point to node j.
That is authority: many rows reference this one. It is a different
meaning from a hub, where this row references many others.
Authority is the right meaning for cognitive keystones. A
conformance vector pins the directed behavior, so it cannot drift
back. For undirected centrality, callers symmetrize the adjacency
first.

Each iteration adds a Perron shift, `xNext += 1.0 * x`. This shift
moves each eigenvalue without changing the eigenvectors. It breaks
the plus-minus oscillation that bipartite graphs, such as stars,
exhibit under raw power iteration. Iteration stops at convergence,
when the L2 difference falls below 1e-6, or at a 100-iteration
safety cap. A zero-norm vector falls back to the uniform
distribution: an edgeless graph makes each row equally non-central.
All three return paths emit a single `centrality.score` telemetry
signal, tagged with the node count and the iterations used.

## RandomWalks.swift

This file gives random-walk-with-restart over the memory graph.
It is the engine behind exploratory recall. That recall wanders
outward from a starting memory and reports where it keeps landing.

`walk(adjacency:start:length:restartProb:seed:)` runs one walk over
a densely indexed weighted adjacency list. Each step does one of two
things. It restarts at the start node, with probability 0.15 by
default. Or it moves to a neighbor, chosen by roulette-wheel weighted
sampling. A dead end, a node with no out-edges, counts as an implicit
restart rather than an
error. The walk is seeded explicitly. Four things pin it: the graph,
the start, the length, and the seed. The same four always reproduce
the same walk on both legs. The file carries its
own public `SplitMix64`. Its `uniform01(_:)` derives a double from
the top 53 bits, exactly as Rust does. Malformed graphs fail hard
preconditions immediately. An out-of-range neighbor fails this way.
A negative or non-finite weight fails this way too. A bad adjacency
cannot represent probabilities, and silence would corrupt results
downstream.

`walkWithRestart(seed:steps:restartProbability:rngSeed:adjacency:)`
is the row-identifier variant. It walks a dictionary adjacency with
uniform neighbor choice. It returns visit counts per row, the shape
the exploratory recall recipe consumes. `sampleWeighted(_:rng:)`
exposes the weighted sampler. It includes pinned fallbacks: a
uniform choice when the total weight is zero, and the last neighbor
on a cumulative rounding shortfall.

## NMFAlternatingLeastSquares.swift

This file gives the canonical non-negative matrix factorization
engine, known as NMF. NMF approximates a non-negative matrix V as
the product of two smaller non-negative matrices, `V ≈ W × H`. Rows
of H are latent themes. Rows of W say how much each original row
loads on each theme. The substrate runs NMF over its field-presence
and co-occurrence matrices. This surfaces themes for recall and for
the Topology theme overlay. The dreaming daemon reruns it monthly.

`factorize(V:rank:maxIterations:tolerance:seed:estate:ts:)` uses the
Lee-Seung multiplicative update rules. These rules keep each entry
non-negative automatically. Each iteration first rescales H by the
ratio of `WᵀV` to `WᵀWH`. It then rescales W by the ratio of `VHᵀ`
to `WHHᵀ`. An epsilon of 1e-9 guards the denominators against
division by zero. W and H start as uniform random values from a
SplitMix64 generator, seeded by the caller with a default of
`0xDEADBEEFCAFEBABE`. Initialization is thus bit-identical
everywhere. Iteration stops when the root-mean-square reconstruction
error stops changing by more than the tolerance, or at a fixed cap.
Entry preconditions enforce a rectangular, finite, non-negative V.
The Lee-Seung theorem is undefined otherwise.

The hot loops run on flat row-major arrays, through unsafe buffer
pointers. This is a documented performance necessity.
Bounds-checked nested-array subscripts block auto-vectorization.
They once left Swift roughly one hundred times behind the Rust port
on large reindex drains. The loop nest and reduction order were
preserved exactly through that rewrite. The NMF conformance vector,
CRC `0x300bf633`, thus remains byte-identical.
`reconstructionError(V:W:H:)` exposes the RMS error on nested
arrays. The nested-array matrix helpers remain for external callers.
On completion, the engine emits the `nmf.factor` telemetry signal,
carrying the final error.

## NMFDoubleFrobeniusSquared.swift

This file gives a parked, non-production NMF variant. It is a
double-precision engine with a raw Frobenius-squared convergence
test. One consumer used it before migrating to the canonical engine
above.

It exists for one reason: honesty in benchmarking. The migration
decision needs a future benchmark. That benchmark must compare the
old f64 Frobenius-squared approach against the canonical f32 RMS
approach. It must compare iteration count, time, memory, and recall
quality. Deleting the old algorithm would make that comparison
impossible. The file is thus preserved verbatim, behind a
production gate. No consumer may wire to this type until the
benchmark passes. Even SIMD acceleration of the f64 path stays
separately gated.

The algorithm is the same Lee-Seung scheme, in scalar f64, with
three deliberate differences. Convergence checks the raw,
unnormalized Frobenius-squared error, rather than RMS.
Initialization floors each starting cell at 1e-3, because a cell at
exact zero can never recover under multiplicative updates. The
default seed is its own historical value, `0xC0FFEE_BABE_BEEF`,
distinct from the package's canonical seeds. `factorize(...)`
returns an `NMFDoubleFrobeniusSquaredFactorization`, whose
`loadings(forRow:)` yields a row's latent factors. The file is
parked, but it stays conformance-gated. Swift and Rust must stay
bit-identical on its named vector, so the eventual benchmark
shows the algorithm, not implementation drift.

## AnomalyDetection.swift

This file gives statistical anomaly scoring for telemetry
streams and matrix cells. It offers the classic z-score, a robust
variant, and a threshold check that turns a score into a flag.

A z-score says how many standard deviations a value sits from the
mean of a baseline window. `zScore(value:mean:stddev:)` is the pure
formula. `rollingZScore(window:current:...)` finds the mean and
deviation from the window first. The robust variant,
`modifiedZScore(value:median:mad:)`, replaces the mean with the
median. It replaces the deviation with the median absolute
deviation. It scales by the standard constant 0.6745, so the same
threshold applies to both variants. Outliers already inside the
window cannot drag the baseline the way they drag a mean.
`isAnomalous(zScore:threshold:)` compares the absolute score against
the default threshold, 3.0. That threshold gives roughly a
one-in-370 false-positive rate on normal data.

`rollingModifiedZScore(...)` sorts a mutable copy of the window in
place, rather than allocating a sorted copy. At large windows this
measured five times faster. The behavior is pinned by a CRC marker,
`0x6c6fda4d`. It must also match Rust's sort semantics for all
non-NaN input. Both rolling functions emit the
`anomaly.flag` telemetry signal, with the absolute score as the
value. The timestamp is always caller-supplied.

## JacobiSVD.swift

This file gives a deterministic truncated singular value
decomposition, or SVD. It is the linear algebra that turns a
term-document matrix into low-dimensional semantic embeddings, for
latent semantic analysis in CorpusKit.

SVD factors a matrix into rotation, scaling, and rotation:
`A = U S Vᵀ`. The implementation is one-sided Jacobi. Each sweep
walks a tournament schedule of rounds, not the old fixed
lexicographic order. Within a round every column pair is disjoint,
so the round's rotations touch no shared column. The implementation
applies two-by-two planar rotations to each pair and accumulates the
rotations into V. After the sweeps, the column norms become the
singular values. The normalized columns become U. A sign convention
forces the largest-magnitude entry of each left vector positive.
This removes the inherent plus-minus ambiguity.

Each convergence-affecting choice is pinned, for cross-port
bit-identity. That is why this exists instead of a call to
Accelerate or LAPACK. The sweep count is fixed, at a default of 30,
and is not convergence-tested. The tournament schedule is a pure
integer function of n. The Rust port twins it exactly in
`JacobiSvd::tournament_rounds`. A shared schedule hash pins both
ports to the identical round order. Cross-port bit-identity holds
exactly as it did under the old lexicographic order. Because a
round's pairs are disjoint, its rotations commute. They may run on
any number of threads with bit-identical output. Output never
depends on thread count. A dense factorization over many columns
once pinned one core for minutes. The tournament order now runs many
rotations at once. All arithmetic is scalar Float32, with no SIMD
and no fused multiply-add. The rotation formula in
`jacobiCS(alpha:beta:gamma:)` keeps a pinned expression tree that
must not be refactored. `decompose(A:rank:sweeps:)` requires at
least as many rows as columns. It returns an `SVDResult`: U,
non-increasing singular values, Vt, and rank. The sweep loops use
unsafe buffer pointers, for the same documented vectorization reason
as the NMF engine. Each index is derived from loop bounds, never
from data.

## FFT.swift

This file gives the discrete Fourier transform, and the rhythm
analysis built on it. The substrate uses this to detect periodicity.
It reads one fingerprint bit across the most recent N time buckets.
The dominant frequency of that zero-or-one series reveals the
rhythm of whatever that bit encodes. Circadian patterns show up in
heart-rate bits. Weekly cycles show up in calendar bits.

`Complex` is a plain double-precision complex number, with the
arithmetic the transform needs. `FFT.forward(real:)` is the
Cooley-Tukey radix-2 algorithm: a bit-reversal permutation, followed
by log2(N) butterfly stages. The input length must be a power of
two. This is a constitutional constraint, and callers zero-pad
shorter windows. The production Apple-silicon path routes to the
Accelerate framework. It must still produce bit-identical output to
this scalar reference, on the conformance vectors. Conformance means
output equality, and never speed parity. `magnitudeSpectrum(real:)`
maps the spectrum to magnitudes.

`RhythmAnalysis.analyze(series:bucketDurationSeconds:)` runs the
transform. It finds the strongest bin among the unique positive
frequencies, bin one through N divided by two. It converts that bin
to a dominant period in seconds. It also reports total spectral
energy, excluding the DC bin. A constant series has no dominant
period, so it reports `nil`. The cookbook treats "no rhythm" as an
honest answer, and a deliberate one. The fingerprint-based overload
extracts the bit series from a list of fingerprints first. The block
and bit position are precondition-checked.

## InformationTheory.swift

This file gives the information-theoretic primitives: entropy,
mutual information, KL divergence, cross-entropy, Jensen-Shannon
divergence, and normalized mutual information. These primitives
quantify uncertainty in bitmap distributions. They also detect drift
between a recent window and a long-term baseline.

All quantities use log base two, so results come out in bits.
Entropy shows how unpredictable a distribution is. Mutual
information shows how much knowing one variable tells you about
another. KL divergence shows how badly a model distribution q
explains an observed distribution p. Terms with probability exactly
zero are skipped everywhere. This implements the standard rule that
zero times log of zero equals zero, without ever producing NaN.

Two honesty notes are documented in the code. When p has mass where
q has none, the true KL divergence is infinite.
`klDivergence(_:_:)` skips such terms, so its result is a lower
bound. Callers must know this. `normalizedMutualInformation(joint:)`
returns a zero sentinel for a ragged joint matrix. This avoids
silently computing a plausible-looking wrong value from corrupted
marginals. Length mismatches between p and q are hard preconditions.
Distribution validity itself, meaning non-negative values that sum
to one, is the caller's responsibility by contract.

## AssociationRuleMining.swift

This file gives pairwise association-rule mining, over
`MatrixO`, the co-occurrence matrix the substrate updates on each
capture. It surfaces rules of the form: when field A has value x,
field B tends to have value y. Each rule carries the standard
support, confidence, lift, conviction, and leverage metrics.

The design leans on how `MatrixO` is built. Its diagonal cells
already hold single-item counts. Its off-diagonal cells hold pair
counts. `mineAssociationRules(matrix:activeRowCount:thresholds:)`
thus needs only two passes over the stored entries. One pass
collects single supports from the diagonal. The other walks the
pairs, finds the five metrics, and keeps rules that clear the
`MiningThresholds` gates. Self-rules, where A implies A, are skipped
as noise. Conviction is defined as positive infinity at confidence
one. The total row count, N, is injected by the caller. The matrix
itself does not know it. The entries iterate in canonical
packed-key order, so the output needs no final sort. Both legs rely
on that order for byte-identical results.

`Item` is the shared atom: a field-and-value byte pair, with a
packed 16-bit ordering key. The Apriori engine reuses it. The
internal `AssociationRuleEngine` holds the actual math, so
conformance tests can drive it directly. Multi-item antecedents fall
outside this file's scope. They belong to `AprioriMining.swift`.

## AprioriMining.swift

This file gives the multi-antecedent generalization: the classic
Apriori algorithm, over `RowAttributeView` rows. It produces rules
such as: if A and B are set, then C is set too. At the minimum
itemset size, it reproduces the pairwise engine's answers. Beyond
that size, it finds combinations the pairwise engine misses.

`AprioriMining.mine(rows:thresholds:)` follows the textbook
outline. Each row becomes an item set. Frequent single items seed
the search. Each level joins frequent item sets of size k minus one
that share a lexicographic prefix. It counts candidate support with
subset tests. It prunes anything below minimum support, up to the
size cap `maxK`, which defaults to three. Rules are extracted from
each frequent itemset of size two or more. Each member is tried as
the consequent. Rules are then filtered by the support, confidence,
and lift gates in `AprioriThresholds`. Requiring a lift of at least
1.0 suppresses anti-correlated coincidences.

Determinism gets specific care here, because Swift dictionaries
iterate in undefined order. Itemsets are processed in canonical
lexicographic order. The final sort uses a four-key total order:
lift, confidence, evidence count, and then a lexicographic
tie-break. Output stays the same across runs and across languages.
One defensive detail matters. The type's custom `Decodable`
initializer routes through the public initializer. A serialized
`maxK` of zero or one thus cannot bypass the floor of two and
corrupt the level loop. `mineAprioriRules(rows:thresholds:)` is a
free-function wrapper, matching the pairwise engine's call style.
`AprioriRule` carries the five metrics, plus a raw `evidenceCount`
for interpretability.

## FormalConceptAnalysis.swift

This file gives bounded Formal Concept Analysis, or FCA. It
finds the exact groupings hidden in a table of rows and attributes.
A formal concept is a maximal pair: a set of rows that share exactly
a set of attributes. The concepts of an estate are data-driven
"kinds of memory." They emerge from what was actually recorded. This
differs from the soft themes NMF finds, and from the graph clusters
Louvain finds.

`FormalAttribute` is one typed attribute: a namespace, a key, and a
value. It is ordered lexicographically, and that order is the
determinism backbone of the whole file. `FormalContext` stores the
row-attribute relation in both directions, as bitsets. The
derivation operators reduce to word-wise bit operations because of
that choice. `extent(of:)` finds rows carrying all given attributes.
`intent(of:)` finds attributes common to all given rows.
`closure(of:)` combines both. Contexts build from raw per-row
attribute lists, or from `RowAttributeView` batches.

Full concept-lattice enumeration is exponential, so
`BoundedConceptMiner` never attempts it. It seeds from attributes
meeting `minSupport`. It can optionally seed from frequent attribute
pairs, in `.multi` seed mode, capped by `maxSeeds`. It takes exactly
one closure per seed. It deduplicates by intent. It truncates at
`maxConcepts`. This gives polynomial cost by construction, with a
fully specified output sort. `ConceptCoverDeltas.covering(concepts:)`
finds the direct-neighbor edges of the concept order, a
structural lens showing which attributes were added. The file
explains that a cover edge shows structure only. It is not a
logical implication. `StabilityEstimator.estimate(...)`
approximates how robust a concept is to noise. It re-closes random
half-subsets of the extent, under a per-concept SplitMix64 stream.
That stream mixes the caller seed with a hash of the concept, using
a default seed of `0xCAFEBABEDEADBEEF`. This trades exactness for a
bounded budget. A zero budget, the default, leaves stability as
`nil`.

## ConceptImplications.swift

This file gives the Duquenne-Guigues canonical basis. That basis
is the minimal set of implications that hold without exception
across a formal context. Each implication reads: each row with
attributes X also has attribute Y. Where FCA finds groupings, this
file finds the rules those groupings obey. That feeds consolidation
and synthesis.

`conceptImplications(over:context:maxImplications:maxPremiseSize:)`
enumerates candidate premises, in strictly increasing size order. It
tests each one for pseudo-intent status. The closure must add
something new. Each smaller pseudo-intent's conclusion must
already sit inside it. Processing in size order makes that
incremental test equivalent to full minimality checking. Two
independent caps keep this NP-hard enumeration bounded.
`maxImplications` is a hard stop, and it sets `isTruncated`, with a
look-ahead so the flag stays honest when the cap lands mid-run.
`maxPremiseSize` silently skips larger premises, without affecting
the soundness of what gets emitted. Each order is pinned: the
attribute universe, the combination enumeration, and the final sort
by premise size then lexicographic content. Both legs thus emit
bit-identical bases. `Implication` carries the premise and
conclusion sets. An empty context short-circuits to an empty,
untruncated result.

## TemporalCausalityFold.swift

This file gives the fold that feeds the temporal-causality
matrix. It scans a time-sorted stream of audit entries. It counts
pairs of the form: field X changed, then field Y changed some
minutes later. Each pair is bucketed by lag.

`fold(entries:windowMinutes:startWatermark:)` processes each entry
newer than the caller's watermark. It evicts buffered entries older
than the window, 256 minutes by default. It pairs the new entry
against each remaining buffered entry. Each minute delta maps to
one of eight log-spaced lag buckets, from one up to 128 minutes,
through `lagBucket(forMinutes:)`. The fold emits one delta increment
per source coordinate, target coordinate, and bucket. The result
carries the deltas plus a new watermark, so the hourly batch run
resumes where it stopped.

Two contracts matter here. Entries must arrive pre-sorted by HLC.
The fold does not re-sort them, and would silently compute wrong
deltas otherwise. The rolling buffer is capped at 512 entries,
`maxWindowOccupancy`. A bulk import can drop tens of thousands of
events into one window, and unbounded pairing is quadratic. Keeping
only the most recent in-window entries preserves the near-lag
signal the causality matrix actually cares about. Output deltas are
aggregated by key, but returned in first-insertion order, so
repeated runs stay bit-identical. The lag bucket table and the
occupancy cap are pinned constants, mirrored exactly in the
downstream matrix type and the Rust port. The input shapes are
`TemporalFieldCoord` and `TemporalAuditEntry`. They are defined
locally, so the kit that owns the real audit types stays upstream.

## TypedDecayWeighting.swift

This file gives type-aware staleness weighting for distillation
features. Different kinds of extracted facts age at different
speeds. A version number, a numerical fact, goes stale within days.
A domain concept, an entity fact, stays true across major changes.

`DistillationFeatureType` names four categories, with wire tags ENT,
REL, TMP, and NUM. It pins each category's decay rate, lambda: 0.1,
0.2, 0.5, and 0.8. This scheme is an analogy with frequency bands in
diffusion noise schedules. `weight(featureType:ageInUnits:)`
finds the exponential weight `exp(-lambda * age)`. It clamps
negative ages to zero, so future-dated observations count as
present. One age unit is one day by convention. That equals `86,400`
seconds, though callers may choose a different unit.

`weightedDocFrequency(featureType:presenceTimestamps:allMemoryTimestamps:referenceDate:timeUnit:)`
finds the decayed document frequency. That frequency is a ratio of
two weights. The first is the summed weight of memories where a
feature is present. The second is the summed weight of all memories
in the cluster. Recent evidence
thus counts more than old evidence, at a rate set by the
feature's type. This is the frequency the distillation pipeline
uses, whenever timestamps are available.

## DeltaFeatureExtractor.swift

This file classifies a feature's value sequence across a cluster,
into four trajectory states. A value can stabilize. A value can
trend. A value can flip-flop. A value can turn to noise. Its job is
rescue. A feature may fail the recurrence threshold on raw counts.
It may still carry a real pattern over time. This classifier finds
that pattern.

`DeltaType` names five verdicts: STATIC, CONVERGENT, MONOTONE,
OSCILLATING, and DIVERGENT.
`analyzeCategorical(sequence:decayLambda:)` handles string-valued
features. All-identical values score STATIC. An A-B-A-B pattern over
the last four observations scores OSCILLATING. Four observations are
required, to avoid false positives on short sequences. Otherwise, the
trailing run of the final value decides the verdict. Its length
counts as a fraction of the sequence. That fraction scores CONVERGENT
at or above a default threshold of 0.5, and DIVERGENT below it.
`analyzeNumerical(sequence:decayLambda:)` handles numbers through
consecutive differences. All-zero differences score STATIC. All one
sign scores MONOTONE, with a slope and a fixed confidence equal to
the type's decay rate, default 0.8. Strict sign alternation scores
OSCILLATING. Anything else scores DIVERGENT.

Both paths return a full `DeltaAnalysis`: verdict, terminal value,
convergence score, optional slope, and confidence. The caller
thus always has a well-formed result. The file is a pure
namespace. It holds no state. It reads no clock. It uses no
randomness at all. The oscillation check looks only at the recent
tail, on purpose. The question this file answers is the current
trajectory, and never the full history.

## DistillationScorer.swift

This file gives the scoring stages of distillation. It decides
whether a cluster of memories is coherent enough to compress. If so,
it decides which features form the core factoid.

A pipeline of pure functions runs in order. A structural threshold,
`structuralThreshold(M:) = 2/M`, splits features into two groups.
Structural features recur in at least two of the M units. Episodic
features appear only once. `applyStructuralThreshold(features:M:)`
performs the split. The file's long comment explains a design
choice: it uses intra-cluster recurrence, rather than a majority
vote. A majority vote was found to discard real distillable cores,
when a single document was split into sentences.
`computeSNR(features:M:)` then gates readiness. The summed frequency
of structural features must be at least twice the episodic sum. That
ratio is the signal-to-noise ratio, floored at a tiny epsilon
against division by zero. `computeStructuralScores(features:)`
assigns each feature a score. The score rewards frequency and
consistency, through binary entropy.

`buildPMIGraph(thresholdFeatures:incidenceMatrix:M:)` builds the
coherence graph. Pointwise mutual information, in log base two,
shows whether two features co-occur more than chance would
predict. Positive-PMI pairs become edges. Connected components are
found by depth-first search, quadratic but acceptable at the
documented cap of about 150 features. Components are sorted by
descending weighted frequency. `selectDominantComponent(graph:)`
returns the top component, the factoid core.
`computeConfidence(selected:allThreshold:)` scores it as the mean
frequency, times a fragmentation penalty: the fraction of threshold
features the core kept. Everything here is deterministic and
log2-based. The Rust port's comments stress that log2, and never
natural log, must match bit-for-bit.

## DistillationPipeline.swift

This file gives the complete five-stage distillation algorithm.
A cluster of raw memory strings goes in. One condensed factoid comes
out. The output type is `DistillationOutput`. It carries the
formatted drawer content, the confidence, the SNR, and an optional
delta verdict. It also carries a success flag and a structural
fingerprint for Hamming-space lookup.

`run(input:extractFeatures:intraItem:)` executes the stages. Stage
one extracts features, per memory and per feature type, through an
injected `FeatureExtractor` closure. Production injects a real
entity tagger. Tests use the bundled capitalization-heuristic
`defaultExtractor`. This stage builds the vocabulary and the
incidence matrix. Document frequencies come from
`TypedDecayWeighting` when timestamps exist. Stage two applies the
SNR gate and the recurrence threshold. Stage two-and-a-half is the
delta pre-pass. Features that failed recurrence are grouped by
predicate key. Their value sequences run through
`DeltaFeatureExtractor`. Converging or trending features get rescued
with their terminal value. Stage three builds the PMI graph and
keeps the dominant component, with two deliberate exceptions. In
intra-item mode, where one document splits into sentences, all
passing features are kept. PMI pruning would wrongly split a single
coherent document otherwise. A ubiquity fix-up also re-adds any
feature present in nearly each member. A ubiquitous feature has zero
PMI with everything, and would otherwise vanish despite being the
semantic spine. Stages four and five score the result, format the
header, and OR-reduce the per-feature hashes into the fingerprint.

`featureHash(_:)` maps a feature string to a `Fingerprint256`,
through SplitMix64, under the pinned seed `featureSimHashSeed`. That
seed, `0x44495354494C4C41`, spells "DISTILLA" in ASCII. Changing the
seed invalidates each stored distillation fingerprint, so it stays
conformance-locked. `queryFingerprint(query:extractFeatures:)`
builds the matching probe fingerprint at query time, with no model
inference. `DistilledHeader.parse` reads the stored header back out
of stored content. Confidence below 0.4 counts as failure.
Confidence between 0.4 and 0.7 marks the output uncertain. One
formatting subtlety is pinned. The `src=` field counts source
drawer IDs, not cluster members. The two differ in intra-item mode.
The `run` body stays one large function, on purpose. Splitting it
would thread the incidence matrix and vocabulary through each
helper signature.

## PairingHandshake.swift

This file gives the pairing handshake. These are the protocol
steps that let two estates establish a shared fingerprint basis, so
their memories become comparable for federation.

Fingerprints are only comparable under the same hyperplane family,
and each estate normally has its own. The handshake fixes that
without a negotiation round trip. The two estates first exchange a
32-byte nonce, out of band.
`generateSharedFamily(nonce:estateA:estateB:density:)` then lets
each side independently derives the same shared family. This
works because the SplitMix64 seed is a pure function of the nonce
and the two estate UUIDs. `sharedFamilyKey(case:peerEstate:)`
finds the canonical manifest key both sides store the family
under. `buildPairEvent(...)` and `buildUnpairEvent(...)` produce the
audit payloads that record the relationship. Dissolving a pairing
keeps the shared family, since historical "as of" queries must
still work. It does stop further sync, however.

The subtlest line in the file is
`PairingNonce.seedWith(estateA:estateB:)`. It orders the two UUIDs by
raw byte comparison, never by their hexadecimal string form. ASCII
comparison of hex strings can rank two UUIDs in the opposite order
from their raw bytes. The Rust leg always compares raw bytes. A
string-ordered Swift would derive a different seed, and the two
devices would build incompatible families. The nonce length, exactly
32 bytes, is precondition-enforced. The FNV-1a constants handle the
seed and family-hash mixing.

## TierContributionFingerprint.swift

This file gives the tier contribution. It is the fixed 64-byte
payload an estate sends up the federation hierarchy. It summarizes
an estate's shareable rows as one OR-reduced fingerprint.

`FederationCase` names the three tiers: household, fleet, and
industry. `build(estateUUID:case:shareableFingerprints:hlc:)`
OR-reduces the shareable fingerprints. This work is routed through
the platform kernel. Federation work thus uses the best
available SIMD backend. The math still stays commutative,
associative, and idempotent. The function wraps the result with the
estate UUID, the case, the row count, and the HLC, into a
`TierContribution`.

`encode(_:)` and `decode(_:)` implement the canonical wire format.
Sixteen UUID bytes come first. Then come the case and row count, as
big-endian 32-bit integers. Then come the four fingerprint blocks,
as big-endian 64-bit integers. Then comes the packed HLC. The total
runs exactly 64 bytes, big-endian throughout. A comment records a
real caught bug. An earlier draft emitted the fingerprint in its
little-endian storage form. That draft diverged from Rust by 32
bytes, until the cross-language conformance gate flagged it. This
layer neither signs nor checksums its output. Authenticity is a
federation-egress concern, applied at the share point. Privacy
noise is applied at the aggregator, and never by the contributor.

## TierAscendingQuery.swift

This file gives the local-scope steps of the tier-ascending
query protocol. It defines how a recall query travels to peer
estates. It also defines how the answers come back combined. Neither
step exposes any single estate's data.

`TierAscendingQuery` models the query itself. It names the
originating estate, the named recall primitive, the target tier, the
privacy budget, and the query HLC. `computeLocal(query:dispatch:)`
runs the local primitive through an injected closure. This keeps the
package independent of the cognition layer. On the peer side,
`applyDPToContribution(_:budget:rngSeed:)` adds Laplace noise to
each score in the peer's result. The noise scale is one over
epsilon, drawn from a seeded SplitMix64. The function attaches a
fixed 95 percent confidence interval, plus or minus 1.96 times the
scale. This lets the requester know how blurry the contribution is.
`combine(local:peers:)` merges the exact local result with the
noised peer results. Scores sum per row. The order runs by
descending score, with row identity as the deterministic tie-break.
The widest confidence interval seen always wins. This choice is
conservative on purpose, since the least certain contribution bounds
the certainty of the union.

`PrivacyLedger` tracks per-peer epsilon and delta consumption
against a daily budget. `canConsume(peer:query:)` gates a query.
`consume(...)` records it. `dailyReset()` starts a new day.
Networking, signing, and scheduling are explicitly the caller's job.
One parity nuance is worth knowing. The Rust leg zeroes the
per-lane distance breakdown in a noised contribution, to close a
side channel. The Swift reference forwards that breakdown unchanged.

## DPORReduction.swift

This file gives the differentially private OR-reduction used at
the aggregation point of federated queries. Many estates'
fingerprints go in. One aggregate fingerprint comes out, with
mathematical privacy protection layered in.

`reduce(fingerprints:params:rngSeed:)` works per bit position. For
each of the 256 positions, it counts how many contributors set that
bit. It adds Laplace noise, at a scale of one over epsilon. The
noise comes from a SplitMix64 generator, seeded by the caller. This
keeps the noise reproducible for testing. It sets the output bit
only if the noised count reaches the k-anonymity threshold. Two
mechanisms stack here. Differential privacy means no observer can
tell whether any one estate contributed. The k-anonymity floor
defaults to three. It means no bit lights up from too few
contributors, even without adversarial analysis. Empty input returns
the zero fingerprint.

`DPParameters` pins the defaults from the cookbook: epsilon 1.0,
delta 1e-9, and k 3, with validity preconditions. One documented
asymmetry stands out. Delta is validated and stored for API
completeness, but the Laplace mechanism itself does not consume it.
The `laplaceNoise(scale:rng:)` helper derives a uniform value from
the top 53 bits of the raw draw. This is the same convention the
package's other samplers use. The helper then inverts the Laplace
distribution's cumulative curve.

## Rust Port and Conformance

The `rust/` directory contains the second leg of the library: the
crate `substrate-ml`. It has one module per Swift file, thirty-eight
modules from `anomaly.rs` through `viz_graph_signals.rs`. These
modules are declared in `rust/src/lib.rs`, with the same layering
notes and the same production gate on the parked NMF variant. The
crate depends on `substrate-types`, `substrate-kernel`, and
`intellectus-lib`. This mirrors the Swift package manifest exactly.

Most modules open with an explicit "mirror of" header, naming their
Swift source. The determinism-bearing files repeat the pinned
constants, seeds, draw orders, and tie-break rules, line for line.
Shared gates prove the agreement property.
`rust/tests/distillation_conformance.rs` locks four things: the
delta extractor, the typed decay weights, `featureHash`, and the
full pipeline. All four are checked against fixed vectors mirrored
in Swift's `DistillationConformanceTests`. The rule there is simple:
fix the algorithm, never the vector.
`rust/tests/float_simhash_kernel_equivalence.rs` proves the kernel
projection reproduces the oracle bit for bit, across seeds and
dimensions. `rust/tests/viz_graph_signals_tests.rs` exercises the
telemetry emit of all five graph algorithms, under a process-wide
lock. Several algorithms carry additional shared JSON vectors under
the engineering test harness. These vectors cover sampling, temporal
causality fold, NMF, and community detection. When you change either
leg, change both, and run both suites. The fixtures are the contract.
