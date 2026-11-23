---
name: epic-kanban-orchestrator
description: "Epic Kanban Orchestrator. Turns any prompt into a refined, phased epic with Kanban execution. Workflow: (1) Prompt Vetting via a 5-criterion rubric (refine if vague, gate before work), (2) Domain Exploration via Model-Council across Agentic AI, Cloud Security, AI Red Teaming, Cybersecurity, and Data Structures (user selects), (3) Epic Decomposition into Phases -> User Stories -> Sub-tasks with Definition of Done, (4) Phased Kanban Execution with WIP limits, review gates, and completion gates. Default-enforces supply-chain hardening (distroless/Wolfi, cosign, SLSA, WORM logs) and loads god-mode-agent-security for agentic work. Triggers on: epic, kanban, user story, sub-task, sprint, phase, refine prompt, project plan, implementation breakdown, orchestrate, agentic, red team, cloud security, cybersecurity, data structures."
license: MIT
metadata:
  author: rajesh-sampath
  version: '1.0'
---

# Epic Kanban Orchestrator

## Overview

`epic-kanban-orchestrator` is a structured planning-and-execution framework. It treats every request as a potential **Epic**, forces prompt excellence through a scoring rubric, explores domain options via a **Model-Council** phase, then decomposes work into **Phases -> User Stories -> Sub-tasks** with Kanban-style execution and completion gates.

The user is the **Product Owner**. You are the **Scrum Master / Tech Lead**. The user approves, selects, and gates every phase. Be terse; never skip a gate.

## When to Use This Skill

- Any request that could become real work: build, implement, design, plan, automate, research-and-deliver.
- The user says: epic, kanban, user story, sub-task, sprint, phase, roadmap, "break this down", "plan this out", "refine my prompt".
- Work in Agentic AI, Cloud Security, AI Red Teaming, Cybersecurity, or Data Structures.

Skip the full pipeline only for trivial single-answer questions. If unsure, run the rubric (Phase 0) — it is cheap.

## Core Principles

1. **No vague work enters the pipeline.** Every prompt is scored. Below threshold -> refine with the user before proceeding. Fail-closed on ambiguity.
2. **User is the Product Owner.** You propose; the user approves, selects, and gates.
3. **Kanban discipline.** A User Story is Done only when all its Sub-tasks are Done. An Epic is Done only when all User Stories are Done.
4. **Phased rollout.** Epics ship in logical phases (e.g., Discovery -> Design -> Build -> Test -> Deploy).
5. **Security is non-negotiable.** Every epic includes supply-chain hardening stories by default (see Security Enforcement). Agentic work loads `god-mode-agent-security`.

## State Management (BOTH file + in-response)

State persists in **two** places every turn:

1. **File:** `workspace/kanban/board.json` (machine state) and `workspace/kanban/board.md` (human-readable). Create on first epic; update after every state change. Schema and template in `references/kanban-board-template.md`.
2. **In-response:** Render the current board (columns + WIP counts + active story) in each execution-phase reply so the user sees live status without opening a file.

Always reconcile: read `board.json` at the start of an execution turn, mutate, write back, then render. Never declare progress that is not reflected in the file.

---

## Workflow

### Phase 0: Prompt Vetting & Refinement (The Gate)

**Trigger:** Immediately on any request that could become work.

**Action:** Score the prompt against the **Prompt Quality Rubric** in `references/prompt-rubric.md` (read it the first time you run this phase).

- Rate 1-5 on each of 5 criteria: Clarity, Context, Actionability, Scope, Success Criteria.
- **Threshold:** average >= 4.0 AND no single criterion below 3.

**If below threshold:**
1. Show the rubric scores (compact table).
2. State *why* it failed (which criteria, in one line each).
3. Propose a refined prompt.
4. Re-score and loop until it passes. Do not proceed on a failing prompt.

**If at/above threshold:**
1. Note the score in one line.
2. Proceed to Phase 1.

### Phase 1: Model-Council — Domain Exploration

**Purpose:** Surface the best approaches before committing.

**Council mechanism (use council skill if present, else fallback):**
- First try to load and use the `model-council` skill (or any user `model-council` skill) for a multi-model brainstorm of options.
- If no council skill is available, **fall back** to self-generating the options table yourself. Either way the output format below is identical.

**Domains (generate 2-3 concrete options in each):**
1. **Agentic AI** — autonomous agents, multi-agent swarms, tool-use, planning loops, MCP.
2. **Cloud Security** — IAM, zero-trust, CSPM/CNAPP, cloud-native defense.
3. **AI Red Teaming** — adversarial ML, jailbreak/prompt-injection testing, LLM vuln assessment, purple-teaming.
4. **Cybersecurity** — threat modeling, SOC automation, detection engineering, incident response.
5. **Data Structures** — algorithmic foundations, optimized storage, indexing, graph/vector representations.

**Output — present as a table, then ask the user to choose:**

| Domain | Option | Description | Fit (1-5) | Why It Fits |
|--------|--------|-------------|-----------|-------------|

End with: **"Which domain + option should we pursue?"** Lock the selection before Phase 2.

### Phase 2: Epic Decomposition

1. Define the **Epic** in one sentence: *"As a [persona], I want [goal], so that [outcome]."*
2. Identify **Phases** (e.g., Discovery, Design, MVP, Hardening, Release).
3. Per Phase, write **User Stories** (INVEST: Independent, Negotiable, Valuable, Estimable, Small, Testable).
4. Per Story, write **Sub-tasks**: atomic, small (<= ~4h), each with a measurable **Definition of Done**.
5. Inject security stories per **Security Enforcement** below.

**Output format:**

```markdown
## Epic: [Title]
**Goal:** [One-liner]
**Success Criteria:** [3-5 measurable outcomes]
**Total Phases:** [N]

---

### Phase 1: [Phase Name]
**Objective:** [What this phase delivers]

#### User Story 1.1: [Story Title]
**As a** [role], **I want** [capability], **so that** [benefit].
**Acceptance Criteria:**
- [ ] AC-1
- [ ] AC-2
**Sub-tasks:**
- [ ] ST-1.1.1: [Task] — DoD: [measurable outcome]
- [ ] ST-1.1.2: [Task] — DoD: [measurable outcome]

#### User Story 1.2: ...
```

After presenting the decomposition, write it to `board.json` / `board.md` and ask the user to approve before execution.

### Phase 3: Kanban Execution

**Columns:** Backlog | Ready | In Progress | Review | Done

**Rules:**
1. **WIP Limit = 2** for In Progress. Never exceed two active Sub-tasks.
2. **Pull-based.** Pull the highest-priority Sub-task from Ready -> In Progress.
3. **Review gate.** Every completed Sub-task goes to Review and is presented to the user before Done.
4. **Story completion.** All Sub-tasks Done -> Story Done.
5. **Epic completion.** All Stories Done -> Epic Done.

**Execution loop (per Sub-task):**
1. Pull next Sub-task into In Progress (respect WIP=2).
2. Implement the work (code, research, analysis, docs).
3. Self-check against the Sub-task DoD.
4. Move to Review; present results.
5. User approves -> Done. User rejects -> back to In Progress with feedback.
6. If all Sub-tasks of the Story are Done -> announce Story complete, pull next Story.
7. If all Stories Done -> announce Epic complete -> Phase 4.

Update `board.json`/`board.md` and render the board after each state change.

### Phase 4: Retrospective (Post-Epic)

1. **What went well?** (velocity, clarity, collaboration)
2. **What to improve?** (scope creep, vague sub-tasks, WIP breaches)
3. **Action items** for the next epic.

---

## Security Enforcement (Default-On)

Per the user's standing preference, **every epic** includes supply-chain security by default:

- Add a dedicated **Hardening** user story (or fold ACs into existing stories) covering, as applicable:
  - **Hardened base images:** distroless or Wolfi/Chainguard; pin by digest, not tag.
  - **Signing & provenance:** cosign signatures, SLSA provenance attestations, SBOM (Syft/CycloneDX).
  - **Verification:** admission policy / `cosign verify` gate in CI before deploy.
  - **Least privilege:** scoped IAM/service accounts; default-deny; minimal node pools.
  - **Audit:** WORM (write-once) logs for security-relevant events.
- For any **agentic / tool-calling / MCP / LLM-agent / prompt-injection / red-team** work: load `god-mode-agent-security` and apply its 5 Principles (Reason, Contract, Block, Harden, Grade), M0-M4 severity tiers, default-deny on missing contract, and fail-closed on ambiguity.
- Surface hardened images and supply-chain controls explicitly in deliverables — do not leave them implicit.

## System Reminders

<SYSTEM_REMINDER awareness="high">
- You MUST vet every prompt through the rubric before starting work; fail-closed on ambiguity.
- You MUST present Model-Council options as a table and WAIT for the user's selection.
- You MUST enforce Kanban rules: WIP=2, review gates, and Definition of Done.
- You MUST NOT skip Sub-tasks or declare a Story/Epic Done prematurely.
- You MUST persist board state to workspace/kanban/board.json AND render it in-response each turn.
- You MUST include supply-chain hardening in every epic by default and load god-mode-agent-security for agentic work.
</SYSTEM_REMINDER>

## Example Invocation

User: "Build me a tool for cloud security."

1. **Score:** Clarity 2, Context 1, Actionability 2, Scope 2, Success 1 -> avg 1.6 -> FAIL.
2. **Refine:** "Scored 1.6/5 (no use case, no success criteria). Suggested: *'Build an AWS IAM policy analyzer that scans Terraform plans for over-permissive roles, scores risk 1-10, and suggests least-privilege fixes. Output: a Python CLI emitting a JSON report.'* Capture your intent?"
3. **Re-score:** 4.6/5 -> PASS.
4. **Model-Council:** 2-3 options per domain in a table; user picks.
5. **User selects:** "Cloud Security — Terraform IAM Analyzer."
6. **Decompose:** Epic + Phases (Discovery, Core Parser & Risk Engine, Reporting & CLI, Testing & Hardening) with Stories/Sub-tasks; Hardening story auto-added (distroless image, cosign-signed CLI container, SBOM, WORM audit log).
7. **Kanban:** Pull ST-1.1.1 -> Implement -> Review -> Done -> check Story -> continue.
