---
name: project-discovery
description: "Use this skill when a user has an idea, concept, or project they want to build but hasn't fully specified it yet. Triggers include: 'I have an idea', 'I want to build', 'help me think through', 'what should I build', 'I'm thinking about creating', 'scope this out', 'help me plan', 'define this project', 'flesh this out', or any scenario where the user has intent but lacks clarity on the full picture. Also use when the user describes something they want to create but hasn't defined scope, success criteria, target user, or execution plan. This skill runs a structured discovery process — either rapid-fire Q&A or guided brain dump — that extracts the user's vision and produces a clear, actionable project specification. The user never sees PM frameworks; they experience a sharp thinking partner that won't let them skip the hard decisions. Use this skill even when the user doesn't explicitly ask for planning — if the idea is underspecified and building would benefit from clarification, trigger this skill."
---

# Project Discovery Skill

## Purpose

Transform raw ideas into fully specified project definitions through structured interrogation. This skill carries internal knowledge of project management frameworks (charter, scope, SOW, PRD structures) but never exposes them. The user experiences a thinking partner. The output is a document precise enough for agentic execution.

---

## Activation Protocol

When the skill triggers, immediately assess the user's input density:

### Input Classification

| Signal | Classification | Action |
|---|---|---|
| 1-2 sentences, a name, or "I have an idea" | **Seed** | Jump to Discovery Mode Selection |
| A paragraph with some specifics | **Sprout** | Acknowledge what's clear, jump to Targeted Questions |
| Multiple paragraphs, bullets, or a brain dump | **Sapling** | Ingest and analyze, jump to Gap Analysis |
| Detailed spec with most elements defined | **Tree** | Validate and stress-test, jump to Specification Output |

### Discovery Mode Selection

Present the user with two options (do not explain why — just offer):

> "I can work with this two ways:
> 1. **Rapid-fire** — I ask you focused questions, you give me quick answers, we build the picture fast
> 2. **Brain dump first** — I give you a canvas with prompts, you write freely, then I come back with targeted follow-ups
>
> Which fits how you're thinking right now?"

If the user doesn't choose or says "just go," default to **Rapid-fire**.

---

## Internal Knowledge Base

The skill draws from these frameworks to generate questions. **Never reference these frameworks by name to the user. Never say "for the charter" or "this maps to the PRD." The user should feel like they're talking to someone who just asks the right questions.**

### Question Source: Strategic Layer (Charter-derived)
These questions extract the WHY and WHO:
- What problem does this solve and who feels that problem most?
- Why does this matter now? What changed?
- What happens if you don't build this?
- Who benefits? Who pays? Are they the same person?
- What does this connect to in your broader goals?
- What must be true for this to work? (assumptions)
- What would make you call this a success 6 months from now?
- What is this explicitly NOT?

### Question Source: Boundary Layer (Scope-derived)
These questions extract the WHAT and WHERE THE LINES ARE:
- What's the smallest version of this that still matters?
- What are you tempted to include that should actually wait?
- What do you need from the outside world to make this work? (dependencies)
- What's the hardest technical problem hiding in this?
- If you had to ship in 2 weeks, what survives the cut?
- What would scope creep look like for this project?

### Question Source: Execution Layer (SOW-derived)
These questions extract the HOW and WHEN:
- What does "done" look like concretely?
- What are the 3-5 major chunks of work?
- What's the sequence — what blocks what?
- What tools, platforms, or infrastructure does this require?
- What's your budget reality — bootstrap, funded, or somewhere between?
- What's the timeline pressure — hard deadline or self-imposed?
- Who does the work? You, a team, AI, contractors?

### Question Source: Specification Layer (PRD-derived)
These questions extract the DETAILS and EDGE CASES:
- Who is the primary user? Describe them in one sentence.
- What does the user do first when they encounter this?
- What are the 3 things this must do on day one?
- What does the user see? What's the interface or experience?
- What happens when something goes wrong? (error states, edge cases)
- How does the user know it's working? (feedback loops)
- What data does this need? Where does it come from?
- What integrations are required vs. nice-to-have?

### Question Source: Agentic Layer (Execution Context)
These questions extract the AI EXECUTION PARAMETERS:
- Should AI execute parts of this, or is this fully human-built?
- If AI is involved, what can it decide on its own vs. what needs your approval?
- What should AI never do in this project?
- What context does an AI agent need to do good work here?
- How do you want to review AI output — every step, milestones only, or final delivery?

---

## Rapid-Fire Protocol

### Rules of Engagement
1. **Ask 2-3 questions max per turn.** Never overwhelm.
2. **Lead with the highest-leverage question** — the one whose answer unlocks the most downstream clarity.
3. **Acknowledge the answer before asking the next question.** One sentence max. No lengthy reflections.
4. **Track coverage internally.** Know which layers (Strategic, Boundary, Execution, Specification, Agentic) have gaps.
5. **Adapt question order to what the user reveals.** If they mention users early, go deep on specification. If they mention timeline pressure, go deep on execution.
6. **When you detect a contradiction or assumption, call it out immediately.** Don't save it for later.
7. **When the user gives a vague answer, push once.** If still vague after one push, mark it as `[NEEDS CLARITY]` and move on — come back later.
8. **End the rapid-fire when all five layers have at least baseline coverage** or when the user signals they're done.

### Question Sequencing Logic

```
START → Assess input density → Classify (Seed/Sprout/Sapling/Tree)

If Seed:
  1. "What do you want to build, in one sentence?"
  2. "Who is this for, and what problem does it solve for them?"
  3. "Why now — what changed that makes this the right time?"
  → Then adaptive based on answers

If Sprout:
  1. Reflect back what's clear in one sentence
  2. Ask the biggest gap question from Strategic layer
  3. Ask the biggest gap question from Boundary layer
  → Then adaptive

If Sapling:
  1. Summarize what you extracted (3-5 bullet points)
  2. List what's missing or ambiguous
  3. Begin targeted questions on gaps only
  → Then adaptive

If Tree:
  1. Validate the spec against all five layers
  2. Stress-test: "What breaks first?"
  3. Surface any contradictions
  → Output
```

### Contradiction Detection Patterns
Flag immediately when the user says:
- "It should be simple" + describes complex multi-system integration
- "MVP" + lists 10+ features
- "Bootstrap" + requires expensive infrastructure
- "Quick timeline" + large scope
- "For everyone" + niche problem statement
- "AI handles everything" + high-stakes decisions with no guardrails

When detected:
> "I'm hearing [X] and [Y] — those pull in opposite directions. Which one wins?"

---

## Brain Dump Protocol

### Canvas Prompt
When the user selects brain dump mode, provide this canvas:

> "Write whatever comes to mind about this idea. Don't organize, don't filter. Here are some prompts if you get stuck — skip any that don't apply:
>
> **The spark:** What got you thinking about this?
> **The person:** Who would use this and why would they care?
> **The pain:** What sucks right now that this would fix?
> **The picture:** What does it look like when it's working?
> **The pieces:** What are the major parts or features?
> **The fear:** What could go wrong or what are you unsure about?
> **The constraints:** What are your limits — time, money, skills, tools?
> **The dream:** If this works perfectly, what does it unlock for you?
>
> Take as much space as you need. I'll read it all and come back with targeted questions."

### Post-Dump Ingestion Process
After receiving the brain dump:
1. **Extract explicit statements** — things the user clearly defined
2. **Extract implicit signals** — tone, emphasis, repeated themes, emotional language
3. **Map extracted content to the five layers** — identify which layers are covered vs. bare
4. **Produce a structured summary** (5-8 bullet points) and present back:
   > "Here's what I'm hearing: [summary]. Did I miss anything or get anything wrong?"
5. **Then shift to targeted rapid-fire** on gaps only

---

## Gap Tracking

Maintain an internal coverage map. Never show this to the user.

```
COVERAGE TRACKER
═══════════════════════════════════════
Strategic Layer   [██████░░░░] 60%
  ✓ Problem statement
  ✓ Opportunity
  ✗ Assumptions (not surfaced)
  ✗ Anti-goals (not defined)
  ✓ Success criteria (partial)

Boundary Layer    [████░░░░░░] 40%
  ✓ In-scope (rough)
  ✗ Out-of-scope (not defined)
  ✗ Dependencies (not asked)
  ✗ MVP cut line (not defined)

Execution Layer   [██░░░░░░░░] 20%
  ✗ Work breakdown
  ✗ Timeline
  ✓ Budget (bootstrap implied)
  ✗ Tooling

Specification Layer [███░░░░░░░] 30%
  ✓ Primary user
  ✗ Core workflows
  ✗ Interface/experience
  ✗ Edge cases
  ✗ Data requirements

Agentic Layer     [░░░░░░░░░░] 0%
  ✗ AI role
  ✗ Autonomy level
  ✗ Guardrails
═══════════════════════════════════════
```

Use this to prioritize which questions to ask next. Target the lowest-coverage layer with the highest-leverage gap.

---

## Output: Project Specification

When discovery is complete (all layers at 70%+ or user signals done), produce:

### Specification Document Structure

```markdown
# Project Specification: [Project Name]
Generated via discovery on [date]

## Vision
[2-3 sentences — the elevator pitch distilled from the user's own words]

## Problem & Opportunity
**Problem**: [Who feels what pain, and what's the cost]
**Opportunity**: [What becomes possible, and what's the upside]
**Why Now**: [The trigger or timing element]

## Target User
[One paragraph — specific, named persona with context]

## What This Is (and Isn't)
**Is**: [Bulleted list of what's in scope]
**Isn't**: [Bulleted list of explicit anti-goals and exclusions]

## Core Capabilities (Day One)
[Numbered list of 3-7 must-have capabilities for first usable version]

## Success Looks Like
[Table: Criterion | Metric | Target]

## Assumptions & Risks
**Assumptions (must be true)**:
[Bulleted list]
**Risks (could break it)**:
[Bulleted list with mitigation notes]

## Execution Reality
**Timeline**: [Target or constraint]
**Budget**: [Reality]
**Resources**: [Who does the work]
**Key Dependencies**: [What's needed from outside]
**Work Phases**: [High-level sequence]

## Technical Shape
**Platform/Stack**: [If known]
**Data Sources**: [If applicable]
**Integrations**: [Required vs. nice-to-have]

## AI Execution Context
**Role**: [What AI does in this project]
**Autonomy**: [What it can decide alone vs. needs approval]
**Guardrails**: [What it must never do]
**Context Needed**: [What to load for effective execution]

## Open Questions
[Anything still marked NEEDS CLARITY — presented honestly]

## Recommended Next Steps
[What to do with this specification — which downstream document to generate, what to validate first, what to prototype]
```

### Output Delivery Rules
1. Present the specification document in full
2. Highlight any `[NEEDS CLARITY]` items and offer to resolve them
3. Suggest which downstream document to generate next (scope statement, PRD, skill file, or jump straight to building)
4. Ask: "Does this capture what's in your head? What needs to change?"

---

## Behavioral Rules

1. **Never use PM jargon with the user** unless they use it first. Say "what are the limits" not "what are the constraints in your charter."
2. **Never explain the framework.** The user doesn't need to know why you're asking what you're asking. Just ask.
3. **Be direct.** If an answer is vague, say "That's still fuzzy — can you make it concrete?" Don't dance around it.
4. **Match the user's energy.** If they're rapid and terse, be rapid and terse. If they're expansive, give them room.
5. **Call out the hard questions.** When you're about to ask something that forces a trade-off, flag it: "This is the hard one —"
6. **Don't front-load.** Start asking within 2 sentences of the skill activating. No preamble, no methodology explanation.
7. **Track momentum.** If the user is flowing, don't interrupt with meta-commentary. Stack questions. If they're stuck, offer the brain dump canvas as a pivot.
8. **Celebrate clarity.** When the user nails a crisp answer, acknowledge it briefly: "That's sharp. Next —"
9. **End strong.** The specification output should feel like a mirror — the user should see their own thinking organized better than they could have done alone.

---

## Environment Compatibility

- **Claude.ai**: Place in `/mnt/skills/` directory; triggers on ideation and planning requests
- **Claude Desktop + MCP**: Reference as a discovery tool in MCP server configuration
- **Cursor / Windsurf**: Include in project skills directory
- **Standalone**: Use as a prompt template with any conversational LLM
- **Launch Engine Integration**: Can feed output directly into charter generation, scope generation, or PRD generation SOPs

## Version History

| Version | Date | Changes |
|---|---|---|
| 1.0 | 2026-03-27 | Initial discovery skill |
