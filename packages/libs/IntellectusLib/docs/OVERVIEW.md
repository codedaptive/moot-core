---
doc: OVERVIEW
package: IntellectusLib
repo: moot-core
authored_commit: b2a5c30b794cf477e18022c55e2fea348614d337
authored_date: 2026-07-04
sources:
  - path: Sources/IntellectusLib/Intellectus.swift
    blob: 47d1dbe60d0ed880eaf657986ef9371f66423fda
  - path: Sources/IntellectusLib/RecentWindowSink.swift
    blob: 275f059bb1c0b5ebea68c7d32d2e6bb5ad5ed034
  - path: Sources/IntellectusLib/StatSample.swift
    blob: 5a63d181680c35a1b79279f83651f8cb62dcd9b7
  - path: Sources/IntellectusLib/StatsSink.swift
    blob: 60dc64a00370e491273c2328409a9d56fc46463b
---

# IntellectusLib Overview

## What This Library Does

IntellectusLib lets any part of the MOOTx01 substrate report a small fact
about itself while it runs: how long an operation took, how many times
something happened, or that a particular lifecycle event occurred. It calls
this small fact a telemetry sample. A telemetry sample is either a named
measurement, such as "capture latency was 42.5 milliseconds," or a record of
a lifecycle transition, such as "row X was captured in estate Y."

The library does not decide what to do with a sample. It hands each sample to
a receiver, called a sink, that the host program installs. A host that wants
to persist samples to disk installs a sink that writes to a database. A host
that only wants a live look at recent activity installs a sink that keeps the
last few hundred samples in memory. IntellectusLib supplies one such sink,
`RecentWindowSink`, and a protocol, `StatsSink`, so any host can supply its
own.

## The Problem It Solves

Substrate code runs on a hot path: the same functions execute many times per
second while a user captures memories or the system dreams through them.
Measuring that code is valuable, but a always-on measurement system is not
free. Building a telemetry payload costs time and memory even if nothing
reads the result afterward.

Most of the time, no one is watching. A production device runs with
monitoring off by default. IntellectusLib is built around one guarantee: when
monitoring is off, reporting a sample costs almost nothing — a single memory
read and a comparison, nothing else. The library never builds a payload, never
takes a lock, and never calls a sink unless a host has explicitly turned
monitoring on. This guarantee is what lets substrate code call
`Intellectus.report(...)` freely, everywhere, without worrying about the cost
of doing so.

The library also has to work correctly when many threads report samples at
once, since the substrate itself runs concurrently. Turning monitoring on or
off, or swapping in a new sink, must never corrupt a sample that is in flight
on another thread.

IntellectusLib keeps this promise in two independent implementations. A Swift
leg serves Apple platforms, and a Rust leg (in `rust/`) mirrors it, section for
section, function for function, for hosts that need a Rust build of the
substrate. Both legs use the same design: a lock-free flag for the on/off
gate, and a lock held only briefly around the installed sink.

## How It Works

Reporting a sample happens in one call: `Intellectus.report(...)`. That call
takes an argument built lazily — Swift's `@autoclosure` — meaning the
expression that builds the sample is not evaluated until `report` decides it
is needed. `report` checks a single on/off flag first. If monitoring is off,
the function returns immediately, and the sample-building expression never
runs at all. Nothing is allocated, and no sink is touched.

If monitoring is on, `report` reads the currently installed sink, briefly
holding a lock so it cannot read a sink in the middle of being replaced, then
releases the lock, builds the sample, and calls the sink's `receive(_:)`
method. The sink runs outside the lock, so a slow or reentrant sink cannot
block a concurrent call that only wants to install a new sink.

A host controls the gate with `Intellectus.setEnabled(_:)` and installs a
receiver with `Intellectus.install(sink:)`. Both are meant to be called
rarely — typically once, at startup, or when an operator toggles monitoring
on or off. The library ships one ready-to-use sink, `RecentWindowSink`, which
keeps a fixed-size, oldest-evicted-first buffer of the most recent samples in
memory. A host can read that buffer back at any time to see recent activity,
which is useful for a live dashboard or a diagnostic command. `RecentWindowSink`
can also wrap a second sink and forward every sample to it after recording it,
so a host can keep a live window and write to durable storage from a single
installed sink.

## How the Pieces Fit

Figure 1 shows the library's topology — its major parts and how a sample
moves through them.

![Figure 1. Topology of IntellectusLib](topology.svg)

*Figure 1. Topology of IntellectusLib. A caller's report flows through the
enabled gate; when open, the sample reaches the installed sink, which may be
the bundled `RecentWindowSink` forwarding on to a second, host-supplied sink.
The dashed region marks the parts a host, not the library, supplies.*

`StatSample.swift` defines the data that flows through the system: the
`StatSample` enum and its two cases, `.metric` and `.event`. `StatsSink.swift`
defines the receiver contract, `StatsSink`, and the default do-nothing
receiver, `NoOpSink`, which is installed before any host calls `install(sink:)`.
`RecentWindowSink.swift` provides the one concrete sink the library ships.
`Intellectus.swift` ties the pieces together: it holds the current sink and
the on/off flag, and exposes the `Intellectus` enum as the single public entry
point that the rest of the substrate calls.

## What Ships in the Package

The package ships four Swift source files and a matching Rust port in
`rust/`. It has no dependency on any other library in the repository — not
even the lowest substrate libraries — so that those libraries can, in turn,
depend on IntellectusLib without creating a dependency cycle. It depends only
on Foundation and on Swift's `Synchronization` module, both supplied by the
platform. The platform floor is macOS 26 and iOS 26, chosen because that is
the first OS release where `Synchronization.Atomic` is available without a
compatibility fallback.
