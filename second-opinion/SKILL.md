---
name: second-opinion
description: >
  Spawns a skeptical critic subagent to pressure-test the current plan, proposal,
  or approach before execution. The critic actively inspects the codebase to verify
  claims, then returns severity-ranked concerns. The main agent must make a hard
  call on each point and show what actually changed.
allowed-tools: "Agent, Read, Grep, Glob, Bash, AskUserQuestion"
---

# Second Opinion

Spawns an isolated critic subagent to find problems with the current plan or proposal.
The critic reads the codebase to verify the plan's claims — it doesn't just reason
about them abstractly. After the critique, the main agent must make hard calls and
show concrete changes, not hand-wave.

---

## Workflow

### Step 1 — Build the briefing

Before spawning the critic, assemble a complete briefing. The critic starts cold with
zero context — anything you don't include, it can't use. Include:

1. **The proposal** — what you intend to do, in full. Not a summary. The actual plan
   with specific files, functions, and approaches.
2. **Assumptions** — what you're taking for granted. Be honest. The critic will find
   them anyway; listing them upfront saves it time and gets you a better critique.
3. **Project context** — language, framework, relevant architectural patterns. Run a
   quick `ls` of the project root and include the output so the critic can orient.
4. **File paths** — every file the plan touches or depends on. The critic will open
   each one.

If the critique target is unclear or too vague to brief against, ask the user to
clarify before proceeding. A vague input produces a vague critique.

### Step 2 — Spawn the critic

Use the Agent tool to spawn the `critic` subagent. Pass the full briefing as the
prompt. Do not duplicate or paraphrase the subagent's own instructions — just send
the briefing.

### Step 3 — Present the raw critique

Show the user the full critique output before you respond to it. Do not filter,
summarize, or editorialize at this stage.

```
-- SECOND OPINION ------------------------------------------------
[full critique output]
------------------------------------------------------------------
```

### Step 4 — Triage and decide

Go through each numbered critique. For every point, make a hard call:

**ACCEPT** — The critique is valid. State the specific change to the plan. Not
"we'll be more careful" — what concretely changes? A file, a step, an approach.

**REJECT** — The critique doesn't hold. Provide evidence: a file you read, a
constraint the critic didn't know about, a reason grounded in the codebase.
"Not relevant" is not a rejection. Show your work.

Format as a numbered list matching the critique. Lead each item with `ACCEPT` or
`REJECT` in bold.

### Step 5 — Show the delta

After triaging, produce two things:

**Changed plan** — If any points were accepted, show the updated plan with changes
marked. Not a description of changes — the actual updated plan. If the user can't
diff the old plan from the new one, this step failed.

**Remaining risks** — What the critique surfaced that you accepted as known risks
rather than changing the plan for. These are things the user should be aware of
going in.

If all points were rejected, state that the plan is unchanged and briefly summarize
why the critique didn't alter the approach.

Then ask the user whether to proceed or if they want to discuss any of the flagged
risks.
