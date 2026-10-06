---
name: clean-core-check
description: Read-only audit of custom ABAP code against SAP's released-objects list — which SAP tables, classes, function modules and CDS views it uses, their clean-core level (A released … D) on the target system type, and the released successors to move to. Use when the user asks for a clean core / ABAP Cloud readiness check, whether code uses unreleased SAP objects, or what to replace an SAP table/FM with. Never writes.
---

# clean-core-check — released-API audit (read-only)

Before the first tool call, read `../../reference/tools.md` and `../../reference/guardrails.md`.
This skill **never writes** to the system.

Requires the **sap-docs** MCP server (`sap_get_object_details`, `sap_search_objects`). Without it,
say so and stop — release states must come from SAP's list, not from memory.

## 1. Frame the audit
Ask (or take from context):
- **Target system type** — `on_premise`, `private_cloud`, `public_cloud` or `btp`. Release states
  differ per type; never assume.
- **Target level** — `A` (released APIs only; ABAP Cloud development) is the default. `B` also
  allows classic APIs. Levels: A = released, B = + classic, C = + internal/stable, D = everything.
- **Scope** — object(s), a package, or a transport.

## 2. Collect what the code uses
For each custom object in scope, read its source (see tools.md). Build the list of **SAP**
objects it references (skip `Z*`/`Y*`/customer namespace and local definitions):
- tables/views in `SELECT … FROM`, `JOIN`, `UPDATE/INSERT/MODIFY/DELETE`;
- function modules in `CALL FUNCTION '…'`;
- classes/interfaces in `TYPE REF TO`, `NEW`, `=>`, `CAST`, `INTERFACES`;
- CDS views/entities and data elements/structures used in `TYPE`.
vsp `GetContext(name, object_type)` lists the referenced **classes and interfaces** with their
contracts — a useful cross-check for those, but it does not list tables or function modules, so
the source read above is what finds them. Record the source line of every usage.

## 3. Look up every distinct SAP object
`sap_get_object_details(object_type, object_name, system_type, target_clean_core_level)`
(TADIR types: `TABL`, `VIEW`, `DDLS`, `CLAS`, `INTF`, `FUGR`, `DTEL`, …). Note `state`,
`cleanCoreLevel`, `complianceStatus` and `successor`/`successorObjects`.
- `found: false` → report as **unclassified** ("not in SAP's released-objects list for
  <system_type>") — do not call it compliant or non-compliant.
- For function modules not found under `FUNC`, try their function group (`FUGR`).
- Cite results as `[sap-docs: released-objects list, <system_type>]`.

## 4. Report
1. Summary: objects audited, SAP objects referenced, counts per clean-core level, unclassified.
2. **Findings above the target level**, worst first: SAP object, level/state, where used
   (object + line), and SAP's named successor(s). If SAP names no successor, say so — don't
   invent one.
3. Unclassified objects to verify manually (ADT → "API State" tab on the object).
4. Optional next step: offer to plan the migration of one finding (read-only plan; actual code
   changes go through the user's normal change process, not this skill). When you suggest
   changing or deleting an object, say whose it is (package responsible — guardrail 5a) and, for a
   deletion, that where-used has to be checked first.
