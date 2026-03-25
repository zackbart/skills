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

</personas>

## What Success Looks Like

[2-4 concrete outcomes that would mean this project achieved its purpose. Not metrics — observable states of the world.]

</vision>
```

### Writing guidance for vision.md

- **Why This Exists** should answer "why" not "what." Don't describe the product — describe the problem and why it's worth solving.
- **Personas** should be specific enough that an agent can make different decisions for different users. "Developers" is too vague. "Solo developers shipping SaaS products who don't have a dedicated ops team" changes how you build.
- **What Success Looks Like** should be falsifiable. "Users are happy" is useless. "A developer can go from zero to deployed in under 10 minutes without reading docs" is actionable.

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

</principles>
```

### Writing guidance for principles.md

- Each principle should resolve a real tradeoff. "Write good code" isn't a principle. "Prefer readability over cleverness — because this codebase will be maintained by people who didn't write it" is.
- The **This means** section is the most important part. It's what agents actually use to make decisions. Be specific: "Choose PostgreSQL over NoSQL for new data models" not "Use the right tool for the job."
- 4-8 principles is the sweet spot. Fewer than 4 means you haven't thought hard enough. More than 8 means some aren't load-bearing — cut them.
- Order by importance. An agent under time pressure may only read the first 3.

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
