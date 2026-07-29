# agent

One **bounded execution**. An agent is invoked with a request, does it, and
ends.

```
src/agent/run.cljc   AgentRun contract, state machine, event fold
```

## Where it sits

```
ao        self-evolves + self-judges   → holds git write authority, needs a lease
yakuwari  self-judges                  → no lease
agent     neither                      → bounded by the request it arrives with
```

An agent does not decide *what* to do — a yakuwari or an operator already
did. Its bound therefore arrives with the invocation instead of having to be
imposed on it, which is why this layer needs no lease and no policy of its
own: goal, budget and capabilities are all fixed before it starts.

Residency is **orthogonal**. An agent kept warm on murakumo is still an
agent; it just does not pay cold start. Being resident changes latency and
cost, never authority.

## Naming

tamaki ADR-0001 calls this an **AgentRun**, not an "Agent", precisely because
"agent" is the most overloaded word in the field. The namespace keeps the
precise name (`agent.run`, `:agent.run/*`) even though the repository uses
the short one.

## The decisions worth knowing

**Refusal is a dead end.** `:rejected` and `:cancelled` have no outgoing
transitions. `:failed` can be requeued because a failure is often retryable,
but re-deriving a run from a human's *no* would launder the refusal.

**Illegal transitions throw.** A run whose history no longer explains its
state is worse than a crash.

**A run needs a stated goal.** Without one it cannot be reviewed, cannot be
judged done, and cannot be explained to the person it acted for.

**Budgets are merged, not replaced.** A caller overriding `:max-turns` keeps
every other ceiling. A run without a ceiling is an unbounded spend against
someone's money and someone's patience.

**Non-run events never materialise as runs.** Loop, role and audit events
share the durable stream; folding must skip them rather than create nil
entries.

## Test

```sh
npm test          # nbb / JS host
clojure -M:test   # JVM host — must agree exactly
```

7 tests, 25 assertions, both hosts.

## Status

Split out of `kotoba-lang/ao` on 2026-07-29, where it had been placed by
mistake: `ao` briefly held all three layers before the axes were separated.
Originally from `kotoba.tamaki.model`. Tamaki now adopts this repository
through a compatibility adapter and supplies its persisted
`:tamaki.event/*` attribute map to `agent.run/event-keys`; existing event
stores therefore require no migration.
