# Tool routing for abap-dev skills

The skills work with whatever ABAP MCP servers the session has. Identify them by **tool name
suffix**, not server name — server names differ per project (`mcp-abap-adt`, `mcp-abap-adt-bgq`,
`vsp-dev`, …). A tool `mcp__<server>__RunATC` belongs to an mcp-abap-adt server; a tool
`mcp__<server>__RunATCCheck` belongs to a vibing-steampunk (vsp) server. If two ABAP servers
point at **different systems**, ask the user which system the task is about before the first call.

| Server | Recognise it by | Project |
|---|---|---|
| mcp-abap-adt | `RunATC`, `GetClass`, `UpdateClass`, `CheckClass` | github.com/fr0ster/mcp-abap-adt |
| vibing-steampunk | `GetSource`, `EditSource`, `RunATCCheck`, `RunUnitTests` | github.com/oisee/vibing-steampunk |
| abaplint | `LintAbap`, `FixAbap`, `ListAbapLintRules` | an abaplint MCP server (optional — the abap-review skill falls back to `npx @abaplint/cli`) |
| sap-docs | `search`, `fetch`, `sap_get_object_details` | github.com/marianfoo/mcp-sap-docs |

If no ABAP system server is connected, say so and ask the user to paste the source / results.
**Never invent source code, findings or test results.**

## Capability → tool

| Capability | mcp-abap-adt | vsp | Notes |
|---|---|---|---|
| Find objects | `SearchObject(object_name, object_type)` | `SearchObject(query)` | mcp-abap-adt answers tab-separated text |
| Objects of a package | `GetPackageContents(package_name, include_subpackages)` | — | |
| Objects of a transport | `GetTransport(transport_number, include_objects=true)` | — | |
| Read source | `GetClass` / `GetInterface` / `GetProgram` / `GetInclude` | `GetSource(name, object_type, method?)` | vsp can read a single method (`method=`) — cheaper for big classes |
| Syntax check | `CheckClass(class_name, source_code?)`, `CheckInterface`, … | `SyntaxCheck(object_url, content)` | `CheckClass` with `source_code` checks a draft without saving |
| Write source | `UpdateClass(class_name, source_code, transport_request?, activate)` / `UpdateProgram` | `EditSource(object_url, old_string, new_string, syntax_check=true)` | vsp `EditSource` = surgical replace, smaller diffs; it **activates by itself** when the syntax check passes |
| Activate | `ActivateClass` / `ActivateObjects` | `Activate(object_name, object_url)` | activation is the real compile check |
| ATC | `RunATC(objects=[{name,type}], check_variant?, wait=true)` → `GetATCFindings(worklist_id)` | `RunATCCheck(object_url, variant?)` | types: class, interface, function_group, package, ddl_source, table, behavior_definition |
| Unit tests | `RunUnitTest(class_name)` | `RunUnitTests(object_url, only_failures?)` | see known issue below |
| Where-used | `GetWhereUsed(object_name, object_type)` | `FindReferences(object_url, line, column)` | |
| Dependencies of a source | — | `GetContext(name, object_type)` | public API contracts of everything it references |
| Lint | — | — | abaplint `LintAbap(source_file, object_name, object_type)` — needs a **file**: write the source to a temp file first |
| Released-API status | — | — | sap-docs `sap_get_object_details(object_type, object_name, system_type, target_clean_core_level)` |
| SAP documentation | — | — | the `sap-kb` skill (KB first, then sap-docs `search`/`fetch`) |

**vsp `object_url` patterns:** classes `/sap/bc/adt/oo/classes/<name>`, interfaces
`/sap/bc/adt/oo/interfaces/<name>`, programs `/sap/bc/adt/programs/programs/<name>`, function
groups `/sap/bc/adt/functions/groups/<name>`. Lower-case the name; encode `/` in namespaced names
as `%2f` (`/sap/bc/adt/oo/classes/%2fui2%2fcl_json`).

## Known issues (check before trusting an error)

- **mcp-abap-adt `RunUnitTest` → 404 on `/sap/bc/adt/abapunit/runs`** on systems that only offer
  `/abapunit/testruns` (measured on S/4HANA 2023 on premise, SAP_BASIS 758; fr0ster/mcp-abap-adt#286).
  The tests are fine — use vsp `RunUnitTests` instead, or tell the user to run them in ADT.
- **vsp in `--read-only` mode** blocks writes but still runs ATC and unit tests.
- **ATC object types** in mcp-abap-adt do not include programs/includes in 17.1.0 — use vsp
  `RunATCCheck` with the program's `object_url`, or check the package.
- **sap-docs `found: false`** means "not in SAP's released-objects list", **not** "compliant".
