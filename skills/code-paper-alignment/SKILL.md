---
name: code-paper-alignment
description: Code and paper alignment audit for an empirical paper. Re-runs the analysis pipeline so all outputs are current, reconciles every number and table in the manuscript against the regenerated output, checks that methodology descriptions (main text, footnotes, table and figure notes, appendices) match what the code actually does, and finds hard-coded numbers that should be code-generated macros. Corrects the paper by default and flags only major discrepancies. Use when the user asks for a "code and paper alignment" check, a "code-to-paper audit", to "make sure the paper matches the code", to "check the numbers in the paper", to "sync the paper with the output", or before submitting or circulating a draft.
---

# Code and Paper Alignment

*Audit that the manuscript says what the code does and reports what the code produces.*

**Argument:** `$ARGUMENTS` (optional): path to the manuscript (`.tex`) and/or the master script. If omitted, locate them from the project README, CLAUDE.md, or memory.

**Flags:**
- `norun`: skip step 1's pipeline run and audit against existing outputs (say so in the report)
- `only:1,2,3`: run a subset of the three components
- `flagall`: flag every discrepancy instead of correcting minor ones

## Correction policy (applies to all components)

The default is to **correct the paper** so that it matches the code and its output. Minor fixes are made directly and logged. **Major** discrepancies are not fixed silently: list them for the user and ask before editing either the paper or the code.

A discrepancy is **major** when any of the following holds:
- A treatment effect changes sign, crosses a conventional significance threshold (10, 5, or 1 percent), or changes in magnitude by more than roughly 20 percent of the old value (or by more than 0.05 SD).
- A sample size changes by more than a handful of observations, or a sample restriction differs from the description in a way that changes who is in the sample.
- The estimator, clustering level, fixed effects, weighting, control set, or multiple-testing correction described differs from what the code does.
- The description implies a departure from (or adherence to) the pre-analysis plan that the code contradicts.
- The fix would change an interpretive claim (a sentence that says an effect is "large", "null", "statistically significant", "robust").
- The code looks wrong, rather than the text (for example, a variable constructed in a way no reasonable description would endorse). Never edit analysis code to make it match the paper without asking.

Everything else (rounding, stale numbers that move slightly, wording of notes, a missing detail in a footnote) is **minor**: fix it and log it.

Never modify data. Never delete or move files. Edits to code are limited to adding macro exports (component 3), and only after the user approves when the change touches an analysis do-file in a substantive way.

## Step 0: Orient

1. Read the project README, any CLAUDE.md, and relevant memory to find: the manuscript and its `\input` files (tables, figures, macro files); the master script and the pipeline order; where outputs are written and how they reach the paper folder (direct write vs. copy); the macro system used for in-text numbers.
2. Check whether the paper folder is under version control (`git status`). If it is, note the starting commit so all edits can be reviewed as a diff. Do not commit unless asked.
3. Snapshot the current outputs before re-running: copy the output tables folder(s) and macro files to the scratchpad, so old and new values can be diffed.
4. Set up the task list with the three components plus the report.

## Component 1: Re-run the pipeline and reconcile outputs

1. Run the master script in batch mode (for Stata: `StataMP-64.exe -b do master.do`, run in the background; long pipelines can exceed the tool timeout). Steps that need confidential data, interactive runs (e.g. R cleaning notebooks), or internet access may be switched off in the master; respect those switches and record which steps did not run.
2. Read the log for errors (`r(...)` codes), and also for silent warnings: dropped observations, `(n missing values generated)` on key variables, failed merges (`_merge` counts), `note: ... omitted because of collinearity`, convergence failures.
3. Diff every regenerated output against the snapshot (tables, macro files, figures by timestamp and size). Classify each change as unchanged, minor, or major using the policy above.
4. Confirm every file the manuscript `\input`s or `\includegraphics` exists and is the freshly generated version. If outputs are copied to the paper folder by hand rather than written there, check that the paper copy matches the output copy; update stale copies (minor) unless the paper copy carries deliberate hand edits (check README notes and the git log before overwriting).
5. Check that every macro key used in the manuscript is defined in the regenerated macro files, and list keys that are defined but unused (possible dead output) and keys that are used but undefined (would print as errors).
6. Compile the paper if a TeX engine is available, with the output going to the scratchpad. **Count every `^! ` line in the log** (`grep -c '^! '`), not just the first screenful of warnings: undefined control sequences inside `\input` tables (for example, esttab's `\sym`) are easy to miss among hundreds of citation warnings. Also check for undefined references and missing files.

### Lessons from earlier runs
- **Verify a variable against its questionnaire item, not its label.** A variable labelled "end of first eligible year" was built from September enrollment items. Read the survey wording (the `.docx` instrument) for every item a key outcome uses.
- **When a check is run in Stata, use `set varabbrev off`.** With abbreviation on, `confirm variable a5_k` matches `a5_k_eq_bl`, and drop lists can remove the wrong variables.
- **Indices.** `swindex` applied to a single summed variable only standardizes it. An Anderson index needs the items as the varlist. Restrict to respondents, so that non-respondents coded 0 by `egen rsum()` do not enter the normalization.
- **Covariates centred for an "effect at the mean"** must be centred within each regression's own sample.
- **Never print data values when scanning for PII.** List variable names, labels, and counts of distinct values only.

## Component 2: Methodology descriptions vs. code

Read the full manuscript, including footnotes, table and figure notes, and appendices. For each methodological statement, find the code that implements it and check that they agree. Cover at minimum:
- **Sample:** inclusion and exclusion rules, subsample definitions, attrition handling, the sample size reported vs. `e(N)`.
- **Variables:** construction of outcomes, indices (components, standardization, reference group), winsorization or trimming, treatment of missing values, top-coding, unit of measurement and recall period.
- **Specification:** estimator, controls (and whether baseline values of the outcome are included), fixed effects (strata, randomization phase), weights, the clustering level, and the inference procedure (randomization inference, wild bootstrap, multiple-testing adjustments such as Anderson q-values or Romano-Wolf).
- **Table and figure notes:** each note's description of the dependent variable, sample, controls, fixed effects, standard errors, significance stars, and what the reported rows are (control mean, SD, p-values) must match the code that writes that table.
- **Cross-references:** numbers of tables, panels, columns, and appendix sections referred to in the text.
- **Pre-analysis plan:** where the paper claims adherence to or lists deviations from the PAP, check the claim against the code and the PAP document if available.

Apply the correction policy. Typical minor fixes: a note that says "robust standard errors" when the code clusters by the same unit described elsewhere; a missing control listed in one note but present in the code; a stale table reference. Typical major flags: a different clustering level or estimator; an undisclosed sample restriction; an index built from components that differ from the description.

## Component 3: Hard-coded numbers that should be macros

1. Scan the manuscript text, footnotes, captions, and notes for literal numbers (regex for digits, percentages, decimals, and spelled-out counts in the vicinity of statistical language).
2. Classify each one:
   - **Analysis-derived** (sample sizes from the data, coefficients, control means, percentages, p-values, shares, counts of observations or clusters, costs computed in code): should be a macro.
   - **Design or context facts** (transfer amounts, dates, number of arms, power-calculation parameters, exchange rates fixed by protocol), **citations and external benchmarks**, and **structural numbers** (table numbers, years, grade levels): leave as typed.
3. For each analysis-derived literal: if a macro already exists for that quantity, swap it in (after confirming the value matches the regenerated output). If none exists, add an export to the do-file that produces the quantity using the project's existing macro system (for example, a helper program that writes `\newcommand` definitions to a `.tex` file), re-run that do-file, and swap it in. If adding the export requires a substantive change to an analysis script, propose it and ask first.
4. Respect project-specific macro syntax (for example, LaTeX macros followed by a space need `{}`).

## Report

Write `code_paper_alignment_<YYYY-MM-DD>.md` in the project root (or the folder the user names). Structure:
1. **Summary:** what ran, what did not run and why, counts of minor fixes made and major items flagged.
2. **Major discrepancies requiring a decision:** each with location (`file:line`), what the paper says, what the code does or produces, old vs. new value, and the options.
3. **Corrections made:** a table of location, old text or value, new text or value, and the reason.
4. **Macro coverage:** literals converted, literals deliberately left as typed (with the category), and new macro exports added.
5. **Pipeline health:** log warnings, missing or orphaned outputs, unused or undefined macro keys, compile warnings.

Write the report in the user's voice conventions if a CLAUDE.md specifies them, but keep it as a working document (lists and tables are fine). Then summarize the major items in chat and ask how to resolve each.
