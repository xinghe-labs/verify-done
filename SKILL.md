---
name: verify-done
description: Hard gate before reporting any task as done. Use when finishing an implementation, bug fix, refactor, or script — and before marking a todo/task completed. Enforces a declared file scope, the verification ladder (static check → build → targeted tests → behavioral repro → read the diff), a leftover scan, and scope-accurate reporting. "Done" without a run check is forbidden. Do not use for reviewing diff quality beyond the self-read, or for the debugging process itself.
---

# Verify Done

"Written" and "verified" are different claims. This skill is a hard gate: **you may not claim done without external evidence** — a passing check you actually ran, a successful command, a real output. "The code looks right" is evidence of nothing.

## The gate — three anchored moments

**1. Before marking any todo/task `completed`.** Run the checklist below first. If verification fails or is missing, the task stays in progress. A task with failing checks is never completed.

**2. Before your final report for a task or session.** Same checklist.

**3. Whenever you catch yourself using an escape phrase.** These phrases are not conclusions — they are signals to stop and go run something:

- "应该可以了" / "should work now" / "that should do it"
- "差不多了" / "looks good to me" / "I think it's fixed"
- "逻辑上是对的" / "logically this is correct" / "理论上没问题"

None of these may appear in a completion report. If you wrote one, the next action is a rung of the ladder, not a summary.

## Proportionality — the ladder scales with the change

The ladder is not all-or-nothing. Match it to the size of the change:

- **Single-line / trivial edit** (typo, comment, log string): static check + read the diff. That is all.
- **Small function-level change**: static check + targeted test/repro + read the diff.
- **Feature / cross-file change**: full applicable ladder.

Judged by what actually changed (lines touched, blast radius), never by how confident you feel.

## The verification ladder

Work down the ladder. Run every rung that has a usable means in this project. Each rung lists its fallback for when the standard means does not exist — a missing facility downgrades the rung, it does not remove it:

1. **Static check** — typecheck / lint on changed files.
   *No linter configured?* Run the interpreter/compiler anyway (`python -m py_compile`, `node --check`, `tsc --noEmit` if tsconfig exists). Zero-config catches syntax and name errors.
2. **Build** — compile / bundle if the project has one.
   *No build step?* Skip silently — but confirm it really has none before deciding.
3. **Targeted tests** — tests covering the changed module or the fixed bug.
   *No test infra?* Write a throwaway repro: a small script or single test file that exercises the changed code path, run it, show the output. Prefer running the user's original failing case over explaining why you can't.
4. **Behavioral check** — reproduce the original symptom and confirm it is gone.
   *Dev server project?* Start it, curl the endpoint or drive the page (Playwright if available). *CLI / script / data-processing project?* Feed a sample input, diff stdout against expected. *Visual change?* Screenshot before/after and compare, don't trust "it should look right". *Notebook?* Re-run the changed cells from a fresh kernel — passing a cell in isolation with drifted state fakes success. *No runnable path?* Say so explicitly — this rung may not be silently skipped.
5. **Read your own diff** — `git diff` before reporting. Skim the full diff early (before running the ladder, to catch obvious breakage cheaply), then read it fully, file by file, before reporting — with extra attention on code you wrote from memory.

A rung fails → fix → re-run that rung. Before any completion report, the whole applicable ladder must be green in a single final pass — so a fix for one rung cannot silently break an earlier one. Exception: a single rung measured in tens of minutes (huge build, full e2e suite) may be documented as "not re-run in the final pass (last green at …)" instead of blocking — named, never silent. Never skip, weaken, or delete a failing test to get past it.

## No side effects — verify without touching the real world

Verification must not hit real users, real money, or real data. Never verify by actually sending emails/SMS, charging payments, posting messages, deleting production rows, or calling paid third-party APIs. Use the safe means instead: dry-run flags, sandbox/staging credentials, mocks, test fixtures, a local copy of the database.

Only the real environment can prove it? Then that rung stays **unverified** — document why, and give the user a safe verification step they control ("flip `DRY_RUN=false` on staging and check the outbox table"). A green check that sent 500 real emails is a failure, not a success.

## The escape hatch — and its anti-escape clause

When a rung genuinely cannot run (no test infra, deps won't install, service won't start), you may report **written, unverified** — but only after documenting, in the report:

- **What you tried**: each command you attempted and the actual error it produced.
- **What is missing**: the specific verification that did not happen ("no test exists for `parser.ts`; repro script failed to import due to missing `foo` package").
- **How the user can verify**: the shortest concrete instruction ("run `python run.py --input sample.csv` and check the second column").

Using unverified with zero documented attempts is a violation. If a means existed and you skipped it, this label is a lie, not a caveat.

## Layering — task level vs session level

- **Per todo/task (the common case):** targeted verification only — static check on changed files + the targeted test or repro that covers the change. Fast, quiet.
- **Session end / user says 收尾 / wrap up / 检查一遍:** the full ladder including build and the whole test suite (if reasonably fast).

Do not run the full suite after every small task; do not let "task-level only" become an excuse to never run it at all.

## User override

The user may explicitly say to skip verification ("先别跑测试", "不用验证，直接交"). The user wins — comply — but the report must then say, in one line, that verification was skipped **at their request**, and name what would have been run:

> Per your request, unverified — would have run `pytest tests/test_parser.py`.

A user override is not a license to self-skip: it applies only when the user says so for this task, never by your own inference from impatience, tone, or context.

## Scope boundary — built in

Scope creep is not only adding features — it is any change the user did not ask for. Drive-by refactors, reformatting, dependency bumps, and "while I'm here" fixes break review, hide real changes, and violate the minimal-change principle.

**Before the first edit of any task**, declare scope — one sentence: which files you intend to change and why. Compare against the user's request: files the user named or that the request directly implies → in scope; anything else → ask first. This gives the leftover scan below its baseline and catches most creep before it happens.

**Green light — change freely, but list in the report:**

- The files and code paths the request directly implies.
- **Mechanical consequences** of an in-scope change: imports updated by a moved symbol, call sites fixed by a renamed function, types adjusted by a changed signature, the one config line the feature requires.

**Stop and ask — hard triggers.** Pause and get explicit approval when the change would:

- Add, remove, or upgrade a dependency (package.json, requirements.txt, go.mod, …)
- Change a public API, exported signature, or shared interface beyond the request
- Touch database schema, migrations, or seed data
- Touch config / CI / build files not required by the request (`.github/`, `Dockerfile`, tsconfig flags, bundler config)
- Reformat or restyle code the request did not cover (prettier sweeping untouched files, quote style, import reordering)
- Refactor "while you're here" — renaming, extracting, reorganizing anything that currently works
- Fix an unrelated bug you happened to notice — report it instead ("BTW, line X looks wrong") and let the user decide

"Obviously better" is not approval. The user may have reasons: generated files, in-flight work, an intentionally frozen area.

**Staying inside the boundary:**

- One task, one intent: never bundle an unrelated fix into the same edit series.
- If a bigger problem blocks the requested change, stop and report the blocker — do not solve around it silently.
- When the request is ambiguous ("clean this up"), confirm what *clean* means and which files are fair game **before** editing, not after.

When anything outside the declared scope was touched, the completion report ends with an explicit statement:

> Also touched (mechanical consequences): `a.ts` (import moved), `b.test.ts` (updated call signature). Nothing else outside the requested files.

## Leftover scan

Before any completion report:

- Debug output you added this session: `console.log`, `print(`, `dbg!`, ad-hoc logging
- Commented-out code and `TODO`/`FIXME` markers you introduced
- Temp/scratch files from this session, including backup copies like `foo.ts.bak`
- `git status` — compare against **the scope you declared before editing**, not against a clean tree. Any file changed beyond that set needs an explicit explanation in the report. The user's pre-existing dirty tree is not yours to fix or flag.
- **No git repo?** Your declared scope *is* the change record: re-read every file you edited end to end, list any file you created this session in the report, and be extra conservative — there is no diff to catch what you forgot.

## Reporting — scope-accurate, not just pass/fail

"Verified" alone is not enough — a claim must carry its scope. Never say "tests pass" when you ran 3 of 40; never say "unverified" when you ran the covering test.

Format for a verified claim:

> Done. Verified: `pytest tests/test_parser.py -k lexer` (4 passed), `tsc --noEmit` clean. **Not** run: full suite, build.

Three states, each with scope:

| State | Requires |
|---|---|
| Verified | Named the commands run + their result + what was *not* covered |
| Written, unverified | Attempts documented + missing verification named + user verification path given |
| Partially done | What works, what doesn't, what's next — never imply more |

Report what you ran and what it printed, not just the conclusion. "Tests pass" without having run them is a lie — even if they would have passed.

## Anti-patterns

- Claiming completion from reading code only
- Marking a todo `completed` while any check is red or unrun
- Running one test and generalizing to "the suite passes"
- Silencing an error (empty catch, ignored promise, `# type: ignore`) to make a check green
- Deleting or skipping a failing test instead of fixing the failure
- "It should work now" — *should* is not evidence
- Using "unverified" as a shortcut to skip verification entirely
