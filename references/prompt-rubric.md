# Prompt Quality Rubric

Score the user's prompt 1-5 on each of the 5 criteria below.

**Pass threshold:** average >= 4.0 AND no single criterion < 3.
On a fail, refine and re-score. Fail-closed: never start work on a failing prompt.

## Criteria

| # | Criterion | 1 (Poor) | 3 (Adequate) | 5 (Excellent) |
|---|-----------|----------|--------------|---------------|
| 1 | **Clarity** | Ambiguous, undefined terms ("a tool", "something for X") | Main intent readable, some vagueness | Single unambiguous reading; key nouns/verbs defined |
| 2 | **Context** | No use case, users, env, or constraints | Some background; gaps remain | Use case, target users, environment, and constraints all stated |
| 3 | **Actionability** | No concrete deliverable or action | Deliverable implied | Explicit deliverable + format (e.g., "Python CLI emitting JSON") |
| 4 | **Scope** | Unbounded or contradictory | Roughly bounded | Clear in/out of scope; blast radius and non-goals named |
| 5 | **Success Criteria** | None | Vague ("works well") | 3-5 measurable, testable outcomes |

## Scoring Output (show this compactly)

```
Clarity:          X/5
Context:          X/5
Actionability:    X/5
Scope:            X/5
Success Criteria: X/5
Average:          X.X/5  -> PASS | FAIL (reason)
```

## Refinement Pattern (on FAIL)

1. Name the failing criteria in one line each.
2. Propose a single rewritten prompt that would score >= 4.0 across the board.
3. Where the user's intent is unknown, ask the minimum clarifying questions (scope, regions, IAM roles, blast radius for infra work) rather than guessing.
4. Re-score the refined prompt and show the new table.

## Notes for Infra / Security Prompts

For Terraform / cloud / cluster work, a passing prompt should name: target cloud + region(s), IAM roles/modules to reuse, blast radius, and a measurable success criterion (e.g., "cluster starts, all nodes Ready, no default-allow security groups"). If these are missing, Context/Scope/Success cannot score >= 3 — ask before proceeding.
