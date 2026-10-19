# Person A metrics: Linux re-run (2IRR80, team 9)

## 1. Commit, OS and tools

| | |
|---|---|
| **Commit measured** | `bf2323b3021c9d3db7a31e1e7d61068c3f704497` (2026-10-13 04:34:03 +0000, "chore: Update mods/"), the pinned commit, so no fallback was needed |
| How | The checkout's HEAD was `b4e46ff` (2026-10-19, "update"). That commit only adds `personA/`, `context final report/` and `script.md` on top of `bf2323b`, so `bf2323b` was already local. Work was done in `git worktree add --detach ../claude-code-pinned bf2323b…` with `personA/` copied in. `run.sh`'s pinned-SHA guard passed unmodified. |
| OS | Ubuntu 22.04.5 LTS, Linux 6.8.0-138-generic x86_64 (HP ZBook Power G10) |
| Node / npm | v25.9.0 / 11.12.1 (Windows run: Node 25) |
| Python | 3.10.12 (Windows run: 3.14.4) |
| Claude Code | **2.1.303** (Windows run: 2.1.300; the newer installed version was kept) |
| lizard / jscpd / madge / typescript | 1.24.1 / 5.4.1 / 8.0.0 / 5.9.3 (same as Windows) |

Full version output is in `versions.txt`. Every tool was installed into a scratch directory, and nothing was installed system-wide.

## 2. Windows vs Linux

### M1: cyclomatic complexity (identical)

| Item | Windows | Linux | |
|---|---:|---:|---|
| `mods/diff` functions / max CCN | 450 / 23 | 450 / 23 | = |
| `mods/sec-default` / `telemetry` / `agents-md` functions | 104 / 125 / 65 | 104 / 125 / 65 | = |
| `scripts/` functions / max CCN | 24 / 19 | 24 / 19 | = |
| `plugins/` Python functions / mean / max CCN | 159 / 7.65 / 80 | 159 / 7.65 / 80 | = |
| `register.ts` (diff) decision points / share | 155 / 24.2 % | 155 / 24.2 % | = |
| `diff` total decision points | 640 | 640 | = |

`m1ts.csv` is byte-identical to the Windows file once `\`/`/` and CRLF are normalised. The 159 Python rows of `m1.csv` are identical. The Windows `m1.csv` also holds 625 TypeScript rows from an earlier lizard run without `-l python`; the current `run.sh` doesn't produce them, and they don't feed any reported number.

### M2: coupling (identical)

| Mod | files / edges / mean fan-out (Win) | Linux | `register` pkg fan-out | |
|---|---|---|---|---|
| diff | 594 / 1 141 / 1.92 | 594 / 1 141 / 1.92 | 21 / 21 | = |
| sec-default | 83 / 130 / 1.57 | 83 / 130 / 1.57 | 8 / 8 | = |
| telemetry | 210 / 365 / 1.74 | 210 / 365 / 1.74 | 6 / 6 | = |
| agents-md | 56 / 103 / 1.84 | 56 / 103 / 1.84 | 6 / 6 | = |

The four madge JSON graphs are identical to the Windows ones after path normalisation. `diff` still has 4 multi-file cycles and 11 of its 28 packages have no fan-out.

### M3: duplication (identical)

| Corpus | Windows | Linux | |
|---|---|---|---|
| `plugins/` | 7 488 lines, 223 dup, 2.98 %, 11 clones | same | = |
| `scripts/` | 749, 37, 4.94 %, 3 | same | = |
| `mods/` | 23 387, 940, 4.02 %, 89 | same | = |
| clones touching `mods/*/hooks/` | 0 | 0 | = |

### M4: tests (**different**)

| Suite | Windows 11 | Linux | |
|---|---|---|---|
| sec-default | 162/162 | 162/162 (6.4 s) | = |
| agents-md | 5/5 | 5/5 (0.4 s) | = |
| diff, run 1 | 160/217 (54 timeouts + 3 assertion failures, 256 s) | **217/217** (14.4 s) | **≠** |
| diff, run 2 | 186/217 (28 timeouts + 3 assertion failures) | **217/217** (14.3 s) | **≠** |
| diff, run 3 | — | **217/217** (13.5 s) | new |
| telemetry | 26/34 (8 assertion failures, no timeouts) | **34/34** (3.7 s) | **≠** |
| timeouts in any suite | 82 across the two diff runs | **0** | **≠** |

- **The 3 diff assertion failures are gone.** `register > a merge finished {0.3, 1.2, 3} s after the open is noticed` passes in all three Linux runs. The draft calls them "pre-existing at the pinned commit on this platform". On Linux they don't occur, so they depend on platform or timing and are not defects at `bf2323b`.
- The Linux-only telemetry test (`on Linux the distribution, kernel, WSL and EC2 come from their files`) runs and passes. So do the other 7 telemetry tests that failed on Windows.
- All three diff runs give the same outcome for every test name (217 distinct names, all pass), so the suite is stable on Linux.
- Every run exited 0 (`m4-exitcodes.txt`).
- **Control for the CLI version.** All four suites were re-run on Linux with Claude Code **2.1.300**, the version the Windows run used (installed in a scratch dir from npm). The result was the same: 5/5, 162/162, 34/34, 217/217, with 0 timeouts, and the 3 `register > a merge finished …` tests and the Linux-only telemetry test pass (`cli-2.1.300/`). So the Windows–Linux difference comes from the platform or the machine, not from the CLI update.
- **Coverage:** `claude plugin test --help` (2.1.303) shows only `[dir]` and `-h, --help`, so there's no coverage option. The draft's statement still holds.
- **CI escape hatch:** `.github/workflows/mod-tests.yml` lines 29–32 still read `if ! claude plugin test --help … | grep -q '^Usage: claude plugin test'; then echo "… skipping the test step"; exit 0; fi` (`mod-tests-L20-35.txt`). Confirmed.

### P2: `scripts/auto-close-duplicates.ts` (identical)

| | Windows before → after | Linux before → after | |
|---|---|---|---|
| functions | 8 → 15 | 8 → 15 | = |
| max CCN | 14 → 8 | 14 → 8 | = |
| mean CCN | 3.62 → 2.47 | 3.62 → 2.47 | = |
| functions CCN > 10 | 1 → 0 | 1 → 0 | = |
| longest function | 174 → 49 lines | 174 → 49 lines | = |
| decision points | 21 → 22 | 21 → 22 | = |
| unit tests | 6 written, 6 pass | 6/6 pass (`node --test`, Node 25.9.0 native TS) | = |
| `tsc --strict --noEmit` | passes | passes with `--module esnext --moduleResolution bundler` (exit 0); the original file passes under the same flags | = (see note) |

Note: with `--module nodenext` and no `package.json` `"type":"module"`, tsc treats the file as CommonJS and rejects the `import.meta.main` guard (TS1470). That comes from the module setting, not from strictness, and the guard already appears in the original script. Exact commands are in `p2-tsc.log`.

### P1: `BodyLoader` extracted from `mods/diff/hooks/register.ts`

| | Windows before → after | Linux before → after | |
|---|---|---|---|
| LOC (sum of lizard per-function NLOC) | 735 → 701 | 735 → 701 | = |
| decision points (ccn.mjs) | 155 → 148 | 155 → 148 | = |
| closure-scope `let` | 20 → 18 | 20 → 18 | = |
| mutable containers | 5 → 4 | 5 → 4 | = |
| `register` package fan-out | 21 → 21 | 21 → 21 | = |
| `diff` total decision points | 640 → 640 | 640 → 640 | = |
| `tsc -p mods/tsconfig.json` | passes | passes (exit 0; the baseline passes too) | = |
| `claude plugin test mods/diff` | 165/217 (49 timeouts + the 3 assertion failures) | **217/217 in each of 3 runs** | **≠** |

Notes on counting rules, so the table stays reproducible:
- "LOC 735" is the **sum of per-function NLOC** from `lizard --csv`. Nested closures are counted inside their parents as well, so this is not a file line count. Lizard's file-level NLOC is 828 → 791, and the non-blank line count is 845 → 808. The draft's label "non-blank, lizard NLOC" should say "sum of function NLOC (lizard)".
- "`let` 20" counts the `let` declarations at the closure scope of `register` (L43–68). A grep over the whole file finds 21, because of a function-local `let result` at L1001.
- The P1 copy has 2 more files and 1 more package (`body-loader`, fan-out 4). Its `diff` graph has 596 files, 1 146 edges and mean fan-out 1.92.

## 3. P1 behaviour check: **yes, it holds**

The outcome sets are identical with and without the patch. The three unmodified baseline runs (`m4-diff-run{1,2,3}.log`) and the three runs on the patched copy (`m4-diff-p1-run{1,2,3}.log`) each report 217 pass, 0 fail, 0 timeouts. Each run's sorted list of `(pass|fail) <test name>` lines is identical to baseline run 1 (`p1-behaviour-check.txt`). There's no new failure and none disappeared. Unlike on Windows, no test timed out, so every test reached its assertions. It is still test evidence, not proof: it shows only that the existing 217 tests can't tell the two versions apart.

## 4. What failed or deviated

- **Nothing that was asked for failed to run.**
- The CLI is 2.1.303, not 2.1.300. The newer installed version was kept, as the instructions allow. A control run of all four suites with 2.1.300 gave identical results (see M4).
- Python is 3.10.12, not 3.14.4. lizard 1.24.1 output is identical anyway.
- `ccn.mjs` resolves `typescript` relative to its own location, so an identical copy (checked with `cmp`) was run from a scratch dir that has `typescript@5.9.3` installed locally.
- `python3-venv` is missing, so lizard was installed with `pip --target` into a scratch dir.
- The first P2 `tsc` attempt (`--module nodenext`) failed with TS1470, as explained above. The second attempt with ESM flags passed.
- The P2 test file was not type-checked by `tsc` because it would need `@types/node`. It ran under `node --test`.
- **The repo state differs from the brief.** `personA/` is *tracked* in this checkout (it was committed in `b4e46ff`), not untracked. Nothing tracked was modified. All work was done in the separate worktree at `bf2323b` and in scratch copies.

## 5. Replacement text for `draft.md`

M4 paragraph (replaces "**Run results on Windows 11 (as measured):** … see §A.9."):

```
**Run results on Linux** (Ubuntu 22.04, Node 25.9.0, Claude Code 2.1.303,
the platform family CI uses): `sec-default` 162/162 pass, `agents-md` 5/5,
`telemetry` 34/34 (including the Linux-only test `on Linux the distribution,
kernel, WSL and EC2 come from their files`), and `diff` 217/217 in each of
three identical runs, with no timeouts and the same outcome for every test in
every run. An earlier run on Windows 11 failed 57/217 and then 31/217 in `diff`
(almost all `timed out after 5000 ms`) and 8/34 in `telemetry`. Three `diff`
assertion failures (`register > a merge finished {0.3, 1.2, 3} s after the
open is noticed`) recurred there but pass on Linux. The failures are therefore
environmental (platform and timing), not defects at the pinned commit, which
is itself a testability finding: the suite's verdict depends on the machine.
```

Re-assessment, P1 behaviour check (replaces "Behaviour check: `claude plugin test diff` on the modified copy gave 165 pass, 52 fail. … Re-run on Linux before claiming it in the report."):

```
Behaviour check: on Linux, `claude plugin test mods/diff` on the modified copy
passed 217/217 in each of three runs, with no timeouts, exactly as the
unmodified baseline did in three runs; the per-test outcomes are identical
with and without the patch. This is evidence, not proof, of unchanged
behaviour: it shows that the existing 217 tests cannot tell the two versions
apart.
```

Other places in `draft.md` that the Linux numbers make stale (not rewritten here):
- §A.7 P3.2: "M4 showed 57 → 31 failures between identical runs". This is still true for Windows, but it should say "on Windows; 0 on Linux".
- §A.8 P1 table: the row label "LOC (non-blank, lizard NLOC)" should read "LOC (sum of function NLOC, lizard)" (see §2).
- §A.9: the open point asking for a Linux M4 run is now resolved.

## Files in this folder

`versions.txt`, `m1.csv`, `m1ts.csv`, `m1-summary.txt`, `m1ts-summary.txt`, `m2-*.json`, `m2-summary.txt`, `m3-{plugins,scripts,mods}/jscpd-report.json`, `m4-{agents-md,sec-default,telemetry}.log`, `m4-diff-run{1,2,3}.log` (`m4-diff.log` is a copy of run 1), `m4-diff-p1-run{1,2,3}.log`, `m4-exitcodes.txt`, `mod-tests-L20-35.txt`, `p1-tsc.log`, `p1-m1ts-{before,after}.csv`, `p1-m2/m2-diff.json`, `p1-summary.txt`, `p1-behaviour-check.txt`, `p2-ccn-{before,after}.csv`, `p2-node-test.log`, `p2-tsc.log`, `p2-summary.txt`.
