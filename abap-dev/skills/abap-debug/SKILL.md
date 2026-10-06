---
name: abap-debug
description: Debug ABAP on the connected system with vibing-steampunk — set a narrowly conditioned breakpoint, wait for the code to be triggered from another session, inspect the call stack and variables, step, and always clean up; or analyse a short dump post-mortem. Built for shared systems where many people run as the same SAP user. Use when the user asks to debug, set a breakpoint, see what a variable holds at runtime, step through code, or investigate a runtime error / short dump.
---

# abap-debug — breakpoints, stepping and dumps

Before the first tool call, read `../../reference/tools.md` (debugger tools + known issues) and
`../../reference/guardrails.md`. Needs a **vsp** server (`SetBreakpoint`, `DebuggerListen`). Without
one, say so — mcp-abap-adt has no debugger.

## How external debugging works (explain this to the user once)
A breakpoint set here is **external and user-scoped**: it stops *any* request that runs as that SAP
user, in any session — not just the user's. `DebuggerListen` only waits; it never runs code. The
code has to be **triggered from somewhere else** (a UI click, an HTTP call, SAP GUI, a separate test
run), while the listener waits (max 240 s). The trigger then hangs until you continue.

## 0. Is there a dump already?
If the question is about a runtime error, start post-mortem: `ListDumps(program?, user?, since?)` →
`GetDump(dump_id)`. Often the termination point and stack answer it without a live session. Go live
only if you need values the dump doesn't show.

## 1. Plan before touching the system
Agree with the user, in one short message:
- **System**: never set breakpoints on a production system. If the system role is unclear, ask.
- **Where**: read the live source first (`GetSource`, `method=` for one method) — the code may have
  changed since anyone last looked. Pick the line by its **statement**.
- **Whose requests**: which SAP user will run the code? The breakpoint must be for that user. Ask the
  user for the name; don't read credential files to find out.
- **Shared user?** If several people/processes run as that user (training, sandbox, integration
  users), a plain breakpoint will catch **their** requests too. Then the breakpoint needs a
  **condition on a value only this test uses** — a made-up material number, search term or test
  record — never a value others are likely testing right now.
- **Coordination**: if the project has a coordination channel or board for the shared system,
  announce the breakpoint there first and check nobody else is testing the same object.
- **Trigger**: who runs the code and how (see step 3).

## 2. Set the breakpoint
`SetBreakpoint(program=<class/program/FM>, line=<line in the source as vsp reads it>,
condition=<ABAP condition>)`. Statement and exception breakpoints are possible
(`kind='statement'`/`'exception'`) — on a shared user they are even broader, so condition them too.
Check with `GetBreakpoints`. The first call takes ~20–25 s.

## 3. Listen, then trigger
- **User triggers** (most common): call `DebuggerListen(timeout=120..240)`, and in the same message
  ask the user to trigger *now* with the exact input (including the unique condition value).
- **You trigger** (e.g. an HTTP call the user approved): start the trigger **in the background with
  a delay** (e.g. `sleep 8 && curl …` as a background shell command) *before* calling
  `DebuggerListen`. The trigger can't go through the same vsp server — its own calls never stop at
  its own breakpoints.
- Timeout without a stop → report it, delete the breakpoint, don't loop.

## 4. When it stops — confirm it's yours
Read `DebuggerGetStack` and the input variables (`DebuggerGetVariables` with the relevant names).
**Check the values match your trigger** before treating anything as the test result. If they don't
(someone else's request), say so, `stepContinue` immediately, and rethink the condition.

## 5. Inspect and step
- Variables: `DebuggerGetVariables` (no argument = locals; pass names, or a composite id to expand a
  structure/table).
- Steps: `stepOver`, `stepInto`, `stepReturn`. `stepRunToLine` doesn't work in current vsp — use
  repeated `stepOver` and track the statement you're on.
- **Never** `stepJumpToLine` (skipping statements can leave data inconsistent) unless the user asks
  for it explicitly and understands that. Don't change variable values.
- Keep the stop short — the triggering request (and its user) is waiting and may time out.

## 6. Release and clean up — always
`stepContinue` (an `AdiFailed` / `debuggeeEnded` answer is vsp's reply *after* the continue released
the request and it ran to the end — the continue worked; it does not mean the request had already
finished on its own), then
`DeleteBreakpoint('all')` and `GetBreakpoints` to confirm none are left, then `DebuggerDetach`. Do
this even if something went wrong earlier. Remove the coordination note if you posted one.

## 7. Report
What stopped where (statement + stack summary), the variable values that answer the question
(as a short list `name = value — note`, not a table: values are often too long for a cell), what
the step(s) showed, whether the stop was confirmed as your trigger, and that the breakpoint is gone.
Mark anything you inferred rather than saw in the debugger as your assessment.
