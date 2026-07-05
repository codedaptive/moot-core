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

This document walks through every source file in the package. Read `OVERVIEW.md` first for the big picture.

Files appear here in dependency order. The row-state automaton comes first, because both write paths consult it. The write gate and the reference verbs come next. These are the two paths that consult the automaton. The content-integrity pipeline comes after that, since it is independent of both. Telemetry comes last, because every other file calls into it.

A note on vocabulary before the walkthrough. A row is one stored memory. A bitmap is a 64-bit integer. It packs several small status fields at fixed bit positions. One integer can carry many independent facts cheaply this way. A verb is one of a fixed set of actions a row can undergo. Capture, mutate, and withdraw are examples. An audit event is the permanent record of one verb happening to one row.

## RowStateAutomaton.swift

This file provides the row-state automaton. The automaton is a transition table. It says which lifecycle-state changes are legal. The file also provides the forbidden-combination checks. These checks catch a legal transition that lands on an illegal field combination.

The data types the automaton operates on live in a sibling package, SubstrateTypes. These types are `RowState` (ten states), `RowVerb` (twelve verbs), and `RowStateError`. Only the compute logic stays in SubstrateLib: the table, the validation function, and the forbidden-combination rules.

`RowState`'s ten values fall into three numeric clusters. These clusters are active, historical, and terminal. The clusters are deliberately spaced apart in the numbering. This lets a caller classify a stored raw value's cluster with one shift-and-mask step. The caller need not decode the full enum.

`RowStateAutomaton.canTransition(from:to:viaVerb:)` answers a yes-or-no question. It uses an older vocabulary of verb names. That vocabulary includes contest, supersede, expunge, and others. This vocabulary comes from an earlier design phase. The function looks the pair up in a private table, `verbTable`. It compares the result to the requested target state. The function exists because `Verbs.swift`'s `mutate` and `withdraw` still use this vocabulary. New call sites should prefer the canonical function described next.

`RowStateAutomaton.transitions` is the canonical table. It is a dictionary keyed by a state-and-verb pair. It uses the `RowVerb` enum rather than raw strings. Any pair absent from the table is illegal by construction. There is no default case to fall through.

`RowStateAutomaton.transition(from:on:)` looks up one pair. It returns the next state, or `nil` if the pair is absent.

`RowStateAutomaton.validate(from:on:targetingFields:)` is the single mutation gate. The design calls it constitutional. It requires a legal transition. It also runs the target's bitmap fields through `ForbiddenCombinations.check`. It returns the next state only if both checks pass.

An earlier design defect let one code path skip the second check. That path could write field combinations the table alone would not catch. Every mutation path in this package now routes through one of these two checks. `AuditGate` does the same before a write counts as final.

`ForbiddenCombinations.check(state:fields:)` enforces four safety rules. The transition table cannot express these rules, because they depend on more than one field at once.

- A row marked secret can never also be marked exportable. This holds regardless of the row's lifecycle state. Sensitivity and exportability are stored as separate six-bit fields in the same bitmap. Nothing about the storage format alone stops both from being set to conflicting extremes. So this rule is checked explicitly on every write.
- An accepted row must carry a trust level of at least canonical. An accepted row is one a person or process explicitly confirmed. It is treated as audit-grade. Accepted status is a promise about reliability. A low trust value would make that promise false.
- A withdrawn or rejected row must actually encode the matching state value in its bitmap. This is a defensive check against corrupted input. It is not a rule a caller could realistically violate through the API.
- An accepted row's sensitivity may not exceed elevated. A higher sensitivity would make an audit-grade row too restricted. That would defeat the purpose accepted status is meant to serve.

A fifth rule the design calls for is not yet implemented here. That rule says a tombstoned row must carry an expunge-completed flag. The source comment explains why. An earlier attempt enforced the wrong invariant. It zeroed bitmap fields that the design says must survive expunge. The attempt was removed until the correct flag location is settled elsewhere in the bitmap layout.

`BitmapFields` and `TransitionKey` are the small carrier types the functions above operate on. `BitmapFields` holds three raw 64-bit integers. `TransitionKey` is a state-and-verb composite key.

## AuditGate.swift

This file provides the production write gate, `AuditGate.admit`. It also provides the vocabulary machinery that arms the gate. Storage kits call this interface on every bitmap mutation. LocusKit is the current example.

A write gate is only as strong as the vocabulary it enforces. So this file first defines what a legal field looks like. `FieldSlot` describes one declared field. It records which of three columns the field lives in: adjective, operational, or provenance. It also records the field's bit position and width within that column. It records a human label too. It records the field's set of legal values as well. An empty legal-value set means any value that fits the width. Basis fields use this feature. The automaton governs their combinations instead of an enumerated list. `FieldSlot.admits(value:)` is the per-field check. The value must fit the width. If the field enumerates specific legal values, the value must be one of them.

`Vocabulary.basis` is the fixed set of fields every SubstrateLib instance must agree on. Every federation peer must agree on it too. The fields are state, sensitivity, exportability, and trust. Each one is six bits wide. A three-bit flags field covers miscellaneous markers. Their legal-value sets come from four local enums: `AuditState`, `AuditSensitivity`, `AuditExportability`, and `AuditTrust`. Adding a case to one of those enums extends the gate's vocabulary automatically. This avoids a second, easy-to-forget update elsewhere. These four enums duplicate raw values defined independently in a higher-level package, LocusKit. SubstrateLib sits below that package in the dependency graph, so it cannot import LocusKit's types. A separate test suite, `GuardianPairParityTests`, checks that the two definitions never drift apart.

Beyond the fixed basis, each SubstrateLib instance may register its own additional fields. These form the union. `VocabularyValidator.freeze(union:)` is the one place a union gets accepted. It checks three things. Every proposed field's width and enumerated values must fit within 64 bits. No proposed field's bits may collide with a basis field's bits. No two proposed fields may collide with each other. Only a union that passes all three checks becomes a `Vocabulary`. The type's initializer is `fileprivate`. So an unvalidated vocabulary cannot reach the gate by any other path. This validation runs once, before any row exists. That timing matters, because a corrupt vocabulary found after data has accumulated would be far harder to recover from.

`AuditGate.admit(...)` is the gate itself. It runs in four steps.

First, every field in the caller's write list must resolve to a declared slot in the vocabulary. Its value must be admissible. An undeclared field or an out-of-range value is rejected immediately, before any bitmap changes happen.

Second, the gate performs a read-modify-write. It starts from the row's prior bitmaps, or zero for a new row. It writes only the addressed bits. It leaves every other bit exactly as it was. A consumer that owns one field can never accidentally clobber a field it does not know about.

Third, the gate checks the resulting state against what the supplied verb was allowed to produce. For a brand-new row, the verb must be `capture`. The encoded state must also be a legal starting state. For an existing row, the verb's transition must match the state the write actually encodes. The forbidden-combination rules must also hold.

Fourth, and only if every prior step succeeded, the gate computes a deterministic content identifier. It returns one canonical `AuditEvent`.

`AuditGate.contentID(...)` computes that identifier. It serializes the identifying fields into a fixed byte layout. It hashes them with SHA-256. It folds the first sixteen bytes of the digest into a UUID. The identifier is a pure function of the event's content, not a random value. So two replicas that independently compute the same logical event produce the same identifier. That sameness is what lets a grow-only set deduplicate the event automatically when replicas merge. Federation depends on this property.

`GateViolation` is the gate's error type. It covers four cases: an undeclared field, an illegal value, a basis violation, and a mismatched state. The basis violation wraps the underlying `RowStateAutomaton` error. Its `description` property produces plain English at the system's outer boundary. No Swift type or case name ever appears in user-visible text. This includes a specific sentinel phrase: "illegal state transition: ...". A downstream parser matches on that phrase.

## Verbs.swift

This file provides the nine substrate verbs. It also provides `Substrate`, the in-memory reference implementation of all nine. This is the scalar oracle. Production storage engines implement the same nine verbs against real persistence. Their behavior gets checked against this reference.

`RowId` is a type alias for `UUID`. It keeps a distinct name so call sites read in the substrate's own vocabulary. Under the hood it stays a plain `UUID`. The Rust port spells the same identifier as a `u128` newtype. The two encodings are byte-identical.

`Substrate` bundles everything one estate's in-memory reference state needs. It holds a dictionary of rows. It holds an append-only list of audit events, treated as a grow-only set. It holds three summary matrices, `matrixF`, `matrixO`, and `matrixT`, all defined in SubstrateTypes. These matrices track aggregate statistics across all rows without re-scanning them. `Substrate` also holds a hybrid logical clock, `hlc`. This clock orders events across replicas without relying on wall time. Finally, it holds a running count of non-tombstoned rows.

Each verb method follows the same shape. It checks preconditions against the row-state automaton. It applies the bitmap change. It updates the two matrices with a symmetric delta. It subtracts the row's old contribution and adds its new one. It appends one audit event. It emits one telemetry count.

Timestamps arrive as a caller-supplied `ts` parameter, defaulting to `0.0`. The method never reads a clock internally. This keeps the verb's functional output independent of when it happens to run.

`capture(...)` creates a new row. It matters because it is the only verb that can originate a row without a prior state to transition from. A proposal noun type starts as `pending`, awaiting confirmation. Every other noun type starts as `active`. A missing lattice anchor is rejected outright. Every row must be locatable in the classification lattice from the moment it exists.

`reanchor(...)` changes only a row's link into the classification lattice. It leaves the row's bitmaps untouched. It exists separately from `mutate`, because reclassifying a memory differs from changing its lifecycle state or its status fields.

`mutate(...)` is the general-purpose state-and-field change. It covers six mutation kinds: confirm, reject, contest, supersede, decay, and expire. It also covers the actuator and automated confirm variants. It matters because it is where the automaton's transition check and the forbidden-combination check both gate a single call. An illegal transition or an illegal resulting combination gets rejected before any row or matrix is touched.

`withdraw(...)` retracts a row by moving it to the `withdrawn` state. It matters as a distinct verb, rather than a case of `mutate`, because withdrawal is available from more starting states than most other transitions. It is also common enough to warrant its own simple call shape.

`expunge(...)` is the legal-compliance hard delete. It moves a row to `tombstoned` and sets its content to `nil`, destroying the verbatim payload. It matters that `expunge` explicitly refuses an `accepted` row before doing anything else. An audit-grade row must survive intact by design. No combination of inputs can route around that refusal.

`recall(...)` is the only read-only verb. It filters rows by a caller-supplied predicate. It can also reconstruct the row set as of a past point in the hybrid logical clock, by intersecting with the audit log. It matters that recall never mutates and never appends an audit event. A memory system that logged every read as if it were a write would make querying create liability.

`propose`, `associate`, and `learn` are thin verbs. Each delegates to `capture` with a fixed `nounType`: proposal, association, and learned reference. They matter as distinct entry points, because each represents a different reason a row came to exist, even though the mechanics underneath are identical. `associate` additionally accepts a computed `weight` parameter. It deliberately discards that parameter after accepting it. The parameter is reserved for a future experiment. The source documents that the value is accepted and thrown away today. It is not silently invented if it were missing instead.

The four private helpers at the bottom of the file are internal plumbing. `appendAudit` advances the clock. It builds one `AuditEvent` using the same content-identifier function `AuditGate` uses. So an event produced by either path is identical in shape. `isLegalRowState` is the verb layer's call into `ForbiddenCombinations.check`. The rest are small bit-manipulation wrappers around the shared `RowBitmaps` type in SubstrateTypes: `rowHasBit`, `extractFieldValues`, `extractState`, and `setStateField`.

## MerkleHash.swift

This file provides the public hash pipeline for the Merkle-style content-integrity tree. It has three functions. They turn a row's content, a set of child hashes, or a tombstoned row into one deterministic SHA-256 value.

A Merkle tree proves that content has not changed. It does this by comparing hashes instead of comparing the content itself. It also lets a parent's hash summarize all of its children's hashes. `MerkleDomain` tags, defined in SubstrateTypes, mark each function's input before hashing. There is a leaf tag, an interior tag, and a tombstone tag. This tagging stops a hash computed one way from ever being mistaken for a hash computed another way.

`MerkleHash.leaf(...)` hashes one memory's content and its vector embeddings into a `ContentHash`. It matters because its byte encoding is versioned. An earlier version wrote each vector's identifying information only as a sort key. That information was which embedding model produced the vector and which slot in a multi-vector row it occupies. The earlier version did not write that information into the hashed bytes themselves. This meant an attacker could substitute one model's vector for another's without changing the hash. The source calls out this security defect by name: finding WS2-F4. The current version writes vector identity into the hash input before the float payload. This closes that gap.

`MerkleHash.interior(childHashes:)` and `interior(childRoots:)` roll up a node's children into one parent hash. They sort children by UUID first. This way, the result never depends on the order children happened to be written or iterated in. Both functions return `MerkleRoot.empty` for a childless node, rather than hashing nothing. This keeps the empty case an explicit, named value instead of an arbitrary hash of zero bytes.

`MerkleHash.tombstone(drawerId:)` hashes an expunged row's identity alone. There is no content and no vectors, because expunge has already destroyed both. It matters that the tombstone hash still exists at all. It lets the Merkle tree keep proving that a specific row occupied a specific position. This holds even after everything about that row except its identity is gone.

`canonicalLeafBytes(...)` is the shared byte-encoding function. Both this file and `KeyedCommitment.swift` call it. It matters that this function is shared rather than duplicated. The content hash and the keyed commitment are deliberately computed from the exact same preimage. They differ only in the domain tag and the algorithm applied to it: a plain hash versus a keyed HMAC. This is what lets the two values be compared meaningfully, as two views of the same underlying commitment to content.

## KeyedCommitment.swift

This file provides the public keyed-commitment API used at expunge time. It offers a way to prove a payload existed, without keeping anything a reader could reconstruct the payload from.

`KeyedCommitmentValue` carries the output. This is thirty-two raw HMAC-SHA256 bytes, plus the key version that produced them. Recording the key version matters, because estate keys rotate over time. A commitment made under an older key must remain verifiable against that older key even after rotation. Without the version, a rotated key would silently invalidate every commitment made before it.

`KeyedCommitment.commit(...)` builds the commitment. It reuses `MerkleHash.canonicalLeafBytes`, the exact same byte encoding `MerkleHash.leaf` uses. But it applies the commitment domain tag instead of the leaf tag. It then runs that byte sequence through an existing HMAC-SHA256 implementation. That implementation comes from the sibling package SubstrateKernel, rather than a new one written for this purpose. The domain separation matters for a specific reason. It guarantees a commitment and a plain content hash of the same underlying payload are always different values. So one can never be mistaken for, or substituted as, the other.

## KeyedCommitmentAudit.swift

This file provides the audit-log entry type and an append-only container for keyed commitments made at expunge time. This record is distinct from the ordinary state-transition audit event. It records that a commitment was made. It does not record that a row's state changed.

`KeyedCommitmentAuditEntry` bundles four things: the drawer identity, the commitment value, the tombstone time, and a human-readable reason. The tombstone time comes from the hybrid logical clock. Its identifier comes from `computeID(...)`. That function computes a SHA-256 hash over the same identifying fields. This matters for the same reason `AuditGate.contentID` matters elsewhere in this package. Two replicas that independently record the same logical commitment compute the same identifier. So a grow-only set deduplicates them without extra coordination.

`CommitmentAuditLog` is that grow-only set. Entries are keyed by their content hash in a dictionary. `add(_:)` inserts one entry idempotently. `merge(_:)` unions two logs together. This is the CRDT join that lets two replicas' commitment logs converge, regardless of the order in which entries arrive. `orderedEntries` and `entries(forDrawer:)` give callers a stable, HLC-ordered view for display or audit review.

## SubstrateLibTelemetry.swift

This file provides the metric name catalogue. It also provides the emit functions every other file in this package calls into when reporting activity.

SubstrateLib's own comment calls this file the determinism floor of the substrate stack. Nothing in this package may read a clock or change its functional output depending on whether monitoring is enabled. This file is where that promise gets implemented, not merely stated.

Every emit function wraps its call in `Intellectus.report(...)`, from the sibling package IntellectusLib. When monitoring is disabled, which is the default, this costs exactly one lock-free flag check and a branch. No metric object gets built. No lock gets taken. Nothing gets allocated. Every timestamp the emit functions accept is a parameter supplied by the caller. It is never read from a clock inside this file. So the substrate's behavior stays identical whether monitoring is on or off.

`SubstrateLibMetric` is a namespace of string constants, one per metric name. This makes the complete list of everything this package can report visible in one place. A typo in a metric name gets caught at compile time. It does not show up as a silently missing metric in production.

The eight `emit...` functions each correspond to one point in the gate or verb flow. `emitAuditGateAdmit` and `emitAuditGateReject` fire from `AuditGate.admit`'s two outcomes. `emitWriteGateAdmitted` and `emitWriteGateRejected` fire from the same gate, tagged by verb and rejection reason. This lets operators break down gate traffic differently than by outcome alone. One function fires per verb: `emitVerbCaptureCount`, `emitVerbMutateCount`, `emitVerbWithdrawCount`, `emitVerbExpungeCount`, `emitVerbRecallCount`, and `emitVerbReanchorCount`. Each is marked `@inline(__always)`. So the disabled-path cost stays a single flag check with no function-call overhead added on top.

## Rust Port and Conformance

The `rust/` directory contains the second leg of the library. Seven source files mirror the Swift implementations one for one: `audit_gate.rs`, `keyed_commitment.rs`, `merkle_hash.rs`, `row_state.rs`, `verbs.rs`, and `substrate_lib_telemetry.rs`. The crate root is `lib.rs`.

Rust has no `CaseIterable`. So the adjective-vocabulary values that the Swift leg derives from enum cases are instead typed constant slices in `audit_gate.rs`. These stay in step with the Swift enums by convention. The same cross-layer parity tests check this.

The two legs share conformance fixtures. These live in `rust/tests/` and `Tests/SubstrateLibConformanceTests/`. The fixtures are recorded input-output pairs. They cover the audit-log fold, bitmap field constants, the SimHash-derived float comparisons, and the wire format. Both implementations must reproduce these byte for byte. When you change either leg, run both test suites. The fixtures are the contract.
