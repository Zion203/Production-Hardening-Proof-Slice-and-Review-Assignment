# Starter — Circuit Breaker Proof Slice (PromptPilot)

This is the starter repository for **Option B (provided scenario)** of the assignment.

Repository: `https://github.com/kalviumcommunity/Production-Hardening-Proof-Slice-and-Review-Assignment` (public, owned by
`kalviumcommunity`). Fork it, implement the breaker, and open a pull request back to your fork.

> The control here is a **circuit breaker**, covered earlier in this module. You are applying a
> concept you have already learned, not a new one.

## Scenario

PromptPilot drafts support replies by calling an external model provider. When the provider is slow
or failing, every request still tries it, so doomed calls pile up and the failure spreads. You will
add a **circuit breaker** so sustained failures short-circuit fast to a safe fallback, then recover
through a probe.

## Your task

Implement the circuit breaker in [`src/circuitBreaker.js`](./src/circuitBreaker.js). The file has a
skeleton and a `TODO`. The behaviour is fully specified by the supplied tests.

Prerequisite: Node.js 18 or newer (the tests use Node's built-in test runner). Then run:

```bash
node --test
```

Make **all tests pass**. Do **not** modify the tests. The reply path in `src/replyService.js` is
already wired to your breaker.

The breaker must:

- start `CLOSED` and count calls and failures;
- move to `OPEN` when `failures / calls >= failureThreshold` **and** `calls >= minimumRequests`;
- while `OPEN` and within `openMillis`, short-circuit (throw `CircuitOpenError` without calling the
  dependency) and increment `metrics.short_circuited_total`;
- after `openMillis` elapses (using the injected `now()`), allow one `HALF_OPEN` probe — success
  closes it and resets counts, failure re-opens it;
- increment `metrics.breaker_open_total` on every trip and call `onStateChange(state)` on every
  transition.

## What to submit

Open a pull request on a branch named `production-hardening-slice`, and in your README include these
labelled sections:

1. **Risk addressed**
2. **Chosen control**
3. **How to run / test**
4. **Evidence** (a reviewer can confirm it works without reading all your code)
5. **Observability note** (which metric moves and which state changes are logged)
6. **Trade-off**
7. **Remaining risk**
8. **How this production control protects my Modules 1–3 design**

Submit the pull-request URL.

## Risk addressed

Prevents repeated provider failures from piling up and spreading latency or errors through PromptPilot.

## Chosen control

A circuit breaker in `src/circuitBreaker.js` opens after the configured failure ratio, probes after a cooldown, and closes on recovery.

## How to run / test

Run `node --test`; the expected result is all supplied circuit-breaker tests passing.

## Evidence

`test/circuitBreaker.test.js` proves threshold opening, dependency-free short-circuiting, recovery probes, and failed-probe reopening.

## Observability note

`metrics.breaker_open_total` and `metrics.short_circuited_total` count protective actions; `onStateChange(state)` reports `OPEN`, `HALF_OPEN`, and `CLOSED` transitions.

## Trade-off

During the cooldown, requests receive the safe fallback instead of attempting the provider, which may delay a recoverable draft.

## Remaining risk

The breaker is process-local, so multiple service instances do not share failure state or a common recovery budget.

## How this production control protects my Modules 1–3 design

It protects PromptPilot's support-reply path by keeping the user-facing workflow available when its external model dependency is unhealthy.
