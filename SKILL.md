---
name: reproduce-my-bug
description: >-
  Turns a vague bug report into a minimal, runnable reproduction BEFORE any
  fix is attempted: sweeps evidence (logs, stack traces, git history around
  first-seen, DB state), ranks hypotheses by evidence, builds the smallest
  failing automated test (or repro script) that demonstrates the bug on
  demand, minimizes it until every remaining step is load-bearing, handles
  intermittent/flaky bugs by forcing determinism and documenting the
  reproduction rate, and delivers a repro dossier that hands the fixer a
  ready-made regression test. Never fixes — reproduces. Use when the user
  reports a bug, says something breaks sometimes or only in production,
  asks to investigate, debug, or find the root cause of an issue, or
  mentions reproduce-my-bug or /reproduce-my-bug.
license: MIT
argument-hint: "[bug description] [project-root]"
---

# Reproduce My Bug

The most expensive words in debugging are "I think I fixed it." A fix
without a reproduction is a guess with a commit message — nobody can prove
it worked, and nothing stops the bug from coming back. This skill does the
half everyone skips: it turns "it breaks sometimes" into a **failing test
you can run on demand** — which makes the eventual fix provable and gives
the repo a permanent regression guard for free.

## The Prime Directive (family rule)

> **No fix without a failing reproduction.** This skill never patches the
> bug — it proves the bug. If the bug "disappears" during investigation,
> that is a finding (flaky, environment-dependent, heisenbug), not a
> resolution. Every claim in the dossier carries evidence: `file:line`,
> a log excerpt, a command output, or an explicit `UNVERIFIED` tag.

## Hard rules

1. **Read-only on app code.** The only writes are the repro artifacts
   (a test file, a repro script, seed data) in clearly marked locations.
2. **Never investigate against production.** Reproduce on local/dev with
   seeded data (the seed-ah skill is the companion for this). Production
   supplies evidence (logs, traces) — never the test bench.
3. **One variable at a time.** Change one thing per attempt and record the
   outcome — including failures to reproduce; an eliminated hypothesis is
   progress, and unrecorded attempts get repeated.
4. **The repro must fail.** Green is not a repro. The dossier includes the
   exact command and the exact failing output, verbatim.

## Progress checklist

Copy this into your response and check items off:

```
Repro Progress:
- [ ] Step 1: Intake — symptom, expected vs actual, environment, frequency, first-seen
- [ ] Step 2: Evidence sweep — logs, traces, git history, data state; hypotheses ranked
- [ ] Step 3: Path trace — code path from symptom backward, file:line map
- [ ] Step 4: Reproduce — failing test or script, one variable at a time
- [ ] Step 5: Minimize — every remaining step load-bearing
- [ ] Step 6: (flaky only) determinism forced or reproduction rate documented
- [ ] Step 7: Dossier delivered — REPRO.md + the failing artifact, handoff stated
```

## Step 1 — Intake

Pin the report down before touching anything:

- **Symptom verbatim** (quote the user/report — don't paraphrase away the
  clue), expected vs actual behavior.
- Environment: version/commit, browser/OS/runtime, user role, locale.
- **Frequency:** always / sometimes / once. "Sometimes" routes Step 6.
- **First seen:** date/version — this anchors the git sweep.
- What already changed since (deploys, config, data migrations)?

Missing answers are fine — mark them `UNKNOWN` and let evidence fill them;
never invent them.

## Step 2 — Evidence sweep

Collect before theorizing — method in
[references/evidence-sweep.md](references/evidence-sweep.md): error logs
and stack traces (exact text), git history around first-seen (what
touched the suspect area — `git log`, and `git bisect` when a
known-good commit exists), the data state that triggers it, config/env
diffs between working and broken environments.

Output: a **ranked hypothesis list** — ranked by what the evidence
supports, not by what training data says usually breaks. Each hypothesis
names the evidence for it and what observation would kill it.

## Step 3 — Path trace

From the symptom backward: the error's `file:line` → what calls it → where
the offending value/state originates. Map the suspect path with citations
before building anything — a repro built on the wrong path reproduces the
wrong bug.

## Step 4 — Reproduce

Build the smallest thing that demonstrates the bug, in this order of
preference (details in
[references/repro-harness.md](references/repro-harness.md)):

1. **A failing automated test** in the repo's own test framework — the
   gold standard: it becomes the regression test the moment the fix lands.
2. **A repro script** (curl sequence, CLI invocation, SQL + request) when
   the bug spans processes or a test harness doesn't reach it.
3. **Documented manual steps** with exact data — last resort, only when
   automation is genuinely impossible, and say why.

Work hypothesis by hypothesis, one variable at a time, recording every
attempt in the dossier's log — reproduced or not.

## Step 5 — Minimize

Shrink until every remaining element is load-bearing: drop steps, shrink
data, remove setup — if removing X still reproduces, X was noise. The
minimal repro is the root cause's silhouette; it often makes the cause
obvious without further investigation.

## Step 6 — Flaky bugs (the "sometimes" path)

Intermittent doesn't mean irreproducible — protocol in
[references/flaky-bugs.md](references/flaky-bugs.md): identify the
nondeterminism source (concurrency, time/clock, ordering, randomness,
network, cache state) and **force it** (fixed seeds, frozen clock, forced
interleaving, repeated runs). If full determinism isn't reachable, a
documented **reproduction rate** (e.g. "fails 8/10 runs with this
recipe") plus the forcing recipe is an acceptable deliverable — say so
plainly.

## Step 7 — The dossier

Write `REPRO.md` (or `docs/repro/<slug>.md`) per
[references/dossier-template.md](references/dossier-template.md): the
symptom, **how to run the repro and the exact failing output**, the
evidence timeline, eliminated hypotheses (with what killed each),
suspected root cause tagged `VERIFIED`/`UNVERIFIED` — root cause is a
bonus; **the repro is the deliverable** — and the handoff: the failing
test is the fix's acceptance criterion and future regression guard. For
fixes that need real design, recommend deep-plan with this dossier as
input.

Report inline: repro status (deterministic / rate-based / not yet),
the one command to run it, and the top suspect with its evidence.
