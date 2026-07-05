---
doc: AGENT_MAP
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

# AGENT_MAP: SubstrateML

PURPOSE: layer-3 cold-path/dreaming algorithm library of the MOOTx01 substrate: learning (decay, Bradley-Terry, calibration, NMF), graph analytics (Louvain, centrality, walks, SVD, FFT, anomaly), pattern mining (association rules, Apriori, FCA, D-G implications, temporal causality), distillation (5-stage cluster→factoid), fingerprint/distance math (FloatSimHash, composite/lattice/partial distances, moment/temporal summaries), and federation privacy math (pairing, tier contribution, tier query, DP OR-reduce). Pure functions + value types only; no storage, no clocks, no hidden state.

DEPS: imports SubstrateTypes (Fingerprint256, HLC, RowId, MatrixO/F/C, SplitMix64, RecallResult…), SubstrateKernel (PortableKernel dispatch: float SimHash projection, orReduce256), IntellectusLib (telemetry; off-path = one atomic-bool load), Foundation. Imported by (per Package.swift/README): LocusKit, CognitionKit, GeniusLocusKit, NeuronKit, dreaming-daemon paths; LatticeLib (moot-semantics) uses EigenvalueCentrality for LexRank; CorpusKit LsaProvider uses JacobiSVD. Within this repo only tests + SubstrateLib's temporary `@_exported` re-export reference it (four-package split mid-migration). Rust port `rust/` = crate `substrate-ml` v1.0.0-skeleton, 38 modules 1:1 with Swift files; conformance tests rust/tests/{distillation_conformance,float_simhash_kernel_equivalence,viz_graph_signals_tests}.rs + shared JSON vectors in the engineering test harness.

ENTRY POINTS (no single facade; per-family):
- DistillationPipeline.swift:277 `DistillationPipeline.run(input:extractFeatures:intraItem:) -> DistillationOutput`: full 5-stage distillation
- CommunityDetection.swift:152 `CommunityDetection.detectFull(adjacency:maxLevels:maxPasses:resolution:estate:ts:) -> [Int]`: multi-level Louvain
- EigenvalueCentrality.swift:87 `EigenvalueCentrality.compute(adjacency:maxIterations:tolerance:estate:ts:) -> [Double]`: keystone scores
- NMFAlternatingLeastSquares.swift:81 `factorize(V:rank:...) -> NMFFactorization`: latent themes
- FloatSimHash.swift:42 `FloatSimHash.project(vector:seed:) -> Fingerprint256`: embedding → fingerprint
- CompositeDistance.swift:50 `CompositeDistance.distance(latticeDistance:fingerprintHammingDistance:...)`: primary recall metric
- AuditLogFold.swift:101 `AuditLogFold.projectAll(events:asOf:nounTypeFor:)`: CRDT state reconstruction
- AprioriMining.swift:167 `AprioriMining.mine(rows:thresholds:) -> [AprioriRule]`: multi-antecedent rules

## Symbol Table

### Telemetry names: VizGraphSignals.swift
- :65 `enum VizGraphSignals`: canonical metric-name constants; one authoritative file prevents orphaned metrics
- :69 `communityAssignment = "community.assignment"` · :73 `centralityScore = "centrality.score"` · :77 `nmfFactor = "nmf.factor"` · :82 `anomalyFlag = "anomaly.flag"` · :86 `edgeDecayedWeight = "edge.decayed_weight"`

### Fingerprint + distance math
- FloatSimHash.swift:30 `enum FloatSimHash`; :42 `project(vector:seed:) -> Fingerprint256`: 256-hyperplane signed projection; empty vector → zero fp; :76 `planes(seed:dim:) -> FloatSimHashPlanes`: deterministic ±1 plane set, cacheable per (seed,dim); :61 private cached kernel (`PortableKernel.kernelForCurrentPlatform()`, selected once)
- CompositeDistance.swift:37 `defaultAlphaLattice = 0.5` / :38 `defaultAlphaFingerprint = 0.5` / :40 `fingerprintTotalBits = 256`; :50 `distance(latticeDistance:fingerprintHammingDistance:alphaLattice:alphaFingerprint:compatibleSeedScope:) -> Double`: preconditions (not clamps); scope-incompatible drops fp term WITHOUT renormalizing (intended)
- LatticeDistance.swift:45 `struct LatticeAnchorStr` (udc + qid; 0 = null); :59 `enum UDCTreeDistance` (:63 `longestCommonPrefixLength`, :83 `distance` clamped [0,1]); :104 `protocol WikidataAdjacencyProvider`; :110 `enum WikidataGraphDistance` (:112 `maxDepth = 4`, :113 `normalizationScale = 3.0`, :118 `shortestPathLength` bounded BFS, :144 `distance` = 1-exp(-len/3), unreachable/null → 1.0); :161 `enum LatticeDistance` (:163/:164 default alphas 0.5, :185 combined `distance(_:_:provider:alphaUDC:alphaQID:)`, :172/:180 legacy hashed-anchor overloads: 0/1 only, compatibility)
- PartialStateRecall.swift:58 `enum PartialStateRecall`; :75 `score(rowFingerprint:anchor:matchBlocks:differBlocks:) -> Double`: matchScore × differScore, blocks ∈ {0,1,2,3} precondition-enforced, empty set → 0; :104 `topK(anchor:rows:matchBlocks:differBlocks:k:)`: linear scan reference; :128 `hammingBlocks(_:_:blocks:)`: block-restricted Hamming
- ShingleSimilarity.swift:53 `enum ShingleSimilarity`; :58 `windowSize = 3`; :69 `shingles(_:) -> Set<String>`: lowercase 3-char window; <3 chars → whole-string shingle; empty → empty set; :96 `similarity(_:_:) -> Float32`: Jaccard; both-empty → 0.0
- MomentSummary.swift:54 `struct RowLite` (fingerprint + captureHLC, 40 bytes); :70 `summarize(rows:[Row]...)` / :89 `summarize(rows:[RowLite]...)`: OR-reduce of rows matching caller predicate; byte-identical, RowLite ~4-5× faster (conformance CRC 0x6762440b); :100 `capturedDuring(_:_:)` convenience predicate; :106 `orReduce(_:)` raw reduction
- TemporalCompression.swift:30 `enum WindowLevel` hour→year (:43 `nextCoarser`, :48 `approxSeconds`: deliberately approximate bucket divisors); :60 `struct TemporalWindow` (:80 `empty(level:)` identity); :93 `compress(rows:startHLC:endHLC:level:)`; :110 `rollup(windows:to:)`: OR fp union, `&+` counts, min/max bounds; :135 `cascadeRollup(hourWindows:upTo:)`: epoch-seconds integer-division bucketing per level

### Ingestion shaping
- FeatureExtractors.swift:41 `enum StreamSourceFlag: UInt8`; :50 `struct AmbientSampleRow`; five sample+extractor pairs, each `extract(_:hlc:rowId:) -> AmbientSampleRow`: :82/:102/:109 HealthKit, :136/:158/:165 CoreLocation (+ :190 `quantizeGeohash` internal), :200/:223/:230 EventKit (attendees sorted before hashing), :257/:279/:286 ScreenTime, :308/:333/:340 SystemTelemetry; :363 internal `encodePayload` key-sorted deterministic binary. Pinned UDC anchors: 613.71 / 914 / 65.012.4 / 004.5 / 004.2. Only HealthKit rows carry payload
- AuditLogFold.swift:44 `struct ProjectedRowState` (no fingerprint: recompute from bitmaps+anchor); :79 `projectCurrentState(rowId:nounType:events:)`; :90 `projectStateAt(...asOf:)`; :101 `projectAll(events:asOf:nounTypeFor:)`; :131 private `foldOrdered`: HLC-sorted sequential fold; tombstone stateRaw == 33 (adjectiveBitmap & 0x3F) is STICKY (I-22); fold is permutation-invariant over the event set (I-21 sync convergence)
- RowAttributeView.swift:51 `enum RowAuditValue` (.bitmap/.integer/.null); :70 `struct RowAuditEntry`; :111 `struct RowAttributeView` (:130 init sorts attributes, :140/:151 manual Eq/Hash); :189 `from(auditEntries:) -> [RowAttributeView]`: vocab sorted+capped 64 (6-bit field), latest-write-wins per fieldPath by HLC, bitmap → one attr per set bit, integer → low byte, null → nothing, empty rows dropped; vocab is EPHEMERAL per call (pre-merge batches for stable indices)

### Learning + decay
- MatrixDecay.swift:39 `struct DecayingMatrix` (values + halfLifeSeconds + lastDecayTimeSeconds; :60 subscript); :84 `MatrixDecay.apply(to:nowSeconds:estate:ts:)`: factor = exp(-dt·ln2/τ), dt ≤ 0 no-op but still emits factor 1.0; saturating Int64 cast for telemetry tag; :148 `decayFactor(elapsedSeconds:halfLifeSeconds:)` pure; :157 `decayAndAdd(...)` decay-then-add; :177/:183/:189 `applyExponentialDecay(to: MatrixF/O/C ...)`: intentional NO-OP reference adapters (real logic in production kit); :199 `Double.ln2 = 0.6931471805599453` (local, matches Rust f64::LN_2); :208 `enum DecayHalfLives`: F 90d, C 180d, O 60d, T 30d, ActionOutcomes 365d, calibration 730d, W_ranking 90d (illustrative; manifest is authoritative)
- ActionOutcomeMatrix.swift:37 `struct ActionOutcomeKey`: 6-bit preconditions (o07/o08), :51 `packed: UInt16`, :55 Comparable; :60 `struct ActionOutcomeCell` (:75 `successRate`, :83 `wilsonLowerBound` 95%, z=1.96); :95 `struct ActionOutcomeMatrix` (:101 `observe(action:outcome:success:at:)` `&+=`, :113 `successRate`, :119 `observationCount`, :139 `topActions(forOutcome:k:minObservations:)`: ranked by Wilson LB, ties count desc then action asc, returns rate+LB+count together, :158 `populatedCellCount`)
- BradleyTerry.swift:31 `struct PreferenceObservation` (winnerID/losers/weight); :53 `struct BradleyTerryEstimator` (`theta: [UUID: Double]` private(set)); :67 `init(learningRate: 0.05, l2: 0.001, theta: [:])`; :78 `observe(_:)`: SGD in log-space, multi-loser SEQUENTIAL against running theta (pinned parity contract); :105 `observeBatch(_:)`; :113 `strength(of:)` = exp(theta); :120 `probability(_:beats:)` = sigmoid(Δtheta). No RNG; warm-restartable; ranking signal only, never truth
- LLMCalibrationCurve.swift:37 `struct LLMCalibrationCurve`; :38 `binCount = 20` / :39 `binWidth = 0.05`; :50 `observe(claimedConfidence:actualOutcome:)`: clamp [0, 0.999999], wrapping counters; :62 `actualRate(in:)`; :72 `midpoint(of:)`; :78 `expectedCalibrationError()`; :92 `brierScore()`; :107 `decay(factor:)` fractional (730d half-life policy, daemon-driven)
- Sampling.swift:57 `enum Sampling`; :75 `sampleNormal(rng: inout SplitMix64)`: Box-Muller COSINE branch only, exactly 2 draws/call, u1 floored at leastNormalMagnitude; :102 `sampleGamma(shape:rng:)`: Marsaglia-Tsang (squeeze 0.0331), shape<1 via Ahrens-Dieter with extra draw BEFORE recursion (pinned order); :157 `sampleBeta(alpha:beta:rng:)`: g1/(g1+g2), alpha-Gamma drawn first, 0.5 degenerate fallback. Conformance vector sampling.json; relies on libm bit-identity

### Graph analytics
- CommunityDetection.swift:80 `typealias Adjacency = [[(neighbor: Int, weight: Double)]]`; :100 `detect(adjacency:maxPasses:estate:ts:)`: Phase 1 only; :152 `detectFull(adjacency:maxLevels:maxPasses:resolution:estate:ts:)`: full Louvain; resolution gamma default 1.0 = classical (IEEE 1.0*x == x, legacy-safe), NeuronKit callers pass 0.05 (pair-lock escape); :368 `canonicalize(_:)`: first-appearance renumbering; private: :202 totalEdgeWeight (zero threshold 1.0e-30 → identity labels, NO emit), :214 detectCore (gain ties → lowest label asc), :317 condense, :345 emit. Defaults maxPasses 10, maxLevels 10; exactly ONE `community.assignment` emit per call
- EigenvalueCentrality.swift:60 `typealias Adjacency`; :62 `defaultMaxIterations = 100` / :63 `defaultTolerance = 1e-6`; :87 `compute(...) -> [Double]`: AUTHORITY via Aᵀx (`xNext[j] += w*x[i]` for edge i→j; design-council 2026-06-04, conformance-locked; symmetrize for undirected), Perron shift 1.0 per iteration (breaks bipartite ±λ oscillation), zero-norm (<1e-30) → uniform 1/√n fallback, all 3 return paths emit `centrality.score` (value 1.0)
- RandomWalks.swift:36 `typealias Adjacency`; :38 `defaultRestartProb = 0.15`; :43 `walk(adjacency:start:length:restartProb:seed:) -> [Int]`: hard preconditions on kernel validity (indices, finite non-negative weights); walk[0] == start; dead end = implicit restart; :99 `sampleWeighted(_:rng:)`: roulette wheel; total ≤ 0 → uniform; rounding shortfall → last neighbor; :127 `uniform01(_:)`: top 53 bits, Rust bit-match; :141 `walkWithRestart(seed:steps:restartProbability:rngSeed:adjacency:) -> [RowId: Int]`: RowId-space uniform-choice visit counts (recall_exploratory); :172 `struct SplitMix64` (public; canonical constants 0x9E3779B97F4A7C15 / 0xBF58476D1CE4E5B9 / 0x94D049BB133111EB)
- NMFAlternatingLeastSquares.swift:45 `struct NMFFactorization` (W m×k, H k×n, rank, iterations, finalError RMS); :81 `factorize(V:rank:maxIterations:100:tolerance:1e-4:seed:0xDEADBEEFCAFEBABE:estate:ts:)`: Lee-Seung multiplicative updates, eps 1e-9, init = SplitMix64 16-bit draws /0xFFFF, preconditions rectangular/finite/non-negative; flat-buffer unsafe-pointer hot loops (loop nest + reduction order preserved EXACTLY; conformance CRC 0x300bf633; ~100× perf vs bounds-checked); emits `nmf.factor` = finalError; :220 `reconstructionError(V:W:H:)` public RMS; internal flat/nested matMul helpers :247/:288/:309/:329/:353/:370/:388
- NMFDoubleFrobeniusSquared.swift:54 `struct NMFDoubleFrobeniusSquaredFactorization` (:93 `loadings(forRow:)`); :113 `enum NMFDoubleFrobeniusSquared`: :116 `defaultMaxIterations = 100`, :122 `defaultTolerance = 1e-6`, eps 1e-9; :153 `factorize(o:rows:cols:rank:seed:0xC0FFEE_BABE_BEEF:...)`: f64 scalar, RAW Frobenius² convergence (not RMS), init floored at 1e-3; :302 `struct NMFDoubleFrobeniusSquaredRNG` (SplitMix64 core, 53-bit output, floor 1e-3). PRODUCTION GATE: no consumer may wire to this pending substrate_math_performance benchmark (ruling 2026-06-13); still conformance-gated vs Rust (nmf_double_frobenius_squared.json)
- AnomalyDetection.swift:26 `enum AnomalyDetection`; :31 `zScore(value:mean:stddev:)` (stddev ≤ 0 → 0); :51 `rollingZScore(window:current:estate:ts:)`: emits `anomaly.flag`; :90 `modifiedZScore(value:median:mad:)`: 0.6745 consistency constant, mad ≤ 0 → 0; :110 `rollingModifiedZScore(...)`: IN-PLACE sort (5× perf, ~400KB saved at 100k; CRC 0x6c6fda4d; must match Rust sort_by partial_cmp for non-NaN); :152 `isAnomalous(zScore:threshold: 3.0)`
- JacobiSVD.swift:74 `struct SVDResult` (U, singularValues non-increasing, Vt, rank); :129 `decompose(A:rank:sweeps:) -> SVDResult`: one-sided Jacobi over a TOURNAMENT sweep schedule (round-robin circle method, replaces the old lexicographic (p,q) nest); sweeps default 30 PINNED (fixed count, not convergence-tested; MUST match across ports); eps 1e-9 skip/zero cutoff; scalar Float32 only, NO SIMD/FMA/Accelerate; sign convention: largest-|entry| of each left vector forced positive; preconditions m ≥ n, rectangular, non-empty; :331 `tournamentRounds(_:)`: pure integer function of n, returns column-DISJOINT (p,q) pairs per round (unit-asserted disjointness), twinned in Rust `JacobiSvd::tournament_rounds` and pinned by a shared schedule hash; each round fans out over `DispatchQueue.concurrentPerform` when `activeProcessorCount` > 1 and the round has 2+ pairs, else runs serially; disjointness makes a round's rotations commute, so output is bit-identical regardless of thread count; :376 `rotatePair(...)`: unsafe-buffer-pointer rotation body, same arithmetic and skip threshold as the pre-tournament serial code; :437 `jacobiCS(alpha:beta:gamma:)`: pinned expression tree (ζ,t,c,s), DO NOT refactor
- FFT.swift:36 `struct Complex` (:51 `magnitude`, :58 `magnitudeSquared`, :63/:68/:73 +,-,*); :83 `enum FFT`: :99 `forward(real:) -> [Complex]` Cooley-Tukey radix-2 DIT, length MUST be power of two (constitutional; callers zero-pad), :162 `magnitudeSpectrum(real:)`; :169/:180 `bitReverse`/`log2Floor` (@usableFromInline); :201 `struct RhythmResult` (dominantPeriodSeconds nil = "no rhythm", honest); :218 `enum RhythmAnalysis`: :229 `analyze(series:bucketDurationSeconds:)` scans bins 1...N/2, energy excludes DC; :281 `analyze(fingerprints:block:bitPosition:bucketDurationSeconds:)`: bit-series extraction, block 0..3 / bit 0..63 preconditions. Production vDSP path must be BIT-IDENTICAL to this scalar reference
- InformationTheory.swift:38 `entropy(_:)`; :48 `mutualInformation(joint:)`; :75 `klDivergence(_:_:)`: length precondition; skips p>0,q=0 terms ⇒ result is a LOWER BOUND; :88 `crossEntropy(_:_:)`; :99 `jensenShannon(_:_:)` bounded [0,1]; :112 `normalizedMutualInformation(joint:)`: ragged matrix → 0 sentinel. All log2 (bits); 0·log0 = 0 by term-skipping; distribution validity is caller's contract

### Pattern mining
- AssociationRuleMining.swift:52 `struct Item` (field/value UInt8; :64 `packed: UInt16` = field<<8|value; :68 Comparable); :76 `struct AssociationRule` (single-Item antecedent + 5 metrics); :107 `struct MiningThresholds` (minSupport/minConfidence); :136 `mineAssociationRules(matrix: MatrixO, activeRowCount:thresholds:)`: N injected (matrix has no row count); activeRowCount ≤ 0 → []; :154/:164 internal `AssociationRuleEngine.mine`: two passes: diagonal = single supports, off-diagonal in canonical packed-key order (NO final sort needed: conformance-relied ordering); self-rules skipped; conviction = +inf at confidence 1; k>2 → AprioriMining
- AprioriMining.swift:55 `struct AprioriThresholds` (minSupport/minConfidence/minLift ≥ 1.0 default/maxK default 3, floored ≥ 2); :89 custom `init(from:)` routes through public init so serialized maxK < 2 cannot bypass the clamp; :115 `struct AprioriRule` (antecedent `[Item]` sorted by packed, + evidenceCount); :167 `AprioriMining.mine(rows:thresholds:)`: join on (k-2)-prefix, subset-count, prune; rule extraction iterates itemsets in lex order (dict order undefined); 4-key sort lift↓ confidence↓ evidence↓ lex↑; :310 `mineAprioriRules(rows:thresholds:)` free-function wrapper; private `HashableItemset`/`itemsetLess`/`aprioriJoin` (:325/:340/:366)
- FormalConceptAnalysis.swift:41 `struct FormalAttribute: Comparable`: lex (namespace,key,value) order = determinism backbone; :70 `enum SeedMode` .single/.multi; :90 `struct CoverDelta` / :117 `struct ConceptCoverDeltas` (:140 `covering(concepts:)`: cover edges ≠ implications; O(n²m) bounded); :210 `struct FormalConcept` (extent/intent/support/stability?); :247 `struct FormalContext`: :251 nested `RowID = UInt32` (avoids LocusKit RowID collision), :285 `from(rowAttributeViews:)` (namespace "row"), :298 `init(rows:)`, :334 `extent(of:)`, :343 `intent(of:)`, :354 `closure(of:)`; internal FCABitSet plumbing :366/:380/:386/:665; :415 `struct BoundedConceptMiner` (minSupport clamped ≥1, maxIntentSize, maxConcepts, seedMode, maxSeeds, stabilityBudget 0 default → stability nil, stabilitySeed 0xCAFEBABEDEADBEEF): :475 `mine(context:)`: one closure per seed, dedup by intent, truncate; sort support↓ intent-size↑ lex; :595 `enum StabilityEstimator` (:616 `estimate(concept:context:budget:seed:)`: Bernoulli-half subsets, per-concept SplitMix64 seed = caller seed XOR FNV(canonical key))
- ConceptImplications.swift:62 `struct Implication` (premise/conclusion; caller ensures disjoint); :96 `struct ConceptImplications` (implications + isTruncated); :148 `conceptImplications(over:context:maxImplications:maxPremiseSize:)`: Next-Closure pseudo-intent enumeration in increasing size order; maxImplications hard cap sets isTruncated (with look-ahead :358 for honesty); maxPremiseSize skips silently (soundness unaffected); output sort premise-size → lex premise → lex conclusion; empty context → empty untruncated
- TemporalCausalityFold.swift:61 `struct TemporalFieldCoord` (fieldPath + valueRepr stable string); :98 `struct TemporalAuditEntry` (hlc + fieldCoords); :121 `struct FoldResult` (deltas in first-insertion order + newWatermark); :139 `struct TemporalCausalityKey` (source/target/lagBucket); :174 `lagBuckets = [1,2,4,8,16,32,64,128]`: must mirror MatrixTier.lagBuckets exactly; :178 `defaultWindowMinutes = 256`; :191 `maxWindowOccupancy = 512`: bulk-import quadratic guard (2026-07-02 decision), keeps most-recent in-window sources; :202 `lagBucket(forMinutes:)`: smallest boundary ≥ delta, clamp 128; zero-ms → minute 1; :241 `fold(entries:windowMinutes:startWatermark:)`: entries MUST be pre-sorted by HLC (not re-sorted; silent corruption otherwise); hourly batch cadence supersedes weekly

### Distillation
- TypedDecayWeighting.swift:32 `enum DistillationFeatureType: String`: ENT/REL/TMP/NUM; :40 `decayLambda` = 0.1/0.2/0.5/0.8 (pinned, diffusion-schedule analogy); :52 `enum TypedDecayWeighting`; :65 `weight(featureType:ageInUnits:)` = exp(-λ·max(0,age)); :86 `weightedDocFrequency(featureType:presenceTimestamps:allMemoryTimestamps:referenceDate:timeUnit: 86_400)`: decayed presence weight / total weight; empty or zero-total → 0
- DeltaFeatureExtractor.swift:29 `enum DeltaType: String` STATIC/CONVERGENT/MONOTONE/OSCILLATING/DIVERGENT; :41 `struct DeltaAnalysis` (deltaType/terminalValue/convergenceScore/slope?/confidence); :86 `analyzeCategorical(sequence:decayLambda: 0.5)`: oscillation = period-2 over LAST 4 obs only (≥4 required); convergence = trailing-run fraction vs λ; :148 `analyzeNumerical(sequence:decayLambda: 0.8)`: diffs all-zero STATIC / one-sign MONOTONE (confidence = λ fixed) / strict alternation OSCILLATING / else DIVERGENT. Pure; zero RNG
- DistillationScorer.swift:16 `struct ExtractedFeature` (:41 init, display defaults to value); :58 `struct DistillationSNR` (readyToDistill iff snr ≥ 2.0); :71 `struct FeatureGraph` (components sorted by weighted df ↓: dominant first); :94 internal `binaryEntropy(_:)`; :114 `structuralThreshold(M:)` = 2/M (M=1 → 2.0, correctly blocks); :120 `applyStructuralThreshold(features:M:)`; :145 `computeSNR(features:M:)`: episodic floor 1e-6; :174 `computeStructuralScores(features: inout)` σ = df·(1−H(df)); :194 `buildPMIGraph(thresholdFeatures:incidenceMatrix:M:)`: PMI log2 ONLY (Foundation.log2 ⇔ f32::log2 bit-match), positive-PMI edges, DFS components O(n²) ok ≤ ~150; :257 `selectDominantComponent(graph:)`; :277 `computeConfidence(selected:allThreshold:)` = meanDf × coherence ratio
- DistillationPipeline.swift:27 `struct DistillationInput` (memoryContents/timestamps?/clusterID/sourceIDs; computed M); :53 `struct DistillationOutput` (drawerContent/confidence/uncertain/snr/deltaType/succeeded/failureReason/featureFingerprint); :81 `struct DistilledHeader` (:102 `parse(_:)`: nil unless "[DIST|" prefix); :172 `typealias FeatureExtractor = @Sendable (String, DistillationFeatureType) -> [ExtractedFeature]`; :184 `featureSimHashSeed = 0x44495354494C4C41` ("DISTILLA"): conformance-critical, changing invalidates ALL stored fingerprints; :193 `featureHash(_:) -> Fingerprint256`; :221 `queryFingerprint(query:extractFeatures:)`: probe fp, no inference; :240 `defaultExtractor` (capitalization heuristic; tests/default); :277 `run(input:extractFeatures:intraItem: false)`: 5 stages; intraItem skips SNR gate + keeps ALL passing features (no PMI pruning of a single doc); ubiquity re-add at df ≥ (M-1)/M (zero-PMI spine rescue); confidence < 0.4 fail, [0.4,0.7) uncertain; `src=` = sourceIDs.count NOT M; deliberate monolithic body

### Federation + privacy
- PairingHandshake.swift:37 `struct PairingNonce`: exactly 32 bytes (precondition); :48 `seedWith(estateA:estateB:)`: UUIDs ordered by RAW BYTES, never uuidString (ASCII hex order ≠ byte order; Rust compares [u8;16]: string order would derive incompatible families); :79 `struct PairingRecord` (:87 `isActive`); :114 `struct PairingAuditPayload`; :133 `enum PairingHandshake`: :145 `generateSharedFamily(nonce:estateA:estateB:density:)` (both sides derive identical family, no round trip), :156 `sharedFamilyKey(case:peerEstate:)` = "H_shared_<case>_<peer8>", :168 `buildPairEvent(...)`, :179 `buildUnpairEvent(...)` (unpair RETAINS family for asOf queries); :192 private `combinedFamilyHash`; :209 fileprivate `lexLessOrEqual` raw-byte compare. FNV-1a 0xCBF29CE484222325 / 0x100000001B3 mixers
- TierContributionFingerprint.swift:45 `enum FederationCase: UInt32` household 1 / fleet 2 / industry 3; :51 `struct TierContribution`; :81 `build(estateUUID:case:shareableFingerprints:hlc:)`: kernel-dispatched orReduce256; :95 `encode(_:) -> Data`: 64-byte canonical wire: UUID(16) | case u32 BE | rowCount u32 BE | 4× fp block u64 BE | hlc.packed u64 BE: BE-UNIFORM (earlier LE-fp draft diverged from Rust by 32 bytes; conformance caught it); :117 `decode(_:)`: nil unless exactly 64 bytes + known case; :148/:154/:160/:166 endian helpers. No signing/checksum at this layer (egress signs; DP at aggregator)
- TierAscendingQuery.swift:35 `enum TargetTier` .peer/.fleetAggregate/.industryAggregate; :41 `struct TierAscendingQuery` (originatingEstate/primitiveName/primitiveInput/targetTier/privacyBudget/queryHLC); :61 `struct PeerResponse`; :68 `enum TierAscendingQueryProtocol`: :72 `computeLocal(query:dispatch:)` (injected closure; no CognitionKit dep), :80 `applyDPToContribution(_:budget:rngSeed:)`: Laplace scale 1/ε per score, CI = ±1.96·scale; NOTE Swift forwards `breakdown` unchanged while Rust zeroes it (side-channel; parity nuance), :100 `combine(local:peers:)`: per-RowId score sum, sort score↓ RowId↑, WIDEST CI wins (conservative); :133 `struct PrivacyLedger` (:141 `remaining`, :147 `canConsume`, :152 `consume`, :159 `dailyReset`: manual, no auto-expiry)
- DPORReduction.swift:31 `struct DPParameters` (:36 init: ε 1.0 > 0, δ 1e-9 ∈ [0,1) VALIDATED BUT UNUSED by Laplace (documented), k 3 ≥ 1); :48 `enum DPORReduction`; :54 `reduce(fingerprints:params:rngSeed:) -> Fingerprint256`: per-bit popcount + Laplace(1/ε) noise + k-anonymity threshold; empty → .zero; fresh SplitMix64(rngSeed) inside; :85 internal `laplaceNoise(scale:rng:)`: inverse CDF from top 53 bits (raw >> 11)

## INVARIANTS / GOTCHAS

- DETERMINISM IS THE CONTRACT. Swift and Rust legs must agree bit-for-bit on shared conformance vectors. Any change to seeds, draw orders, tie-breaks, loop-nest/reduction order, or pinned constants must be mirrored in `rust/src/` and pass both test suites. SplitMix64 is the only PRNG; all uniform doubles come from the top 53 bits.
- NO CLOCKS, NO STATE, NO I/O anywhere in the package. `ts`, `nowSeconds`, `referenceDate`, watermarks: all caller-supplied. Everything is Sendable value types or pure static functions.
- Pinned seeds: canonical conformance seed 0xCAFEBABEDEADBEEF (FCA stability, temporal-fold vectors); NMF default 0xDEADBEEFCAFEBABE; parked-NMF 0xC0FFEE_BABE_BEEF (historical, deliberate); distillation featureSimHashSeed 0x44495354494C4C41: changing it orphans every stored distillation fingerprint.
- Pinned constants: do not change without conformance regen: JacobiSVD sweeps 30; FFT power-of-two input; Louvain zero-weight 1e-30 + gamma 1.0 legacy identity; centrality Perron shift 1.0 + tolerance 1e-6 + cap 100; restartProb 0.15; Marsaglia-Tsang 0.0331; anomaly threshold 3.0 + MAD 0.6745 (CRC 0x6c6fda4d); shingle window 3; lag buckets {1..128} + window 256 min + occupancy 512; SNR gate 2.0 + τ 2/M + confidence 0.4/0.7 + ubiquity (M-1)/M; decay λ ENT/REL/TMP/NUM 0.1/0.2/0.5/0.8; half-life table (90/180/60/30/365/730/90 d); calibration 20 × 0.05 bins; Wilson z 1.96; DP defaults ε 1.0, δ 1e-9, k 3; CompositeDistance 256 bits, alphas 0.5/0.5; UDC clamp, BFS depth 4, scale 3.0; NMF eps 1e-9 + tolerance 1e-4 (CRC 0x300bf633); MomentSummary CRC 0x6762440b.
- NMFDoubleFrobeniusSquared is PRODUCTION-GATED: no consumer may wire to it until the substrate_math_performance benchmark passes. Do not "clean it up" into use; do not delete it either (it is the benchmark baseline).
- EigenvalueCentrality is AUTHORITY (Aᵀx), locked by conformance vector. Do not flip to hub semantics. Symmetrize adjacency for undirected use.
- MatrixDecay is constitutional: decay only shrinks; the three typed applyExponentialDecay overloads are intentional no-ops at reference level. dt ≤ 0 is a no-op that still emits factor 1.0.
- AuditLogFold tombstone (stateRaw 33) is sticky forever; fold is event-set permutation-invariant: that IS the sync-convergence proof. Fingerprint deliberately absent from projections.
- TemporalCausalityFold requires pre-sorted input by HLC; violation silently corrupts (no crash). RowAttributeView vocab (≤64 fields) is per-call ephemeral: merge batches first for stable indices.
- PairingHandshake seed ordering compares raw UUID bytes, never uuidString: ASCII hex order can invert byte order and desync from Rust. Nonce exactly 32 bytes.
- TierContribution wire format is BE-uniform 64 bytes (fingerprint blocks BE, not the LE storage form). Unsigned at this layer by design.
- TierAscendingQuery parity nuance: Rust zeroes DistanceBreakdown in DP contributions; Swift forwards it unchanged. Flag before relying on either behavior.
- CompositeDistance with incompatible seed scope DROPS the fingerprint term without renormalizing: smaller distance is intended, not a bug. Preconditions, not clamps, on inputs.
- InformationTheory.klDivergence is a LOWER BOUND when q has zeros where p has mass. NMI returns 0 sentinel on ragged input. All log2.
- Apriori maxK floor 2 is enforced in init AND in custom Decodable (synthesized decode would bypass it). Association mining needs caller-injected N; matrix diagonal doubles as single-item support.
- Hard preconditions (crash, not error) on malformed inputs throughout: RandomWalks kernel validity, NMF domain (rectangular/finite/non-negative), SVD shape (m ≥ n), FFT length, block IDs {0..3}, 6-bit action/outcome keys, 32-byte nonce.
- Unsafe-buffer hot loops (NMF, SVD, moment/anomaly paths) exist for auto-vectorization; all indices are loop-derived, never data-derived. Preserve loop nest and reduction order exactly: Float32 lowest-bit rounding is conformance-visible.
- Telemetry: exactly five signals (VizGraphSignals); one emit per invocation; off-path = single atomic-bool load. Degenerate zero-weight Louvain emits nothing. Only AnomalyDetection/CommunityDetection/EigenvalueCentrality/NMF/MatrixDecay emit.
- Distillation `src=` field counts sourceIDs, not cluster members (differs in intra-item mode). intraItem skips the SNR gate and PMI pruning by design.
