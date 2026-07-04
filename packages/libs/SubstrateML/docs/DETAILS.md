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

# SubstrateML Details

This document walks through every source file in the package. Read
`OVERVIEW.md` first for the big picture. Files appear in working-group
order: the telemetry names, the fingerprint and distance math, the
ingestion shapers, the learning and decay machinery, the graph analytics,
the pattern miners, the distillation stages, and finally the federation
and privacy layer.

## VizGraphSignals.swift

This file provides the canonical metric names for the package's telemetry
layer: five string constants used as signal names when the five
graph-analytic algorithms report completion.

Telemetry is optional observation data for the Topology visualization — a
live picture of the memory graph. Each algorithm emits exactly one metric
per invocation through `IntellectusLib`, and only when monitoring is
enabled. When monitoring is off (the default), the emit costs a single
atomic boolean load and a branch: no clock read, no allocation. Keeping
the names in one authoritative file prevents a typo from producing an
orphaned metric that no dashboard ever finds.

The five constants are `communityAssignment` ("community.assignment",
value is the community count), `centralityScore` ("centrality.score",
value 1.0 as a completion marker), `nmfFactor` ("nmf.factor", value is the
final reconstruction error), `anomalyFlag` ("anomaly.flag", value is the
absolute z-score), and `edgeDecayedWeight` ("edge.decayed_weight", value
is the applied decay factor). The file also records the caller contract:
callers supply the estate tag and the timestamp, because SubstrateML never
reads a clock.

## FloatSimHash.swift

This file provides `FloatSimHash`, which projects a dense float embedding
vector — for example a 384-dimension MiniLM or 768-dimension BERT vector
from an external model — into the substrate's canonical 256-bit
fingerprint type, `Fingerprint256`.

The substrate compares memories by Hamming distance, the number of
differing bit positions between two equal-length codes. External
embeddings are floats, not bits, so they cannot join that comparison
directly. `FloatSimHash` bridges the gap with signed random-hyperplane
projection, the classic SimHash construction: generate 256 pseudo-random
"+1 or -1" direction vectors (hyperplanes) matching the input dimension,
take the sign of each hyperplane's dot product with the input, and pack
the 256 sign bits into four 64-bit blocks. Vectors that point the same way
land on the same side of most hyperplanes, so cosine similarity in float
space is approximately preserved as Hamming closeness in bit space.

`project(vector:seed:)` is the entry point. The hyperplanes come from a
SplitMix64 generator seeded by the caller, drawn in a pinned order (256
planes outer, coordinates inner) with a pinned sign convention (low bit 1
means +1). The same seed always yields the same planes, which matters
because stored fingerprints must stay comparable to freshly computed ones.
Different embedding providers use different seeds, so each provider's
fingerprints live in their own namespace. The dot-product work itself is
dispatched to a `SubstrateKernel` backend, selected once per process, so a
future SIMD kernel can drop in without touching this file.
`planes(seed:dim:)` materializes the hyperplane set for callers, such as
the kernel-equivalence conformance test, that need it directly.

## CompositeDistance.swift

This file provides `CompositeDistance.distance`, the substrate's primary
retrieval metric: one number in the range zero to one that says how far
apart two memory rows are.

The score is a weighted blend of two normalized parts:
`alphaLattice * latticeDistance + alphaFingerprint * (hamming / 256)`.
The lattice part measures topical distance in the classification
hierarchy. The fingerprint part is Hamming distance divided by the fixed
256-bit width. Both weights default to 0.5, but the defaults are only the
starting point; the weights are learned per user over time through
Bradley-Terry feedback (see `BradleyTerry.swift`).

One edge case is engineered, not accidental. Two fingerprints are only
comparable when they were computed under the same hyperplane family — the
`compatibleSeedScope` argument. That always holds inside one estate, but
across estates it requires a completed pairing handshake. When the scopes
are not compatible, the function keeps only the lattice term and does not
rescale it. The resulting smaller distance is the documented intended
behavior. Inputs are checked with preconditions rather than clamped, so an
out-of-range value fails loudly at the source.

## LatticeDistance.swift

This file provides the two conceptual distance axes that feed the
composite metric: tree distance in the UDC classification hierarchy and
graph distance in the Wikidata concept graph.

`LatticeAnchorStr` pairs a UDC code string with an optional Wikidata Q-ID
(zero means none). `UDCTreeDistance.distance(_:_:)` compares two code
strings by their longest common prefix: the more leading characters two
codes share, the closer their subjects sit in the hierarchy. The raw
formula can reach two, so the result is clamped to the zero-to-one range
the composite metric requires. `WikidataGraphDistance` runs a
breadth-first search over a caller-injected `WikidataAdjacencyProvider`
(the "is-a" and "part-of" edges), bounded at depth 4, and normalizes the
path length with `1 - exp(-length / 3)`. Unreachable concepts, and the
null Q-ID zero, score the maximum 1.0. The graph is directed as given, so
symmetry holds only if the provider supplies reverse edges.

`LatticeDistance.distance(_:_:provider:alphaUDC:alphaQID:)` blends the two
axes with weights that default to 0.5 each. A legacy overload also accepts
the older integer-hashed anchor type; hashing destroys prefix structure,
so that overload can only answer zero (identical) or one (different) and
is kept purely for compatibility with earlier call sites.

## PartialStateRecall.swift

This file provides the "same in one way, different in another" recall
primitive. A row fingerprint has four 64-bit blocks, and different blocks
encode different aspects of a memory. `PartialStateRecall` scores a
candidate row by how well it matches an anchor on one chosen block subset
while differing from it on another — for example, same topic but different
behavioral state.

`score(rowFingerprint:anchor:matchBlocks:differBlocks:)` computes a
restricted Hamming distance over each block subset. The match score is one
minus the normalized distance on the match blocks; the differ score is the
normalized distance on the differ blocks. The final score is their
product, so a candidate must satisfy both constraints at once: a row
identical to the anchor everywhere scores zero (it fails to differ), and a
row unrelated on the match blocks also scores zero. Block identifiers must
come from the set {0, 1, 2, 3}; a precondition enforces this because an
out-of-range block would silently corrupt the denominator and produce a
subtly wrong score instead of a crash. `topK(anchor:rows:...)` scores every
row in a linear scan and returns the best k — the reference form of the
`recall_partial_match` query. `hammingBlocks(_:_:blocks:)` exposes the
block-restricted Hamming count itself.

## ShingleSimilarity.swift

This file provides the substrate's one character-shingle Jaccard
similarity: how alike two strings are, as the overlap of their
three-character substring sets.

Two consuming kits needed exactly this function for recall de-duplication,
and neither may depend on the other. Following the "one implementation per
substrate atomic" rule, the shared math lives here, below both.
`shingles(_:)` lowercases the input and slides a three-character window
across it, collecting each substring into a set; strings shorter than the
window collapse to a single whole-string shingle, and the empty string
yields the empty set. `similarity(_:_:)` returns intersection size divided
by union size, with "both empty" defined as 0.0 rather than undefined. No
stemming, tokenization, or locale-sensitive transform is applied, which is
what keeps Swift and Rust byte-identical on the content the recall path
actually compares. The window size 3 is pinned: long enough to
discriminate near-duplicates, short enough to survive paraphrase.

## MomentSummary.swift

This file provides the moment summary: one 256-bit fingerprint that
stands for everything active during a time window, built by OR-reducing
the fingerprints of the matching rows.

OR-reduction sets a bit in the output when any input has it set. The
choice of OR gives the summary useful algebra for free: it is
order-independent, idempotent, and monotonic — more rows can only add
bits, never clear them. An empty match yields the all-zero fingerprint.
Recall can then find similar past moments by Hamming search over cached
window summaries.

What counts as "active during" the window is deliberately not hardcoded.
The cookbook defines three legitimate meanings, so both `summarize`
overloads take the predicate as a caller-supplied closure. One overload
works on full `Row` values; the other works on `RowLite`, a 40-byte
fingerprint-plus-timestamp pair, and runs four to five times faster on
large windows purely because the loop touches less memory. Both paths are
verified byte-identical against a conformance vector (CRC `0x6762440b`).
`capturedDuring(_:_:)` is the common convenience predicate, and
`orReduce(_:)` exposes the raw reduction for callers that already hold a
fingerprint list.

## TemporalCompression.swift

This file provides hierarchical time roll-up: hour summaries roll into
days, days into weeks, and onward through months and quarters to years.
Each level is a `TemporalWindow` — a summary fingerprint, a row count, and
start and end timestamps.

Because the summary operator is OR-reduction, rolling up is associative
and commutative: the result does not depend on the order windows arrive.
`compress(rows:startHLC:endHLC:level:)` builds a base window from raw row
fingerprints. `rollup(windows:to:)` merges finer windows into one coarser
window, summing row counts with overflow-safe addition and taking the
outermost time bounds. `cascadeRollup(hourWindows:upTo:)` automates the
whole ladder: it buckets the current level's windows by which coarser slot
they fall into (integer division of epoch seconds by the coarser level's
approximate duration) and rolls each bucket, one level at a time.

The `WindowLevel` durations are deliberately approximate — a month is
thirty days, a year 365 — because they serve only as bucket divisors, not
calendar math. Storage and scheduling belong to the caller (the dreaming
daemon); this file is pure computation and never reads a clock.

## FeatureExtractors.swift

This file provides the ingestion boundary for ambient signals: five
encoder types that turn raw OS sensor samples into `AmbientSampleRow`
values — fingerprinted, classification-anchored rows ready for storage
and recall.

Each extractor covers one stream: `HealthKitExtractor` (biometrics),
`CoreLocationExtractor` (position), `EventKitExtractor` (calendar),
`ScreenTimeExtractor` (app usage), and `SystemTelemetryExtractor` (system
load). The OS-specific capture code lives in the host apps; only the pure
encoding lives here. Each `extract` call computes four 64-bit subhashes
from the sample's salient fields, then feeds them with a hyperplane family
into the substrate's SimHash to produce the 256-bit fingerprint. Numeric
fields are quantized to integers before hashing (for example, coordinates
times one million) so floating-point representation can never change a
fingerprint. Attendee lists are sorted before hashing so the same meeting
fingerprints identically regardless of listing order. Each stream carries
a fixed UDC lattice code (for example "613.71" for health, "914" for
location) as its coarse topical anchor.

Only HealthKit rows carry a `payload` — a small, key-sorted, deterministic
binary encoding for human-readable recall detail. The other streams emit
no payload by design: the fingerprint already carries all recall-relevant
signal, and less stored raw data means less to protect. The FNV-1a hash
constants serve as the file's general 64-bit mixers.

## AuditLogFold.swift

This file provides `AuditLogFold`, which reconstructs what a memory row
looked like at any point in time by replaying its audit log.

The audit log is a grow-only set (a G-Set, a conflict-free replicated data
type): events are only ever added, never edited. Current state is defined
as the fold — the sequential replay — of a row's events sorted by their
hybrid logical clock (HLC) timestamps, and "state as of time T" is the
same fold truncated at T. Because the fold depends only on the set of
events, not their arrival order, two devices that have exchanged the same
events converge to identical state. That property is what makes sync
correct without coordination.

`projectCurrentState(rowId:nounType:events:)` folds a full history;
`projectStateAt(...)` folds up to a cutoff; `projectAll(events:asOf:...)`
groups a whole log by row and folds each. The result type,
`ProjectedRowState`, carries the row's bitmaps, lattice anchor, tombstone
flag, and last-event time. One rule is deliberately sticky: once the
tombstone marker (state raw value 33) appears, the row stays tombstoned
even if a later, illegal event tries to revive it. The fingerprint is
intentionally absent from the projection; callers recompute it from the
bitmaps and anchor when needed.

## RowAttributeView.swift

This file provides the bridge from the audit log to the pattern miners:
`RowAttributeView`, one row's field writes flattened into sorted
`(field, value)` byte pairs — the exact shape Apriori and Formal Concept
Analysis consume.

`from(auditEntries:)` runs a four-step pipeline. It first builds a
vocabulary of all distinct field paths in the batch, sorted
alphabetically and capped at 64 entries, because the miners' item type
stores the field index in six bits. It then groups entries by row, keeps
only the latest write per field by HLC order (the same last-write-wins
rule the audit fold uses, so the two subsystems never disagree), and
expands each surviving value: a bitmap value becomes one attribute per set
bit, an integer contributes its low byte, and a null contributes nothing.
Rows left with no attributes are dropped. Attributes are sorted within a
view and views are sorted by row identity, so the output is deterministic.

One subtlety is documented rather than hidden: the vocabulary is ephemeral
to a single call. Two separate calls can assign different field indexes,
so callers who need a stable vocabulary must merge their entry lists
before calling. The input types `RowAuditEntry` and `RowAuditValue` are
defined here, in a lower layer than the kit that owns the real audit
types, precisely so the dependency arrow keeps pointing downward.

## MatrixDecay.swift

This file provides the substrate's forgetting mechanism: exponential
half-life decay for every matrix in the matrix tier — the accumulated
statistics tables such as field presence, correlation, co-activation, and
action outcomes.

The rule is constitutional: decay may only shrink values. New evidence
enters through separate write operations, never through this file.
`DecayingMatrix` is a plain row-major matrix that carries its own
half-life and its last-decay timestamp. `apply(to:nowSeconds:estate:ts:)`
computes the elapsed time, derives one shared factor
`exp(-elapsed * ln2 / halfLife)`, and multiplies every cell by it. A
non-positive elapsed time is a documented no-op — decay never runs
backward — but it still emits its telemetry metric with factor 1.0, so a
watcher can tell "daemon ran, nothing to do" from "daemon never ran."
Using one scalar factor per pass (rather than per-cell clocks) is what
makes decay commute with addition, a property the file states formally.

`decayFactor(elapsedSeconds:halfLifeSeconds:)` is the pure projection
helper, and `decayAndAdd(...)` is the decay-then-add step the online write
path uses. `DecayHalfLives` records the per-matrix schedule from the
cookbook: 90 days for field presence, 180 for correlation, 60 for
co-activation, 30 for temporal causality, 365 for action outcomes, and
730 for calibration. Two details protect determinism and safety: `ln2` is
a local constant chosen to match Rust's value bit-for-bit, and the
elapsed-seconds cast for the telemetry tag saturates instead of trapping
on absurdly large values. Three `applyExponentialDecay` overloads for the
typed matrices are deliberate no-op adapters at this reference level; the
production per-matrix routine lives in the consuming kit.

## ActionOutcomeMatrix.swift

This file provides the action-outcome matrix: the substrate's
reinforcement record of which actions succeed. Each cell, keyed by a
six-bit action kind and a six-bit outcome category, accumulates a success
count, a total count, and the last-update HLC.

`ActionOutcomeKey` enforces the six-bit ranges with preconditions and
packs both bytes into one `UInt16` ordering key. `observe(...)` increments
a cell with overflow-safe arithmetic. `successRate(...)` and
`observationCount(...)` read cells back. The interesting method is
`topActions(forOutcome:k:minObservations:)`: it ranks actions not by raw
success rate but by the Wilson lower bound — the bottom of a 95 percent
confidence interval around the rate. A cell with two observations and a
perfect record should not outrank a cell with two hundred observations and
a 95 percent record; the Wilson bound encodes exactly that caution. The
method returns rate, bound, and count together so callers see the value
the ranking actually used. Ties break by total count, then by ascending
action, keeping output deterministic. Decay for this matrix (365-day
half-life) is applied lazily by the dreaming daemon through
`MatrixDecay`, not here.

## BradleyTerry.swift

This file provides an online Bradley-Terry estimator: it learns a
per-row "strength" score from pairwise preference feedback, and that
strength reorders future recall candidates.

Every time recall surfaces a candidate set and the user or agent picks
one, that pick is an observation: the winner beat the losers. The
Bradley-Terry model says the probability that row i beats row j is
`w_i / (w_i + w_j)`. The implementation works in log-space (`theta = log
w`), so the win probability becomes a sigmoid of the theta difference and
gradient updates never have to guard positivity. `observe(_:)` performs
one stochastic gradient step per observation with L2 regularization
pulling theta toward zero, which keeps early sparse data from overfitting.
For multi-loser observations the winner's theta is updated sequentially
against each loser, recomputing against the running value — a pinned
ordering that the Rust port mirrors exactly.

`observeBatch(_:)` applies a day's worth of observations in order.
`strength(of:)` and `probability(_:beats:)` read the model. There is no
randomness anywhere: the same observation sequence always produces
bit-identical theta, and a serialized theta dictionary warm-restarts the
estimator exactly. The defaults (learning rate 0.05, L2 0.001) are tuned
for the ten-to-one-hundred observations per day a personal estate
produces. Strength is a ranking signal only; it never changes what is
stored or filters what exists.

## LLMCalibrationCurve.swift

This file provides a twenty-bucket calibration histogram that tracks how
well an LLM's stated confidence matches reality.

When the cognition tier resolves a probabilistic claim — the model said
"80 percent sure" and the claim turned out true or false —
`observe(claimedConfidence:actualOutcome:)` clamps the confidence into
the range zero to just under one, maps it to one of twenty 0.05-wide
buckets, and bumps that bucket's predicted and hit counters. Counters use
wrapping arithmetic on purpose: this is a long-lived accumulator, and
overflow should wrap rather than crash.

`actualRate(in:)` reads one bucket's observed hit rate.
`expectedCalibrationError()` measures the weighted average gap between
each bucket's observed rate and its midpoint — the "perfectly calibrated"
diagonal. `brierScore()` is the same with squared gaps. `decay(factor:)`
scales every counter by a fraction so stale calibration evidence fades
smoothly; the dreaming daemon drives it on the 730-day half-life schedule.
The result lets recall annotate model-claimed confidence with how much
that model's confidence has historically been worth.

## Sampling.swift

This file provides the three deterministic distribution samplers —
Normal, Gamma, and Beta — that sit under Thompson sampling. Thompson
sampling is a decision strategy that picks among options by drawing from a
belief distribution over each option's payoff; the dreaming daemon uses it
to choose maintenance strategies. The policy lives in the consuming kit;
the sampling math is centralized here so every consumer shares one
implementation.

All three samplers take the caller's SplitMix64 generator by `inout`
reference, so the random stream advances visibly and reproducibly.
`sampleNormal(rng:)` uses the Box-Muller transform and deliberately keeps
only the cosine branch, consuming exactly two uniform draws per call —
caching the sine branch would desynchronize the stream between ports.
`sampleGamma(shape:rng:)` implements Marsaglia-Tsang rejection sampling
for shape at least one, with the standard squeeze constant 0.0331, and
reduces smaller shapes through the Ahrens-Dieter identity; the reduction's
extra uniform draw happens before the recursive call, an order pinned for
cross-port stream identity. `sampleBeta(alpha:beta:rng:)` derives Beta
from the ratio of two Gamma draws, alpha first. Given the same seed and
inputs, both legs consume the same words in the same order and return the
same values, gated by a shared conformance vector.

## CommunityDetection.swift

This file provides Louvain community detection over the estate graph: it
labels every node with a community so related memories cluster together.
Auto-rooming, keystone recall, and the daily dreaming refresh all consume
the partition.

Louvain works in two phases. Phase 1 repeatedly tries moving each node
into a neighboring community and keeps the move that most improves
modularity — a score that rewards putting well-connected nodes together
relative to chance. Phase 2 condenses each community into a supernode and
repeats on the smaller graph. `detect(...)` runs Phase 1 only;
`detectFull(...)` runs the full multi-level algorithm and is the normal
entry point.

Two design choices deserve explanation. First, determinism: candidate
communities are evaluated in ascending label order, so score ties always
resolve to the lowest label instead of depending on hash-map iteration
order; a `canonicalize(_:)` step then renumbers labels by first
appearance, so Swift and Rust emit identical partitions. Second, the
resolution parameter: plain Louvain can get "pair-locked" on graphs with
strongly bonded pairs plus weak star edges, unable to escape a local
optimum. The Reichardt-Bornholdt gamma scales only the degree-penalty
term; small gamma (consumers use 0.05) forces pairs to merge into hub
communities, while the default 1.0 reproduces classical Louvain exactly —
IEEE arithmetic guarantees `1.0 * x == x`, so the refactor cannot perturb
legacy results. The implementation is intentionally the compact
O(N·E)-per-pass reference rather than the fastest possible variant. One
`community.assignment` telemetry signal fires per call when monitoring is
on; a degenerate all-zero-weight graph returns identity labels and emits
nothing.

## EigenvalueCentrality.swift

This file provides eigenvalue centrality: a per-row authority score
computed by power iteration over the estate graph, cached as each row's
keystone score for `recall_keystone`.

Power iteration repeatedly multiplies a score vector by the graph's
adjacency and renormalizes; the vector converges to the principal
eigenvector, where a node scores highly when high-scoring nodes connect to
it. The direction matters and is a locked design decision: `compute(...)`
multiplies by the transpose, accumulating at node j the influence of nodes
that point TO j. That is authority ("many rows reference this one"), not
hub ("this row references many"), and it is the right meaning for
cognitive keystones. A conformance vector pins the directed behavior so it
cannot drift back. For undirected centrality, callers symmetrize the
adjacency first.

Each iteration adds a Perron shift (`xNext += 1.0 * x`), which shifts
every eigenvalue without changing eigenvectors and thereby breaks the
plus-minus oscillation that bipartite graphs such as stars exhibit under
raw power iteration. Iteration stops at convergence (L2 difference below
1e-6) or at the 100-iteration safety cap. A zero-norm vector falls back to
the uniform distribution — an edgeless graph makes every row equally
non-central. All three return paths emit the single `centrality.score`
telemetry signal, tagged with the node count and iterations used.

## RandomWalks.swift

This file provides random-walk-with-restart over the memory graph — the
engine behind exploratory recall, which wanders outward from a starting
memory and reports where it keeps landing.

`walk(adjacency:start:length:restartProb:seed:)` runs one walk over a
densely indexed weighted adjacency list. Each step either restarts at the
start node (probability 0.15 by default) or moves to a neighbor chosen by
roulette-wheel weighted sampling. A dead end — a node with no out-edges —
is an implicit restart rather than an error. The walk is seeded
explicitly: the same graph, start, length, and seed reproduce the same
walk on both legs, because the file carries its own public `SplitMix64`
and a `uniform01(_:)` that derives a double from the top 53 bits exactly
as Rust does. Malformed graphs (out-of-range neighbor, negative or
non-finite weight) fail hard preconditions immediately, because a bad
adjacency cannot represent probabilities and silence would corrupt
results downstream.

`walkWithRestart(seed:steps:restartProbability:rngSeed:adjacency:)` is the
row-identifier variant: it walks a dictionary adjacency with uniform
neighbor choice and returns visit counts per row, the shape the
exploratory recall recipe consumes. `sampleWeighted(_:rng:)` exposes the
weighted sampler, including its pinned fallbacks (uniform choice when
total weight is zero; last neighbor on cumulative rounding shortfall).

## NMFAlternatingLeastSquares.swift

This file provides the canonical non-negative matrix factorization (NMF)
engine. NMF approximates a non-negative matrix V as the product of two
smaller non-negative matrices, V ≈ W × H. Rows of H are latent "themes";
rows of W say how much each original row loads on each theme. The
substrate runs NMF over its field-presence and co-occurrence matrices to
surface themes for recall and for the Topology theme overlay; the
dreaming daemon reruns it monthly.

`factorize(V:rank:maxIterations:tolerance:seed:estate:ts:)` uses the
Lee-Seung multiplicative update rules, which keep every entry non-negative
automatically: each iteration rescales H by the ratio of `WᵀV` to `WᵀWH`,
then W by the ratio of `VHᵀ` to `WHHᵀ`, with epsilon 1e-9 in denominators
against division by zero. W and H start as uniform random values from a
SplitMix64 seeded by the caller (default `0xDEADBEEFCAFEBABE`), so
initialization is bit-identical everywhere. Iteration stops when the
root-mean-square reconstruction error stops changing by more than the
tolerance, or at the cap. Entry preconditions enforce a rectangular,
finite, non-negative V, because the Lee-Seung theorem is undefined
otherwise.

The hot loops run on flat row-major arrays through unsafe buffer pointers.
This is a documented performance necessity: bounds-checked nested-array
subscripts block auto-vectorization and left Swift roughly one hundred
times behind the Rust port on large reindex drains. The loop nest and
reduction order were preserved exactly through that rewrite, so the NMF
conformance vector (CRC `0x300bf633`) remains byte-identical.
`reconstructionError(V:W:H:)` exposes the RMS error on nested arrays, and
the nested-array matrix helpers remain for external callers. On
completion the engine emits the `nmf.factor` telemetry signal carrying
the final error.

## NMFDoubleFrobeniusSquared.swift

This file provides a parked, explicitly non-production NMF variant: the
double-precision engine with a raw Frobenius-squared convergence test
that one consumer used before migrating to the canonical engine above.

It exists for one reason: honesty in benchmarking. The migration decision
requires a future benchmark comparing the old f64/Frobenius-squared
approach against the canonical f32/RMS approach on iteration count, time,
memory, and recall quality. Deleting the old algorithm would make that
comparison impossible, so it is preserved verbatim behind a production
gate: no consumer may wire to this type until the benchmark passes, and
even SIMD acceleration of the f64 path is separately gated.

The algorithm is the same Lee-Seung scheme in scalar f64, with three
deliberate differences: convergence checks the raw (unnormalized)
Frobenius-squared error rather than RMS; initialization floors every
starting cell at 1e-3, because a cell at exact zero can never recover
under multiplicative updates; and the default seed is its own historical
value (`0xC0FFEE_BABE_BEEF`), distinct from the package's canonical seeds.
`factorize(...)` returns an `NMFDoubleFrobeniusSquaredFactorization` whose
`loadings(forRow:)` yields a row's latent factors. Despite being parked,
the file is still conformance-gated: Swift and Rust must stay
bit-identical on its named vector, so the eventual benchmark measures the
algorithm, not implementation drift.

## AnomalyDetection.swift

This file provides statistical anomaly scoring for telemetry streams and
matrix cells: the classic z-score and a robust variant, plus the
threshold check that turns a score into a flag.

A z-score says how many standard deviations a value sits from the mean of
its baseline window. `zScore(value:mean:stddev:)` is the pure formula;
`rollingZScore(window:current:...)` computes mean and deviation from the
window first. The robust variant, `modifiedZScore(value:median:mad:)`,
replaces mean with median and deviation with the median absolute
deviation, then scales by the standard constant 0.6745 so the same
threshold applies to both variants; outliers already inside the window
cannot drag the baseline the way they drag a mean.
`isAnomalous(zScore:threshold:)` compares the absolute score against the
default threshold 3.0, roughly a one-in-370 false-positive rate on normal
data.

`rollingModifiedZScore(...)` sorts a mutable copy of the window in place
rather than allocating a sorted copy; at large windows this measured five
times faster, and the behavior is pinned by a CRC marker (`0x6c6fda4d`)
plus a note matching Rust's sort semantics for all non-NaN input. Both
rolling functions emit the `anomaly.flag` telemetry signal with the
absolute score as the value; the timestamp is always caller-supplied.

## JacobiSVD.swift

This file provides a deterministic truncated singular value
decomposition (SVD) — the linear algebra that turns a term-document
matrix into low-dimensional semantic embeddings for latent semantic
analysis in CorpusKit.

SVD factors a matrix into rotation, scaling, and rotation: `A = U S Vᵀ`.
The implementation is one-sided cyclic Jacobi: it repeatedly applies
two-by-two planar rotations to every pair of columns in a fixed cyclic
order, accumulating the rotations into V; after the sweeps, column norms
become the singular values and normalized columns become U. A sign
convention (largest-magnitude entry of each left vector forced positive)
removes the inherent plus-minus ambiguity.

Every convergence-affecting choice is pinned for cross-port bit-identity,
which is why this exists instead of a call to Accelerate or LAPACK: the
sweep count is fixed (default 30, not convergence-tested), the rotation
order is fixed, all arithmetic is scalar Float32 with no SIMD or fused
multiply-add, and the rotation formula in `jacobiCS(alpha:beta:gamma:)`
keeps a pinned expression tree that must not be refactored.
`decompose(A:rank:sweeps:)` requires at least as many rows as columns and
returns an `SVDResult` (U, non-increasing singular values, Vt, rank). The
sweep loops use unsafe buffer pointers for the same documented
vectorization reason as the NMF engine; every index is derived from loop
bounds, never from data.

## FFT.swift

This file provides the discrete Fourier transform and the rhythm analysis
built on it. The substrate uses it to detect periodicity: read one
fingerprint bit across the most recent N time buckets, and the dominant
frequency of that 0/1 series reveals the rhythm of whatever that bit
encodes — circadian patterns in heart-rate bits, weekly cycles in
calendar bits.

`Complex` is a plain double-precision complex number with the arithmetic
the transform needs. `FFT.forward(real:)` is the Cooley-Tukey radix-2
algorithm: a bit-reversal permutation followed by log2(N) butterfly
stages. Input length must be a power of two — a constitutional constraint;
callers zero-pad shorter windows. The production Apple-silicon path routes
to the Accelerate framework but must produce bit-identical output to this
scalar reference on the conformance vectors; conformance is output
equality, not speed parity. `magnitudeSpectrum(real:)` maps the spectrum
to magnitudes.

`RhythmAnalysis.analyze(series:bucketDurationSeconds:)` runs the
transform, finds the strongest bin among the unique positive frequencies
(bin 1 through N/2), and converts it to a dominant period in seconds,
alongside total spectral energy excluding the DC bin. A constant series
has no dominant period and reports `nil` — the cookbook treats "no rhythm"
as an honest answer, not a default. The fingerprint-based overload
extracts the bit series from a list of fingerprints first, with the block
and bit position precondition-checked.

## InformationTheory.swift

This file provides the information-theoretic primitives — entropy, mutual
information, KL divergence, cross-entropy, Jensen-Shannon divergence, and
normalized mutual information — used to quantify uncertainty in bitmap
distributions and to detect drift between a recent window and a long-term
baseline.

All quantities use log base 2, so results are in bits. Entropy measures
how unpredictable a distribution is. Mutual information measures how much
knowing one variable tells you about another. KL divergence measures how
badly a model distribution q explains an observed distribution p.
Everywhere, terms with probability exactly zero are skipped, implementing
the standard `0 · log 0 = 0` limit without ever producing NaN.

Two honesty notes are documented in the code. When p has mass where q has
none, true KL divergence is infinite; `klDivergence(_:_:)` skips such
terms and its result is therefore a lower bound, which callers must know.
And `normalizedMutualInformation(joint:)` returns the zero sentinel for a
ragged joint matrix rather than silently computing a plausible-looking
wrong value from corrupted marginals. Length mismatches between p and q
are hard preconditions; distribution validity itself (non-negative,
summing to one) is the caller's responsibility by contract.

## AssociationRuleMining.swift

This file provides pairwise association-rule mining over `MatrixO`, the
co-occurrence matrix the substrate updates on every capture: it surfaces
rules of the form "when field A has value x, field B tends to have value
y," with the standard support, confidence, lift, conviction, and leverage
metrics.

The design leans on how `MatrixO` is built. Its diagonal cells already
hold single-item counts, and its off-diagonal cells hold pair counts, so
`mineAssociationRules(matrix:activeRowCount:thresholds:)` needs only two
passes over the stored entries: one to collect single supports from the
diagonal, one to walk the pairs, compute the five metrics, and keep rules
that clear the `MiningThresholds` gates. Self-rules (A implies A) are
skipped as noise, conviction is defined as positive infinity at
confidence one, and the total row count N is injected by the caller
because the matrix itself does not know it. Because the entries iterate
in canonical packed-key order, the output needs no final sort — an
ordering both legs rely on for byte-identical results.

`Item` is the shared atom: a `(field, value)` byte pair with a packed
16-bit ordering key, reused by the Apriori engine. The internal
`AssociationRuleEngine` holds the actual math so conformance tests can
drive it directly; multi-item antecedents are out of scope here and
belong to `AprioriMining.swift`.

## AprioriMining.swift

This file provides the multi-antecedent generalization: the classic
Apriori algorithm over `RowAttributeView` rows, producing rules such as
"{A, B} implies C." At its minimum itemset size it reproduces the pairwise
engine's answers; beyond that it finds combinations the pairwise engine
cannot see.

`AprioriMining.mine(rows:thresholds:)` follows the textbook outline. Each
row becomes an item set. Frequent single items seed the search; each
level joins frequent (k-1)-item sets that share a lexicographic prefix,
counts candidate support with subset tests, and prunes below minimum
support, until the size cap `maxK` (default 3). Rules are then extracted
from every frequent itemset of size two or more, with each member tried
as the consequent, and filtered by the support, confidence, and lift
gates in `AprioriThresholds`. Requiring lift at least 1.0 suppresses
anti-correlated coincidences.

Determinism gets specific care because Swift dictionaries iterate in
undefined order: itemsets are processed in canonical lexicographic order,
and the final sort uses a four-key total order (lift, confidence,
evidence count, then lexicographic tie-break), so output is identical
across runs and across languages. One defensive detail: the type's custom
`Decodable` initializer routes through the public initializer so a
serialized `maxK` of zero or one cannot bypass the floor of two and
corrupt the level loop. `mineAprioriRules(rows:thresholds:)` is a free
function wrapper matching the pairwise engine's call style; `AprioriRule`
carries the five metrics plus a raw `evidenceCount` for interpretability.

## FormalConceptAnalysis.swift

This file provides bounded Formal Concept Analysis (FCA): it finds the
exact groupings hidden in a table of rows and attributes. A formal
concept is a maximal pair — a set of rows that share exactly a set of
attributes — and the concepts of an estate are the data-driven "kinds of
memory" that emerged from what was actually recorded, as opposed to the
soft themes NMF finds or the graph clusters Louvain finds.

`FormalAttribute` is one typed attribute (namespace, key, value), ordered
lexicographically — that ordering is the determinism backbone of the whole
file. `FormalContext` stores the row-attribute relation in both directions
as bitsets, so the derivation operators `extent(of:)` (rows carrying all
given attributes), `intent(of:)` (attributes common to all given rows),
and `closure(of:)` reduce to word-wise bit operations. Contexts build from
raw per-row attribute lists or from `RowAttributeView` batches.

Full concept-lattice enumeration is exponential, so `BoundedConceptMiner`
never attempts it. It seeds from attributes meeting `minSupport` (and
optionally frequent attribute pairs in `.multi` seed mode, capped by
`maxSeeds`), takes exactly one closure per seed, deduplicates by intent,
and truncates at `maxConcepts` — polynomial cost by construction, with a
fully specified output sort. `ConceptCoverDeltas.covering(concepts:)`
computes the direct-neighbor edges of the concept order, the "what
attributes were added" structural lens; the file is explicit that a cover
edge is not a logical implication. `StabilityEstimator.estimate(...)`
approximates how robust a concept is to noise by re-closing random
half-subsets of its extent under a per-concept SplitMix64 stream (caller
seed mixed with a hash of the concept; default seed
`0xCAFEBABEDEADBEEF`), trading exactness for a bounded budget. A zero
budget, the default, leaves stability `nil`.

## ConceptImplications.swift

This file provides the Duquenne-Guigues canonical basis: the minimal set
of implications — "every row with attributes X also has attribute Y" —
that hold without exception across a formal context. Where FCA finds the
groupings, this finds the rules the groupings obey, feeding consolidation
and synthesis.

`conceptImplications(over:context:maxImplications:maxPremiseSize:)`
enumerates candidate premises in strictly increasing size order and tests
each for pseudo-intent status: its closure must add something, and every
smaller pseudo-intent's conclusion must already be inside it. Processing
in size order makes that incremental test equivalent to full minimality
checking. Two independent caps keep an NP-hard enumeration bounded:
`maxImplications` is a hard stop that sets `isTruncated` (with a
look-ahead so the flag is honest when the cap lands mid-run), and
`maxPremiseSize` silently skips larger premises without affecting the
soundness of what is emitted. Every ordering — the attribute universe,
the combination enumeration, and the final sort by premise size then
lexicographic content — is pinned, so both legs emit bit-identical bases.
`Implication` carries the premise and conclusion sets; an empty context
short-circuits to an empty, untruncated result.

## TemporalCausalityFold.swift

This file provides the fold that feeds the temporal-causality matrix: it
scans a time-sorted stream of audit entries and counts "field X changed,
then field Y changed N minutes later" pairs, bucketed by lag.

`fold(entries:windowMinutes:startWatermark:)` processes each entry newer
than the caller's watermark. It evicts buffered entries older than the
window (256 minutes by default), then pairs the new entry against every
remaining buffered entry, mapping each minute delta to one of eight
log-spaced lag buckets — 1, 2, 4, up to 128 minutes, via
`lagBucket(forMinutes:)` — and emitting one delta increment per source
coordinate, target coordinate, and bucket. The result carries the deltas
plus a new watermark, so the hourly batch run resumes where it stopped.

Two contracts matter. Entries must arrive pre-sorted by HLC; the fold
does not re-sort and would silently compute wrong deltas otherwise. And
the rolling buffer is capped at 512 entries (`maxWindowOccupancy`),
because a bulk import can drop tens of thousands of events into one
window and unbounded pairing is quadratic; keeping only the most recent
in-window entries preserves the near-lag signal the causality matrix
actually cares about. Output deltas are aggregated by key but returned in
first-insertion order, so repeated runs are bit-identical. The lag bucket
table and the occupancy cap are pinned constants mirrored exactly in the
downstream matrix type and the Rust port. The input shapes
(`TemporalFieldCoord`, `TemporalAuditEntry`) are defined locally so the
kit that owns the real audit types stays upstream.

## TypedDecayWeighting.swift

This file provides type-aware staleness weighting for distillation
features. Different kinds of extracted facts age at different speeds: a
version number (numerical) goes stale in days, while a domain concept
(entity) stays true across major changes.

`DistillationFeatureType` names the four categories with their wire tags —
ENT, REL, TMP, NUM — and pins each one's decay rate lambda: 0.1, 0.2,
0.5, and 0.8 respectively, by analogy with frequency bands in diffusion
noise schedules. `weight(featureType:ageInUnits:)` computes the
exponential weight `exp(-lambda * age)`, clamping negative ages to zero
so future-dated observations count as present. One age unit is one day by
convention (86,400 seconds), though callers choose the unit.

`weightedDocFrequency(featureType:presenceTimestamps:allMemoryTimestamps:
referenceDate:timeUnit:)` computes the decayed document frequency: the
summed weight of the memories where a feature is present, divided by the
summed weight of all memories in the cluster. Recent evidence therefore
counts more than old evidence, at a rate set by the feature's type. This
is the frequency the distillation pipeline uses whenever timestamps are
available.

## DeltaFeatureExtractor.swift

This file provides trajectory classification for a single feature's value
sequence across a cluster: has the value stabilized, is it trending, is
it flip-flopping, or is it noise? Its job is rescue — a feature that
fails the recurrence threshold on raw counts may still carry a real
pattern over time, and this classifier finds it.

`DeltaType` names the five verdicts: STATIC, CONVERGENT, MONOTONE,
OSCILLATING, DIVERGENT. `analyzeCategorical(sequence:decayLambda:)`
handles string-valued features: all-identical values are STATIC; an
A-B-A-B pattern over the last four observations (four required, to avoid
false positives on short sequences) is OSCILLATING; otherwise the length
of the trailing run of the final value, as a fraction of the sequence,
decides CONVERGENT (at least the threshold, default 0.5) or DIVERGENT.
`analyzeNumerical(sequence:decayLambda:)` handles numbers via consecutive
differences: all zero is STATIC, all one sign is MONOTONE (with slope and
a fixed confidence of the type's decay rate, default 0.8), strict sign
alternation is OSCILLATING, anything else DIVERGENT.

Both paths return a full `DeltaAnalysis` — verdict, terminal value,
convergence score, optional slope, confidence — so the caller always has
a well-formed result. The file is a pure namespace: no state, no clock,
no randomness at all. The oscillation check looks only at the recent
tail on purpose; the question is the current trajectory, not history.

## DistillationScorer.swift

This file provides the scoring stages of distillation: the math that
decides whether a cluster of memories is coherent enough to compress and,
if so, which features form the core factoid.

The pipeline of pure functions runs in order. A structural threshold,
`structuralThreshold(M:)` = 2/M, splits features into structural
(recurring in at least two of the M units) and episodic (one-off);
`applyStructuralThreshold(features:M:)` performs the split. The file's
long comment explains why this is intra-cluster recurrence rather than a
majority vote: majority voting was found to discard real distillable
cores when a single document is split into sentences.
`computeSNR(features:M:)` then gates readiness: the summed frequency of
structural features must be at least twice the episodic sum (the
signal-to-noise ratio, floored at a tiny epsilon against division by
zero). `computeStructuralScores(features:)` assigns each feature a score
rewarding frequency and consistency via binary entropy.

`buildPMIGraph(thresholdFeatures:incidenceMatrix:M:)` builds the
coherence graph: pointwise mutual information (PMI), in log base 2,
measures whether two features co-occur more than chance; positive-PMI
pairs become edges, and connected components are found by depth-first
search (quadratic, acceptable at the documented cap of about 150
features) and sorted by descending weighted frequency.
`selectDominantComponent(graph:)` returns the top component — the factoid
core — and `computeConfidence(selected:allThreshold:)` scores it as mean
frequency times a fragmentation penalty (the fraction of threshold
features the core kept). Everything is deterministic and log2-based; the
Rust port's comments stress that log2, not natural log, must match
bit-for-bit.

## DistillationPipeline.swift

This file provides the complete five-stage distillation algorithm: a
cluster of raw memory strings in, one condensed factoid out. The output
`DistillationOutput` carries the formatted drawer content
(`"[DIST|conf=…|src=…|snr=…|delta=…] prose"`), the confidence, the SNR,
an optional delta verdict, a success flag, and a structural fingerprint
for Hamming-space lookup.

`run(input:extractFeatures:intraItem:)` executes the stages. Stage 1
extracts features per memory per feature type through an injected
`FeatureExtractor` closure — production injects a real entity tagger,
tests use the bundled capitalization-heuristic `defaultExtractor` — and
builds the vocabulary and incidence matrix. Document frequencies come
from `TypedDecayWeighting` when timestamps exist. Stage 2 applies the SNR
gate and the recurrence threshold. Stage 2.5 is the delta pre-pass: 
features that failed recurrence are grouped by predicate key and their
value sequences run through `DeltaFeatureExtractor`; converging or
trending features are rescued with their terminal value. Stage 3 builds
the PMI graph and keeps the dominant component, with two deliberate
exceptions: in intra-item mode (one document split into sentences) all
passing features are kept, because PMI pruning would wrongly split a
single coherent document; and a ubiquity fix-up re-adds any feature
present in nearly every member, because a ubiquitous feature has zero PMI
with everything and would otherwise vanish despite being the semantic
spine. Stages 4 and 5 score, format the header, and OR-reduce per-feature
hashes into the fingerprint.

`featureHash(_:)` maps a feature string to a `Fingerprint256` via
SplitMix64 under the pinned seed `featureSimHashSeed`
(`0x44495354494C4C41`, ASCII "DISTILLA"); changing that seed invalidates
every stored distillation fingerprint, so it is conformance-locked.
`queryFingerprint(query:extractFeatures:)` builds the matching probe
fingerprint at query time with no model inference. `DistilledHeader.parse`
reads the `[DIST|...]` header back out of stored content. Confidence
below 0.4 is failure; between 0.4 and 0.7 the output is marked uncertain.
One formatting subtlety is pinned: the `src=` field counts source drawer
IDs, not cluster members — the two differ in intra-item mode. The `run`
body is deliberately one large function; splitting it would thread the
incidence matrix and vocabulary through every helper signature.

## PairingHandshake.swift

This file provides the pairing handshake: the protocol steps that let two
estates establish a shared fingerprint basis so their memories become
comparable for federation.

Fingerprints are only comparable under the same hyperplane family, and
each estate normally has its own. The handshake fixes that without a
negotiation round trip: after the two estates exchange a 32-byte nonce
out of band, `generateSharedFamily(nonce:estateA:estateB:density:)` lets
each side independently derive the identical shared family, because the
SplitMix64 seed is a pure function of the nonce and the two estate UUIDs.
`sharedFamilyKey(case:peerEstate:)` computes the canonical manifest key
(`H_shared_<case>_<peer8>`) both sides store it under, and
`buildPairEvent(...)` and `buildUnpairEvent(...)` produce the audit
payloads that record the relationship. Dissolving a pairing keeps the
shared family — historical "as of" queries must still work — but stops
further sync.

The subtlest line in the file is `PairingNonce.seedWith(estateA:estateB:)`:
it orders the two UUIDs by raw byte comparison, never by their hexadecimal
string form. ASCII comparison of hex strings can rank two UUIDs in the
opposite order from their raw bytes, and the Rust leg always compares raw
bytes — a string-ordered Swift would derive a different seed and the two
devices would build incompatible families. The nonce length (exactly 32
bytes) is precondition-enforced, and the FNV-1a constants do the seed and
family-hash mixing.

## TierContributionFingerprint.swift

This file provides the tier contribution: the fixed 64-byte payload an
estate sends up the federation hierarchy, summarizing its shareable rows
as one OR-reduced fingerprint.

`FederationCase` names the three tiers (household, fleet, industry).
`build(estateUUID:case:shareableFingerprints:hlc:)` OR-reduces the
shareable fingerprints — routed through the platform kernel so federation
work uses the best available SIMD backend while preserving the
commutative, associative, idempotent math — and wraps the result with the
estate UUID, the case, the row count, and the HLC into a
`TierContribution`.

`encode(_:)` and `decode(_:)` implement the canonical wire format: 16
UUID bytes, then case and row count as big-endian 32-bit integers, then
the four fingerprint blocks as big-endian 64-bit integers, then the
packed HLC — 64 bytes exactly, big-endian throughout. The comment records
a real caught bug: an earlier draft emitted the fingerprint in its
little-endian storage form and diverged from Rust by 32 bytes until the
cross-language conformance gate flagged it. This layer neither signs nor
checksums; authenticity is a federation-egress concern applied at the
share point, and privacy noise is applied at the aggregator, not by the
contributor.

## TierAscendingQuery.swift

This file provides the local-scope steps of the tier-ascending query
protocol: how a recall query travels to peer estates and how the answers
come back combined, without exposing any single estate's data.

`TierAscendingQuery` models the query itself — the originating estate,
the named recall primitive to run, the target tier, the privacy budget,
and the query HLC. `computeLocal(query:dispatch:)` runs the local
primitive through an injected closure, keeping this package independent
of the cognition layer. On the peer side,
`applyDPToContribution(_:budget:rngSeed:)` adds Laplace noise (scale one
over epsilon, drawn from a seeded SplitMix64) to every score in the
peer's result and attaches a fixed 95 percent confidence interval of plus
or minus 1.96 times the scale, so the requester knows how blurry the
contribution is. `combine(local:peers:)` merges the exact local result
with the noised peer results: scores sum per row, ordering is descending
score with row identity as the deterministic tie-break, and the widest
confidence interval seen wins — a deliberately conservative choice, since
the least certain contribution bounds the certainty of the union.

`PrivacyLedger` tracks per-peer epsilon and delta consumption against a
daily budget: `canConsume(peer:query:)` gates a query, `consume(...)`
records it, and `dailyReset()` starts a new day. Networking, signing, and
scheduling are explicitly the caller's job. One parity nuance is worth
knowing: the Rust leg zeroes the per-lane distance breakdown in a noised
contribution to close a side channel, while the Swift reference forwards
the breakdown unchanged.

## DPORReduction.swift

This file provides the differentially private OR-reduction used at the
aggregation point of federated queries: many estates' fingerprints in,
one aggregate fingerprint out, with mathematical privacy protection
layered in.

`reduce(fingerprints:params:rngSeed:)` works per bit position. For each
of the 256 positions it counts how many contributors set that bit, adds
Laplace noise with scale one over epsilon (drawn from a SplitMix64 seeded
by the caller, so the noise is reproducible for testing), and sets the
output bit only if the noised count reaches the k-anonymity threshold.
The two mechanisms stack: differential privacy means no observer can tell
whether any one estate contributed, and the k-anonymity floor (default
k=3) means no bit lights up from too few contributors even without
adversarial analysis. Empty input returns the zero fingerprint.

`DPParameters` pins the defaults from the cookbook: epsilon 1.0, delta
1e-9, k 3, with validity preconditions. One documented asymmetry: delta
is validated and stored for API completeness but is not consumed by the
Laplace mechanism itself. The `laplaceNoise(scale:rng:)` helper derives a
uniform value from the top 53 bits of the raw draw — the same convention
the package's other samplers use — and inverts the Laplace distribution's
cumulative curve.

## Rust Port and Conformance

The `rust/` directory contains the second leg of the library: crate
`substrate-ml`, with one module per Swift file — thirty-eight modules from
`anomaly.rs` through `viz_graph_signals.rs`, declared in `rust/src/lib.rs`
with the same layering notes and the same production gate on the parked
NMF variant. The crate depends on `substrate-types`, `substrate-kernel`,
and `intellectus-lib`, mirroring the Swift package manifest exactly.

Most modules open with an explicit "mirror of" header naming their Swift
source, and the determinism-bearing files repeat the pinned constants,
seeds, draw orders, and tie-break rules line for line. Shared gates prove
the agreement property: `rust/tests/distillation_conformance.rs` locks the
delta extractor, the typed decay weights, `featureHash`, and full
pipeline runs against fixed vectors mirrored in Swift's
`DistillationConformanceTests` ("fix the algorithm, not the vector");
`rust/tests/float_simhash_kernel_equivalence.rs` proves the kernel
projection reproduces the oracle bit for bit across seeds and dimensions;
and `rust/tests/viz_graph_signals_tests.rs` exercises the telemetry emit
of all five graph algorithms under a process-wide lock. Several
algorithms carry additional shared JSON vectors under the engineering
test harness (sampling, temporal causality fold, NMF, community
detection). When you change either leg, change both and run both suites;
the fixtures are the contract.
