---
name: abap-review
description: Review ABAP code for Clean ABAP and robustness — abaplint for the mechanical rules, then a reasoned read for what linters miss (naming, method size, error handling, SQL, side effects), each rule cited from the Clean ABAP style guide. Use when the user asks to review, critique or check the quality of an ABAP class, program or snippet, or before a pull request / transport. Read-only unless the user asks for fixes.
---

# abap-review — Clean ABAP review

Before the first tool call, read `../../reference/tools.md` and `../../reference/guardrails.md`.
Read-only: report findings; change code only if the user asks, then follow the batch rules of the
`atc-fix` skill (diff → OK → write → syntax check → activate → re-check).

## 1. Get the code
From the system (see tools.md) or from what the user pasted/attached. Note the object type and
the ABAP language version if the user knows it (ABAP Cloud vs Standard) — some rules differ.

## 2. Mechanical pass — abaplint
Write the source to a temp file named like abapGit does (`zcl_x.clas.abap`, `zprog.prog.abap`).
- **abaplint MCP server connected** (`LintAbap` tool): `LintAbap(source_file, object_name,
  object_type)`; `GetRuleDetails(rule_key)` explains an unfamiliar rule.
- **Otherwise, if Node is available:** run the public CLI on a folder holding just that file:
  `npx @abaplint/cli` with a minimal `abaplint.json` (ask before installing anything; run it in
  a fresh temp folder).
- **Neither:** skip this pass and say the review has no linter results.
Keep rule key, line and message of every issue. Don't repeat abaplint's findings in your own
words as if you found them — group them under "abaplint".

## 3. Reasoned pass — what linters miss
Read the whole source. Look for, and only report with line numbers:
- names that don't say what things are/do; Hungarian/prefix noise beyond the team convention;
- methods doing several things, deep nesting, long parameter lists, boolean flag parameters;
- error handling: swallowed exceptions (`CATCH` with no handling), `sy-subrc` not checked,
  classic exceptions where class-based ones fit;
- SQL: `SELECT *` where fields suffice, `SELECT` in loops, missing `WHERE` on writes;
- hidden side effects, global state, code that's hard to unit-test (no seams for dependencies);
- dead code, commented-out code, comments that restate the code.

**Cite the rule** for each reasoned finding from the Clean ABAP style guide via the `sap-kb`
skill (it reaches the guide through sap-docs, source `sap-styleguides`). If you can't find the
rule in the guide, label the finding "assessment" rather than presenting it as a guideline.
Respect a team convention the user states over the guide, and say where they differ.

## 4. Report
Ranked: **bugs/risks** (behavioural problems) → **clean code** (maintainability) → **style**
(abaplint, naming). Per item: line(s), what, why (cited), a concrete suggestion (small code
snippet). End with 2–3 highest-value changes. With the abaplint MCP server, offer
`FixAbap(..., dry_run=true)` for fixable issues — never `write_back` without an explicit OK.
