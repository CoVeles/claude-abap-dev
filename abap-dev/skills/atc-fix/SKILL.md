---
name: atc-fix
description: Run ATC (ABAP Test Cockpit) on a class, package or transport, group the findings by check, explain each with a cited source, and fix them in confirmed batches — re-running ATC after every batch. Use when the user asks to run ATC, fix/remediate/triage ATC findings, clean up check results, or prepare an object for release. Never suppresses findings with pseudo-comments unless the user explicitly asks.
---

# atc-fix — ATC findings, fixed methodically

Before the first tool call, read `../../reference/tools.md` (which tool does what, known issues)
and `../../reference/guardrails.md` (rules that win over every step below).

## 1. Scope
- Pin down **what** to check: one object, several, a package (`GetPackageContents`), or a
  transport (`GetTransport(..., include_objects=true)`). Confirm the list if it has more than ~10
  objects or contains anything outside the customer namespace (those are check-only, never fixed).
- **Ownership** (guardrail 5a): note each object's package and responsible person
  (`GetPackage(package_name)` shows the package's; ask if no tool shows it). Objects that aren't
  the user's are triaged but not fixed unless the user confirms they may change them.
- Check variant: use the system default unless the user names one. (There is no tool to list
  variants yet — ask the user if they need a specific one.)

## 2. Run
- mcp-abap-adt: `RunATC(objects=[{name, type}], check_variant?, wait=true)` → take `worklist_id`
  → `GetATCFindings(worklist_id)`.
- Programs/includes, or no mcp-abap-adt: vsp `RunATCCheck(object_url, variant?)`.
- Keep the raw counts (by priority) — they are the "before" baseline.

## 3. Triage (show this before touching anything)
Group findings by **check title**, sorted by priority (1 = error, 2 = warning, 3 = info), then by
count. Show a compact table — **Prio | Check | Count | Class** (short labels; guardrail 10) — and
under it, per group, the affected objects/lines and a short reason for the classification:

| Class | Meaning | Typical handling |
|---|---|---|
| **Mechanical** | Fix is local and behaviour-neutral (unused variable, obsolete statement form, missing text element) | batch-fixable |
| **Behavioural** | Fix changes runtime behaviour (missing `WHERE` in `UPDATE`/`DELETE`, missing authority check, performance) | one at a time, explain impact, unit tests after |
| **Needs a decision** | Possibly intended (e.g. a deliberate full-table delete in a reset utility) | ask the user; an ATC exemption in ADT is *their* call |

Read the source lines of each finding before classifying (`GetClass`, or vsp `GetSource` with
`method=` for big classes). Don't classify from the message text alone.

For *why* a check matters or what SAP recommends, use the `sap-kb` skill and cite it. If nothing
citable is found, say "assessment, not from docs".

## 4. Fix in batches
Propose one batch at a time (all mechanical findings of one object, or a single behavioural
finding). For each batch show:
- the finding(s) it addresses,
- the exact diff,
- for behavioural changes: what changes at runtime and how it can be verified.

**Wait for an explicit OK.** Then, per guardrail 8:
1. write (`UpdateClass(..., activate=false)` then `CheckClass`, or vsp `EditSource(..., syntax_check=true)`),
2. activate (`ActivateClass` / vsp `Activate`) — activation errors stop the batch,
3. re-run ATC on that object and compare with the baseline,
4. if the object has unit tests, run them (skill `abap-unit`).

If the re-run shows **new** findings or tests fail: stop, show the delta, ask how to proceed.

## 5. Report
Before/after counts per priority, what was fixed, what was left and why (needs decision, outside
namespace, user declined), and any finding you could not verify. Mention the transport used for
each changed object, or that it stayed in `$TMP`.
