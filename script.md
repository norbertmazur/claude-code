# Project Quality (45 pts) — Two-Person Work Division

Capstone Software Development (2IRR80), team 9 — analysis of the `anthropics/claude-code` repository.
Grounded in the interim report (27 September 2026).

---

## What the interim report already settles

Three things that were open questions in the generic version of this plan are now answered, and they change the split.

**1. The measurement scope is known.** The repo does not contain the CLI source (interim §1.1). The measurable corpus is:

| Corpus | Size / character | Metric suitability |
|---|---|---|
| `mods/` TypeScript | >10 000 LOC; `mods/diff/` alone is 577 files / ~9 000 LOC | Full static analysis: complexity, coupling, duplication, coverage |
| `mods/*/tests/` | 30 test files, ~4 000 LOC | Coverage and test-structure metrics |
| `plugins/` Python + Bash | ~9 000 LOC executable, **zero** automated tests | Complexity, duplication, error-handling metrics |
| `scripts/` TypeScript | Bun/TS maintenance utilities | Complexity, duplication |
| `.github/workflows/` | 13 workflows | Supply-chain / CI hardening metrics |
| `mods/types/claude-code.d.ts` | >10 000 generated lines | **Exclude** from all metrics — generated, would swamp every number |

So static metrics are feasible. Say in §2's preamble that the generated `.d.ts`, the marketplace manifest and Markdown-only plugin assets are excluded, and why.

**2. The ISO 25010 set from the interim report is the starting point.** Interim §2 used Maintainability (modularity, analysability, testability), Security (confidentiality, integrity, resistance, accountability/non-repudiation) and Compatibility (co-existence, interoperability). Step 1 of the assignment says you "likely need to extend and revise" — two extensions are already justified by evidence you collected:

- **Reliability → Fault tolerance**, motivated by the `hookify` global fail-open (interim §2.2.3) and the CI gate that exits 0 when the test command is missing (§2.1.3). Both are failure-masking behaviours, which is fault tolerance, not security.
- **Flexibility → Installability**, motivated by the Windows quick-start failure (§5.3) and the npm-deprecation contradiction between root README, `plugins/README.md` and `mod-tests.yml` (§1.6 item 4).

Adding `Modifiability` and `Reusability` under Maintainability is also cheap to justify — §3.1.3 (DRY) and §3.2.6 (OCP via `INSTALLED_BACKEND_PROBES`) are already modifiability/reusability arguments filed under the wrong label.

**3. Half of step 3 exists in draft.** Interim §3.2 already identifies the architectural styles (microkernel, tiered interceptor chain), the architectural pattern (MVU) and four GoF patterns, and §3.1 covers SRP, DIP and DRY. Two UML figures exist (Fig. 1.1 package, Fig. 3.1 class), both PlantUML, both with provenance sentences already written. The new work in step 3 is therefore **evaluation rather than identification**, plus role stereotypes (3c), which nobody has done.

---

## Ground rules

**Pin one commit — and note that the interim report did not.** Figure 1.1 cites `f96c3b4`; §4.1 permalinks `56f36532530f88b572854538d685fcf781141e8c`. Pick one SHA for the whole quality section, re-verify every line number carried over from the interim report against it, and put it in a shared `\newcommand{\repocommit}{...}`. This repo moves fast (interim §6.3: v2.1.280 landed with a large commit cluster in late September), so line numbers from three weeks ago are not safe to reuse unchecked.

**Split vertically by quality characteristic, not by step.** Step 4 must be traceable to step 2–3 findings and step 5 re-runs step 2's metrics. If one person owns metrics and the other owns improvements, the second is blocked until the first finishes and the traceability is second-hand. Each person carries their own characteristics through all five steps.

**Reuse, but re-verify.** Interim material is a head start, not evidence. Anything carried across gets its line numbers and claims re-checked at the pinned commit before it goes in.

**Note on the third author.** The report has three authors but this section is for two. The natural fit, from the interim AI-use statement: whoever wrote chapters 1–3 already has the `mods/` and architecture context and should take Person A; whoever wrote §§4–6 has the `scripts/`, documentation and tooling context and should take Person B. The third author is best used for S8 cross-review and the collaboration section, not for a third slice of this one.

---

## Division 1 — the short version

| | **Person A — "Structure"** | **Person B — "Ecosystem"** |
|---|---|---|
| Characteristics | Maintainability: modularity, analysability, modifiability, testability | Security: integrity, resistance, accountability · Compatibility: co-existence, interoperability · Reliability: fault tolerance · Flexibility: installability |
| Corpus | `mods/`, `mods/*/tests/`, `scripts/` | `plugins/`, `.github/workflows/`, `examples/`, docs |
| Metrics | M1–M4 (complexity, coupling, duplication, test coverage) | M5–M7 (doc/structure conformance, CI supply-chain, fail-open rate) |
| Qualitative (step 3) | (a) architectural style, (d) design patterns, (e) test half | (b) design principles, (c) role stereotypes, (e) documentation half |
| Improvements | P1–P3: architecture + test set | P4–P8: implementation, CI, docs |
| Re-assessment | Re-runs M1–M4 | Re-runs M5–M7 |
| Infrastructure | PlantUML diagrams (all figures) | LaTeX build, bibliography, consistency pass |
| Shared | §1 characteristic selection and justification, trade-off synthesis, cross-review, final edit | |

Roughly 1500 words each.

---

## Division 2 — the detailed version

### Phase 0 — Joint kickoff (both, ~2 hours)

- **S1.** Pick and freeze the analysis commit; record SHA, date, permalink prefix. Re-verify the handful of interim line references you intend to reuse.
- **S2.** Write the measurement-scope paragraph from the corpus table above, including the exclusions.
- **S3.** Finalise the revised ISO 25010 set. Proposed: keep the interim nine sub-characteristics, add modifiability and reusability (Maintainability), fault tolerance (Reliability) and installability (Flexibility); each justified by a specific interim finding, not by generic reasoning. Drop confidentiality if words get tight — it was the one sub-characteristic where the interim found no defect (`checkedValue`, `isAnalyticsOff`), so it generates no step-4 proposal.
- **S4.** Numbering: metrics **M1–M7**, qualitative findings **Q-a1, Q-b1, …**, proposals **P1–P8**. Step 4 requires every proposal to name the finding that motivated it.
- **S5.** LaTeX skeleton, one `.tex` per section. B owns the build.
- **S6.** Word budget: §1 ≈ 350, §2 ≈ 800, §3 ≈ 800, §4 ≈ 700, §5 ≈ 350.

---

### Person A — Structure & static quality

**Owns:** Maintainability (modularity, analysability, modifiability, testability). Corpus: `mods/`, `mods/*/tests/`, `scripts/`. All diagrams.

#### A1 — Metrics M1–M4

**M1 — Cyclomatic complexity and function length.** Tool: `lizard` (language-agnostic, handles TS and Python in one run; record `lizard --version`). Run over `mods/*/hooks/`, `plugins/**/*.py`, `scripts/*.ts`, excluding `mods/types/`. Report the distribution plus the worst offenders — which the interim already predicts: `mods/diff/hooks/register.ts` (874 LOC, 17 mutable closure variables, 12 registered hooks), `security_reminder_hook.py` (2 192 LOC with a single 477-line handler at lines 901–1377), `autoCloseDuplicates` (lines 91–230). Indicates analysability and modifiability; also testability, since the interim §4.2 finding is precisely that the logic cannot be unit-tested without mocking the whole API environment.

**M2 — Coupling (fan-in / fan-out).** Tool: `dependency-cruiser` or `madge` over `mods/`. This is the metric that tests the interim's strongest claim — that `mods/diff/` is "exceptionally modular" because 9 000 LOC is spread over 577 single-export files. File count is not modularity. Fan-in/fan-out will show whether the folder-per-export convention plus the barrel files actually reduce coupling, or whether `register.ts` is a hub that every feature package feeds into. Either result is a good finding, and the second one is the more interesting.

**M3 — Duplication ratio.** Tool: `jscpd` (multi-language, so one run covers `plugins/hookify/hooks/*.py` and `scripts/*.ts`). Quantifies the two DRY violations the interim identified qualitatively: the ~15-line bootstrap block copied verbatim across four `hookify` entry scripts, and `githubRequest<T>()` + `GitHubIssue` copied across three maintenance scripts. Indicates modifiability and reusability.

**M4 — Test coverage and test distribution.** Two numbers: coverage over `mods/` via the engine's own test command (`claude plugin test`, record the CLI version), and a test-to-source ratio per top-level directory. The expected result is sharply bimodal — ~4 000 test LOC for `mods/` against 0 for ~9 000 LOC of executable `plugins/` code. Add the effectiveness caveat as part of the metric, not a footnote: `.github/workflows/mod-tests.yml` lines 29–32 exit 0 when the CLI lacks the test command, so a green pipeline does not imply tests ran. A coverage number without that caveat overstates testability.

*If words run short, M3 is the one to cut — the duplication is already demonstrated qualitatively in interim §3.1.3 and five metrics is the floor.*

#### A2 — Qualitative (a): architectural style

Carry over the microkernel and tiered-interceptor identification (interim §3.2.1–3.2.2), then do what the interim did not: **evaluate suitability**. The real argument is that the microkernel split is well-suited to a closed-core product with an external plugin ecosystem, but it pushes all the risk onto the extension boundary, and the repo governs that boundary unevenly — `mods/` gets a typed contract and tests, `plugins/` gets neither. Also assess whether the five-tier ordering (`prepend → user → append → builtin → core`) is verifiable from this repo at all; interim §2.2.2 already notes the tier guarantee is documented but unverifiable because the runtime is closed. That is an architectural evaluation finding, not just a caveat.

#### A3 — Qualitative (d): design patterns

Carry over MVU, Chain of Responsibility, Decorator, Strategy/Factory Method, Adapter (interim §3.2.3–3.2.7), then judge **correct and effective application**. The ready-made critical finding: MVU is implemented cleanly for Model (`PaneState.PaneModel`), Update (pure reducers like `afterFetch`) and View (`paneView`), but the Controller role is not a component at all — it is 874 lines of closure inside `register.ts`. So the pattern is three-quarters applied, and the quarter that is missing is exactly where M1 reports the complexity. That single observation ties Q-d to M1 to P1 and is worth building the section around.

#### A4 — Qualitative (e), test half

Structure and coverage of the test suite: the symmetric `hooks/` ↔ `tests/` mirroring and `tests/fixtures/` isolation (interim §1.2, §1.4) are genuinely good and should be credited; the absence of any `plugins/` tests and the fail-open CI gate are the defects. Assess the level mix — the mock kit (`mock.clock`, `mock.env`, `mock.store`) supports unit testing well, but note whether anything tests the integration between a registered hook and the engine contract.

#### A5 — Diagrams (all figures)

Reuse Fig. 1.1 (package) and Fig. 3.1 (class) where they still serve, re-rendered at the pinned commit. New figures needed:

- A **sequence diagram** of one `/diff` lifecycle flow (session.start → probe → fetch → reducer → view), which makes the god-controller problem visible in a way the class diagram does not.
- An **"after" class or component diagram** for P1.
- Whatever B specifies for role stereotypes (3c) — B specifies content, A draws, so the visual style stays uniform.

Each figure keeps a 1–2 sentence provenance note in the interim's existing style ("manual static analysis of X, rendered using PlantUML vN").

#### A6 — Improvements P1–P3

- **P1** (from M1, M2, Q-a, Q-d): decompose `register.ts` into an explicit Controller plus coordinators, completing the MVU structure. Show the "after" UML. This is the headline proposal of the whole section.
- **P2** (from M1, interim §4.2): extract the closure-decision logic in `autoCloseDuplicates` into small pure predicates, making it unit-testable without mocking the GitHub API.
- **P3** (from M4): a test set for `plugins/`, and removal of the `exit 0` escape in `mod-tests.yml`. Specify which characteristic each addition targets rather than saying "add tests".

#### A7 — Re-assessment of M1–M4

Implement P2 for real — it is a single ~140-line function and the honest before/after is cheap. Re-run `lizard` with the identical command and report the delta. For P1, implement one extracted coordinator (not the whole refactor) and report the change in `register.ts`'s complexity and fan-out. Where you estimate rather than measure, say so explicitly and justify why the estimate is trustworthy; the brief asks for this and most teams skip it.

---

### Person B — Ecosystem, security & process quality

**Owns:** Security (integrity, resistance, accountability), Compatibility (co-existence, interoperability), Reliability (fault tolerance), Flexibility (installability). Corpus: `plugins/`, `.github/workflows/`, `examples/`, documentation. Plus the LaTeX build.

#### B1 — Metrics M5–M7

**M5 — Documented-structure conformance.** `plugins/README.md` lines 49–61 specify a mandatory plugin structure; the interim found three classes of violation. Turn that into a number: of the 13 marketplace plugins, what fraction has the required `.claude-plugin/plugin.json`, what fraction places executables only in documented directories, and how many documented capabilities match the actual `hooks.json` registrations. `plugin-dev` has no `.claude-plugin/` at all; `security-guidance` is documented as a PreToolUse hook and registers zero PreToolUse hooks. A short script over the manifests gives a conformance percentage, and the script in an appendix is your reproducibility evidence. Indicates analysability and integrity of the published contract.

**M6 — CI supply-chain hardening.** Across the 13 workflows: fraction of third-party action references pinned to a full commit SHA rather than a mutable tag, and the count of workflows that combine write permissions with a publicly triggerable event. The interim's `actions/checkout@v4` finding in `claude-issue-triage.yml` is one data point; the metric says whether it is systemic or isolated. Easy to script with a YAML parser. Indicates integrity.

**M7 — Fail-open rate.** Across every executable hook entry point in `plugins/`, count the handlers whose exception path exits 0 or otherwise swallows the error, as a fraction of all handlers. Python's `ast` module makes this a short script; shell hooks need a grep plus manual check, which you state as part of the method. The interim found this pattern in all four `hookify` entry points, so the expected number is high and damning. Indicates fault tolerance and resistance simultaneously — one metric covering two sub-characteristics is efficient use of your word budget.

**Optional M8 — cross-platform invocation safety.** Fraction of hook commands across all `hooks.json` files that quote `${CLAUDE_PLUGIN_ROOT}` and use a launcher-aware Python discovery (as `sg-python.sh` does) rather than a bare `python3`. Quantifies interoperability and installability, and directly extends interim §2.3.2. Include if the words are there; M5–M7 plus A's four already clear the five-metric floor.

#### B2 — Qualitative (b): design principles

SRP, DIP and DRY are drafted (interim §3.1) and should be compressed, not re-expanded. Extend to the principles the interim mentioned only in passing inside the patterns section: OCP (`INSTALLED_BACKEND_PROBES` is a textbook-clean example, §3.2.6), ISP (the narrow `Host` interface against the engine's 22 subsystems, §3.2.7), plus KISS and separation of concerns. The strongest new argument available: the same codebase contains both an exemplary OCP/ISP implementation in `mods/` and a 2 192-line procedural dispatcher in `plugins/` — so the finding is not "the project follows principles well or badly" but that principle adherence tracks exactly the `mods/` ↔ `plugins/` governance boundary A identifies in Q-a. Cross-reference it.

#### B3 — Qualitative (c): role stereotypes

Entirely new work — nothing in the interim covers it, and it is a named deliverable. Classify the key modules and assess whether responsibilities are cleanly assigned. Starting candidates from what the interim already documents:

| Module | Stereotype(s) | Assessment |
|---|---|---|
| `mods/diff/hooks/register.ts` | Controller **+** Coordinator **+** Information Holder | Three stereotypes in one unit — the role-stereotype statement of the SRP violation |
| `backend/backend-of.ts` | Service provider / factory | Clean — single stereotype |
| `pane-state/` | Information holder + structurer | Clean |
| `host/host.ts` | Interfacer | Clean; the adapter in `register.ts` is where it blurs |
| `sec-default/hooks/register.ts` | Controller + policy enforcer | Mostly clean; silent denial is the accountability gap |
| `scripts/gh.sh` | Interfacer / guard | Clean — a good example to credit |
| `security_reminder_hook.py` | Controller + service provider + interfacer | Worst case in the repo |

Any module holding two or more stereotypes is a step-4 proposal in waiting; say so explicitly so the traceability is visible.

#### B4 — Qualitative (e), documentation half

The interim already has four documentation discrepancies (§1.6) and a usability assessment (§5). Compress them, and add the judgement the interim stopped short of: the documentation is accurate for end users (the "Get started" section is good, modulo the missing PATH step in §5.3) and inaccurate for extension authors (`mods/` unadvertised despite holding most of the typed source and 100% of the tests; the plugin structure spec not matched by the plugins themselves). That audience split is the finding, and M5 is its number.

#### B5 — Improvements P4–P8

- **P4** (from M3, interim §3.1.3): a shared bootstrap module for the `hookify` entry scripts and a shared GitHub API utility for `scripts/`.
- **P5** (from M7, interim §2.2.4): fail-closed-with-audit for hook error paths, plus an audit event on `sec-default`'s `TOOL_REGISTER_REFUSAL` denial, which currently rejects silently.
- **P6** (from M6): SHA-pin third-party actions, scope workflow permissions.
- **P7** (from M8, interim §2.3.2): quote `${CLAUDE_PLUGIN_ROOT}` and reuse the `sg-python.sh` discovery pattern across all hook invocations.
- **P8** (from M5, interim §1.6): a CI conformance check that validates each plugin against the documented structure and that documented hook events match `hooks.json`, so the four discrepancies cannot recur.

Each must name its motivating metric or finding by number. P5 and P6 are the two where the trade-off discussion gets interesting — see S7.

#### B6 — Re-assessment of M5–M7

Three of B's proposals are genuinely cheap to implement and re-measure, which is unusual and worth exploiting:

- P6: pin the actions in one workflow, re-run the M6 script, report the new percentage.
- P8 / P5: add the missing `.claude-plugin/plugin.json` to `plugin-dev` or correct one documented hook event, re-run M5.
- P5: convert one `hookify` entry point to fail-closed, re-run M7.

Report before/after with the same provenance detail as step 2.

#### B7 — LaTeX, bibliography, figure placement, cross-references, word count

Build daily once writing starts.

---

### Phase 3 — Joint closing (both, ~3 hours)

- **S7 — Trade-off synthesis.** The section neither person can write alone, and where step 4's fourth bullet and step 5 are marked. Walk each proposal against the *other* person's characteristics. The genuinely non-obvious ones in this project:
  - **P5 (fail-closed) vs co-existence.** The `hookify` fail-open exists so that a malformed rule file does not break the user's session. Making it fail-closed improves fault tolerance and resistance, and degrades co-existence and usability — a broken rule file now blocks work. The honest proposal is fail-closed for security-relevant hooks and fail-open-with-warning elsewhere, and saying why.
  - **P1 (decompose `register.ts`) vs performance and co-existence.** Splitting a closure that holds 17 shared variables into coordinators means passing state across boundaries; argue whether that affects the debounce/polling behaviour that `/diff` relies on to stay non-intrusive (interim §2.3.1).
  - **P3 (test `plugins/`) vs maintainability of the ecosystem.** Mandatory tests raise the barrier for third-party plugin authors — a maintainability gain for the repo, an extensibility cost for the ecosystem.
  - **P6 (SHA-pinning) vs maintainability.** Pinned actions stop receiving patch updates automatically, so integrity improves and modifiability degrades unless Dependabot-style automation is added.

  Write it as a proposals × characteristics matrix, then prose.
- **S8 — Cross-review** against a fixed checklist: every metric has tool + version + exact command + commit; every qualitative claim has a file and a permalink *re-verified at the pinned commit*; every proposal names its motivating finding; every diagram has a provenance sentence. These four are the explicit reproducibility and traceability demands in the brief and are the cheapest marks available. The interim's own AI-use statement notes that AI-supplied line citations were frequently wrong — keep that verification habit.
- **S9 — Voice pass.** One person (B, who owns the build) reads through for one voice.
- **S10 — Word count and point balance.** Cut to 3000, from whichever section is most over its §-budget.
- **S11 — AI use statement.** Extend the interim's appendix B for this section's work.

---

### Suggested sequencing

| Phase | Person A | Person B |
|---|---|---|
| 1 | Kickoff S1–S6 (joint) | |
| 2 | M1–M4 | M5–M7 (+ M8 if taken) |
| 3 | Q-a, Q-d, Q-e (tests) | Q-b, Q-c, Q-e (docs) |
| 4 | Diagrams (A5) | LaTeX scaffold + §1 draft from S3 |
| 5 | P1–P3 | P4–P8 |
| 6 | Re-measure (A7) | Re-measure (B6) |
| 7 | S7–S11 (joint) | |

Metrics first is deliberate: M2 may contradict the interim's modularity claim, and if it does, that reshapes Q-a, P1 and the whole §5 narrative. Finding that out in phase 2 is cheap; finding it out in phase 6 is not.

**Dependencies to watch:** phase 2 before phase 5 for both people, since proposals must cite metric numbers; S7 needs both proposal sets; and A3's MVU-controller finding and B3's role-stereotype table describe the same defect from two angles — compare notes at the end of phase 3 so §3 states it once and §4 cites both.