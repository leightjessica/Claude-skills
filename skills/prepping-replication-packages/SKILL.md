---
name: prepping-replication-packages
description: >
  Audit and prepare a Stata replication package for sharing with external
  researchers (or for journal/data-editor submission). Use this skill whenever
  the user asks to "proofread," "audit," "prep," "clean up," "sanity check,"
  or "get ready to share" a replication package, replication folder,
  replication archive, or set of do-files plus data. Also trigger when the
  user mentions "replication files," "replication package," "AEA data
  editor," "JDE replication," "data editor," or asks whether code "still
  runs," whether documentation is complete, or whether the package contains
  PII. Walks the user through master-script verification, path harmonization,
  do-file audit, output verification, README completeness, PII audit and
  removal, and final cleanup. Designed to be reused across papers.
---

# Prepping a Replication Package for Sharing

This skill is a structured audit checklist for getting a Stata replication
package ready to share with other researchers or to submit to a journal data
editor. It assumes the package is already roughly complete (do-files exist
and produce some output) and the goal is to certify that it is reproducible,
documented, portable, and free of PII.

The standard end-state of a clean package is:

- One `MASTER.do` at the top of the folder that runs the entire pipeline.
- A `README.md` that lists every file, every output, every required package,
  and every known discrepancy from the published paper.
- All paths set via a single `$repdir` global edited in one place.
- Every do-file uses the same `$repdir` pattern, with a fallback for
  standalone runs.
- A `Data/` folder that contains only the variables actually needed for the
  analysis — no GPS, names, addresses, free-text comments, or other PII.
- A `tables/` folder with one `.tex` file per table in the paper.
- No stray `.Rhistory`, `.DS_Store`, orphaned `.dta`, or developer-laptop
  artifacts.
- No internal-only backup folders, log files with hardcoded paths, or
  half-finished side scripts.

Work through the seven stages below in order. Each stage has a concrete
check, the typical defect found at that stage, and the standard fix. Report
findings to the user *before* applying destructive fixes — the package
contains the user's research data and most fixes touch real files.

---

## Stage 1: Verify the master script runs end-to-end

**Check.** Confirm a recent `MASTER.log` exists and ends with a "complete"
banner (or, if not, run `MASTER.do` fresh in batch mode and watch for any
`r(NNN)` errors). Do NOT trust individual sidecar `.log` files as proof of
end-to-end reproduction; only the master log matters.

```bash
"/c/Program Files/StataNow19/StataMP-64.exe" -b do MASTER.do
```

**Check the log for real errors.** Most "(file ... not found)" warnings
inside the log are benign (e.g. `est2tex` probing tempfiles before writing,
or `reshape long` noting that `cost2` exists but `revenue2` does not).
Real errors look like `r(NNN)` codes outside the controlled scope of a
`capture` block, or `invalid syntax`, or "no observations." Filter:

```
Grep pattern: ^r\(|invalid syntax|no observations|conformability error
```

**Typical defect.** A child do-file aborts because of a missing variable
or a missing user-written package that was never SSC-installed.

**Standard fix.** Add `ssc install` calls to MASTER.do for every external
command the package depends on. Confirm with a foreach loop guarded by
`capture which`.

---

## Stage 2: Harmonize all paths

**Check.** Grep the entire package for hardcoded paths and confirm every
do-file uses a single global. The user's preferred convention is `$repdir`.

```
Grep pattern: C:\\Users|/Users/|<username>
Files: *.do
```

Every match outside `MASTER.do` (and inside the `if "$repdir" == ""`
fallback block of each child do-file) is a defect.

**Check folder name case.** The actual folder name on disk must match the
case used in the do-file `use` statements. Windows is case-insensitive so
`Data\` vs `data\` works locally, but it breaks on Mac/Linux. Use
PowerShell to read the canonical case:

```powershell
(Get-Item "Replication files\Data").Name
```

Rename with a two-step pattern to force case change on Windows:
`Rename-Item data data_tmp; Rename-Item data_tmp Data`.

**Standard MASTER.do header.**

```stata
capture clear all
set more off

global repdir "C:\Users\<user>\...\Replication files"
global rawdir "C:\Users\<user>\...\clean data"   // optional; only for rebuild step

foreach pkg in est2tex parmest qqvalue diff {
    capture which `pkg'
    if _rc {
        di as txt "Installing required package: `pkg'"
        ssc install `pkg', replace
    }
}
```

**Standard child do-file header.**

```stata
capture clear all
set more off

if "$repdir" == "" {
    global repdir "C:\Users\<user>\...\Replication files"
    di as txt "Note: \$repdir not set externally; using default ${repdir}"
}
global tex "$repdir\tables"
```

This pattern lets MASTER.do drive the package end-to-end and also lets a
recipient run any single child do-file standalone (after one path edit).

---

## Stage 3: Audit each do-file

For every do-file, confirm it:

1. Has a header docstring listing inputs, outputs, and which paper table
   it produces.
2. Uses `${repdir}/Data/...` for inputs (matching the disk case from
   Stage 2).
3. Writes outputs to `${repdir}/tables/` (or `${repdir}/Data/` if it
   produces an intermediate `.dta`).
4. Does not write to absolute paths or to ad-hoc working directories.
5. Does not contain commented-out experimental code that confuses a
   reader. Move dead code to an `Archive/` folder outside the shipped
   package.

**Cross-check inventory.** Build a table of every `use`, `merge ... using`,
`append ... using`, and `save` line across the package:

```
Grep pattern: ^[^*]*(use |merge .* using |append .* using |save )
Files: *.do
```

This tells you exactly which `.dta` files the code reads and writes.
Cross-reference against the README's data-file table — every read should be
listed; every file in `Data/` not in this list is an orphan candidate.

---

## Stage 4: Verify outputs

**Check 1: every documented output exists.** The README should list every
`.tex` table and which paper table it corresponds to. Glob the `tables/`
folder and confirm the file count matches the README's count, and that
every documented filename is on disk.

```
Glob pattern: tables/*.tex
```

**Check 2: every output is non-empty.** A 0-byte `.tex` indicates a script
exited mid-table.

```bash
find tables -name "*.tex" -size 0
```

**Check 3: spot-check a few outputs against the published paper.** Open
two or three representative `.tex` files and verify the column headers,
row labels, and at least one numeric cell match what the published table
shows. Differences should be documented in the README's "Known
discrepancies" section.

---

## Stage 5: Cross-check the README

The README must contain, at minimum:

1. **Citation of the paper** with DOI.
2. **Quick start** instructions — how to set `$repdir`, how to run
   `MASTER.do`, what Stata version is required.
3. **Required user-written packages** with the SSC names.
4. **Inventory of do-files**: which paper table each one produces.
5. **Inventory of data files** in `Data/`: file name, row unit, brief
   description.
6. **Inventory of outputs** in `tables/`: which paper table each `.tex`
   corresponds to.
7. **Known discrepancies** vs. the published paper, with a one-line
   explanation per discrepancy. This is the most-read section of the
   README and the most underwritten — push the user to be thorough.
8. **Specification notes** on weighting, clustering, and any non-obvious
   choices (e.g. cash-only vs. with-own-consumption variables).

If a backup folder like `_original_backup_<date>/` exists, decide with the
user whether to keep it (transparency) or delete it (less clutter for
external researchers).

---

## Stage 6: PII audit and removal

**This is the highest-stakes stage.** Survey-derived analysis files almost
always carry survey-collection metadata that the analysis code never
references but that cannot be shipped: GPS coordinates, village
identifiers, photo-tracking-sheet filenames, free-text respondent
justifications (which routinely contain names), enumerator comments, and
exact visit timestamps.

**Step 1: write and run a PII audit do-file** that, for every shipped
`.dta`, lists (a) variable labels matching PII keywords, (b) variable
names matching PII keywords, and (c) every string variable with its first
3 distinct values. The script must be crash-resistant — wrap `levelsof`
calls in `capture` because pathological string values can break local
macro expansion.

A working template (forward slashes in paths to avoid backslash-escape
issues with Stata local macros, and individual `f1`/`f2`/... locals to
preserve multi-word filenames):

```stata
local nfiles 3
local f1 "analysis_endline"
local f2 "analysis_individual"
local f3 "program_exposure"

forvalues i = 1/`nfiles' {
    local f "`f`i''"
    use "$repdir/Data/`f'", clear

    qui ds
    foreach v of varlist `r(varlist)' {
        local lbl : variable label `v'
        if regexm(lower("`lbl'"), ///
            "name|phone|telephon|address|gps|latitud|longitud|coord|" + ///
            "enumerator|interview|respondent|nom |prenom|adresse|" + ///
            "email|village|locality|hamlet") {
            di as txt "  `v' : `lbl'"
        }
    }

    qui ds, has(type string)
    foreach v of varlist `r(varlist)' {
        di as txt "  STRING `v'"
        capture levelsof `v', local(vals) clean
        if !_rc {
            local n : word count `vals'
            local show = min(3, `n')
            forvalues j = 1/`show' {
                local val : word `j' of `vals'
                capture noisily di as txt "    [`j'] `val'"
            }
        }
    }
}
```

**Step 2: report findings to the user before dropping anything.** Group
the findings by file and by category (direct identifier / geographic /
free-text PII / quasi-identifier). For each finding, note whether the
variable is referenced anywhere in the do-files (it almost never is,
because PII variables are survey-collection metadata, not analysis
inputs).

**Step 3: ask the user which approach they want:**

- **Conservative (drop-list):** drop only the explicitly PII-flagged
  variables, leave everything else. Defensible, minimal disruption,
  doesn't risk dropping a control the user forgot to mention.
- **Aggressive (keep-list):** rebuild each `.dta` with only the
  variables actually referenced by some do-file. Maximum privacy, but
  brittle if any reference was missed.

The conservative approach is the default; only escalate to the
aggressive approach if the user explicitly asks or if the data is
exceptionally sensitive.

**Step 4: write a `Remove PII.do` script** that, for each affected file:

1. Backs up the original to `Data/_pre_pii_backup/` (non-destructive).
2. Loads the file.
3. Drops each PII variable using `capture drop` so re-runs are safe.
4. Saves over the shipped file.
5. Prints a count of variables dropped.

Group drop blocks by reason and put a one-line comment on each block:

```stata
* --- Direct identifiers: GPS coordinates ---
capture drop gps_lat gps_lon

* --- Geographic identifier ---
capture drop VillageID

* --- Free-text respondent-justification fields (contain person names) ---
capture drop q1_explain q5_explain_1 q14_explain_1 q14_explain_2
```

**Step 5: re-run MASTER.do on the cleaned data** and confirm every
output table still reproduces. If a do-file fails with `r(111) variable
not found`, the dropped variable was a *source* for a derivation in the
analysis code — not a column the regressions read directly, but a string
field that some `gen` line parses (a common case is `surveymonth =
substr(visit_date_str, 4, 2)` from a visit-date string). Two-part fix:

1. In `Remove PII.do`, **pre-compute** the derived variable as a numeric
   field *before* the `capture drop` of the source. Survey month or year
   alone is far less identifying than the raw timestamp.
2. In the analysis do-file, wrap the original `gen` in a `capture
   confirm variable` guard so the derivation runs only if the
   pre-computed variable is missing (i.e., on raw, pre-cleanup data).
   This keeps the do-file usable both on the shipped cleaned data and
   on a researcher's raw rebuild.

If a regression itself breaks (not a derivation), restore from
`_pre_pii_backup/` and reconsider whether that variable should have
been dropped.

**Step 6: document the PII removal in the README** — add a line under
"Data files" noting that the shipped files have been stripped of survey
metadata and listing the broad categories dropped (no need to list every
individual variable; the script is the audit trail).

**Common PII categories to look for** in any survey-derived analysis
file:

- **Direct identifiers:** GPS lat/lon, village name, household name,
  respondent name, phone number, address, email, photo URLs.
- **Free-text fields:** any string variable that holds enumerator
  comments or "please justify" answers — these almost always contain
  names. Sample a few values to confirm.
- **Quasi-identifiers:** exact visit dates and times, photo-tracking
  filenames, "index child still in village" / "why mother not in HH"
  fields, ethnicity in small subsamples.
- **Geographic:** VillageID, EnumeratorID, ClusterID at a finer
  granularity than the analysis cluster level. The cluster ID actually
  used for SE clustering must be preserved (e.g. CommuneID); finer
  identifiers usually can be dropped.

---

## Stage 7: Final cleanup

Before the user zips and ships the archive, sweep for:

- `.Rhistory`, `.DS_Store`, `Thumbs.db`, `.ipynb_checkpoints/` — IDE/OS
  artifacts.
- `_pii_audit.do`, `_pii_audit.log`, `_repro_test/`, scratch files —
  proofreading leftovers.
- Sidecar `.log` files for individual do-files. The `MASTER.log` is
  enough; individual logs duplicate it and bake in local paths. Either
  delete them or keep only `MASTER.log`.
- `_pre_pii_backup/`, `_original_backup_<date>/` — internal-only
  backups. Keep a copy outside the share folder; remove from the
  share folder.
- Any `.do` file inside `_original_backup_*` that contains a hardcoded
  pre-cleanup path — these will be grep-flagged later and worry
  recipients even though they don't run.

Final acceptance check: re-run MASTER.do one last time on the cleaned
package, confirm every documented `.tex` output exists in `tables/`, and
confirm the master log ends with the "complete" banner.

---

## Reporting style

When walking the user through findings, **report, do not argue.** Group
issues into:

1. **Issues that affect external researchers** — must fix (broken paths,
   missing files, PII).
2. **Issues worth deciding on** — judgment calls (delete backup folder?
   keep individual logs?).
3. **Things that look correct** — what passed, so the user knows the scope
   of the audit.

Do not apply destructive fixes (deleting files, overwriting `.dta` files)
without first listing what will be done and getting explicit user approval.
The package contains real research data and most fixes are hard to undo.
