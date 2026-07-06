# Evidence Sweep Guide

Collect first, theorize second. A hypothesis formed before the sweep
selects its own evidence; a hypothesis formed after the sweep is selected
BY the evidence.

## Where to look, in order of signal density

1. **The exact error** — full stack trace / log line, quoted verbatim in
   the dossier. The exact wording matters: "connection refused" and
   "connection reset" are different bugs. Grep the codebase for the
   message to find the throwing site (`file:line`).
2. **Logs around the incident** — not just the error line: the 30 seconds
   before it. What request, what user, what data, what else was running?
   Note what is ABSENT too (an expected log line missing is a clue).
3. **Git history around first-seen** —
   `git log --oneline --since=<first-seen minus a few days> -- <suspect paths>`:
   what changed right before it started? Deploys, dependency bumps
   (lockfile diffs!), config changes, migrations.
   **`git bisect`** when a known-good commit exists and the repro is
   cheap — it converts history into a binary search and often ends the
   investigation outright.
4. **The data** — bugs that "only happen to some users" are usually data:
   find the triggering row/document and diff it against a working one
   (null where code assumes value, unexpected enum, boundary length,
   unicode, timezone).
5. **Environment diffs** — works-on-my-machine = diff the machines:
   versions (runtime, dependencies via lockfile), config/env vars,
   locale/timezone, data volume.

## Building the hypothesis list

For each candidate cause:

| Hypothesis | Evidence FOR | What would kill it | Cost to test |
|------------|--------------|--------------------|--------------|
| Null `discount` on legacy rows reaches `calc()` unchecked | log shows NPE at Calc.php:88; migration 2026_05 backfilled only active rows | a failing row with non-null discount | low — query + unit test |

Ranking rules:

- Evidence-supported beats plausible-from-memory. "This framework
  usually…" is a hypothesis generator, never a ranking criterion.
- Prefer cheap-to-kill hypotheses first when ranks are close —
  elimination is progress.
- **Correlation trap:** "started after deploy X" makes deploy X a
  suspect, not a culprit — the deploy may have merely increased traffic
  or shifted timing on an older bug.

## Recording

Every sweep artifact goes into the dossier's evidence section with a
citation (log path + timestamp, commit hash, query). Investigations
without a written trail loop: you will re-check the same log twice and
re-suspect the same innocent commit.
