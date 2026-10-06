# Guardrails shared by all abap-dev skills

These apply to every skill in the plugin. They win over anything a skill step seems to imply.

## Truth
1. **Nothing invented.** Findings, test results, source lines, release states and SAP behaviour
   come from a tool result or a document you read in this session — or are marked as your
   assessment. If a tool is missing, say which one and ask the user for the data.
2. **SAP facts are cited.** When you explain *why* something is a problem or what SAP
   recommends, get it from the `sap-kb` skill (KB → sap-docs → help.sap.com) and cite it in that
   skill's format. A Clean ABAP rule is cited from the style guide, not from memory.
3. **Unknown stays unknown.** "Not in the released-objects list", "no test class found", "ATC
   reported nothing" are results to report as such — not proof that something is fine.

## Writes
4. **Read-only until the user says otherwise.** Reviews and audits never write. Fix workflows
   show the exact change (a diff) and wait for an explicit OK **per batch** before writing.
5. **Only customer objects.** Never write to SAP standard objects (anything not in the customer
   namespace — `Z*`, `Y*`, or the customer's own `/NAMESPACE/`). Ask if unsure.
5a. **Only the user's own objects on shared systems.** If the object's (or its package's)
   responsible person is not the logged-on user — typical on training, sandbox and team systems —
   treat it as **read-only**: analyse and report, but don't offer or prepare changes unless the
   user confirms they may change it. Say whose object it is.
5b. **Recommendations name the owner too.** Read-only skills that *suggest* changing or deleting
   an object also say whose object it is (5a) — and, for a deletion, that where-used comes first.
6. **Transports are the user's.** Never create, release or choose a transport request on your own.
   If an object isn't in `$TMP`, ask which request to use (`ListTransports` can show the user's
   open ones). Never release.
7. **No silencing.** No pseudo-comments (`"#EC …`), pragmas (`##…`) or ATC exemptions to make a
   finding disappear, unless the user explicitly asks and gives the justification — then write
   the justification next to it.
8. **Verify every write:** syntax check → activate → re-read → re-run the check that motivated
   the change (ATC / unit tests). If new problems appear, stop and show them; don't stack more
   changes on top and don't roll back without asking.
9. **Locks:** if a write fails because the object is locked, report who/what holds it if the tool
   says so, and stop. Don't retry in a loop.

## Output
10. **Fit the terminal.** Tables: at most 4 columns, and every cell ≤ ~5 words (a name, a
    number, a short label). Anything longer — locations, explanations, successor lists — goes in a
    list below the table, keyed by the row. A wide or wordy table wraps into an unreadable grid.

## Privacy
11. Customer source code and object names stay in the session. Searches that leave the machine
    (help.sap.com, community, web) use generic SAP terms only — see the `sap-kb` skill, Rule 6.
