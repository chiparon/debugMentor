# Backend debugger playbook

Use only the section that matches the learner's stack. Exact labels vary by IDE version; describe the underlying action as well as the UI label.

## Contents

- Universal debugger controls
- Java and Kotlin
- Python
- JavaScript and TypeScript on Node.js
- C# and .NET
- Go
- Rust
- Backend observation boundaries
- When a debugger is the wrong tool

## Universal debugger controls

- **Line breakpoint:** pause before an executable line runs.
- **Conditional breakpoint:** pause only when an expression is true. Keep expressions side-effect free.
- **Logpoint or tracepoint:** record a message without intentionally pausing.
- **Exception breakpoint:** pause when an exception is thrown or becomes unhandled.
- **Data breakpoint or watchpoint:** pause when a memory location or supported field changes.
- **Step over:** execute the current line without entering its calls.
- **Step into:** enter the specific call on the current line; use smart step-into when several calls share a line.
- **Step out:** finish the current function and return to its caller.
- **Call stack:** show the chain of callers. Select an older frame to inspect the values that led to the current call.
- **Watch or evaluate:** inspect a targeted expression. Avoid evaluation that mutates state or performs I/O.

Check these first when a breakpoint is hollow, unbound, or never reached: correct process, correct build, loaded source version, debug symbols or source maps, route and input, child process, container or remote mapping, and breakpoint condition.

## Java and Kotlin

Typical tools: IntelliJ IDEA, Android Studio for applicable services, or a Java debugger client in another IDE.

- Pause at controller or handler entry, service boundaries, repository calls, and exception throw sites.
- Use an exception breakpoint for a specific exception type when ordinary line breakpoints stop too late.
- Inspect both the current frame and the first application caller. Framework and proxy frames often surround the useful frame.
- For Kotlin, inspect nullable values, generated coroutine state, extension receivers, and the actual runtime type behind interfaces.
- Be alert to lazy ORM proxies, transaction boundaries, and expressions whose evaluation can trigger database access.
- Remote JVM debugging commonly uses JDWP. Bind it only to a trusted interface or tunnel and never expose an unauthenticated debug port publicly.

Useful checks: request DTO after binding, validated values, security principal, transaction active state, SQL parameters and returned row count, and response DTO before serialization.

## Python

Typical tools: PyCharm, VS Code with a Python debugger, or built-in `breakpoint()` for a local interactive run.

- Enable pause on raised or uncaught exceptions according to the question being tested.
- Inspect locals, closure values, instance attributes, and the selected caller frame.
- Confirm whether the server reloads into a child process; attach to or launch the process that actually handles the request.
- For async code, track the request or task and distinguish an awaited result from a coroutine object.
- Avoid evaluating properties or representations that perform database queries or other I/O.
- Native extensions, optimized code, multiprocessing, greenlets, and framework reloaders may require stack-specific debugger configuration.

Useful checks: parsed request data, dependency injection output, ORM session and transaction state, query filters, returned model versus schema, and exception chaining (`__cause__` and `__context__`).

## JavaScript and TypeScript on Node.js

Typical tools: VS Code or a client attached to Node's inspector.

- Ensure source maps map the running JavaScript artifact back to the TypeScript source.
- Confirm the correct worker or child process; a development watcher may restart it after attachment.
- Inspect `undefined` versus `null`, string versus number IDs, promise state, closure values, and the first relevant async caller.
- Pause on caught or uncaught exceptions selectively; libraries may intentionally throw and catch many exceptions.
- Use `--inspect` or `--inspect-brk` only in a trusted development environment. Do not expose the inspector port publicly.

Useful checks: middleware-mutated request fields, route parameters, validation output, awaited service result, database query arguments, event handler registration, and serialized response.

## C# and .NET

Typical tools: Visual Studio, Rider, or VS Code with a compatible .NET debugger.

- Configure exception settings to break when the relevant exception is thrown, not only when it becomes user-unhandled.
- Inspect autos, locals, watches, call stack, tasks, threads, and parallel stacks when applicable.
- Check nullable values, boxed runtime types, deferred LINQ execution, async continuations, and cancellation tokens.
- “Just My Code” and optimized Release builds can hide or rearrange frames and locals; record any setting changed to investigate.
- Property evaluation may execute code. Disable or avoid it when it changes timing or performs I/O.

Useful checks: model binding and validation state, claims principal, dependency scope, EF query inputs and materialized results, transaction state, cancellation, and response mapping.

## Go

Typical tool: Delve, directly or through an IDE.

- Prefer a binary built with usable debug information. If optimization obscures stepping or variables, use an appropriate local debug build rather than changing production binaries.
- Inspect goroutines and select the one serving the failing request.
- Check interface dynamic types, nil interfaces versus typed nil pointers, error values, slice length and capacity, map membership, and context cancellation.
- Stepping through goroutine scheduling can alter timing. Use targeted logging, traces, or race detection for race symptoms.
- Data breakpoints depend on architecture and debugger support and may be limited.

Useful checks: decoded request, context deadline, error wrapping chain, repository arguments, rows iteration and `rows.Err()`, shared map access, and response encoding.

## Rust

Typical tools: CodeLLDB, LLDB, or GDB with Rust-aware IDE integration.

- Use a debug build with symbols when practical; optimized code may inline functions or make variables unavailable.
- Inspect `Option` and `Result` variants, borrowed versus owned values, trait-object runtime behavior, and error source chains.
- For panics, pause at the panic path and move to the first application frame. A backtrace often gives a faster initial boundary.
- Async state machines can produce unfamiliar frames; follow the task and identify the application future or handler frame.
- Evaluate expressions cautiously because debugger support for Rust expressions varies.

Useful checks: deserialized extractor values, match-arm selection, error conversion, database result variants, lock guards, channel operations, and serialization.

## Backend observation boundaries

Choose two boundaries around the suspected divergence and bisect further:

1. transport receives bytes
2. routing selects a handler
3. middleware, authentication, or authorization transforms context
4. decoding and validation produce application input
5. controller or handler calls the domain service
6. domain logic chooses a branch or mutates state
7. repository constructs a query and receives results
8. external client sends a request and receives a response
9. transaction commits or rolls back
10. cache reads, writes, or invalidates
11. job or event is published and consumed
12. response model is mapped and serialized

At every boundary, compare identity, value, type, count, order, time representation, error state, and correlation ID as relevant.

## When a debugger is the wrong tool

Prefer other evidence when pausing changes behavior, the failing process cannot safely stop, or the failure spans services:

- structured logs for discrete values and branch outcomes
- distributed traces for call order, cross-service context, and latency
- metrics and profiles for frequency, resource use, or performance distributions
- record-and-replay or deterministic tests for timing-sensitive logic
- race detectors and concurrency analyzers for shared-memory races
- database query logs or execution plans for query correctness and performance

Instrument the smallest boundary that can decide a hypothesis. Include correlation data, redact sensitive fields, and remove temporary high-volume probes after use.
