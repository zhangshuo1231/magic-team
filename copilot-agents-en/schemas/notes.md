# Notes Schema

This file defines the structure for project Notes. Any agent that uses Notes - especially Conductor, Analyst, and Note Keeper - must follow this schema.

The goal of Notes is to give any reading agent a project understanding close to that of a developer who has maintained the project for years: where code lives, how data flows, why major choices appear to exist, how to run the system, and where risk areas are.

---

## 0. Design Philosophy (Do Not Remove)

Understanding the design intent of Notes is essential to maintaining them correctly. Keep this section with the schema.

1. **The authors are AI plus user; the readers are AI.** Notes are not ordinary human-facing documentation. They are a structured knowledge base used by agents to search, judge readiness, and work reliably.

2. **Knowledge sources are mixed, so reliability labels are mandatory.** Some content comes from Analyst reading code, some is AI inference, and some comes from the user or external materials such as Confluence/Jira, diagrams, design docs, chat excerpts, skills, and operational experience. The labels `[Confirmed]`, `[Inferred]`, and `[Needs input]` let Conductor decide whether work conditions are mature.

3. **Notes are a living knowledge base, not a one-time snapshot.** Initial Analyst learning creates a skeleton. Later, the user may add small pieces of knowledge from code reading, coworker conversations, meetings, Confluence pages, diagrams, design docs, chat excerpts, fragments, prewritten skills, or production experience. These materials must be digested and merged into the right sections over time, not appended as loose fragments or pasted wholesale.

4. **New knowledge should be verified when possible.** Pure intent or historical context may not be verifiable in code, and should be labeled accordingly. But if a user statement or new material includes a claim about current implementation, Analyst should usually verify it against code before Note Keeper records it as fact. This prevents stale memory or stale documents from becoming `[Confirmed]` documentation.

5. **Updates must include consistency review.** If new information conflicts with an existing `[Confirmed]` fact, Note Keeper must not overwrite it directly. Conductor should arrange code verification or user clarification first.

6. **The document's own evolution should preserve rationale.** Structural decisions such as this design philosophy should remain visible so future maintainers understand why the Notes system works this way.

---

## 1. File Location Convention

```text
{project root}/.copilot-notes/
    project-notes.md       -> project-level Notes describing current state; updated in place
    adr.md                 -> incremental decision records; append-only, newest first
```

---

## 2. Reliability Labels

Every content block in Notes must include a reliability label.

- **[Confirmed]**: Based on code facts, configuration, tests, explicit user confirmation, or verified external documentation. Safe to rely on.
- **[Inferred]**: A reasonable inference from code, but the true intent is not confirmed. Use with caution.
- **[Needs input]**: Not knowable from code or currently missing. Requires user, Jira, Confluence, diagrams, design docs, skills, or other external input.

Labels should appear at the start of each bullet or paragraph.

Example:

```markdown
- [Confirmed] The service uses Spring Boot 3.2, as shown in pom.xml.
- [Inferred] Kafka appears to be used for decoupling and asynchronous processing; the design intent still needs user confirmation.
- [Needs input] The full list of local environment variables is not documented in the repository.
```

**How Conductor uses labels:** If task-critical information is mostly `[Inferred]` or `[Needs input]`, Conductor should treat the work conditions as immature and ask the user or Analyst for clarification before implementation.

**About `[Label]` placeholders below:** In templates, `[Label]` is only a placeholder. Real Notes must replace it with `[Confirmed]`, `[Inferred]`, or `[Needs input]`.

---

## 3. `project-notes.md` Structure

```markdown
# Project Notes

## Metadata
- last_updated_at: YYYY-MM-DD HH:mm
- source_branch: {branch read when Notes were generated or updated}
- source_commit_id: {commit ID used as the code-fact basis for this Notes version}
- notes_scope: {whole-project / module:{module name} / limited:{scope description}}
- workspace_state: {clean / dirty / unknown; if dirty, summarize relevant uncommitted changes}
- author: {updater ID}

## 1. Project Overview
[Label] What this project does, what problem it solves, and where it sits in the larger system.

## 2. Technology Stack and Versions
[Label] Main languages, frameworks, runtime versions, build tools, and major dependencies. Include versions when visible from code/config.

## 3. Code Map
This section helps Planner locate code quickly.
- [Label] Top-level directory responsibilities.
- [Label] Key entry points: main classes, handlers, controllers, commands, jobs, or functions.
- [Label] Mapping from common change goals to files or modules to inspect.

## 4. Module Boundaries and Responsibilities
[Label] What each module does and how modules depend on each other.

## 5. Core Data Flows
[Label] Trace the most important end-to-end paths:
where requests/events/jobs enter, which layers/classes they pass through, how data transforms, and where results land.

## 6. Key Design Decisions
This section often contains more `[Inferred]` and `[Needs input]` content because code may not explain why choices were made.
- [Label] Important architecture choices and likely rationale.
- [Label] Historical or sensitive areas that should not be changed casually.

## 7. Data Model
[Label] Core entities, DTOs, schemas, tables, relationships, and important invariants.

## 8. External Dependencies and Integrations
[Label] External systems, databases, message queues, third-party services, APIs, and integration patterns.

## 9. Build and Runtime
This section supports local debugging and validation.
- [Label] How to build, run, and debug locally.
- [Label] Required configuration, environment variables, secrets, and local setup notes.

## 10. Testing Strategy
[Label] How tests are organized, how to run them, major coverage areas, and known gaps.

## 11. Risk Areas and Cautions
This section often combines code-observed risk with user experience.
- [Label] Fragile, complex, highly coupled, or surprising areas.
- [Label] Known traps, conventions, or counterintuitive design details.
```

---

## 4. `adr.md` Structure

```markdown
# Incremental Decision Records (ADR)

Newest entries always appear at the top. Old entries are not deleted.

---

## {YYYY-MM-DD} | {Jira Ticket ID or Task ID} | {one-line description}

### Metadata
- branch: {feature branch}
- base_commit_id: {base commit for this change, usually the merge-base with target branch}
- working_tree_state: {uncommitted / pending-commit / committed}
- diff_summary: {summary of files or modules changed}
- final_commit_id: {final commit ID; use pending if not committed yet}
- author: {user ID}

### Change Scope
{Modules and files changed.}

### Reason for Change
{Why the change was needed and what task goal it served.}

### Technical Decisions
{Important technical choices made, and why these choices were made instead of alternatives.}

### Risks and Cautions
{Risks introduced by the change or follow-up cautions.}

---

{next ADR entry}
```

---

## 5. Conductor Rules for Judging Notes Completeness

After reading `project-notes.md` metadata, Conductor should judge:

1. **Notes missing** -> ask Analyst to learn the project.

2. **`source_commit_id` is current or an ancestor of current target branch/HEAD, and `notes_scope` covers the task area** -> Notes are basically usable. Conductor may inspect the diff from `source_commit_id` to the current workspace:
   - Small delta not touching task area -> proceed with Notes.
   - Large delta or task-area delta -> tell the user and consider Analyst incremental update.

3. **`source_commit_id` missing, not in current history, or `notes_scope` does not cover the task area** -> Notes are stale or insufficient. Recommend Analyst refresh or bounded investigation.

4. **`workspace_state` is `dirty` or `unknown`** -> do not automatically invalidate Notes, but lower confidence around areas affected by uncommitted changes. If relevant to the task, ask Analyst to re-check.

Even when freshness is acceptable, Conductor must check reliability labels. If task-critical content is mostly `[Inferred]` or `[Needs input]`, conditions are not mature enough for implementation.

---

## 6. Targeted Update and Consistency Protocol

The user may ask to supplement or correct Notes at any time, not necessarily as part of a development task. The user may also provide a new document, diagram, Confluence page, screenshot/PDF, chat excerpt, fragment, prewritten skill, or other artifact to improve the knowledge base. This is normal because Notes are a living knowledge base. These updates must be digested and merged, not appended to the end or pasted wholesale.

### Step 1: Classify the Material and Decide Whether Code Verification Is Needed

Conductor first identifies the material source, target project, format, and accessibility. If the material is inaccessible, permission-blocked, unreadable, visually unclear, or missing context, Conductor should ask for readable content or clarification. Then Conductor decides whether the user's new information or material includes an implementation claim.

- If it contains a claim that can be checked in code, Conductor should usually ask Analyst to verify it. Analyst reads relevant Notes, inspects code evidence, and reports confirmed/disproved/partially confirmed/not determinable.
- If it is pure intent, history, or context that code cannot verify, skip code verification but label the source and confidence honestly.
- Intent and implementation are often mixed. Verify the implementation part when possible.
- For large, mixed, or uncertain-format material, Conductor should usually ask Analyst for document digestion first: extract candidate knowledge, classify it, and identify which claims need code verification.

### Step 2: Locate

Decide which `project-notes.md` section should contain the new information. If it affects multiple sections, update each relevant section consistently.

### Step 3: Merge, Do Not Append

Integrate new information into the document. Do not dump raw documents into Notes. Notes store digested and structured knowledge, not a raw document archive.

- **Fill**: complete a `[Needs input]` gap.
- **Deepen**: add detail to an existing point by rewriting or extending it.
- **Correct**: fix an `[Inferred]` point or an outdated fact after consistency review.
- **Cross-link**: reflect information across multiple sections when it affects more than one area.

After merging, the Notes should still read as a coherent document.

### Step 4: Consistency Review

Compare new information against existing Notes:

- If it fills a gap or agrees with existing content, merge it and choose the correct label.
- If it conflicts with `[Inferred]`, prefer verified or user-confirmed information and adjust the label.
- If it conflicts with an existing `[Confirmed]` fact, do not overwrite directly. Conductor must arrange Analyst verification or user clarification first.

### Step 5: Update Metadata

After any write, Note Keeper updates the file metadata.

---

## 7. Important Rules

- Notes are the structured cross-session knowledge base.
- `project-notes.md` describes current state and is updated in place.
- `adr.md` is append-only; newest entries go first.
- ADR can be written before a change is committed. In that case, `final_commit_id` is `pending`; after the user commits, Note Keeper may backfill it or add a follow-up note when Conductor explicitly asks.
- Every Note Keeper update must update metadata.
- Every content block must include `[Confirmed]`, `[Inferred]`, or `[Needs input]`.
- Analyst must not pretend uncertainty is certainty.
