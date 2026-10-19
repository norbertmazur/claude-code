# Person A — Structure & static quality (draft)

Project Quality section, team 9, 2IRR80. Owner: Person A. Scope from `script.md`:
Maintainability (modularity, analysability, modifiability, testability); corpus
`mods/`, `mods/*/tests/`, `scripts/` (+ `plugins/` Python for M1/M3/M4 because the
metrics cover it); all figures.

> **Status of this draft.** Every number below was measured by me at the pinned
> commit and is reproducible with `personA/measure/run.sh`; raw outputs are in
> `personA/results/`. Items marked **[CONFIRM]** need a team decision or a
> check I could not do alone; they are collected in §A.9. Text in
> "Report text" blocks is written to drop into the LaTeX with light editing;
> everything else is working notes for the team.

---

## A.0 Pinned commit and corrections to the interim report

**Pinned commit (proposed, S1):** `bf2323b3021c9d3db7a31e1e7d61068c3f704497`
(2026-10-13, "chore: Update mods/"), on `upstream/main`. **[CONFIRM]** with B.

```latex
\newcommand{\repocommit}{bf2323b}
\newcommand{\repocommitfull}{bf2323b3021c9d3db7a31e1e7d61068c3f704497}
\newcommand{\permalink}[2]{\href{https://github.com/anthropics/claude-code/blob/\repocommitfull/#1\##2}{\texttt{#1}}}
```

The repo has moved since the interim report (16 commits on `mods/` since
27 Sept). Re-verification of the interim claims that Person A's sections reuse:

| Interim claim | At `bf2323b` | Action |
|---|---|---|
| `mods/diff/` = 577 files / ~9 000 LOC | **594 files / 9 512 LOC** (`.ts`+`.tsx` under `hooks/`) | update |
| `mods/*/tests/` = 30 files, ~4 000 LOC | **417 files / 14 730 LOC**, of which 38 are `*.test.ts(x)` and the rest fixtures; 418 `test()` cases | update; the "~4 000" number is badly stale |
| `register.ts` 874 LOC, 17 mutable closure vars, 12 hooks | **1 062 LOC, 20 `let` + 5 mutable containers, 10 `on(...)` registrations** | update |
| `security_reminder_hook.py` 2 192 LOC, handler L901–1377 | **confirmed**: 2 192 LOC; `handle_commit_review_posttooluse` L901–1376 (476 lines) | keep |
| `autoCloseDuplicates` L91–230 | **L99–272** | update |
| `mod-tests.yml` L29–32 exits 0 without the test command | **confirmed** L29–32 | keep |
| `githubRequest<T>()` + `GitHubIssue` copied across 3 scripts | 2 verbatim copies (`auto-close-duplicates.ts:28`, `backfill-duplicate-comments.ts:26`) + 1 **divergent** copy (`sweep.ts:14`, different signature, swallows 404 as `{}`) | refine: the third copy is worse than a copy |
| ~15-line bootstrap copied across 4 `hookify` entry scripts | **confirmed** by jscpd (27–28-line clones, `hookify/hooks/*.py` L4–32) | keep |

---

## A.1 Characteristic selection — Person A's contribution to S3

Report text (≈120 words, for §1):

> Under **Maintainability** we keep the interim's *modularity*, *analysability*
> and *testability* and add *modifiability*. The interim argued `mods/diff/` is
> modular because its code is spread over hundreds of single-export files; file
> count is evidence about modularity only if coupling is low, so we measure it
> (M2). Modifiability is added because the interim's DRY findings (§3.1.3) and
> the concentration of change in one file (37 of 62 commits that touched
> `mods/diff/hooks` since 9 Sept also touched `register.ts`) are arguments about
> the cost of change, not about structure. *Reusability* is folded into
> modifiability: in this repository it shows up only as duplication (M3), and
> a separate sub-characteristic would generate no separate proposal.

**[CONFIRM]** with B whether reusability is dropped as a separate heading (the
script suggests adding it; I recommend folding it, for the word budget).

---

## A.2 Metrics M1–M4 (step 2)

Measurement scope (for the S2 paragraph): `mods/{agents-md,diff,sec-default,telemetry}/hooks`
(TS/TSX), `mods/*/tests`, `scripts/*.ts`, `plugins/**/*.py`. Excluded:
`mods/types/claude-code.d.ts` (13 440 generated lines), Markdown plugin assets,
JSON manifests.

### M1 — Cyclomatic complexity and function length

*Tools:* lizard 1.24.1 for Python; for TypeScript a 60-line McCabe counter on
the TypeScript compiler AST (`personA/measure/ccn.mjs`), because lizard 1.24.1
mis-parses TypeScript. It ends a function at a multi-line signature with a
generic return type (`): Promise<boolean> {`), so in `register.ts` it drops
`fetchBodies` entirely and misses `githubRequest` in the scripts. Counting
rule: CCN = 1 + `if`, `?:`, `case`, loops, `catch`, `&&`, `||`, `??` (incl.
assignment forms); nested functions are counted separately.

| Corpus | functions | mean CCN | p90 | max | CCN > 10 | CCN > 15 |
|---|---:|---:|---:|---:|---:|---:|
| `mods/diff` | 450 | 2.42 | 5 | 23 | 10 | 3 |
| `mods/sec-default` | 104 | 1.88 | 4 | 8 | 0 | 0 |
| `mods/telemetry` | 125 | 2.43 | 5 | 16 | 2 | 1 |
| `mods/agents-md` | 65 | 2.25 | 5 | 22 | 2 | 1 |
| `scripts/` | 24 | 4.21 | 13 | 19 | 4 | 1 |
| `plugins/` (Python) | 159 | 7.65 | 14 | **80** | 28 | 15 |

Worst offenders: `handle_commit_review_posttooluse` (CCN 80, 476 lines,
`security_reminder_hook.py:901`), `handle_push_sweep_posttooluse` (62, 321
lines, `:1378`), `agentic_review` (61, 509 lines, `llm.py:1059`). In TS:
`dockPlanOf` (23, `views/body/plan/dock-plan-of.ts:18`), `backfillDuplicateComments`
(19, 137 lines), `autoCloseDuplicates` (14, 174 lines, `scripts/auto-close-duplicates.ts:99`).

**The finding that matters is not in the table.** No single function in
`mods/diff/hooks/register.ts` is extreme (max 14). The file, though, defines
116 functions (closures) and holds **155 decision points**, against a median
of 2 and a p90 of 9 for the other 160 files of `mods/diff` that contain
functions. It is 3.4× the next most complex TS file in the repo. Function-level
CCN hides this because the complexity is spread across closures that share 25
pieces of mutable state. Report the file-level number next to the function-level
one, and say why.

Also worth one sentence: the distribution is sharply split by governance
boundary. `mods/` mean CCN is ≈2.3 and 0 functions in `sec-default` exceed 10;
`plugins/` Python has mean 7.65 and 15 functions above 15.

*Indicates:* analysability (how much must be understood to change a unit),
modifiability (hotspot × churn, see M2), testability (decision points per unit
that a test must reach).

### M2 — Coupling (fan-in / fan-out)

*Tool:* madge 8.0.0, `--ts-config mods/tsconfig.json`, over each mod's `hooks/`.
Every barrel does `export * as default from '.'`, which madge reports as 267
"circular dependencies"; these are 1-node self-loops and are removed before
computing anything (state this in the method; otherwise a reader re-running
madge sees 267 cycles and distrusts the number).

| Mod | files | import edges | mean fan-out | packages | leaf packages (fan-out 0) | `register` pkg fan-out | real multi-file cycles |
|---|---:|---:|---:|---:|---:|---:|---:|
| `diff` | 594 | 1 141 | 1.92 | 28 | 11 | **21 of 27** | 4 |
| `sec-default` | 83 | 130 | 1.57 | 10 | 7 | 8 of 9 | 0 |
| `telemetry` | 210 | 365 | 1.74 | 17 | 7 | 6 of 16 | 1 |
| `agents-md` | 56 | 103 | 1.84 | 8 | 4 | 6 of 7 | 0 |

Highest file fan-in in `diff`: `git/types/index.ts` (54), `git/index.ts` (35),
`limits/index.ts` (34). Highest package fan-in: `git` (9), `limits` (8), `host` (6).

**Interpretation, honestly stated.** The interim's modularity claim survives
for the leaves. Coupling is low (mean fan-out < 2 per file), 11 of 28 packages
import nothing else, and no package other than `register` depends on more than
8 others. High fan-out of `register` by itself is *not* a defect: a composition
root is supposed to import everything, and `sec-default`'s register has an even
higher fan-out ratio. The defect is the combination that only `diff` has.
`register.ts` is a composition root **and** a stateful controller **and** a
change hotspot:

| `register.ts` of | LOC | decision points | `let` | share of mod's decision points |
|---|---:|---:|---:|---:|
| `diff` | 1 062 | 155 | 20 | 24.2 % |
| `agents-md` | 306 | 45 | 7 | 55.6 % |
| `sec-default` | 208 | 25 | 0 | 27.2 % |
| `telemetry` | 369 | 6 | 2 | 3.4 % |

`telemetry` is the in-repo counter-example. Its `register` only wires events to
an object built by `telemetryOf(...)`, which owns the behaviour. That is the
target shape for P1. (`agents-md`'s high share comes from a small mod; at 45
decision points it is not a priority.)

The 4 real cycles in `diff` (e.g. `names/texts/untracked-withheld-text-of.ts` ↔
`pane-state/empty-state-of.ts`) are `import type` edges, erased at runtime.
They cost analysability, not correctness. One sentence suffices.

*Indicates:* modularity, analysability.

### M3 — Duplication ratio

*Tool:* jscpd 5.4.1, `--min-tokens 50`, formats typescript/tsx/python.

| Corpus | lines scanned | duplicated lines | ratio | clones |
|---|---:|---:|---:|---:|
| `plugins/` (Python) | 7 488 | 223 | 2.98 % | 11 |
| `scripts/` | 749 | 37 | 4.94 % | 3 |
| `mods/` (hooks + tests) | 23 387 | 940 | 4.02 % | 89 |
| — of which `mods/*/hooks` | — | **0** | **0 %** | 0 |

All 940 duplicated lines in `mods/` are in test files (repeated setup inside
`register.test.ts` / `gate.test.ts`), which is conventional and mostly harmless.
The production code of `mods/` has **zero** clones ≥ 50 tokens, which is the
strongest quantitative support for the interim's DRY praise of `mods/`.

The interim's two DRY violations are confirmed and quantified. They are the
`hookify` bootstrap (clones of 27–28 lines between `posttooluse.py`,
`userpromptsubmit.py`, `stop.py`, `pretooluse.py`) and `githubRequest`. The
largest single clone is new: 73 lines duplicated between
`security-guidance/hooks/llm.py:1455` and `review_api.py:211`.

Caveat for the method paragraph: jscpd finds token-identical (Type-1/2) clones
only. It misses the third `githubRequest` in `sweep.ts`, which has diverged
(different signature, and it returns `{}` on 404 where the other two throw).
Divergent copies are a worse modifiability problem than identical ones,
because a fix applied to one no longer applies mechanically to the others.

*Indicates:* modifiability (incl. reusability). *If words run short, cut M3
to two sentences; the plan already marks it as the cuttable metric.*

### M4 — Tests: distribution, level mix, and effectiveness

*Tool:* Claude Code 2.1.300, `claude plugin test mods/<mod>` with
`CLAUDE_CODE_ENABLE_FUNCTION_HOOKS=1`. **The runner has no coverage option**
(`claude plugin test --help` lists only `[dir]`). Line coverage via the
engine's own test command, as the plan proposed, is therefore not obtainable.
I report structural proxies instead and say so. **[CONFIRM]** whether the team
wants me to attempt coverage via another runner (see §A.9).

| Corpus | source LOC | test LOC (incl. fixtures) | test : source | test cases | packages with a dedicated test file |
|---|---:|---:|---:|---:|---:|
| `mods/diff` | 9 512 | 7 910 | 0.83 | 217 | 10 of 27 |
| `mods/sec-default` | 1 320 | 4 498 | 3.41 | 162 | 2 of 8 |
| `mods/telemetry` | 3 061 | 1 952 | 0.64 | 34 | 1 of 14 |
| `mods/agents-md` | 1 071 | 370 | 0.35 | 5 | 0 of 6 |
| `plugins/` (py+sh) | 9 088 | **0** | **0** | 0 | 0 of 13 plugins |
| `scripts/` | 748 | **0** | **0** | 0 | — |

Packages without a dedicated test file are not necessarily untested. Each mod's
`register.test.ts` drives the whole mod through the engine (see A.5), so they
are exercised transitively. The column measures *direct, isolatable* tests.

**Run results on Windows 11 (as measured):** `sec-default` 162/162 pass,
`agents-md` 5/5. `diff` failed 57/217 on the first run and 31/217 on a second
identical run from a non-synced folder. All but 3 of those were `timed out
after 5000 ms`, and all fell in 6 of the 30 test files. The same 3 assertion
failures recur in every run (`register > a merge finished {0.3, 1.2, 3} s after
the open is noticed`), so they are pre-existing at the pinned commit on this
platform. `telemetry` failed 8/34, including a Linux-only test (`on Linux
the distribution, kernel, WSL and EC2 come from their files`). Failures that
change between identical runs are environmental (timing), not defects. The
report should cite a **Linux run**, which CI uses (`ubuntu-latest`). **[CONFIRM]**,
see §A.9.

**Effectiveness caveat, as part of the metric:** `mod-tests.yml` L29–32 exits 0
when the installed CLI lacks `plugin test`. With 2.1.300 the command exists, so
CI does run the tests today. The escape hatch remains, though, and a CLI
regression would turn the gate green without running anything. A pass rate is
evidence only alongside that caveat.

*Indicates:* testability.

---

## A.3 Q-a — Architectural style: suitability (step 3a)

Carry over the interim §3.2.1–3.2.2 identification (microkernel; tiered
interceptor chain) in 2–3 sentences, then:

Report text (≈200 words):

> The microkernel split suits a product whose core is closed and whose
> extensions are developed in the open. The contract between them is a single
> typed declaration file (`mods/types/claude-code.d.ts`), against which every
> mod type-checks in CI (`mod-tests.yml` L24–25). The style moves risk onto the
> extension boundary, and the repository governs that boundary unevenly.
> `mods/` gets a typed contract, a test kit that runs each mod through the
> engine's real hook chain, 418 test cases and no production-code duplication
> (M3). `plugins/` gets a prose structure specification, no types and no tests
> (M4), and holds the 15 functions with CCN > 15 (M1). The style is therefore
> well chosen, but its quality guarantee covers only one of the two extension
> mechanisms the repository ships.
>
> The five-tier ordering is visible in the contract
> (`TIERS = ["prepend","user","append","builtin","core"]`,
> `claude-code.d.ts:9958`) and partly testable: the test kit's `tier(...)`
> seats a mod in a tier, and `sec-default`'s tests use `prepend` five times.
> What cannot be verified from this repository is that the closed runtime
> *enforces* the order between plugins. That is an architectural limitation of
> evaluating a microkernel from the extension side, not a missing test.

Cross-reference for B: this "governance boundary" is the same split B's Q-b
finds for principle adherence. **Agree on one name for it** (I use "the
`mods/`↔`plugins/` governance boundary").

---

## A.4 Q-d — Design patterns: correct and effective application (step 3d)

Carry over MVU, Chain of Responsibility, Decorator, Strategy/Factory Method,
Adapter (interim §3.2.3–3.2.7) as a 5-row table (pattern | where | verdict),
then build the prose around MVU:

Report text (≈220 words):

> MVU is applied correctly for three of its four roles. The **Model** is an
> immutable record (`PaneState.PaneModel`, `pane-state/pane-model/pane-model.ts:16`).
> **Update** is a set of pure reducers (`afterFetch`, `after-fetch.ts:14`;
> `afterNewSession`), each unit-tested without the engine. The **View** is a
> pure function of the model (`paneView`, `views/pane-view.tsx:29`). The
> fourth role, the runtime that owns the model, sequences effects and feeds
> messages to Update, is not a component. It is the body of `register(on)`:
> 1 062 lines, 20 mutable `let`s plus 5 mutable containers, 10 hook
> registrations and 116 closures, which reassign `model` directly in 22
> places. Six of these are in the View's own action callbacks (`actionsOf`,
> L553–631), which mutate `model` instead of dispatching a message. The pattern is therefore three
> quarters applied. The missing quarter is where M1 puts 155 decision points
> and where 37 of the last 62 `diff` commits landed. Figure A.1 shows it: every
> step of a `/diff` lifecycle between the engine and the pure modules passes
> through closure-local state.
>
> The remaining patterns are applied effectively. Strategy over git backends
> via `INSTALLED_BACKEND_PROBES` lets a backend be added without editing the
> caller. The Adapter from `$` to the narrow `Host` interface (21 members,
> L676–706) is what makes the pure modules testable with plain fakes.

Verified: `grep -cE "^\s*model = " mods/diff/hooks/register.ts` → 22; 6 of
them within L553–631.

---

## A.5 Q-e (tests half) — Structure and coverage of the test suite (step 3e)

Report text (≈150 words):

> The suite mirrors the source (`tests/git/parse/parse-numstat.test.ts` ↔
> `hooks/git/parse/`), keeps shared setup in `tests/fixtures/` one export per
> file, and mixes two levels deliberately. Pure modules get unit tests with no
> engine at all (`header-totals-of.test.ts`: 3 tests, 0 mocks). Each mod's
> `register.test.ts` is an integration test through the real hook chain: the
> test calls the engine's `$`, every hook of the mod runs as shipped, and
> anything the test leaves unanswered throws (`diff`: 45 such tests, 134 calls
> on `$.session/command/tool/ui`). This answers the interim's open question:
> the hook ↔ engine contract *is* tested, at integration level.
> The defects are elsewhere. `plugins/` (9 088 LOC) and `scripts/` (748 LOC)
> have no tests (M4). The CI gate can pass without running tests. In `diff`,
> 17 of 27 packages carry no direct test and depend on the integration-level
> tests, which are the slowest and most timing-sensitive. On our machine every
> failure in three runs fell in 6 of the 30 test files, the integration-heavy
> `register`, `views` and `pane-view` among them.

**[CHECK]** against a Linux run before submitting.

---

## A.6 Figures (A5)

| Fig. | File | Status |
|---|---|---|
| 1.1 package (interim) | — | **[CONFIRM]** need the interim `.puml` source to re-render at `bf2323b`. Note `mods/` now has 4 mods (`agents-md`, `telemetry` are new since the interim figure) |
| 3.1 class (interim) | — | same |
| A.1 `/diff` lifecycle sequence | `personA/diagrams/diff-lifecycle-sequence.puml` | drafted, line refs at `bf2323b` |
| A.2 P1 before/after component | `personA/diagrams/p1-after-component.puml` | drafted; shaded box = prototyped |
| B's role-stereotype figure | — | waiting on B's content (B3) |

Provenance sentences are in each `.puml` header in the interim's style. Render
with `plantuml -tsvg personA/diagrams/*.puml` and record the PlantUML version.
I have not rendered them (no PlantUML or Java here). **[CHECK]** they render.

---

## A.7 Improvements P1–P3 (step 4)

**P1. Complete MVU in `mods/diff`: decompose `register.ts`** (from M1, M2, Q-a,
Q-d; targets modularity, analysability, modifiability, testability).
`register.ts` becomes wiring only, as `telemetry`'s already is. A
`PaneController` owns `model` and dispatches each hook to one of five
coordinators, each owning the closure variables that today belong to it:

| Coordinator | Closure state it takes over (register.ts L43–71) |
|---|---|
| `BodyLoader` *(prototyped)* | `bodyStamp`, `bodyBase`, `bodyLoads` |
| `RefreshScheduler` | `isRefreshing`, `isRefreshQueued`, `generation`, `landed`, `polled`, `timers` |
| `BackendPin` | `backend`, `probing`, `pin`, `loggedBaseKinds` |
| `PaneLifecycle` | `isPaneOpen`, `awaited`, `dialogRows`, `hasAutoOpened`, `hasRestoredEdits`, `columns`, `shownSessionId`, `sessionStartMs` |
| `AskComposer` | `armed`, `carrying` |

View actions return messages to the controller instead of reassigning `model`.
Figure A.2. Trade-off (for S7): coordinators must read and write `model` across
a boundary (the prototype passes `modelOf`/`setModel` closures). The debounce
and coalesce timers that keep `/diff` non-intrusive (interim §2.3.1) move
intact into `RefreshScheduler`, so their timing is unchanged, but the
`isRefreshing` ↔ `loadBodies` interleaving becomes a cross-object protocol that
needs its own test.

**P2. Make `autoCloseDuplicates` testable** (from M1, interim §4.2; targets
testability, analysability). Extract the closure decision into pure predicates:
`isDupeDetectionComment`, `hasActivityAfter`, `hasAuthorDisagreed`, and two
verdict functions. The comment-only verdict decides before reactions are
fetched, so API calls are not added. The I/O loop keeps only fetching and
acting. Implemented (A.8).

**P3. Make test results trustworthy, then extend them to `plugins/`** (from
M4; targets testability, plus fault tolerance via B's characteristics).
1. Delete the `exit 0` branch in `mod-tests.yml` L29–32 and pin the CLI
   version installed at L22, so a missing test command fails the job
   (testability: a green gate implies tests ran).
2. Raise the test kit's per-test timeout, or fix the timing-sensitive tests,
   so results are stable across machines (M4 showed 57 → 31 failures between
   identical runs).
3. Add `pytest` suites for the two highest-risk plugin units: `hookify`'s rule
   engine (`rule_engine.py`, `config_loader.extract_frontmatter`, CCN 32), which
   targets fault tolerance of rule parsing, and `security_reminder_hook.py`'s
   dispatch, which targets testability of the CCN-80 handler and needs P2-style
   extraction first.
4. Add unit tests for `scripts/` predicates (enabled by P2).

Trade-off (for S7): mandatory tests raise the bar for third-party plugin
authors, a repo-maintainability gain against an ecosystem-extensibility cost.
Recommend requiring them for first-party `plugins/` only.

---

## A.8 Re-assessment (step 5)

Both prototypes are in `personA/prototypes/`, applied to scratch copies, never
to the repo, and measured with the identical commands.

**P2: implemented and measured** (`ccn.mjs`, same counting rule as M1):

| `scripts/auto-close-duplicates.ts` | before | after |
|---|---:|---:|
| functions | 8 | 15 |
| max CCN | 14 | **8** |
| mean CCN | 3.62 | 2.47 |
| functions with CCN > 10 | 1 | **0** |
| longest function (lines) | 174 | **49** |
| total decision points | 21 | 22 |
| unit tests possible without mocking the GitHub API | 0 | **6 written, 6 pass** (`node --test`) |

Type-checks under `tsc --strict`. The total decision count does not fall; the
refactor relocates complexity into units that can be tested, which is the
honest claim to make.

**P1: one coordinator implemented and measured.** Extracting `BodyLoader`
(`personA/prototypes/p1-body-loader/`, patch + new module); `tsc -p
mods/tsconfig.json` passes.

| `mods/diff/hooks/register.ts` | before | after |
|---|---:|---:|
| LOC (non-blank, lizard NLOC) | 735 | 701 |
| decision points | 155 | 148 |
| mutable `let` | 20 | 18 |
| mutable containers | 5 | 4 |
| package fan-out | 21 | 21 |
| `diff` total decision points | 640 | 640 |

Behaviour check: `claude plugin test diff` on the modified copy gave 165 pass,
52 fail. That is 49 timeouts plus **the same 3 assertion failures as both
baseline runs and no new one**. This is evidence, not proof, of unchanged
behaviour: a test that timed out never reached its assertions. Re-run on Linux
before claiming it in the report.

**Estimate for full P1** (stated as an estimate): the five coordinators take
over 22 of the 25 mutable items, leaving `model` (in the controller) and the
`host` and `columns` bindings. By the same per-item ratio the prototype showed
(−7 decision points for 3 items), `register.ts` would drop to roughly
**100 decision points or fewer**. That figure is a lower bound on the
reduction, because the 10 hook bodies would also shrink to one-line dispatches,
and moving them out is where most of the remaining 148 sit (the largest is the
`command.run{diff}` hook at 14). Package fan-out would stay at about 21 for
`register` + controller together. The improvement is in state-per-unit, not
coupling, and the report should say that M2 is *not* expected to move.

---

## A.9 Open points — decisions for you / the team

1. **Pinned commit.** I used `bf2323b` (the commit you checked out). B must
   use the same SHA for M5–M7. Confirm.
2. **M4 authoritative run.** The `diff` and `telemetry` suites are flaky on
   Windows (timeouts) and one test is Linux-only. WSL Ubuntu is installed here
   but has no Node or Claude CLI. Options: (a) I install Node + Claude CLI in
   WSL and re-run; (b) cite the CI run of the pinned commit; (c) report the
   Windows numbers with the caveat. I recommend (a) or (b).
3. **Coverage.** `claude plugin test` has no coverage flag. Either accept the
   structural proxies above (my recommendation, fits the words) or ask the
   course whether a proxy is acceptable.
4. **Interim figures 1.1 / 3.1.** I need their `.puml` sources to re-render
   at `bf2323b`. Where are they?
5. **Reusability** as a separate sub-characteristic or folded into
   modifiability (A.1)?
6. **Prototypes on a branch?** P1/P2 currently live only in `personA/`. If you
   want them as real commits (e.g. for a permalinked "after" state), I can put
   them on a branch of your fork. They should not go upstream: `mods/` is
   published from a closed source ("chore: Update mods/" bot), so a PR there
   would not be the right channel anyway.

## A.10 Word budget (Person A ≈ 1 500)

| Part | target words | in this draft (report text only) |
|---|---:|---:|
| §1 maintainability justification | 120 | ≈120 |
| §2 M1–M4 (tables + 1 para each) | 450 | tables + notes; to be compressed |
| §3 Q-a, Q-d, Q-e tests | 550 | ≈570 |
| §4 P1–P3 | 250 | ≈400, trim P3 list |
| §5 re-assessment | 150 | tables + ≈150 |
