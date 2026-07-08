# Repro Harness Guide

Building the smallest thing that fails on demand.

## Form 1 — Failing test (gold standard)

Use the repo's own framework and conventions (find an existing test as
the style template, `file:line`). Name it after the bug, not the fix:
`test_order_total_negative_when_discount_exceeds_price`, and reference
the issue/report in a comment.

Rules:

- The test asserts the CORRECT behavior — so it fails now (red) and
  passes when fixed. Never assert the buggy behavior to make it green.
- Run it and capture the failing output verbatim into the dossier.
- Keep it at the lowest layer that still exhibits the bug: unit if the
  broken logic is pure, integration if it's wiring, E2E only if it's
  genuinely emergent — lower layers run faster and flake less.
- Put it where the repo's tests live; mark it clearly
  (`@group repro` / tag / comment) so the fixer finds it.

## Form 2 — Repro script

When the bug spans processes (HTTP + queue + DB) or the test harness
can't reach it: a committed script (`repro/<slug>.sh` or `.php`/`.py`)
that seeds the minimal state, performs the exact actions (curl calls,
CLI commands), and **checks the outcome itself** — exits non-zero with a
message on bug-present, so it's still "runnable and fails on demand".

## Form 3 — Manual steps (last resort)

Numbered steps with exact data values, plus WHY automation wasn't
possible (device-only, third-party sandbox, visual-only symptom). Even
then: capture the state programmatically where possible so a future
harness can be built.

## The one-variable loop

Attempt #0 is always the **runnability spike**: prove the suspect path
executes locally (run the module's existing tests, hit the endpoint
once) before building anything on it. A path that can't run locally is
the first blocker to report, not a silent detour.

```
pick top hypothesis → design smallest attempt that isolates it
→ run → record (reproduced? exact output?)
→ reproduced: minimize (below)   |   not: record, kill/demote hypothesis, next
```

Record failed attempts in the dossier log — they are eliminations, and
an unrecorded attempt WILL be repeated.

## Minimization

From a working repro, remove one element at a time and re-run:

- Steps: drop each; if it still fails, the step was noise.
- Data: shrink values, drop fields, reduce rows toward the boundary —
  often revealing THE triggering property (null, length 256, 0, DST
  timestamp).
- Environment: drop env vars/config until only the load-bearing ones
  remain.

Stop when removing ANY remaining element makes it pass. That set is the
bug's true precondition list — paste it into the dossier as "triggering
conditions", each citing the attempt # whose removal-then-pass run
proved it load-bearing. Minimal is a measured property, not an
impression: an element with no removal attempt on record is not proven
load-bearing. Delta debugging in spirit; discipline over tooling.

## Traps

- **Fixing while reproducing:** you notice the likely cause and itch to
  patch it. Don't — note it as the suspect and finish the repro. The fix
  without the red test is exactly the guess this skill exists to prevent.
- **Repro that tests the mock:** if the failing layer is mocked, the test
  reproduces your mock, not the bug. Reproduce against the real component
  at least once.
- **Environment drift:** a repro that only fails on your machine is data —
  diff the environments (see evidence-sweep) rather than shipping a
  "works for me".
