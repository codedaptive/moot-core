---
doc: AGENT_MAP
package: SubstrateLib
repo: moot-core
authored_commit: b2a5c30b794cf477e18022c55e2fea348614d337
authored_date: 2026-07-04
sources:
  - path: Sources/SubstrateLib/AuditGate.swift
    blob: 1f5ea748a82d21756518ed2dc4dda9e0a2ceed4a
  - path: Sources/SubstrateLib/KeyedCommitment.swift
    blob: 2b93f766914a2261cc4d63067469a0609157665f
  - path: Sources/SubstrateLib/KeyedCommitmentAudit.swift
    blob: afdac27147578c4b0cd2324e3db861812f89097a
  - path: Sources/SubstrateLib/MerkleHash.swift
    blob: 2a7e1b957d0f38633ee9048108fb9b59f8ee5648
  - path: Sources/SubstrateLib/RowStateAutomaton.swift
    blob: b3111584e1495a41d66e51de620e27e55f31d4cd
  - path: Sources/SubstrateLib/SubstrateLibTelemetry.swift
    blob: b76dd8280311aeb67d8f5499108a0c0b080dc787
  - path: Sources/SubstrateLib/Verbs.swift
    blob: b3f6b0db6b099d6932f7e57c017637193e2a4be6
---

# AGENT_MAP — SubstrateLib

PURPOSE: orchestration layer of the substrate stack. Single write gate (AuditGate.admit) + row-state automaton (legal transitions, I-22/S-1/S-2/S-4 forbidden combinations) + reference impl of the nine substrate verbs (Substrate struct) + Merkle content-hash pipeline + keyed-commitment pipeline for expunge provenance. Value types (Row, RowState, NounType, LatticeAnchor, AuditEvent, HLC, MatrixF/O/T, Fingerprint256) live in SubstrateTypes.

DEPS: imports SubstrateTypes (value types), SubstrateKernel (BitField, SHA256, GrantHKDF.hmac), SubstrateML (transitively via Package.swift, not used directly in these 7 files), IntellectusLib (Intellectus.report telemetry sink). Imported by: LocusKit (AuditGate.admit is the sole bitmap write path), other kits per DECISION_KIT_GRAPH_REFACTOR_2026-05-19. Sibling packages in the four-way split: SubstrateTypes (data), SubstrateKernel (hot-path kernels), SubstrateML (cold-path/ML). Rust port in rust/src/ mirrors all 7 files 1:1; conformance fixtures in rust/tests/ + Tests/SubstrateLibConformanceTests/ gate byte-identity.

ENTRY POINTS (most callers need only these):
- AuditGate.swift:317 `AuditGate.admit(estateUuid:rowId:nounType:verb:prior:priorLatticeAnchor:writes:afterLatticeAnchor:vocabulary:hlc:actor:) -> Result<AuditEvent, GateViolation>` — THE single production write gate for row bitmaps
- AuditGate.swift:222 `VocabularyValidator.freeze(union:) -> Result<Vocabulary, VocabularyError>` — one-time vocabulary construction, required before any Vocabulary exists
- RowStateAutomaton.swift:213 `RowStateAutomaton.validate(from:on:targetingFields:) throws -> RowState` — the constitutional mutation gate (legality + I-22/S-1/S-2/S-4)
- Verbs.swift:76 `struct Substrate` — in-memory reference impl of the nine verbs; scalar oracle for production storage engines

## Symbol Table

### Row-state automaton — RowStateAutomaton.swift
- :50 `enum RowStateAutomaton` — namespace; RowState/RowVerb/RowStateError types live in SubstrateTypes, imported here
- :72 `canTransition(from:to:viaVerb:) -> Bool` — §10 STRING-verb vocabulary (legacy); looks up :84 `verbTable` (fileprivate `[VerbKey: RowState]`); used by Verbs.swift mutate/withdraw
- :78 `fileprivate struct VerbKey` — (RowState, String) composite key for verbTable
- :127 `static let transitions: [TransitionKey: RowState]` — CANONICAL enum-keyed table (§9 lifecycle vocabulary); absence from map = illegal
- :204 `transition(from:on:) -> RowState?` — single lookup against `transitions`
- :213 `validate(from:on:targetingFields:) throws -> RowState` — transition() + ForbiddenCombinations.check(); THE gate; bypassing is forbidden (v0.36 C1 resolution)
- :227 `struct TransitionKey` — (from: RowState, verb: RowVerb)
- :240 `struct BitmapFields` — adjective/operational/provenance UInt64 triple
- :260 `enum ForbiddenCombinations`
- :270 `check(state:fields:) throws` — I-22 (secret+exportable forbidden), S-1 (accepted⇒trust≥canonical), S-2 (defensive state-encoding check), S-4 (accepted⇒sensitivity≤elevated); S-5 (tombstone completion flag) NOT YET IMPLEMENTED (queued, see comment at file end)

### Write gate — AuditGate.swift
- :51 `struct FieldSlot: Hashable, Sendable` — one declared field-slot: column/shift/width/label/legalValues
- :52 `enum Column` — .adjective | .operational | .provenance
- :72 `capacity: Int64` — 1<<width upper bound
- :76 `bitMask: UInt64` — used for overlap detection
- :84 `admits(value:) -> Bool` — width + enumerated-legal-value check
- :106 `enum AuditState` / :113 `AuditSensitivity` / :118 `AuditExportability` / :123 `AuditTrust` — SubstrateLib-local mirrors of LocusKit's Adjectives.swift enums (raw values MUST match; can't import upward); parity enforced by GuardianPairParityTests
- :134 `struct Vocabulary: Sendable`
- :145 `static let basis: Set<FieldSlot>` — universal fields: state(0,w6) sensitivity(6,w6) exportability(12,w6) trust(18,w6) flags(24,w3, bits 24/25/26 = state_extension/lineage_clustering/dreaming_recalc_required); legalValues derived from allCases.map{rawValue}
- :188 `union: Set<FieldSlot>` — per-instance consumer fields, frozen
- :192 `fileprivate init(validatedUnion:)` — ONLY VocabularyValidator.freeze can construct
- :196 `slot(for:) -> FieldSlot?` — basis-first lookup, then union
- :206 `enum VocabularyError` — .overlap / .basisCollision / .malformedWidth / .valueExceedsWidth
- :222 `enum VocabularyValidator`
- :224 `freeze(union:) -> Result<Vocabulary, VocabularyError>` — 3-step validation (width-sane → no basis collision → no union-union overlap); run ONCE before any data exists
- :258 `struct FieldWrite: Sendable` — (slot, value) one consumer-requested change
- :269 `enum GateViolation: Error, Sendable` — .undeclaredField / .illegalValue / .basisViolation(Error) / .stateInconsistentWithVerb
- :292 `description: String` — English-only at ARIA boundary; NO Swift case-name leakage; carries the "illegal state transition: ..." sentinel the describe_gate_rejection parser matches
- :311 `enum AuditGate`
- :317 `admit(...) -> Result<AuditEvent, GateViolation>` — see ENTRY POINTS; 4 steps: vocab+value gate → read-modify-write (preserves unaddressed bits) → basis gate (state must equal verb's authorized output) → contentID + AuditEvent
- :421 `static func contentID(estateUuid:rowId:hlc:verb:after:afterAnchor:) -> UUID` — SHA-256(stable byte encoding) folded to UUID; NAME-keyed (not ordinal), vocabulary-set-independent; also called directly by Verbs.swift's appendAudit so both paths produce identical event IDs

### Verb orchestration — Verbs.swift
- :48 `typealias RowId = UUID` — wire-compatible with Rust's RowId(u128)
- :55 `enum SubstrateError: Error, Equatable` — invalidStateTransition / missingLatticeAnchor / invalidNounType / rowNotFound / forbiddenStateCombination / alreadyTombstoned / proposalRequired / nonProposalCannotUseProposalVerb
- :76 `struct Substrate` — see ENTRY POINTS; fields :77-84 (estateUuid, rows, auditEvents(G-Set), matrixF/O/T, hlc, rowCountActive)
- :86 `init(estateUuid:hlc:)`
- :110 `capture(nounType:adjectiveBitmap:operationalBitmap:provenanceBitmap:latticeAnchor:fingerprint:lineageId:content:actor:ts:) -> Result<UUID, SubstrateError>` — ONLY verb that originates a row w/o prior state; proposal⇒pending else active; rejects null latticeAnchor
- :180 `reanchor(rowId:newLatticeAnchor:actor:ts:) -> Result<(), SubstrateError>` — lattice-link change only, bitmaps untouched
- :213 `enum MutationKind: String` — confirm/reject/contest/supersede/automated_confirm/decay/expire/lineage_advance/actuator_confirm
- :222 `mutate(rowId:mutationKind:newAdjectiveBitmap:newOperationalBitmap:newProvenanceBitmap:actor:ts:) -> Result<(), SubstrateError>` — canTransition() (§10 string verbs) + isLegalRowState() gate; general state+field change
- :304 `withdraw(rowId:actor:ts:) -> Result<(), SubstrateError>` — → .withdrawn; setStateField raw=18
- :365 `expunge(rowId:reason:actor:ts:) -> Result<(), SubstrateError>` — → .tombstoned, content=nil; REFUSES .accepted rows (S-3, audit-grade survive intact) before any other check
- :425 `recall(matching:asOf:ts:) -> [Row]` — READ-ONLY; no mutation, no audit event; asOf reconstructs via audit-log HLC filter
- :457 `propose(...) -> Result<UUID, SubstrateError>` — capture(nounType: .proposal, ...)
- :484 `associate(rowA:rowB:signalSourcesBitset:weight:...) -> Result<UUID, SubstrateError>` — capture(nounType: .association, ...); `weight` param ACCEPTED THEN DISCARDED (vestigial, reserved for pre-2.0 gauntlet experiment) — do not silently wire it up or silently drop the parameter
- :531 `learn(...) -> Result<UUID, SubstrateError>` — capture(nounType: .learnedReference, ...)
- :555 `private appendAudit(verb:rowId:before:after:beforeAnchor:afterAnchor:actor:)` — advances hlc; calls AuditGate.contentID (same fn as the gate path) so both paths produce identical event IDs
- :596 `private isLegalRowState(state:adjective:operational:) -> SubstrateError?` — wraps ForbiddenCombinations.check; provenance passed as 0 (unused by check)
- :615/:624/:632/:637 private bit-twiddling wrappers around SubstrateTypes.RowBitmaps (rowHasBit, extractFieldValues, extractState, setStateField)

### Merkle hash pipeline — MerkleHash.swift
- :31 `struct MerkleVectorInput: Sendable` — modelID, vectorIndex, floats (Float32 LE)
- :51 `enum MerkleHash`
- :76 `leaf(drawerId:content:vectors:) -> ContentHash` — v2 format: vector identity (modelID+vectorIndex) written BEFORE float payload (v1 gap = WS2-F4 vector-substitution finding, fixed 2026-06-28)
- :101 `interior(childHashes: [(UUID, ContentHash)]) -> MerkleRoot` — sorted by UUID BE bytes; empty ⇒ MerkleRoot.empty
- :130 `interior(childRoots: [(UUID, MerkleRoot)]) -> MerkleRoot` — same as above, wing/estate-level roll-up (children already MerkleRoots)
- :157 `tombstone(drawerId:) -> ContentHash` — domain tag only + drawer id; no content/vectors (already destroyed)
- :178 `static func canonicalLeafBytes(drawerId:content:vectors:domainTag:) -> [UInt8]` — SHARED encoding with KeyedCommitment.commit; domainTag is the only difference between leaf hash and commitment preimage
- :226/:230/:236 private byte-encoding helpers (uuidBytes, appendU64BE, appendU32BE)

### Keyed commitment — KeyedCommitment.swift
- :26 `struct KeyedCommitmentValue: Hashable, Sendable, Codable` — hmacBytes (32B, precondition-enforced) + keyVersion
- :40 `hexString: String`
- :49 `enum KeyedCommitment`
- :66 `commit(key:keyVersion:drawerId:content:vectors:) -> KeyedCommitmentValue` — MerkleHash.canonicalLeafBytes(domainTag: .commitment=0x03) → GrantHKDF.hmac (SubstrateKernel, no new HMAC impl)

### Commitment audit log — KeyedCommitmentAudit.swift
- :27 `struct KeyedCommitmentAuditEntry: Hashable, Sendable` — id (content-hash), drawerId, commitment, tombstoneHLC, reason
- :40 `init(drawerId:commitment:tombstoneHLC:reason:)` — computes :62 `private static computeID(...) -> [UInt8]` (SHA-256 over identifying fields) at construction
- :92 `struct CommitmentAuditLog: Sendable` — G-Set keyed by content hash
- :103 `add(_:)` — idempotent insert
- :108 `merge(_:)` — CRDT join = set union
- :114 `count: Int`
- :117 `orderedEntries: [KeyedCommitmentAuditEntry]` — sorted by tombstoneHLC
- :122 `entries(forDrawer:) -> [KeyedCommitmentAuditEntry]`

### Telemetry — SubstrateLibTelemetry.swift
- :78 `enum SubstrateLibMetric` — string-constant namespace, `substratelib.*` metrics
- :83/:86 auditGateAdmitCount / auditGateRejectCount
- :91/:94/:97/:100/:103/:106 verbCaptureCount / verbMutateCount / verbWithdrawCount / verbExpungeCount / verbRecallCount / verbReanchorCount
- :111/:114 writeGateAdmittedCount / writeGateRejectedCount
- :132 `emitAuditGateAdmit(nounTypeRaw:ts:)`, :152 `emitAuditGateReject(violationName:ts:)`
- :170 `emitWriteGateAdmitted(verb:ts:)`, :187 `emitWriteGateRejected(verb:reason:ts:)`
- :204/:219/:233/:247/:266/:280 `emitVerb{Capture,Mutate,Withdraw,Expunge,Recall,Reanchor}Count(...)` — one per Verbs.swift verb
- ALL emit fns `@inline(__always)`; wrap `Intellectus.report(...)`; disabled-path cost = one atomic load + branch, zero allocation

## INVARIANTS / GOTCHAS

- DETERMINISM IS THE CONTRACT (same as LatticeLib). No file in this package may read a clock. Every `ts` parameter is caller-supplied epoch seconds; monitoring on/off must never change functional output. Rust mirrors every algorithm; conformance fixtures in rust/tests/ + Tests/SubstrateLibConformanceTests/ gate byte-identity — run both suites on any change to audit_gate/merkle_hash/keyed_commitment/row_state/verbs.
- AuditGate and Substrate do NOT call each other. Two independent consumers of the same RowStateAutomaton rules, not a pipeline. Do not wire one to call the other — that would let a fast-path bug hide behind a passing reference-path test.
- AuditGate.admit and Verbs.appendAudit call the SAME contentID function (AuditGate.contentID) so both paths produce byte-identical event IDs for the same logical event. If you change the byte layout in contentID, both paths and the Rust mirror change together.
- Content IDs (AuditGate.contentID, KeyedCommitmentAuditEntry.computeID) are NAME/CONTENT-keyed, never random UUIDs. This is what lets a G-Set (grow-only set) dedupe on federation merge. Do not reintroduce `UUID()` random defaults on any audit-adjacent identity.
- S-3 (cookbook §9.5.3): accepted rows CANNOT transition to tombstoned. Enforced in TWO places independently: RowStateAutomaton.transitions (no TransitionKey(.accepted, .tombstone) entry) AND Verbs.expunge's explicit early-return guard. Do not remove either; they are a deliberate belt-and-suspenders pair.
- S-5 (tombstone ⇒ expunge_completed_flag) is NOT YET IMPLEMENTED in ForbiddenCombinations.check — queued pending F17 cookbook location decision (adjective bit 24 or operational reserved bits). A previous incorrect implementation (zeroing bitmaps on tombstone) was removed 2026-05-27; do not reintroduce that specific check.
- AuditState/AuditSensitivity/AuditExportability/AuditTrust (AuditGate.swift) are LOCAL COPIES of LocusKit/Adjectives.swift enums — raw values must match exactly (SubstrateLib cannot import LocusKit; dependency graph points the other way). GuardianPairParityTests enforces this; the `@guardian-pair` comment annotations mark every duplicated literal (also present in RowStateAutomaton.swift's ForbiddenCombinations.check: 48/32/3/16 = secret/public/canonical/elevated raw values).
- Vocabulary can ONLY be constructed via VocabularyValidator.freeze (fileprivate init). Never bypass with a literal struct construction — that skips overlap/collision/width validation entirely.
- FieldSlot.legalValues empty ⇒ "any value that fits width" (used for state, governed by the automaton instead of an enumerated set... actually state DOES enumerate via AuditState.allCases; empty-legalValues is used only by the flags field). Do not conflate "empty" with "no restriction beyond width" vs "enumerated" — check the specific basis slot.
- MerkleHash leaf format is v2 (vector identity before float payload). v1 had a vector-substitution security gap (WS2-F4, fixed 2026-06-28). KeyedCommitment.commit shares this exact byte encoding via canonicalLeafBytes — a leaf-format change without updating both Rust and Swift, and both the hash and commitment call sites, reopens or diverges the fix.
- MerkleHash.interior sorts children by UUID ascending (lexicographic 16-byte BE) — required for roll-up to be write-order-independent. Do not sort by insertion order or hash value.
- Verbs.associate's `weight` parameter is accepted and explicitly discarded (`_ = weight`) — vestigial, reserved for a future experiment. Do not silently start persisting it without a matching bitmap/column change, and do not remove the parameter from the signature without checking callers.
- Package.swift depends on SubstrateTypes + SubstrateKernel + SubstrateML + IntellectusLib directly (not transitively) per DECISION_LIFT_PACKAGE_SWIFT_RULE_2026-05-28 — IntellectusLib sits below SubstrateLib in the topology (depends on nothing in-repo), so this is not a layering violation.
- No pinned data artifacts ship in this package (unlike LatticeLib) — every guarantee here is computational (fixed table, fixed byte encoding, cryptographic hash), not reference-data-driven.
