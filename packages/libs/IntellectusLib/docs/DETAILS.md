---
doc: DETAILS
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

# IntellectusLib Details

This document walks through every source file in the package. Read
`OVERVIEW.md` first for the big picture. Files appear here in the order data
flows through the library: the sample type, the receiver contract, the one
bundled receiver, and finally the global facade that ties everything
together for callers.

## StatSample.swift

This file provides `StatSample`, the telemetry datum that flows through the
whole library, and `EventKind`, a small enum used by one of its two cases.

`StatSample` is a value type — an enum with associated values, not a class —
and it conforms to `Sendable`, meaning the Swift compiler checks that it is
safe to pass between threads or concurrent tasks without extra
synchronization. Value types copy instead of sharing memory, which is what
makes that safety check possible: no two threads can race on the same
storage.

The enum has two cases. `.metric` carries a dot-separated name (such as
`"locus.capture.latency_ms"`), a floating-point value, an optional
dictionary of string tags for context (such as `["kit": "LocusKit"]`), and a
timestamp. `.event` carries a lifecycle event: an `EventKind` (`.capture` or
`.think`), the integer form of a substrate row's noun type, the row's
identifier string, the estate identifier string, and a timestamp. `EventKind`
distinguishes a caller-driven write (`.capture`) from a substrate-driven
autonomous transition (`.think`) — the two broad classes of activity the
substrate's Brain layer can trigger.

Every timestamp in the library is a caller-supplied number of seconds since
the epoch, never a value read from the system clock inside this package. This
matters for determinism: a caller that replays the same sequence of events
gets the same timestamps back, and tests can construct samples with fixed,
predictable times instead of racing against the real clock.

`ts`, a computed property added in an extension, returns the timestamp
regardless of which case the sample is. This spares every caller that only
cares about ordering samples in time from writing a `switch` over both cases.

## StatsSink.swift

This file provides `StatsSink`, the protocol every receiver of telemetry
samples must conform to, and `NoOpSink`, the receiver installed before any
host supplies its own.

A protocol in Swift is a contract: it lists the methods and properties a
conforming type must supply, without dictating how. `StatsSink` requires
exactly one method, `receive(_:)`, and requires conformers to be `Sendable`,
because the one sink installed at any moment is held in global state and
called from whatever thread happens to be reporting a sample at the time.

`receive(_:)` delivers one `StatSample` to the sink. The file documents an
expectation, not an enforced rule: implementations should return quickly and
should not block. Anything slow — writing to a database, sending data over a
network — belongs in the sink's own background work, scheduled from inside
`receive(_:)`, not run synchronously inside it. A slow sink would otherwise
add latency to whatever substrate code path triggered the sample.

`NoOpSink` is the simplest possible conformer: `receive(_:)` does nothing and
returns immediately. It exists so the global holder in `Intellectus.swift`
always has a valid sink installed, even before a host calls `install(sink:)`.
Because monitoring is off by default, `NoOpSink.receive(_:)` is not expected
to run in ordinary use — the enabled gate in `Intellectus.swift` stops a
sample before it ever reaches a sink — but the no-op is still the correct,
safe answer if a host enables monitoring before installing a real sink.

## RecentWindowSink.swift

This file provides `RecentWindowSink`, the one concrete `StatsSink`
implementation the library ships: a fixed-size, in-memory buffer of the most
recently received samples.

The buffer is what programmers call a ring buffer, or FIFO ring (first in,
first out): a fixed-length array where new entries overwrite the oldest slot
once the array is full, so the memory used never grows past the buffer's
capacity no matter how many samples arrive over the buffer's lifetime. This
bound is a deliberate contract (documented in the file as SPEC I-8): a
component that is supposed to help observe the system must not itself become
a source of unbounded memory growth.

`RecentWindowSink` optionally wraps a second sink, called the forward sink.
Every sample the window records is, after recording, also handed to the
forward sink if one was supplied at construction. This decorator arrangement
— one sink wrapping another — lets a host keep a live, readable window of
recent activity and simultaneously persist every sample to durable storage,
using a single sink installed with `Intellectus.install(sink:)`.

`init(capacity:forward:)` builds the sink. The requested capacity is clamped
to a minimum of one: a zero or negative capacity would make the window
useless, so the initializer silently corrects it rather than crashing or
throwing, treating the input as a caller mistake rather than a fatal
condition.

`receive(_:)` is the `StatsSink` method. It takes an internal lock, writes the
sample into the ring buffer's next slot, advances the write position, and
counts the sample toward a running total, then releases the lock before
calling the forward sink. Running the forward sink outside the lock matters
for the same reason `Intellectus.report(_:)` calls its sink outside its own
lock: a forward sink that itself needs to take a lock — its own, or even this
same one, if it happened to be reinstalled recursively — cannot deadlock
against this method, because this method is not holding anything by the time
the forward sink runs.

`snapshot()` returns a copy of the buffer's current contents, oldest sample
first. Because it is a copy taken while holding the lock only long enough to
read the buffer, a caller can freely inspect or iterate the returned array
without any risk of it changing underneath them, and without blocking
further calls to `receive(_:)` for any longer than the copy itself takes.

`count` reports how many samples the buffer currently holds, which is at most
its capacity. `totalReceived` reports how many samples have ever been handed
to `receive(_:)`, including ones already evicted from the buffer. The
distinction matters for a host reading the window: a `count` of zero could
mean either "monitoring just started" or "monitoring has been off the whole
time," and `totalReceived` tells them apart — a nonzero total with an empty
buffer would be a contradiction that never actually occurs, but a zero total
is the library's explicit signal that nothing has crossed the gate yet.

## Intellectus.swift

This file provides the library's single entry point, the `Intellectus`
public enum, together with its private backing implementation,
`_IntellectusHolder`.

`Intellectus` is what the file calls a stateless namespace: an enum with no
cases, used purely to group related static functions under one name, because
Swift has no notion of free-standing module-level functions. Every piece of
mutable state — the installed sink, and the on/off flag — lives instead in a
single global instance of `_IntellectusHolder`, a class not exposed outside
the package. This split keeps the public surface simple (four static
members) while keeping the actual synchronization logic in one well-tested
place.

The on/off flag is stored as a `Synchronization.Atomic<Bool>`, a type
supplied by the platform that lets a single boolean be read and written
directly by the processor without an operating-system lock. The file
explains the reasoning in a comment: an uncontended `NSLock` still costs
roughly six nanoseconds to acquire and release on Apple Silicon, while a
single atomic read costs roughly one nanosecond and can sometimes be
optimized away entirely inside a tight loop. Since the flag is checked on
every single call to `report(_:)` — the library's hottest path by far — this
difference compounds across a busy substrate.

Reading and writing the flag use matched atomic orderings, `.acquiring` for
reads and `.releasing` for writes. These orderings are a promise to the
compiler and processor: once a read observes the flag as `true`, it is also
guaranteed to observe any other data a writer set up before flipping the flag
to `true`. Without that promise, a thread could theoretically see the flag
turn on before it saw the sink that was installed alongside it, even though
program order says the sink was installed first.

The installed sink, in contrast, is protected by an ordinary `NSLock`, because
an "any `StatsSink`" value — the type-erased container that lets any
conforming type be stored — cannot be swapped as a single atomic operation
the way a boolean can. The lock is held only long enough to copy the sink
reference out; the sink's `receive(_:)` method always runs after the lock has
already been released, matching the same outside-the-lock discipline
`RecentWindowSink.receive(_:)` uses for its own forward sink.

`_IntellectusHolder.install(sink:)` replaces the currently installed sink.
Because it is expected to run rarely — typically once, at process startup —
it can afford the ordinary lock; a report already in flight on another
thread finishes with whichever sink it captured, and the very next report
sees the new one.

`_IntellectusHolder.setEnabled(_:)` flips the atomic flag. `isEnabled` reads
it back. Both are simple one-line wrappers, but their simplicity is the
point: this is the only place the ordering guarantees above need to be
stated and reasoned about.

`_IntellectusHolder.report(_:)` is the function every other function in this
file exists to support quickly. It takes its sample-building argument as an
`@autoclosure`, a Swift feature that wraps an expression in a closure
automatically at the call site, so the expression is not evaluated as soon as
it is written — only when and if the closure is actually called. `report(_:)`
reads the atomic flag first; if it is `false`, the function returns
immediately, and the argument expression is never evaluated at all, which is
exactly the "off-path" cost the whole file is designed around: one atomic
load and a branch, nothing more. Only when the flag is `true` does the
function take the sink lock, copy the sink reference, release the lock, then
finally evaluate the sample-building expression and hand the result to the
sink.

`Intellectus.install(sink:)`, `Intellectus.setEnabled(_:)`, and
`Intellectus.isEnabled` are thin public wrappers over the matching
`_IntellectusHolder` members, forwarding to the single global instance,
`_intellectus`. `Intellectus.report(_:)` does the same for the reporting
path, and repeats the `@autoclosure` on its own parameter so the
short-circuit behavior holds all the way from a substrate call site down to
the atomic check, with no intermediate function ever forcing early
evaluation of the sample expression.

## Rust Port and Conformance

The `rust/` directory contains the second leg of the library, structured to
mirror the Swift files one for one: `sample.rs` matches `StatSample.swift`,
`sink.rs` matches `StatsSink.swift`, `window.rs` matches
`RecentWindowSink.swift`, and `holder.rs` together with `global.rs` matches
`Intellectus.swift`. `lib.rs` assembles the crate's public surface and adds
one addition the Swift side does not need: a `report!` macro, which gives
Rust callers the same short-circuit evaluation that Swift gets for free from
`@autoclosure` — the macro expands to an `if` check around the sample
expression, so the expression is only compiled into a branch that runs when
monitoring is enabled.

The two legs use different low-level tools for the same guarantees. Swift
uses `Synchronization.Atomic<Bool>` with `.acquiring`/`.releasing` orderings;
Rust uses `std::sync::atomic::AtomicBool` with `Ordering::Acquire` and
`Ordering::Release` — the same memory-ordering concept under the standard
library's own name. Swift protects the installed sink with `NSLock`; Rust
protects it with `std::sync::Mutex`, wrapping the sink in an `Arc` (Rust's
shared, reference-counted pointer) so it can be cloned out from under the
lock the same way Swift copies the "any `StatsSink`" existential out from
under its own lock. `RecentWindowSink`'s ring buffer is a fixed-size Swift
array with a manually tracked head position on the Swift side, and a
`VecDeque` — a double-ended queue supplied by Rust's standard library — on
the Rust side; both enforce the same bound and the same oldest-first eviction
rule.

The package's `Tests/IntellectusLibTests/IntellectusLibTests.swift` suite and
the Rust suite in `rust/tests/intellectus_lib_tests.rs` are organized around
matching sections — gating behavior, the no-op sink, thread safety, the
sample type, `EventKind`, a performance smoke check, and the recent-window
sink — so a change in one leg has a direct counterpart to check in the other,
even though, unlike LatticeLib, this package has no shared byte-for-byte
conformance fixtures: nothing here produces a value that two platforms need
to agree on, since a `StatSample` is opaque data the caller constructs, not a
value this library computes from an input.
