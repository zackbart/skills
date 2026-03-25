---
name: ethos
description: >
  Generate an ethos/ folder that captures a project's vision, principles, personas,
  and non-goals — the 50k-foot "why behind the what." Use when the user wants to
  establish project philosophy, define guiding principles, document who the product
  is for, or create foundational context that agents and humans can reference for
  decision-making. Triggers: "create an ethos", "define project principles",
  "document our vision", "set up project philosophy", "who is this product for."
argument-hint: "<init | audit | refresh [vision|principles|non-goals]>"
allowed-tools: "Write, Read, Glob, Grep, AskUserQuestion"
---

# Project Ethos Generator

Generate and maintain an `ethos/` folder that gives agents and humans the strategic
context behind a project. Three documents, all interview-generated:

- **vision.md** — Why this exists, who it's for, what success looks like
- **principles.md** — Technical and product tradeoffs, guardrails, beliefs
- **non-goals.md** — What the project intentionally avoids

## Modes

Detect the correct mode from context. If the user says "ethos" without specifying,
check whether `ethos/` exists in the project root:
- If no: run `/ethos:init`
- If yes: run `/ethos:audit`

### Mode 1: `/ethos:init` — Generate Ethos

**Trigger:** No `ethos/` folder exists, or user explicitly asks to create one.
**What it does:** Adaptive interview → generates all 3 files → adds CLAUDE.md pointer.
**Read before running:** `references/output-formats.md`

### Mode 2: `/ethos:audit` — Audit Existing Ethos

**Trigger:** `ethos/` folder exists and user wants to check if it's still accurate.
**What it does:** Reads ethos docs, scans codebase, reports completeness and relevance.

### Mode 3: `/ethos:refresh` — Update a Specific Document

**Trigger:** User wants to update a specific ethos doc (e.g., `/ethos:refresh vision`).
**What it does:** Reads the existing doc → targeted re-interview → surgical update.
**Read before running:** `references/output-formats.md`

---

## Mode 1: Init — Full Protocol

This skill is fundamentally an interview tool. Unlike clarification questions in other
skills, the interview here IS the product — it surfaces beliefs the user hasn't
articulated yet. The adaptive flow asks core questions first, then follows up where
answers reveal something interesting.

### Step 1: Check for Existing Ethos

Use Glob to check if `ethos/` already exists in the project root.
- If it exists: warn the user and ask if they want to overwrite or run audit instead.
- If not: proceed.

### Step 2: Read Output Templates

Read `references/output-formats.md` to understand the target format for all 3 documents.

### Step 2.5: Codebase Reconnaissance

Before starting the interview, silently scan the project to inform your questions.
Read/check (don't narrate this to the user):

- `README.md` — project description, stated audience, tech stack
- `CLAUDE.md` / `AGENTS.md` — existing conventions or philosophy
- `package.json`, `pyproject.toml`, `Cargo.toml`, or equivalent — dependencies, scripts
- Top-level directory structure via Glob (`*`) — what modules/areas exist

Use these findings to:
- Generate persona options grounded in the actual project (not generic archetypes)
- Pre-fill non-goal options based on what the project clearly isn't
- Ask about things you noticed rather than generic tradeoffs
- Skip questions the codebase already answers (e.g., don't ask "who is this for?" if
  the README has a clear audience statement — instead confirm and ask follow-ups)

### Step 3: Interview — Vision

Ask 2-4 questions using AskUserQuestion. Core questions:

1. **Why does this project exist?** What problem does it solve, and why is that problem
   worth solving? (Free text via "Other" — don't constrain with options here)
2. **Who is the primary user?** Offer 3-4 persona archetypes based on what you can
   infer from the codebase (package.json, README, etc.), plus "Other."
3. **What does success look like?** Not metrics — what observable outcome would mean
   this project achieved its purpose?

**Contextual follow-ups** — ask these when the codebase reconnaissance (Step 2.5) suggests relevance:

4. **Are there sub-groups within this persona?** If a persona has distinct variants that share
   a role but differ in context or needs (e.g., small-team admin vs. enterprise admin), ask
   the user to describe each sub-group. Only probe when the codebase or answers suggest the
   persona isn't monolithic.
5. **What platforms does this target?** Ask when the project supports multiple platforms or
   when the codebase structure might mislead about platform priorities. Clarify which platforms
   are primary, which are secondary/support surfaces, and correct any misconceptions.
6. **What are the 3-4 things this product must do excellently before adding anything new?**
   Ask for user-facing products to establish a positive scope boundary. Skip for libraries
   or developer tools where capabilities are obvious from the API.
7. **How should this product talk to users?** Ask for user-facing products: what tone, what
   framing choices (e.g., "boundaries not surveillance"), what words to avoid. Skip for
   libraries, CLIs, or internal tools unless brand voice matters.

Based on answers, ask 1-2 follow-up questions if something is ambiguous or interesting.
Then ask: **"Is there another distinct user group?"** Repeat until the user says no.

### Step 4: Interview — Principles

Ask 2-3 questions using AskUserQuestion:

1. **What tradeoff would a new contributor get wrong?** e.g., "They'd over-engineer
   the auth system" or "They'd optimize for performance when we care about DX."
   Offer 3-4 common tradeoff pairs as options.
2. **When you disagree with a PR, what's usually the reason?** This surfaces unstated
   beliefs about code quality, architecture, or product direction.
3. **What's a technical decision you've made that surprised people?** This often
   reveals the most distinctive principles.

Based on answers, ask 1-2 follow-ups if a principle needs clarification. Then:

3.5. **What tradeoffs in this project never fully resolve?** These are tensions where both
   sides are valid and the answer requires judgment every time — unlike principles (clear
   stance) or non-goals (clear boundary). Offer examples based on what you've learned:
   e.g., "security vs. onboarding simplicity," "school authority vs. family autonomy."

4. **When two principles conflict, which one wins?** e.g., "Security makes something
   harder to use — do you choose security or simplicity?" This produces a priority
   stack that agents can use as a tiebreaker for real decisions.

### Step 5: Interview — Non-Goals

Ask 1-2 questions using AskUserQuestion:

1. **What has this project intentionally said no to?** Offer 3-4 options based on
   common non-goals for the project type (e.g., "Enterprise features," "Multi-tenancy,"
   "Backwards compatibility with v1"), plus "Other."
2. **What should an agent working on this project never optimize for?** This catches
   non-goals that aren't obvious from the codebase.

### Interview Guidance: Multi-Select and "All of the Above"

Many interview questions benefit from multi-select. When offering options, explicitly
tell the user they can pick more than one: "Pick all that apply (or add your own via
Other)." After they select multiple options, follow up to rank them: "You picked A, B,
and C — if you had to order them by importance, what's the ranking?"

If the user selects ALL options or says "all of the above," follow up with a forcing
question: "If you had to pick the ONE that matters most, which would it be?" The goal
is to surface priority, not just agreement. Every option being equally important means
none of them are — push for differentiation.

### Step 5.5: Synthesis Playback

Before generating files, present a brief synthesis of what you heard:

> "Here's what I took away from our conversation — [3-5 bullet points covering the
> key vision, principles, and non-goals]. Anything I'm missing or getting wrong?"

Use AskUserQuestion with options like "Looks good, generate the files" / "Let me
clarify something." This catches misinterpretations before they're baked into prose —
especially important when the user gave rich free-text answers.

### Step 6: Generate Files

Using the interview answers and the templates from `references/output-formats.md`,
generate all three files:

1. Write `ethos/vision.md`
2. Write `ethos/principles.md`
3. Write `ethos/non-goals.md`

**Generation rules:**
- Use the user's own words where possible — don't sanitize their voice into corporate speak
- Every principle must have a concrete **This means** section
- Every non-goal must have a **What to do instead** redirect
- Personas must be specific enough to change agent behavior
- Keep each file focused — vision.md should be under 100 lines, principles.md under 140,
  non-goals.md under 70
- **Optional sections:** Platform Scope, Core Capabilities, How We Talk About This (vision.md)
  and Tensions We Live With (principles.md) are contextual — only generate them if the
  interview surfaced relevant content. Don't force these on every project.
- **Deduplication pass:** After drafting all three files, check for concepts that appear
  in multiple documents. If something is a scoping principle (what we focus on), keep it
  in principles. If it's a boundary (what we refuse to do), keep it in non-goals. A tension
  belongs in principles if it describes a tradeoff requiring ongoing judgment — but if one
  side of the tension is something you've firmly decided against, that side belongs in
  non-goals instead. No concept should appear in more than one document.

### Step 6.5: Decision Tests

Generate 2-3 "decision tests" — hypothetical scenarios the ethos should resolve — and
present them to the user:

> "To make sure this captures what you meant, here are a few test decisions:"
> - Q: Should we add a GraphQL layer? → A: Principle "Simplicity over flexibility" says no — stick with REST unless a specific consumer needs it.
> - Q: Should we support IE11? → A: Non-goal "Legacy browser support" says no — redirect to the progressive enhancement approach.

Ask: "Do these answers match your intuition?" If any feel wrong, revisit the relevant
principle or non-goal before finalizing. These tests also get appended to principles.md
as a "Decision Tests" section (see output-formats.md template).

**Codebase grounding:** Where possible, ground each decision test in real code. After
generating a test scenario, grep the codebase for implementations that embody the answer —
e.g., point to how rules are exposed in the frontend routes, or how schedule policies work
in backend services. This makes tests verifiable, not just philosophical. Follow the format
guidance in `references/output-formats.md`.

### Step 7: Add CLAUDE.md Pointer

Check if CLAUDE.md (or AGENTS.md) exists in the project root:

- **If it exists:** Check if an ethos section is already present. If not, append:

```markdown

## Project Ethos

Read the `ethos/` directory for the project's vision, principles, personas,
and non-goals. Consult these before making architectural or UX decisions.
```

- **If it doesn't exist:** Skip. Don't create CLAUDE.md — that's the user's decision.
  Instead, tell the user: "Consider adding a pointer to ethos/ in your CLAUDE.md."

### Step 8: Summary

Present what was generated:
- List the 3 files with a 1-line summary of each
- Note the CLAUDE.md status (updated / already had pointer / no CLAUDE.md found)
- Offer: "Want me to adjust anything?"

---

## Mode 2: Audit — Full Protocol

### Step 1: Read Existing Ethos

Read all files in `ethos/`. If any of the 3 expected files are missing, flag them.

### Step 2: Check Completeness

For each file, verify:
- **vision.md:** Has a Why section, at least one persona, and success criteria
- **principles.md:** Has at least 3 principles, each with Why and This means sections
- **non-goals.md:** Has at least 2 non-goals, each with Why not and What to do instead

Flag any missing or empty sections.

Additionally, check for optional sections that the current templates support but the
existing ethos may predate:
- **vision.md:** Platform Scope, Core Capabilities, How We Talk About This
- **principles.md:** Tensions We Live With

Flag these as "available but not present" rather than missing — they are enhancements,
not requirements. Mention them in the report as potential additions.

### Step 3: Check Relevance

Scan the codebase for signals that the ethos may have drifted:

- **Tech stack changes:** Use Glob/Grep to check if technologies mentioned in principles
  still appear in the codebase (e.g., a principle about PostgreSQL when the project
  moved to SQLite)
- **Structural shifts:** Check if the project's scope has visibly expanded or narrowed
  since the ethos was written (new top-level directories, major dependency changes)
- **Persona validity:** Check if the README or docs still describe the same target users

Don't flag style or wording issues — only substantive drift.

### Step 4: Report

Output a structured report:

```
## Ethos Audit

### Completeness
- [x] vision.md — all sections present
- [ ] principles.md — missing "This means" on principle 3
- [x] non-goals.md — all sections present

### Relevance Flags
- [flag]: principles.md mentions Express but package.json uses Fastify
- [flag]: No persona matches the new API consumer audience added in v2

### Recommendations
- [action]: Update principles.md to reflect Fastify migration
- [action]: Consider adding an API consumer persona to vision.md
```

If everything checks out, say so briefly and don't invent issues.

---

## Mode 3: Refresh — Full Protocol

### Step 1: Parse Target

The user specifies which doc to refresh: `vision`, `principles`, or `non-goals`.
If not specified, ask which one.

### Step 2: Read Existing Document

Read the target file from `ethos/`. Present a brief summary of what's currently there.

### Step 3: Read Output Templates

Read `references/output-formats.md` for the target format.

### Step 4: Targeted Re-Interview

Ask 2-3 questions focused on what might have changed:

1. **What's different now?** Present the current content summary and ask what no
   longer holds or what's missing.
2. **What triggered this refresh?** Understanding the motivation helps target the update
   (e.g., "we pivoted audiences" vs. "we changed our tech stack" leads to very
   different questions).

3. **New sections available:** If the target document is missing optional sections that the
   current format supports (e.g., Platform Scope, Core Capabilities, or How We Talk About
   This for vision.md; Tensions We Live With for principles.md), mention them and ask if
   the user wants to add them as part of the refresh.

Follow up based on answers — but keep it focused. This isn't a full init interview.

### Step 5: Surgical Update

Rewrite only the sections that changed. Preserve the user's original voice in
sections that still hold. Don't reformat or rephrase content that isn't being updated.

### Step 6: Summary

Show what changed with a brief diff summary. Offer to adjust.

---

## Reference Files

| File | When to Read |
|------|-------------|
| `references/output-formats.md` | Before generating files (init) or updating files (refresh) |
