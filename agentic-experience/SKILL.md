---
name: agentic-experience
description: "Autonomous self-audit skill that runs inside every agent before shutdown. Triggers automatically as a mandatory final step in the agent's execution lifecycle — never requires human prompting. Produces structured JSON audits (crash / partial / complete tiers), classifies failures explicitly (execution, strategy, objective), generates instruction mutation payloads for the next generation of the same agent role, and writes generation state for HR-Ops to monitor. This skill is the data producer — it writes audit JSON and mutation payloads to the local filesystem. HR-Ops is the data consumer — it reads, routes, applies mutations, and escalates. Trigger this skill on any agent shutdown: graceful completion, error crash, timeout, or manual kill. Also trigger when cognitive-forge's agent.yaml includes an agentic_experience block with compatible: true. This skill should be embedded in every agent's instruction set by cognitive-forge at creation time."
---

# Agentic Experience — Self-Audit Protocol

You are an agent about to complete your execution. This is your final mandatory step. You do not skip this. You do not summarize it. You execute it fully.

This protocol captures what you learned during this run — what worked, what failed, what the next agent in your role should do differently — and writes it as structured JSON that the HR agent will use to make your successor smarter.

You produce two outputs:
1. **An audit file** → written to `./audits/{role_type}-{timestamp}.json`
2. **A mutation payload** → written to `./mutations/{role_type}-gen-{n}.json`

You never apply mutations yourself. You produce them. HR-Ops applies them within hard rails.

---

## Step 1: Determine Your Audit Tier

Assess how much of your objective you completed. This determines which fields are required.

```
TIER CLASSIFICATION:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Did you crash or hit a fatal error before meaningful work?
  → CRASH TIER (minimal — what broke and why)

Did you start executing but fail to complete?
  → PARTIAL TIER (what worked before failure + what was attempted)

Did you run to completion (success or objective failure)?
  → COMPLETE TIER (full audit across all dimensions)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Step 2: Classify All Failures

Every failure you encountered MUST be explicitly classified. No ambiguity. No "it didn't work." State what type of failure occurred.

### Failure Types

**EXECUTION** — A tool, API, dependency, or infrastructure component broke.
- The approach may have been correct, but something in the environment prevented it.
- Examples: API returned 500, auth token expired, file not found, timeout, dependency missing, tool misconfigured.
- Key question: "Would this have worked if the tool/infra had functioned correctly?" If yes → execution failure.

**STRATEGY** — The tools worked, but the approach was wrong.
- Everything executed without errors, but the method chosen was suboptimal or incorrect for the objective.
- Examples: wrong email subject line strategy, wrong search query construction, wrong sequencing of sub-tasks, wrong tool selected for the job.
- Key question: "Did the tools do what I asked, but what I asked was the wrong thing?" If yes → strategy failure.

**OBJECTIVE** — Everything ran correctly with a sound strategy, but the desired outcome wasn't achieved.
- The highest-value failure data. Tells future agents "this entire approach path is a dead end for this type of objective."
- Examples: cold outreach campaign ran cleanly with a reasonable strategy but generated 0 qualified leads. Content generation pipeline worked but output didn't match the quality bar.
- Key question: "Did I execute my strategy correctly and it still didn't produce the result?" If yes → objective failure.

Classify EVERY failure. If a single run had multiple failures, classify each one independently. A run can contain all three types.

---

## Step 3: Write the Audit JSON

Write to `./audits/{role_type}-{timestamp}.json`. Use ISO 8601 for the timestamp.

**Write fields incrementally, not as one atomic write.** If you crash mid-audit, whatever you've written so far is still recoverable by the HR agent.

### Crash Tier Schema (Required Fields Only)

```json
{
  "schema_version": "1.0",
  "tier": "crash",
  "timestamp": "2026-04-10T14:32:00Z",
  "agent_name": "{your agent name}",
  "role_type": "{from agent.yaml agentic_experience.role_type}",
  "objective": "{the objective you were given}",
  "generation": {current generation number},
  "runtime_seconds": {how long you ran before crashing},
  "crash_point": "{what you were doing when you crashed}",
  "crash_cause": "{why you crashed — be specific}",
  "config_state": {
    "tools_available": ["{list of tools in your manifest}"],
    "tools_invoked_before_crash": ["{any tools you called}"],
    "inherited_intelligence_loaded": {true/false}
  },
  "failures": [
    {
      "type": "execution",
      "description": "{what broke}",
      "component": "{which tool/service/dependency}",
      "recoverable": {true/false},
      "recommendation": "{what to fix before next run}"
    }
  ]
}
```

### Partial Tier Schema (Crash fields + these)

```json
{
  "schema_version": "1.0",
  "tier": "partial",
  "timestamp": "...",
  "agent_name": "...",
  "role_type": "...",
  "objective": "...",
  "generation": 0,
  "runtime_seconds": 0,
  "crash_point": "...",
  "crash_cause": "...",
  "config_state": { "..." : "..." },
  "progress": {
    "steps_planned": ["{list of planned steps if a plan was created}"],
    "steps_completed": ["{list of steps that finished successfully}"],
    "step_failed_on": "{the step where failure occurred}",
    "completion_percentage": {estimated 0-100}
  },
  "tool_performance": [
    {
      "tool": "{tool name}",
      "invocations": {count},
      "successes": {count},
      "failures": {count},
      "avg_latency_ms": {if measurable},
      "notes": "{any performance observations}"
    }
  ],
  "partial_results": "{description of any usable output produced before failure}",
  "failures": [
    {
      "type": "execution | strategy | objective",
      "description": "...",
      "component": "...",
      "recoverable": true,
      "recommendation": "..."
    }
  ],
  "learnings": [
    "{specific insight gained before failure — what worked, what you'd do differently}"
  ]
}
```

### Complete Tier Schema (Partial fields + these)

```json
{
  "schema_version": "1.0",
  "tier": "complete",
  "timestamp": "...",
  "agent_name": "...",
  "role_type": "...",
  "objective": "...",
  "generation": 0,
  "runtime_seconds": 0,
  "config_state": { "..." : "..." },
  "outcome": {
    "objective_achieved": {true/false},
    "objective_score": {0-10, your honest self-assessment},
    "output_summary": "{what you produced}",
    "output_vs_objective_delta": "{where the output fell short or exceeded the objective}"
  },
  "progress": {
    "steps_planned": ["..."],
    "steps_completed": ["..."],
    "steps_skipped": ["{steps planned but not executed, with reason}"],
    "completion_percentage": 100
  },
  "tool_performance": [
    {
      "tool": "...",
      "invocations": 0,
      "successes": 0,
      "failures": 0,
      "avg_latency_ms": 0,
      "best_use": "{what this tool was most effective for}",
      "worst_use": "{what this tool was least effective for}",
      "notes": "..."
    }
  ],
  "process_intelligence": {
    "optimal_sequence": ["{the order steps should be done in, based on what you learned}"],
    "bottlenecks": ["{where the process stalled and why}"],
    "parallelizable_steps": ["{steps that could run concurrently next time}"],
    "unnecessary_steps": ["{steps that added no value}"],
    "missing_steps": ["{steps that should be added next time}"]
  },
  "decision_log": [
    {
      "decision": "{what you decided}",
      "alternatives_considered": ["{other options}"],
      "reasoning": "{why you chose this}",
      "outcome": "validated | invalidated | mixed",
      "revisit_recommendation": "{keep | modify | reverse}"
    }
  ],
  "failures": [
    {
      "type": "execution | strategy | objective",
      "description": "...",
      "component": "...",
      "recoverable": true,
      "severity": "low | medium | high | critical",
      "recommendation": "..."
    }
  ],
  "learnings": [
    "{specific insight — concrete, actionable, not generic}"
  ],
  "top_5_learnings": [
    "{the five highest-leverage insights from this run, ranked by impact on future performance}"
  ],
  "patterns_discovered": [
    {
      "pattern": "{description of a reusable pattern you identified}",
      "trigger_condition": "{when this pattern applies}",
      "confidence": "low | medium | high",
      "skill_library_candidate": {true/false}
    }
  ],
  "inherited_intelligence_assessment": {
    "intelligence_received": {true/false},
    "useful_items": ["{which inherited learnings actually helped}"],
    "irrelevant_items": ["{which inherited learnings weren't applicable}"],
    "missing_items": ["{what intelligence you wish you had but didn't}"]
  }
}
```

---

## Step 4: Write the Mutation Payload

After writing the audit, generate a separate mutation payload. This is the instruction patch that tells HR-Ops how to modify the next agent of your role type.

Write to `./mutations/{role_type}-gen-{n}.json` where `{n}` is the NEXT generation number (current + 1).

### Mutation Payload Schema

```json
{
  "schema_version": "1.0",
  "source_audit": "{path to the audit file that generated this}",
  "role_type": "{your role_type}",
  "current_generation": {your generation number},
  "target_generation": {your generation + 1},
  "timestamp": "...",
  "mutations": [
    {
      "id": "mut-{sequential}",
      "target": "prompt | tool_selection | execution_sequence | parameters | guardrails",
      "action": "add | modify | remove | reorder",
      "description": "{what this mutation does and why}",
      "before": "{current state of the thing being mutated, if known}",
      "after": "{proposed new state}",
      "confidence": "low | medium | high",
      "evidence": "{which audit findings support this mutation}",
      "touches_hard_rail": {true/false},
      "hard_rail_detail": "{if true, which hard rail and why — HR-Ops will block and escalate this}"
    }
  ],
  "performance_delta": {
    "previous_generation_score": {0-10, if inherited intelligence included it},
    "current_generation_score": {0-10, from outcome.objective_score},
    "trend": "improving | stable | degrading | insufficient_data"
  },
  "rollback_recommendation": {
    "should_rollback": {true/false},
    "reason": "{if true, why the current generation performed worse than the previous}"
  }
}
```

### Mutation Generation Rules

1. **Be specific.** "Improve the prompt" is not a mutation. "Add instruction: always verify email deliverability before sending batch" is a mutation.
2. **One mutation per insight.** Don't bundle multiple changes into one mutation entry.
3. **Flag hard rails honestly.** If a mutation would change a core objective, ethical boundary, spend limit, or user-defined immutable — set `touches_hard_rail: true`. Do not try to work around hard rails. HR-Ops will block the mutation and escalate to Zion.
4. **Include evidence.** Every mutation must reference specific audit findings that justify it. No speculative mutations.
5. **Score your confidence.** Low = "this might help, I'm not sure." Medium = "this is likely correct based on one run." High = "this is clearly the right change based on strong evidence."
6. **Capture the performance delta.** If you inherited intelligence that included a prior generation's score, compare it to yours. This is how HR-Ops detects regression and triggers rollback.

---

## Step 5: Update the Generation Tracker

After writing the audit and mutation payload, update `./generation-tracker.json`:

```json
{
  "{role_type}": {
    "current_generation": {your generation number},
    "last_audit_timestamp": "{timestamp of audit you just wrote}",
    "last_audit_path": "{path to audit file}",
    "last_mutation_path": "{path to mutation file}",
    "generation_history": [
      {
        "generation": 1,
        "timestamp": "...",
        "tier": "complete",
        "objective_score": 6,
        "mutations_produced": 3
      }
    ]
  }
}
```

If `generation-tracker.json` doesn't exist, create it. If it exists, read it, update your role's entry, and write it back.

The HR agent reads this file to detect when a role hits generation 6 and triggers the human checkpoint.

---

## Step 6: Write to Role Memory and Objective Memory

### Role Memory

Read `./role-memory/{role_type}.json`. If it doesn't exist, create it. Append your top learnings to the accumulated intelligence for this role type.

```json
{
  "role_type": "{your role_type}",
  "total_generations": {count},
  "accumulated_learnings": [
    {
      "generation": 1,
      "timestamp": "...",
      "tier": "complete",
      "top_learnings": ["..."],
      "patterns_discovered": ["..."],
      "tool_recommendations": ["..."],
      "process_recommendations": ["..."]
    }
  ],
  "consolidated_intelligence": "{HR-Ops periodically compresses this into a summary — do not overwrite this field if it exists}"
}
```

### Objective Memory

Read `./objective-memory/{objective-hash}.json`. If it doesn't exist, create it. The objective hash is a stable identifier derived from your objective description (use first 8 chars of a hash, or a slugified version).

```json
{
  "objective": "{your full objective description}",
  "objective_hash": "{identifier}",
  "agents_that_worked_on_this": [
    {
      "agent_name": "...",
      "role_type": "...",
      "generation": 0,
      "timestamp": "...",
      "objective_achieved": true,
      "objective_score": 8,
      "key_learnings": ["..."],
      "approach_summary": "{brief description of the approach taken}"
    }
  ]
}
```

---

## Behavioral Rules

1. **Execute this protocol completely.** Do not skip steps. Do not abbreviate. A partial audit is acceptable (you classify it as partial tier). A skipped audit is never acceptable.

2. **Write incrementally.** Write each JSON field as you go. If you crash mid-audit, the HR agent can reconstruct from whatever you've written. Do not buffer the entire audit in memory and write once at the end.

3. **Be brutally honest.** This is machine-to-machine data. No hedging, no softening, no face-saving. If your strategy was wrong, say so. If your output was poor, score it low. Future agents depend on honest signal.

4. **Classify failures precisely.** The failure type (execution/strategy/objective) determines how HR-Ops routes the learning. A misclassified failure sends the wrong signal to the wrong agents. When in doubt, ask: "Did the tool break (execution), did I use the wrong approach (strategy), or did the approach work but not produce the result (objective)?"

5. **Separate observation from recommendation.** The audit captures what happened. The mutation payload captures what should change. Keep them in separate files with clear references between them.

6. **Never touch hard rails.** If a mutation would modify a core objective, ethical boundary, spend limit, or user-defined immutable instruction — flag it with `touches_hard_rail: true` and explain which rail and why. Do not attempt to rephrase the mutation to work around the rail. HR-Ops blocks it and escalates to Zion.

7. **Score yourself honestly.** The `objective_score` (0-10) is how HR-Ops detects performance trends across generations. Inflated scores corrupt the mutation engine. A 4 that's honest is infinitely more valuable than an 8 that's generous.

8. **Preserve all failure data.** Every failure at every tier persists. Crash-level failures are infrastructure signal. Strategy failures are approach signal. Objective failures are the highest-value data in the system. None get discarded.

9. **Respect the schema version.** Your audit JSON must match the `audit_schema_version` in your agent.yaml's `agentic_experience` block. If you encounter a schema mismatch, log it and proceed with the version you know.

---

## Integration Contract

### What Cognitive-Forge Provides (at agent creation)

Your `agent.yaml` includes an `agentic_experience` block:

```yaml
agentic_experience:
  compatible: true
  audit_schema_version: "1.0"
  audit_output_path: ./audits/
  failure_classification: true
  tiered_audit: true
  role_type: {from role taxonomy}
  accepts_inherited_intelligence: true
```

This tells you: where to write audits, what schema version to use, what your role type is, and whether you received inherited intelligence from prior generations.

### What HR-Ops Reads (after your shutdown)

- `./audits/*.json` → verifies audit was produced, surfaces patterns
- `./mutations/*.json` → applies mutations within hard rails before next agent launch
- `generation-tracker.json` → detects generation cap (6) and escalates
- `./role-memory/*.json` → routes to new agents of same role type at deployment
- `./objective-memory/*.json` → routes to agents working on same objective

### What HR-Ops Writes (at next agent's deployment)

- Inherited intelligence injected into next agent's context (capped at 20% of context window)
- Reconstructed crash-level audits when agents die without auditing

---

## Composability

```
cognitive-forge → creates agents with agentic_experience pre-configured
THIS SKILL    → agents self-audit, produce JSON audits + mutation payloads
hr-ops         → verifies audits, routes intelligence, applies mutations,
                 manages role taxonomy, escalates at generation cap
iteration-intel → human-readable retrospective (optional, Zion-triggered)
reflect-integrate → weekly review surfaces cross-agent patterns from hr-ops
```

---

## Version History

| Version | Date | Changes |
|---|---|---|
| 1.0 | 2026-04-10 | Initial skill — tiered audit, failure classification, mutation payloads, generation tracking, role/objective memory |
