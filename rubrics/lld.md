# LLD rubric

Low-level design rubric for the critical-path module the candidate is asked to drill into. Modes cite as `rubrics/lld.md → <dimension>`.

Skeletal version — Session 7 expands.

## Dimensions

### interface-design
Are the public methods/endpoints minimal, well-named, and orthogonal? Are inputs and outputs typed? Are error cases explicit (return Result, throw typed exception, status codes)?

### data-model
Schema for the chosen module: tables, indexes, constraints, denormalization choices. Is the schema queryable for the access patterns? Are indexes justified? Is there a write amplification concern that's been thought about?

### error-handling
What happens when the upstream is slow, when a dependency 500s, when a write partially succeeds, when input is malformed? Does the candidate distinguish "user error" from "system error" from "transient"? Retries — bounded? Idempotent?

### concurrency
Is there shared state? Is access to it serialized correctly (lock, queue, single-writer, optimistic concurrency)? Are race conditions named and addressed, or hand-waved?

### observability
What does the candidate log, count, and trace? Can you tell from telemetry alone whether this module is healthy? Are there RED/USE-style metrics? Is there a story for debugging a single failed request?

## Citation format

```
rubrics/lld.md → <dimension>: <quote> | <judgment>
```
