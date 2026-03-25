# Ethos Output Formats

These templates define the structure for each generated ethos document. Use hybrid format: XML wrapper tags for agent parsing, markdown content inside for human readability.

---

## vision.md

```markdown
<vision>

# Project Vision

## Why This Exists

[1-3 paragraphs: the problem this project solves, why it matters, what motivated its creation]

## Who It's For

<personas>

### [Primary Persona Name]

**Role:** [What they do]
**Context:** [Their environment, constraints, daily reality]
**Core need:** [The job they're hiring this product to do]
**Frustration without this:** [What their life looks like without this project]

### [Secondary Persona Name] (if applicable)

[Same structure — repeat for each distinct user group]

### [Persona Name] (with sub-groups)

When a persona has distinct sub-groups that share a role but differ in context or needs:

#### [Sub-Group A]

**Role:** [What they do]
**Context:** [Their specific environment and constraints]
**Core need:** [Their specific job-to-be-done]
**Frustration without this:** [Their specific pain point]

#### [Sub-Group B]

[Same structure]

</personas>

## What Success Looks Like

[2-4 concrete outcomes that would mean this project achieved its purpose. Not metrics — observable states of the world.]

## Platform Scope (if applicable)

[Primary platforms and the role each plays. Correct any misconceptions the codebase structure might create about platform priorities.]

- **Primary platforms:** [e.g., iOS and Android — mobile-first]
- **Secondary surfaces:** [e.g., desktop apps are install surfaces only, not management interfaces]
- **Key clarification:** [e.g., this is not Apple-only despite heavy MDM usage — Android is first-class]

## Core Capabilities (if applicable)

[3-4 bullet points stating what this product does well. This is a positive scope boundary — these things must be excellent before adding anything new.]

- [Capability 1]
- [Capability 2]
- [Capability 3]

## How We Talk About This (if applicable)

[Brand voice and framing choices. Agents writing copy, error messages, and UI text should follow these guidelines.]

- **[Framing choice]:** [e.g., "Boundaries, not surveillance" — we never position the product as monitoring]
- **[Framing choice]:** [e.g., "Rules, not restrictions" — positive framing in the UI]
- **Tone:** [e.g., warm, approachable, not corporate or surveillance-themed]
- **Visual reference:** [e.g., reference design tokens, brand colors, typography from project docs]

</vision>
```

### Writing guidance for vision.md

- **Why This Exists** should answer "why" not "what." Don't describe the product — describe the problem and why it's worth solving.
- **Personas** should be specific enough that an agent can make different decisions for different users. "Developers" is too vague. "Solo developers shipping SaaS products who don't have a dedicated ops team" changes how you build.
- **What Success Looks Like** should be falsifiable. "Users are happy" is useless. "A developer can go from zero to deployed in under 10 minutes without reading docs" is actionable.
- **Sub-personas** should only be used when a persona genuinely has sub-groups with different needs. Don't split a persona just because you can — split when the sub-groups would lead to different product decisions.
- **Platform Scope** is most valuable for multi-platform products or projects where the codebase structure might mislead about platform priorities. Skip for single-platform projects.
- **Core Capabilities** creates a positive scope boundary. Each capability should be specific enough to evaluate ("content filtering" not "security"). Skip for libraries or tools where capabilities are obvious from the API.
- **How We Talk About This** matters for user-facing products where agents write copy, error messages, or UI text. Skip for developer tools, libraries, or internal projects unless brand voice is important.

---

## principles.md

```markdown
<principles>

# Technical & Product Principles

## [Principle Name]

[1-2 sentence statement of the belief]

**Why:** [The reasoning, experience, or tradeoff that led to this principle]

**This means:** [Concrete implications — what you will and won't do because of this principle]

## [Next Principle Name]

[Same structure — aim for 4-8 principles total]

## Tensions We Live With (if applicable)

These are real tradeoffs that don't resolve cleanly — they require judgment every time they come up. Unlike principles (which have a clear stance) or non-goals (which have a clear boundary), tensions are permanent and both sides are valid.

### [Tension Name]

[1-2 sentence description of the tension]

**Current stance:** [How the project currently navigates this — e.g., "most restrictive wins via policy synthesis"]
**Why it's permanent:** [Why this can't be fully resolved — both sides have legitimate claims]

## When Principles Conflict

When two principles pull in opposite directions, use this priority stack (highest wins):

1. [Highest-priority principle] — always wins in a direct conflict
2. [Second-priority principle]
3. [Third-priority principle]
[... ordered by the user's stated priorities from the interview]

**Example:** [Principle A] says keep it simple, but [Principle B] says make it secure.
Because security ranks higher, add the security measure even if it adds complexity —
but look for the simplest secure option first.

## Decision Tests

These scenarios validate that the principles above produce the right answers:

- **Q:** Should we [scenario]? → **A:** [Principle X] says [answer because reason].
- **Q:** Should we [scenario]? → **A:** [Principle Y] says [answer because reason].
- **Q:** Should we [scenario]? → **A:** [Non-goal Z] says [answer because reason].

</principles>
```

### Writing guidance for principles.md

- Each principle should resolve a real tradeoff. "Write good code" isn't a principle. "Prefer readability over cleverness — because this codebase will be maintained by people who didn't write it" is.
- The **This means** section is the most important part. It's what agents actually use to make decisions. Be specific: "Choose PostgreSQL over NoSQL for new data models" not "Use the right tool for the job."
- 4-8 principles is the sweet spot. Fewer than 4 means you haven't thought hard enough. More than 8 means some aren't load-bearing — cut them.
- Order by importance. An agent under time pressure may only read the first 3.
- **When Principles Conflict** must reflect the user's actual priority order from the interview. The example should use two real principles from the generated list.
- **Decision Tests** should be 2-3 scenarios that a contributor might actually face. Generate these from the interview answers — they double as verification that the ethos captures what the user meant. Where possible, reference actual code patterns: after generating a test scenario, grep the codebase for real implementations that embody the answer. This makes tests verifiable, not just aspirational.
- **Tensions We Live With** should capture tradeoffs where both sides are legitimate and the answer requires judgment every time. If one side is clearly wrong, it's a principle, not a tension. If one side is something you've firmly decided against, that side belongs in non-goals. 2-4 tensions is the sweet spot — if you have more, some are probably principles in disguise.

---

## non-goals.md

```markdown
<non-goals>

# Non-Goals

These are things this project intentionally does not do, optimize for, or support. They are not failures or limitations — they are deliberate choices that keep the project focused.

## [Non-Goal Name]

**What we don't do:** [Clear statement of the boundary]

**Why not:** [The reasoning — what we'd lose by pursuing this, or why it conflicts with our vision]

**What to do instead:** [Redirect — if someone wants this, where should they go or what approach should they take]

## [Next Non-Goal Name]

[Same structure — aim for 3-6 non-goals]

</non-goals>
```

### Writing guidance for non-goals.md

- Non-goals should prevent the most common wrong turns. Think: "What would a well-meaning contributor add that would actually hurt this project?"
- **What to do instead** is critical for agents. Without it, hitting a non-goal is a dead end. With it, the agent can redirect its approach.
- Frame positively where possible. Not "we don't care about performance" but "we optimize for developer experience over runtime performance — and here's what to do if you hit a perf issue."
- 3-6 non-goals is the sweet spot. If you have more than 6, some might belong as constraints in principles.md instead.
