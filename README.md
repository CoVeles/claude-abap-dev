# abap-dev — ABAP workflows for Claude Code

A Claude Code plugin with five ABAP development workflows that run on the community MCP
servers you may already use — no SAP-specific IDE plugin needed:

| Skill | What it does | Writes? |
|---|---|---|
| `/abap-dev:atc-fix` | Run ATC on an object, package or transport; group findings by check; explain each (cited); fix in batches you approve; re-run ATC after every batch | only after your OK, per batch |
| `/abap-dev:clean-core-check` | Audit which SAP tables, classes, FMs and CDS views the code uses, their clean-core level (A–D) for your system type, and SAP's named successors | never |
| `/abap-dev:abap-review` | Clean ABAP review: abaplint for mechanical rules + a reasoned read for what linters miss, each rule cited from the Clean ABAP style guide | never, unless you ask |
| `/abap-dev:abap-unit` | Run ABAP Unit with the right tool for your system, explain failures from the test and the code under test | only after your OK |
| `/abap-dev:abap-debug` | Breakpoints with narrow conditions, wait for the trigger from another session, stack/variables/stepping, guaranteed cleanup; or short-dump analysis | never changes code or variables |

The skills also trigger on plain requests ("fix the ATC findings in ZCL_ORDER", "is this package
clean core?", "review this class", "run the unit tests", "debug this method with input X").

## Built on

| Server | Used for | Required? |
|---|---|---|
| [mcp-abap-adt](https://github.com/fr0ster/mcp-abap-adt) and/or [vibing-steampunk](https://github.com/oisee/vibing-steampunk) | system access: read/write source, ATC, unit tests, activation | at least one |
| an abaplint MCP server (tools `LintAbap`, `FixAbap`) — or the public [abaplint CLI](https://github.com/abaplint/abaplint) via `npx @abaplint/cli` | lint + fix previews (abap-review) | optional |
| [mcp-sap-docs](https://github.com/marianfoo/mcp-sap-docs) | SAP's released-objects list (clean-core-check), style guide and docs | for clean-core-check |
| [claude-sap-kb](https://github.com/CoVeles/claude-sap-kb) | cited SAP documentation answers (`sap-kb` skill) | recommended |

The skills detect the servers by their tool names, so server names in your config don't matter.
`abap-dev/reference/tools.md` maps every step to the tool on each server and lists known issues
(e.g. the ABAP Unit endpoint gap on S/4HANA 2023 — fr0ster/mcp-abap-adt#286 — which the
`abap-unit` skill works around via vsp).

## Install

```bash
claude plugin marketplace add CoVeles/claude-abap-dev
claude plugin install abap-dev@claude-abap-dev
```
Then open a new Claude Code session in a project where your ABAP MCP servers are configured.

## Guardrails

All skills share `abap-dev/reference/guardrails.md`. In short:
- **Nothing invented** — findings, results and release states come from a tool or a document read
  in the session; SAP facts are cited; "not found" is reported as unknown, not as fine.
- **Read-only by default** — writes only after you approve the exact diff, per batch; only
  customer-namespace objects; activation and a re-run of the check verify every write.
- **Transports are yours** — the skills never create, choose or release a transport request.
- **No silencing** — no `"#EC`, pragmas or exemptions to hide a finding unless you ask and
  give the justification.
- **Privacy** — customer code and names never go into web searches.
- **Shared systems** — breakpoints are user-scoped and catch *every* request of that SAP user, so
  `abap-debug` insists on a condition with a value only your test uses, confirms the stop came
  from your trigger, and always deletes its breakpoints.

## Tested against

S/4HANA 2023 on premise (SAP_BASIS 758) with mcp-abap-adt 17.1.0, vibing-steampunk 2.60.0,
an abaplint MCP server (core 2.120) and mcp-sap-docs 0.3.55:

- every tool call the skills prescribe, including a full ATC → fix → activate → re-check cycle on
  a throwaway `$TMP` class;
- the four v0.1 skills in real Claude Code sessions on real custom code — ATC triage of a demo-data
  class, a clean-core audit of a 33-object RAP package (spot-checked against SAP's list), an
  18-test ABAP Unit run, and a Clean ABAP review whose top finding (a material-number conversion
  bug) was then proven on the system.

Feedback from those runs is already in the skills (ownership checks on shared systems, narrow
tables for terminal output, a section on writing new tests). `abap-debug` (v0.2.0) comes from live
debugging on the same system — including the lesson that on a shared SAP user a breakpoint can catch
someone else's request — and was then run as a skill: a breakpoint conditioned on a unique test
value, triggered over HTTP from a second session, confirmed as the right request, cleaned up.

## Credits

Workflow ideas (batched ATC remediation without suppressions, clean-core report structure) were
inspired by [matt1as/claude-abap-skills](https://github.com/matt1as/claude-abap-skills)
(Apache-2.0); the skill texts here are written from scratch for these MCP servers.

## License

MIT — see [LICENSE](LICENSE).
