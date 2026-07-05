---
doc: OVERVIEW
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

# SubstrateLib Overview

## What This Library Does

SubstrateLib is the orchestration layer of MOOTx01's storage substrate. It decides what counts as a legal change to a memory. It refuses every other kind of change.

MOOTx01 is an on-device AI memory system. It stores what an AI observes over time. Later, the AI can recall that content.

Each stored memory is one row. A row is an entry with a lifecycle state. A row also links into a classification lattice. LatticeLib builds that lattice.

Each row carries three 64-bit integers called bitmaps. A bitmap packs many small status fields into one compact form. These fields include sensitivity, trust, and exportability.

SubstrateLib supplies three things a row needs to stay trustworthy.

First is a single write gate, `AuditGate.admit`. Every bitmap change must pass through this gate.

Second is a row-state automaton. It lists which of ten lifecycle states a row may hold. It also lists which state changes are legal.

Third is a reference set of the nine verbs. A verb is one fixed action a row can undergo. Capture, mutate, and withdraw are three examples. Production storage kits check their own faster code against this reference.

Two supporting pipelines round out the package.

A Merkle hash pipeline computes deterministic content hashes. These hashes let the system prove a memory's content has not changed. The system proves this without reading the whole memory store again.

A keyed-commitment pipeline proves that a piece of content once existed. It does this after the content has been destroyed. Destruction happens to satisfy a deletion request.

The value types SubstrateLib works with live in a sibling package. That package is called SubstrateTypes. Its types include `Row`, `RowState`, `NounType`, `LatticeAnchor`, and `AuditEvent`. SubstrateLib imports these types. It adds the logic on top: the rules, the gate, and the verbs.

## The Problem It Solves

MOOTx01 estates can federate. Federation means separate devices exchange and merge memories. Merging works cleanly only when two devices agree on one thing. They must compute the same event identifier for the same logical event.

SubstrateLib solves this with a deterministic content identifier. Each audit event gets a SHA-256 hash of its stable fields. That hash is folded into a UUID, instead of a random one. Two replicas that reach the same write independently produce the same identifier.

This sameness matters for a grow-only set. A grow-only set, often called a G-Set, can gain entries but never lose them. Because the identifier is deterministic, a G-Set simply keeps one copy. A random identifier would defeat this design. The same logical event would look like two different events on two devices. Federation would then double-count it.

A second problem is structural corruption. A row's status lives in three 64-bit integers. Each integer packs several unrelated fields at fixed bit positions. Nothing at the type level stops a careless write from causing harm. A careless write could touch the wrong bits. It could write a value wider than its field. It could move a row into a combination the design forbids. One example is marking a secret memory as exportable.

SubstrateLib closes this gap with one gate that every write must pass. A write must land as a legal value in a declared field. It must produce a legal state and a legal combination of fields. Otherwise, the gate refuses it. Corruption of this kind becomes structurally impossible this way. Convention alone would only discourage it, never prevent it.

A third problem is privacy against accountability. When a memory is expunged, MOOTx01 destroys its content for good. This satisfies a legal-compliance requirement. An audit trail that simply drops the event creates a puzzle, though. Nothing happening and something being hidden look the same from the outside. An absence proves nothing on its own.

SubstrateLib's keyed-commitment pipeline resolves this problem. It commits, cryptographically, to the fact that a payload once existed. It uses a secret key the estate holds. It retains nothing the payload could be rebuilt from.

The mathematics in this package is conformance-gated. A Rust port in `rust/` mirrors every algorithm. Shared test fixtures require both legs to produce identical output. A device running the Swift leg and a device running the Rust leg must agree. They must agree on every hash, every content identifier, and every gate decision.

## How It Works

The write gate and the verb reference sit side by side. Both build on the same row-state automaton. But they serve different callers.

`AuditGate.admit` is the gate production kits call directly. A caller supplies the row's identity. A caller also supplies the prior snapshot of its bitmaps. A caller supplies only the field values it owns.

The gate looks up each written field in the instance's admitted vocabulary. That vocabulary holds the substrate's universal fields: state, sensitivity, exportability, and trust. It also holds whatever fields the calling kit registered for itself. The gate rejects any field that is undeclared. It rejects any value that is out of range or not in the field's legal set.

The gate then reads the prior bitmaps. It writes only the addressed bits. It leaves everything else untouched. This is a read-modify-write, never a blind overwrite.

Before returning, the gate re-derives the encoded state. It checks that state against what the supplied verb was allowed to produce. It also checks the whole result against the automaton's forbidden-combination rules. Only then does it compute the deterministic content identifier. Only then does it emit one canonical audit event.

The row-state automaton itself is a fixed transition table. For each pair of a state and a verb, either a next state is defined, or the transition is illegal. The automaton reports illegal transitions as such.

A second check sits on top of the table: the forbidden-combination rules. A transition can be legal on its own. Yet it can still produce a field combination the design never allows. One example is a secret memory that is also exportable. Both checks together define what "legal state" means in this package.

`Substrate` is the reference verb implementation. It offers a second, independent route to the same guarantees. It is an in-memory struct. It implements all nine verbs end to end.

`Substrate` validates a mutation against the automaton directly. It applies the bitmap change. It updates two summary structures called matrices. These matrices track how many rows carry each field value. It appends an audit event. All of this happens inside one call.

Production storage engines reimplement these same nine verbs against real storage. Examples include a SQLite tail and a memory-mapped bit-slice tensor. `Substrate` is the scalar oracle these engines are tested against. It is not itself the production path.

The two supporting pipelines are simpler and pure.

The Merkle hash pipeline builds a fixed byte encoding of a memory's content. It also encodes the memory's vector embeddings. It tags this encoding with a domain byte. That tag stops a leaf hash from being mistaken for an interior or tombstone hash. The pipeline runs the tagged bytes through SHA-256.

The keyed-commitment pipeline reuses that same byte encoding. It applies a different domain tag. It runs the bytes through a keyed HMAC instead of a plain hash. This binds the commitment to a secret only the estate holds.

Every one of these code paths can report activity to an injected telemetry sink. This includes the gate, the verbs, and both hash pipelines. But none of them ever reads a clock. None of them changes its output because monitoring is on. Timestamps arrive as a caller-supplied parameter. When the sink is disabled, the entire telemetry call collapses to one lock-free flag check.

## How the Pieces Fit

Figure 1 shows the library's topology. It shows the major parts and how a write moves through them.

![Figure 1. Topology of SubstrateLib](topology.svg)

*Figure 1. Topology of SubstrateLib. Two independent write paths exist: the production `AuditGate` and the reference `Substrate` verbs. Both consult the shared row-state automaton. A separate, unrelated pipeline computes content hashes and keyed commitments. These give the integrity and privacy guarantees. Both paths report to the telemetry sink. The sink is shown as a dashed external region because it lives in the sibling package IntellectusLib.*

`AuditGate` and `Substrate` do not call one another. They are two consumers of the same rules. Neither feeds the other in a pipeline. A kit such as LocusKit calls `AuditGate.admit` directly on every bitmap write. `Substrate` exists for a different reason. Alternative or future storage engines need a known-correct in-memory reference to test against.

Keeping the two paths independent, rather than layering one on the other, is deliberate. It guarantees that a bug in the fast path cannot hide behind a passing reference-path test. The two paths share only the rules. They do not share code.

## What Ships in the Package

The package ships seven Swift source files, listed above. It also ships a Rust port in `rust/`. The Rust port mirrors each Swift file one for one.

The package ships no pinned data artifacts of its own. There is no bundled JSON and no trained model. SubstrateLib's guarantees are entirely computational. They rest on a fixed transition table, a fixed byte encoding, and a cryptographic hash. All of these are deterministic by construction. None depend on shipped reference data.

Conformance fixtures live in `Tests/SubstrateLibConformanceTests/` and in `rust/tests/`. Both suites must pass before either leg changes ship.
