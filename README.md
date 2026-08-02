# What is it, and what does it do?

A small service status monitor. Polls a list of HTTP endpoints on a schedule,

classifies each as Up / Degraded / Down, and reports p95 latency.

## Example

-coming soon

## Configuration

-coming soon

## Running

    -coming soon

## Design notes

- **Stateless checks, single evaluator.** Endpoint checks hold no state and

  return results; one evaluator applies the health rules afterward. This means

  parallel checks never write to shared memory, and the health logic is testable

  without any HTTP.

- **Concurrency.** The first version fired every check simultaneously with

  `Task.WhenAll`. Under load - it became a learning experience... Concurrency is

  now capped with a `SemaphoreSlim` and failures retry with exponential backoff

  plus jitter.

- **Flap protection.** An endpoint must fail N consecutive checks before being

  reported Down, and only state transitions are announced. A single timeout is

  not an outage.

- **p95 over average**, computed over a fixed window of the last 100 checks.

  Averages hide tail latency, and an unbounded window both leaks memory and

  gets less meaningful over time.

- **Local network detection.** If every endpoint fails at once, that's reported

  as a probable local issue rather than a global outage.

- **Retry is hand-rolled** to understand the mechanics. In production I would

  use `Microsoft.Extensions.Http.Resilience`.

## Non-goals

Deliberately out of scope: authentication, multi-region probing, alerting

integrations, a web dashboard, and distributed coordination. This is a

single-node monitor for a handful of endpoints.
