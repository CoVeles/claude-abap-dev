---
name: abap-unit
description: Run ABAP Unit tests for a class, program or function group on the connected system, with the right tool for the system (works around the /abapunit/runs 404 on S/4HANA 2023), explain failures by reading the failing test and the code under test, and write new local test classes on request (shown first, test include only, harmless risk level). Use when the user asks to run, write or fix unit tests, check if tests pass, or investigate a failing test.
---

# abap-unit — run and explain ABAP Unit tests

Before the first tool call, read `../../reference/tools.md` and `../../reference/guardrails.md`.

## 1. Run
- **Class, mcp-abap-adt present:** `RunUnitTest(class_name)`.
  If it fails with **404 on `/sap/bc/adt/abapunit/runs`**, that's the known endpoint issue
  (tools.md) — not a test problem. Switch to vsp.
- **vsp present (or after the 404):** `RunUnitTests(object_url)`. Default risk levels only:
  pass `include_dangerous=true` / `include_long=true` **only** if the user explicitly asks — tests
  with RISK LEVEL DANGEROUS may change data.
- **Neither works:** say what failed and ask the user to run the tests in ADT (Ctrl+Shift+F10)
  and paste the result.

## 2. Read the result honestly
- Report counts: test classes, methods, passed, failed, errors, not run.
- **No test class found / 0 methods ran is not "green".** Say the object has no (runnable) tests.
- A test class that aborts (setup failure, runtime error) is a failure even if no assertion failed.

## 3. Explain failures
For each failing method: read the test method (local test classes via `GetLocalTestClass` or
vsp `GetSource(..., include='testclasses')`) and the production code it exercises. State:
- what the test expects (the assertion, `exp`/`act`),
- what the code does instead, with line numbers,
- the most likely cause — marked as your assessment unless the code makes it certain.

## 4. Fixing
Propose fixes to **production code** first. Changing a test's expectation to make it pass needs
the user's explicit confirmation that the old expectation was wrong. Writes follow the batch rules
of the `atc-fix` skill (diff → OK → write → check → activate), then re-run the tests.

## 5. Writing new tests (when the user asks for them)
- Read the class first; list the behaviours worth testing and which need the database. Show the
  **complete test class** and wait for an explicit OK before writing (guardrails 4, 5a).
- Touch only the **test-class include** (mcp-abap-adt `UpdateLocalTestClass`, or vsp
  `WriteSource(..., include='testclasses')` / `test_source`). The production class stays unchanged;
  to reach private methods use `CLASS <cut> DEFINITION LOCAL FRIENDS <test class>.`
- `RISK LEVEL HARMLESS`, `DURATION SHORT`. No real database writes: test DB-free logic directly;
  for SQL use the ABAP SQL test double framework (`cl_osql_test_environment`) or CDS test doubles —
  look up the API with the `sap-kb` skill before using it.
- After writing: activate, run the tests (section 1), report counts.
- If the project keeps ABAP sources in a git repo, point out that the new test include exists only
  on the system until it is added to the repo.
