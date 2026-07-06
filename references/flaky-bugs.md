# Flaky Bugs — the "sometimes" protocol

Intermittent means nondeterministic input you haven't identified yet.
The job: find the nondeterminism source, then FORCE it.

## Step A — Measure before forcing

Run the current best repro attempt N times (default 10–20) and record
the rate. "Fails 7/20" is a baseline; changes against it tell you
whether a forcing experiment moved anything. Zero failures in 20 runs
= your repro is missing a precondition — back to the evidence sweep.

## Step B — Identify the nondeterminism source

The usual suspects, with their tells:

| Source | Tell | How to force it |
|--------|------|-----------------|
| **Concurrency / races** | fails under load, passes stepped-through in a debugger | force interleaving: sleeps/barriers at the suspect points, run handlers in controlled order, hammer with parallel requests |
| **Time / clock** | fails at midnight, month-end, DST, or "only in prod (different TZ)" | freeze/set the clock (the stack's time-mocking: Carbon::setTestNow, timecop, fake timers) to the failing instant |
| **Ordering** | depends on which row/file/test ran first | fix the order explicitly; or run permutations to find the failing order |
| **Randomness** | seeds, UUID collisions, sampled behavior | fix the seed; iterate seeds to find a failing one, then pin it |
| **External services / network** | timeouts, rate limits, slow responses | stub with forced latency/errors at the boundary |
| **Shared state / cache** | first-run passes, second fails (or inverse) | control the cache explicitly: cold vs warm as separate repro cases |
| **Test pollution** (flaky tests) | test passes alone, fails in suite | bisect the suite order to find the polluting predecessor |

## Step C — Force determinism

Apply the forcing lever for the identified source and re-measure. Success
states, in order of preference:

1. **Deterministic repro** — fails every run with the forcing recipe
   applied (frozen clock, fixed seed, forced interleaving). State the
   recipe in the dossier; it's part of the repro.
2. **Rate-based repro** — full determinism unreachable (true races
   sometimes are): deliver the recipe + measured rate ("8/10 under
   `repro/hammer.sh` with 50 parallel requests") and a loop script that
   runs until failure. Honest and actionable.
3. **Not yet reproduced** — also a valid (interim) deliverable: the
   dossier ships the evidence, eliminated hypotheses, and the monitoring
   to add (targeted logging around the suspect path) so the next
   occurrence carries the missing evidence. Say this plainly; never
   dress it up as reproduced.

## Heisenbugs

If observing changes the behavior (debugger passes, logging shifts
timing): prefer passive capture — trace-level logs already in prod,
core dumps, record-replay tooling where available — and lean on forcing
levers that don't change timing (seeds, clock) before ones that do
(sleeps).
