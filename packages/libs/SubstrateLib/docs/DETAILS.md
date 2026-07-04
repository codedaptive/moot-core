---
doc: DETAILS
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

# SubstrateLib Details

This document walks through every source file in the package. Read
`OVERVIEW.md` first for the big picture. Files appear here in dependency
order: the row-state automaton first, because both write paths consult it;
then the write gate and the reference verbs, the two paths that consult
it; then the content-integrity pipeline, which is independent of both; and
finally telemetry, which every other file calls into.

A note on vocabulary before the walkthrough: a row is one stored memory. A
bitmap is a 64-bit integer that packs several small status fields at fixed
bit positions, so one integer can carry many independent facts cheaply. A
verb is one of the fixed set of actions a row can undergo — capture,
mutate, withdraw, and so on. An audit event is the permanent record of one
verb having happened to one row.

## RowStateAutomaton.swift

This file provides the row-state automaton: the transition table that
says which lifecycle-state changes are legal, and the forbidden-combination
checks that catch a legal transition landing on an illegal field
combination.

The data types the automaton operates on — `RowState` (ten states),
`RowVerb` (twelve verbs), and `RowStateError` — are defined in the sibling
package SubstrateTypes and imported here. Only the compute stays in
SubstrateLib: the table, the validation function, and the forbidden-
combination rules. `RowState`'s ten values are deliberately scale-gapped
into three numeric clusters — active (0–3), historical (16–19), and
terminal (32–33) — so a caller can classify a stored raw value's cluster
with one shift-and-mask, without decoding the full enum.

`RowStateAutomaton.canTransition(from:to:viaVerb:)` answers a yes-or-no
question for one older vocabulary of verb names (contest, supersede,
expunge, and others) inherited from an earlier design phase. It looks the
pair up in a private table, `verbTable`, and compares the result to the
requested target state. It exists because `Verbs.swift`'s `mutate` and
`withdraw` still speak this vocabulary; new call sites should prefer the
canonical function below instead.

`RowStateAutomaton.transitions` is the canonical table: a dictionary
keyed by `(from state, verb)`, using the `RowVerb` enum rather than raw
strings. Any pair absent from the table is illegal by construction — there
is no default case to fall through. `RowStateAutomaton.transition(from:on:)`
looks up one pair and returns the next state, or `nil` if the pair is
absent. `RowStateAutomaton.validate(from:on:targetingFields:)` is the
single mutation gate the design calls constitutional: it requires a legal
transition, then also runs the target's bitmap fields through
`ForbiddenCombinations.check`, and only returns the next state if both
pass. An earlier design defect let one code path skip this second check
and write field combinations the table alone would not catch; every
mutation path in this package and in `AuditGate` now routes through one
of these two checks before a write is considered final.

`ForbiddenCombinations.check(state:fields:)` enforces four safety rules
that the transition table cannot express, because they depend on more than
one field at once:

- A row marked secret can never also be marked exportable, regardless of
  its lifecycle state. Sensitivity and exportability are stored as
  separate 6-bit fields in the same bitmap; nothing about the storage
  format alone prevents both being set to conflicting extremes, so this
  rule is checked explicitly on every write.
- An accepted row — one a person or process explicitly confirmed, treated
  as audit-grade — must carry a trust level of at least "canonical."
  Accepted status is a promise about reliability, and a low trust value
  would make that promise false.
- A withdrawn or rejected row must actually encode the matching state
  value in its bitmap, a defensive check against corrupted input rather
  than a rule that a caller could realistically violate through the API.
- An accepted row's sensitivity may not exceed "elevated," because a
  higher sensitivity would make an audit-grade row too restricted to
  serve the purpose accepted status is meant to guarantee.

A fifth rule the design calls for — that a tombstoned row must carry an
"expunge completed" flag — is deliberately not yet implemented here; the
comment in the source explains that an earlier attempt enforced the wrong
invariant (zeroing bitmap fields that the design says must survive
expunge) and was removed until the correct flag location is settled
elsewhere in the bitmap layout.

`BitmapFields` and `TransitionKey` are the small carrier types the
functions above operate on: three raw 64-bit integers, and a
`(state, verb)` composite key, respectively.

## AuditGate.swift

This file provides the production write gate, `AuditGate.admit`, and the
vocabulary machinery that arms it. This is the interface a storage kit —
LocusKit, in the current design — calls on every bitmap mutation.

A write gate is only as strong as the vocabulary it enforces, so this file
first defines what a legal field looks like. `FieldSlot` describes one
declared field: which of three columns it lives in (adjective,
operational, or provenance), its bit position and width within that
column, a human label, and its set of legal values. An empty legal-value
set means "any value that fits the width," which basis fields use because
the automaton, not an enumerated list, governs their combinations.
`FieldSlot.admits(value:)` is the per-field check: the value must fit the
width, and if the field enumerates specific legal values, the value must
be one of them.

`Vocabulary.basis` is the fixed set of fields every SubstrateLib instance
and every federation peer must agree on: state, sensitivity, exportability,
and trust, each a 6-bit field, plus a 3-bit flags field for miscellaneous
markers. Their legal-value sets are derived automatically from four local
enums — `AuditState`, `AuditSensitivity`, `AuditExportability`,
`AuditTrust` — so adding a case to one of those enums extends the gate's
vocabulary without a second, easy-to-forget update elsewhere. These four
enums duplicate raw values defined independently in a higher-level package
(LocusKit), because SubstrateLib sits below that package in the
dependency graph and cannot import its types; a separate test suite
(`GuardianPairParityTests`) checks that the two definitions never drift
apart.

Beyond the fixed basis, each SubstrateLib instance may register its own
additional fields — the union. `VocabularyValidator.freeze(union:)` is
the one place a union is accepted: it checks that every proposed field's
width and enumerated values fit within 64 bits, that no proposed field's
bits collide with a basis field's bits, and that no two proposed fields
collide with each other. Only a union that passes all three checks becomes
a `Vocabulary`; the type's initializer is `fileprivate`, so an unvalidated
vocabulary cannot reach the gate by any other path. This validation runs
once, before any row exists, which matters because a corrupt vocabulary
discovered after data has accumulated would be far harder to recover from
than one rejected up front.

`AuditGate.admit(estateUuid:rowId:nounType:verb:prior:priorLatticeAnchor:writes:afterLatticeAnchor:vocabulary:hlc:actor:)`
is the gate itself, and it runs in four steps. First, every field in the
caller's write list must resolve to a declared slot in the vocabulary, and
its value must be admissible; an undeclared field or an out-of-range value
is rejected immediately, before any bitmap changes. Second, the gate
performs a read-modify-write: it starts from the row's prior bitmaps (or
zero, for a new row), writes only the addressed bits, and leaves every
other bit exactly as it was — a consumer that owns one field can never
accidentally clobber a field it does not know about. Third, the gate
checks the resulting state against what the supplied verb was allowed to
produce: for a brand-new row, the verb must be `capture` and the encoded
state must be a legal starting state; for an existing row, the verb's
transition must match the state the write actually encodes, and the
forbidden-combination rules must hold. Fourth, and only if every prior
step succeeded, the gate computes a deterministic content identifier and
returns one canonical `AuditEvent`.

`AuditGate.contentID(estateUuid:rowId:hlc:verb:after:afterAnchor:)`
computes that identifier: it serializes the identifying fields into a
fixed byte layout, hashes them with SHA-256, and folds the first sixteen
bytes of the digest into a UUID. Because the identifier is a pure function
of the event's content — not a random value — two replicas that compute
the same logical event independently produce the same identifier. That
sameness is what lets a grow-only set deduplicate the event automatically
when replicas merge, which is the property federation depends on.

`GateViolation` is the gate's error type, covering an undeclared field, an
illegal value, a basis violation (wrapping the underlying
`RowStateAutomaton` error), and a state that does not match its verb. Its
`description` property is written to produce plain English at the
system's outer boundary — no Swift type or case name ever appears in
user-visible text — including a specific sentinel phrase,
"illegal state transition: ...", that a downstream parser matches on.

## Verbs.swift

This file provides the nine substrate verbs and `Substrate`, the in-memory
reference implementation of all of them. This is the scalar oracle:
production storage engines implement the same nine verbs against real
persistence, and their behavior is checked against this reference.

`RowId` is a typealias for `UUID`, kept as a distinct name so call sites
read in the substrate's own vocabulary while staying a plain `UUID` under
the hood; the Rust port spells the same identifier as a `u128` newtype,
and the two encodings are byte-identical.

`Substrate` bundles everything one estate's in-memory reference state
needs: a dictionary of rows, an append-only list of audit events (treated
as a grow-only set), three summary matrices (`matrixF`, `matrixO`,
`matrixT`, defined in SubstrateTypes) that track aggregate statistics
across all rows without re-scanning them, a hybrid logical clock (`hlc`)
that orders events across replicas without relying on wall time, and a
running count of non-tombstoned rows.

Each verb method follows the same shape: check preconditions against the
row-state automaton, apply the bitmap change, update the two matrices with
a symmetric delta (subtract the row's old contribution, add its new one),
append one audit event, and emit one telemetry count. Timestamps arrive as
a caller-supplied `ts` parameter defaulting to `0.0`, never read from a
clock inside the method, which keeps the verb's functional output
independent of when it happens to run.

`capture(nounType:adjectiveBitmap:operationalBitmap:provenanceBitmap:latticeAnchor:fingerprint:lineageId:content:actor:ts:)`
creates a new row. It matters because it is the only verb that can
originate a row without a prior state to transition from: a proposal noun
type starts `pending`, awaiting confirmation, and every other noun type
starts `active`. A missing lattice anchor is rejected outright, because
every row must be locatable in the classification lattice from the moment
it exists.

`reanchor(rowId:newLatticeAnchor:actor:ts:)` changes only a row's link
into the classification lattice, leaving its bitmaps untouched. It exists
separately from `mutate` because reclassifying a memory is a different
kind of change from changing its lifecycle state or its status fields.

`mutate(rowId:mutationKind:newAdjectiveBitmap:newOperationalBitmap:newProvenanceBitmap:actor:ts:)`
is the general-purpose state-and-field change, covering confirm, reject,
contest, supersede, decay, expire, and the actuator/automated confirm
variants. It matters because it is where the automaton's transition check
and the forbidden-combination check both gate a single call: an illegal
transition or an illegal resulting combination is rejected before any row
or matrix is touched.

`withdraw(rowId:actor:ts:)` retracts a row by moving it to the
`withdrawn` state. It matters as a distinct verb, rather than a case of
`mutate`, because withdrawal is available from more starting states than
most other transitions and is a common enough action to warrant its own
simple call shape.

`expunge(rowId:reason:actor:ts:)` is the legal-compliance hard delete: it
moves a row to `tombstoned` and sets its content to `nil`, destroying the
verbatim payload. It matters that `expunge` explicitly refuses an
`accepted` row before doing anything else — an audit-grade row must
survive intact, by design, and no combination of inputs can route around
that refusal.

`recall(matching:asOf:ts:)` is the only read-only verb: it filters rows
by a caller-supplied predicate, optionally reconstructing the row set as
of a past point in the hybrid logical clock by intersecting with the
audit log. It matters that recall never mutates and never appends an audit
event — a memory system that logged every read as if it were a write
would make querying create liability.

`propose`, `associate`, and `learn` are thin verbs that each delegate to
`capture` with a fixed `nounType` — proposal, association, and learned
reference, respectively. They matter as distinct entry points because each
represents a semantically different reason a row came to exist, even
though the mechanics underneath are identical. `associate` additionally
accepts a computed `weight` parameter that it deliberately discards after
accepting it: the parameter is reserved for a future experiment, and the
source documents that the value is accepted and thrown away today rather
than silently invented if it were missing.

The four private helpers at the bottom of the file —
`appendAudit`, `isLegalRowState`, `rowHasBit` /
`extractFieldValues` / `extractState` / `setStateField` — are internal
plumbing: `appendAudit` advances the clock and builds one `AuditEvent`
using the same content-identifier function `AuditGate` uses, so an event
produced by either path is identical in shape; `isLegalRowState` is the
verb layer's call into `ForbiddenCombinations.check`; the rest are small
bit-manipulation wrappers around the shared `RowBitmaps` type in
SubstrateTypes.

## MerkleHash.swift

This file provides the public hash pipeline for the Merkle-style
content-integrity tree: three functions that turn a row's content, a set
of child hashes, or a tombstoned row into one deterministic SHA-256 value.

A Merkle tree proves that content has not changed by comparing hashes
instead of comparing the content itself, and by letting a parent's hash
summarize all of its children's hashes. `MerkleDomain` tags (defined in
SubstrateTypes) — a leaf tag, an interior tag, and a tombstone tag — are
prepended to each function's input before hashing, so a hash computed one
way can never be mistaken for, or substituted as, a hash computed another
way.

`MerkleHash.leaf(drawerId:content:vectors:)` hashes one memory's content
and its vector embeddings into a `ContentHash`. It matters because its
byte encoding is versioned: an earlier version wrote each vector's
identifying information (which embedding model produced it, and which
slot in a multi-vector row) only as a sort key, not into the hashed bytes
themselves, which meant an attacker could substitute one model's vector
for another's without changing the hash — a security defect the source
calls out by name (finding WS2-F4). The current version writes vector
identity into the hash input before the float payload, closing that gap.

`MerkleHash.interior(childHashes:)` and `interior(childRoots:)` roll up a
node's children into one parent hash, sorting children by UUID first so
the result never depends on the order children happened to be written or
iterated in. Both return `MerkleRoot.empty` for a childless node rather
than hashing nothing, which keeps the empty case an explicit, named value
instead of an arbitrary hash of zero bytes.

`MerkleHash.tombstone(drawerId:)` hashes an expunged row's identity alone,
with no content and no vectors, because expunge has already destroyed
both. It matters that the tombstone hash still exists at all: it lets the
Merkle tree continue to prove that a specific row occupied a specific
position, even after everything about that row except its identity is
gone.

`canonicalLeafBytes(drawerId:content:vectors:domainTag:)` is the shared
byte-encoding function both this file and `KeyedCommitment.swift` call.
It matters that it is shared rather than duplicated: the content hash and
the keyed commitment are deliberately computed from the exact same
preimage, differing only in the domain tag and the algorithm applied to
it (a plain hash versus a keyed HMAC), which is what lets the two values
be compared meaningfully as two views of the same underlying commitment
to content.

## KeyedCommitment.swift

This file provides the public keyed-commitment API used at expunge time:
a way to prove a payload existed without keeping anything a reader could
reconstruct the payload from.

`KeyedCommitmentValue` carries the output: 32 raw HMAC-SHA256 bytes plus
the key version that produced them. Recording the key version matters
because estate keys rotate over time, and a commitment made under an
older key must remain verifiable against that older key even after
rotation; without the version, a rotated key would silently invalidate
every commitment made before it.

`KeyedCommitment.commit(key:keyVersion:drawerId:content:vectors:)` builds
the commitment. It reuses `MerkleHash.canonicalLeafBytes`, the exact same
byte encoding `MerkleHash.leaf` uses, but with the commitment domain tag
instead of the leaf tag, and then runs that byte sequence through an
existing HMAC-SHA256 implementation from the sibling package
SubstrateKernel rather than a new one written for this purpose. The domain
separation matters for a specific reason: it guarantees a commitment and a
plain content hash of the same underlying payload are always different
values, so one can never be mistaken for, or substituted as, the other.

## KeyedCommitmentAudit.swift

This file provides the audit-log entry type and append-only container for
keyed commitments made at expunge time — a record distinct from the
ordinary state-transition audit event, because it records that a
commitment was made, not that a row's state changed.

`KeyedCommitmentAuditEntry` bundles the drawer identity, the commitment
value, the hybrid logical clock time the tombstone was applied, and a
human-readable reason. Its identifier, computed by
`computeID(drawerId:commitment:tombstoneHLC:reason:)`, is a SHA-256 hash
over the same identifying fields, which matters for the same reason
`AuditGate.contentID` matters elsewhere in this package: two replicas
that independently record the same logical commitment compute the same
identifier, so a grow-only set deduplicates them without extra
coordination.

`CommitmentAuditLog` is that grow-only set: entries are keyed by their
content hash in a dictionary, `add(_:)` inserts one entry idempotently,
and `merge(_:)` unions two logs together — the CRDT join that lets two
replicas' commitment logs converge regardless of the order in which
entries arrive. `orderedEntries` and `entries(forDrawer:)` give callers a
stable, HLC-ordered view for display or audit review.

## SubstrateLibTelemetry.swift

This file provides the metric name catalogue and the emit functions every
other file in this package calls into when reporting activity.

SubstrateLib's own comment calls it the "determinism floor" of the
substrate stack: nothing in this package may read a clock or change its
functional output depending on whether monitoring is enabled. This file
is where that promise is implemented, not merely stated. Every emit
function wraps its call in `Intellectus.report(...)` (from the sibling
package IntellectusLib), which — when monitoring is disabled, the
default — costs exactly one lock-free flag check and a branch: no metric
object is built, no lock is taken, nothing is allocated. Every timestamp
the emit functions accept is a parameter supplied by the caller, never
read from a clock inside this file, so the substrate's behavior is
byte-identical whether monitoring is on or off.

`SubstrateLibMetric` is a namespace of string constants — one per metric
name — so the complete list of everything this package can report is
visible in one place, and a typo in a metric name is caught at compile
time rather than showing up as a silently missing metric in production.

The eight `emit...` functions each correspond to one point in the gate or
verb flow: `emitAuditGateAdmit` and `emitAuditGateReject` fire from
`AuditGate.admit`'s two outcomes; `emitWriteGateAdmitted` and
`emitWriteGateRejected` fire from the same gate but tagged by verb and
rejection reason, for operators who want to break down gate traffic
differently than by outcome alone; and one function per verb —
`emitVerbCaptureCount`, `emitVerbMutateCount`, `emitVerbWithdrawCount`,
`emitVerbExpungeCount`, `emitVerbRecallCount`, `emitVerbReanchorCount` —
fires from the matching method in `Verbs.swift`. Each is marked
`@inline(__always)` so the disabled-path cost stays a single flag check
with no function-call overhead added on top.

## Rust Port and Conformance

The `rust/` directory contains the second leg of the library: seven
source files mirroring the Swift implementations one for one —
`audit_gate.rs`, `keyed_commitment.rs`, `merkle_hash.rs`, `row_state.rs`,
`verbs.rs`, and `substrate_lib_telemetry.rs`, plus `lib.rs` as the crate
root. Rust has no `CaseIterable`, so the adjective-vocabulary values that
the Swift leg derives from enum cases are instead typed constant slices in
`audit_gate.rs`, kept in step with the Swift enums by convention and
checked by the same cross-layer parity tests. The two legs share
conformance fixtures in `rust/tests/` and
`Tests/SubstrateLibConformanceTests/` — recorded input-output pairs for
the audit-log fold, bitmap field constants, the SimHash-derived float
comparisons, and the wire format — that both implementations must
reproduce byte for byte. When you change either leg, run both test suites;
the fixtures are the contract.
