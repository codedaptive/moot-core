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
flows through the library: the sample type, the receiver contract, and the
one bundled receiver. The global facade comes last. It ties everything
together for callers.

## StatSample.swift

This file provides `StatSample`, the telemetry datum that flows through the
whole library. It also provides `EventKind`, a small enum used by one of
`StatSample`'s two cases.

`StatSample` is a value type. It is an enum with associated values, not a
class. It conforms to `Sendable`. That means the Swift compiler checks that
it is safe to pass between threads or concurrent tasks without extra
synchronization. Value types copy instead of sharing memory. That copying is
what makes the safety check possible, since no two threads can then race on
the same storage.

The enum has two cases. `.metric` carries a dot-separated name, such as
`"locus.capture.latency_ms"`. It also carries a floating-point value, an
optional dictionary of string tags for context, and a timestamp. An example
tag dictionary looks like `["kit": "LocusKit"]`. `.event` carries a
lifecycle event instead. It carries an `EventKind`, either `.capture` or
`.think`. It also carries the integer form of a substrate row's noun type,
the row's identifier string, the estate identifier string, and a timestamp.
`EventKind` distinguishes two broad classes of activity. `.capture` marks a
caller-driven write. `.think` marks a substrate-driven autonomous
transition. Both classes can be triggered by the substrate's Brain layer.

Every timestamp in the library is a caller-supplied number of seconds since
the epoch. No timestamp is ever read from the system clock inside this
package. This matters for determinism. A caller that replays the same
sequence of events gets the same timestamps back. Tests can construct
samples with fixed, predictable times instead of racing against the real
clock.

`ts` is a computed property added in an extension. It returns the timestamp
regardless of which case the sample is. This spares every caller that only
cares about ordering samples in time. Such a caller need not write a
`switch` over both cases.

## StatsSink.swift

This file provides `StatsSink`, the protocol every receiver of telemetry
samples must conform to. It also provides `NoOpSink`, the receiver installed
before any host supplies its own.

A protocol in Swift is a contract. It lists the methods and properties a
conforming type must supply, without dictating how. `StatsSink` requires
exactly one method, `receive(_:)`. It also requires conformers to be
`Sendable`, because the one sink installed at any moment lives in global
state. That sink is called from whatever thread happens to be reporting a
sample at the time.

`receive(_:)` delivers one `StatSample` to the sink. The file documents an
expectation rather than an enforced rule. Implementations should return
quickly. They should not block. Anything slow, such as writing to a database
or sending data over a network, belongs in the sink's own background work.
That background work should be scheduled from inside `receive(_:)`, not run
synchronously inside it. A slow sink would otherwise add latency to whatever
substrate code path triggered the sample.

`NoOpSink` is the simplest possible conformer. Its `receive(_:)` does
nothing and returns immediately. It exists so the global holder in
`Intellectus.swift` always has a valid sink installed. This holds even
before a host calls `install(sink:)`. Because monitoring is off by default,
`NoOpSink.receive(_:)` is not expected to run in ordinary use. The enabled
gate in `Intellectus.swift` stops a sample before it ever reaches a sink.
Still, the no-op is the correct, safe answer if a host enables monitoring
before installing a real sink.

## RecentWindowSink.swift

This file provides `RecentWindowSink`, the one concrete `StatsSink`
implementation the library ships. It is a fixed-size, in-memory buffer of
the most recently received samples.

The buffer is what programmers call a ring buffer, or FIFO ring. FIFO means
first in, first out. It is a fixed-length array. New entries overwrite the
oldest slot once the array is full. So the memory used never grows past the
buffer's capacity, no matter how many samples arrive over the buffer's
lifetime. This bound is a deliberate contract, documented in the file as
SPEC I-8. A component that helps observe the system must not itself become a
source of unbounded memory growth.

`RecentWindowSink` optionally wraps a second sink, called the forward sink.
Every sample the window records is, after recording, also handed to the
forward sink if one was supplied at construction. This is a decorator
arrangement, one sink wrapping another. It lets a host keep a live, readable
window of recent activity. At the same time, the host can persist every
sample to durable storage. Both effects come from a single sink installed
with `Intellectus.install(sink:)`.

`init(capacity:forward:)` builds the sink. The requested capacity is clamped
to a minimum of one. A zero or negative capacity would make the window
useless. So the initializer silently corrects it rather than crashing or
throwing. It treats the input as a caller mistake, not a fatal condition.

`receive(_:)` is the `StatsSink` method. It takes an internal lock. It writes
the sample into the ring buffer's next slot. It advances the write position.
It counts the sample toward a running total. Then it releases the lock
before calling the forward sink. Running the forward sink outside the lock
matters for the same reason `Intellectus.report(_:)` calls its sink outside
its own lock. A forward sink might itself need to take a lock, its own or
even this same one if reinstalled recursively. That sink cannot deadlock
against this method, because this method holds nothing by the time the
forward sink runs.

`snapshot()` returns a copy of the buffer's current contents, oldest sample
first. It is a copy taken while holding the lock only long enough to read
the buffer. So a caller can freely inspect or iterate the returned array. No
risk exists of it changing underneath them. No call blocks further calls to
`receive(_:)` for any longer than the copy itself takes.

`count` reports how many samples the buffer currently holds. This is at most
its capacity. `totalReceived` reports how many samples have ever been handed
to `receive(_:)`, including ones already evicted from the buffer. The
distinction matters for a host reading the window. A `count` of zero could
mean monitoring just started. Or it could mean monitoring has been off the
whole time. `totalReceived` tells the two apart. A nonzero total with an
empty buffer would be a contradiction that never actually occurs. A zero
total is the library's explicit signal that nothing has crossed the gate
yet.

## Intellectus.swift

This file provides the library's single entry point, the `Intellectus`
public enum. It also provides its private backing implementation,
`_IntellectusHolder`.

`Intellectus` is what the file calls a stateless namespace. It is an enum
with no cases, used purely to group related static functions under one
name, because Swift has no notion of free-standing module-level functions.
Every piece of mutable state lives instead in a single global instance of
`_IntellectusHolder`, a class not exposed outside the package. That state
includes the installed sink and the on/off flag. This split keeps the
public surface simple, at four static members. It also keeps the actual
synchronization logic in one well-tested place.

The on/off flag is stored as a `Synchronization.Atomic<Bool>`. This type is
supplied by the platform. It lets a single boolean be read and written
directly by the processor, without an operating-system lock. The file
explains the reasoning in a comment. An uncontended `NSLock` still costs
roughly six nanoseconds to acquire and release on Apple Silicon. A single
atomic read costs roughly one nanosecond. It can sometimes be optimized away
entirely inside a tight loop. Since the flag is checked on every single call
to `report(_:)`, the library's hottest path by far, this difference compounds
across a busy substrate.

Reading and writing the flag use matched atomic orderings. Reads use
`.acquiring`. Writes use `.releasing`. These orderings are a promise to the
compiler and processor. Once a read observes the flag as `true`, it is also
guaranteed to observe any other data a writer set up before flipping the
flag to `true`. Without that promise, a thread could theoretically see the
flag turn on before it saw the sink installed alongside it. This could
happen even though program order says the sink was installed first.

The installed sink, by contrast, is protected by an ordinary `NSLock`. An
"any `StatsSink`" value is a type-erased container that lets any conforming
type be stored. It cannot be swapped as a single atomic operation the way a
boolean can. The lock is held only long enough to copy the sink reference
out. The sink's `receive(_:)` method always runs after the lock has already
been released. This matches the same outside-the-lock discipline
`RecentWindowSink.receive(_:)` uses for its own forward sink.

`_IntellectusHolder.install(sink:)` replaces the currently installed sink.
It is expected to run rarely, typically once, at process startup. So it can
afford the ordinary lock. A report already in flight on another thread
finishes with whichever sink it captured. The very next report sees the new
one.

`_IntellectusHolder.setEnabled(_:)` flips the atomic flag. `isEnabled` reads
it back. Both are simple one-line wrappers. Their simplicity is the point,
though: this is the only place the ordering guarantees above need to be
stated and reasoned about.

`_IntellectusHolder.report(_:)` is the function every other function in this
file exists to support quickly. It takes its sample-building argument as an
`@autoclosure`. This Swift feature wraps an expression in a closure
automatically at the call site. So the expression is not evaluated as soon
as it is written, only when and if the closure is actually called.
`report(_:)` reads the atomic flag first. If it is `false`, the function
returns immediately. The argument expression is never evaluated at all. This
is exactly the off-path cost the whole file is designed around: one atomic
load and a branch, nothing more. Only when the flag is `true` does the
function take the sink lock. It copies the sink reference, releases the
lock, then finally evaluates the sample-building expression and hands the
result to the sink.

`Intellectus.install(sink:)`, `Intellectus.setEnabled(_:)`, and
`Intellectus.isEnabled` are thin public wrappers. Each forwards to the
matching `_IntellectusHolder` member, on the single global instance,
`_intellectus`. `Intellectus.report(_:)` does the same for the reporting
path. It repeats the `@autoclosure` on its own parameter. This lets the
short-circuit behavior hold all the way from a substrate call site down to
the atomic check. No intermediate function ever forces early evaluation of
the sample expression.

## Rust Port and Conformance

The `rust/` directory contains the second leg of the library. It is
structured to mirror the Swift files one for one. `sample.rs` matches
`StatSample.swift`. `sink.rs` matches `StatsSink.swift`. `window.rs` matches
`RecentWindowSink.swift`. `holder.rs`, together with `global.rs`, matches
`Intellectus.swift`. `lib.rs` assembles the crate's public surface. It adds
one addition the Swift side does not need: a `report!` macro. This macro
gives Rust callers the same short-circuit evaluation that Swift gets for
free from `@autoclosure`. The macro expands to an `if` check around the
sample expression. So the expression compiles into a branch that only runs
when monitoring is enabled.

The two legs use different low-level tools for the same guarantees. Swift
uses `Synchronization.Atomic<Bool>` with `.acquiring` and `.releasing`
orderings. Rust uses `std::sync::atomic::AtomicBool` with `Ordering::Acquire`
and `Ordering::Release`, the same memory-ordering concept under the standard
library's own name. Swift protects the installed sink with `NSLock`. Rust
protects it with `std::sync::Mutex`, wrapping the sink in an `Arc`. That is
Rust's shared, reference-counted pointer. It lets the sink be cloned out
from under the lock, the same way Swift copies the "any `StatsSink`"
existential out from under its own lock. `RecentWindowSink`'s ring buffer is
a fixed-size Swift array on the Swift side, with a manually tracked head
position. On the Rust side, it is a `VecDeque`, a double-ended queue
supplied by Rust's standard library. Both sides enforce the same bound and
the same oldest-first eviction rule.

The package's `Tests/IntellectusLibTests/IntellectusLibTests.swift` suite
and the Rust suite in `rust/tests/intellectus_lib_tests.rs` are organized
around matching sections. Those sections cover gating behavior, the no-op
sink, thread safety, the sample type, `EventKind`, a performance smoke
check, and the recent-window sink. So a change in one leg has a direct
counterpart to check in the other. This differs from LatticeLib in one way,
though: this package has no shared byte-for-byte conformance fixtures.
Nothing here produces a value that two platforms need to agree on, since a
`StatSample` is opaque data the caller constructs. It is not a value this
library computes from an input.
