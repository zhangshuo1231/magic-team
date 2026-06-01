# Planner Output Schema

This file defines the standard format for Planner's implementation-plan output.

**Upstream**: Conductor provides Planner with a structured task package.

**Downstream consumers**:

- Coder, who implements according to the plan.
- Verifier, who uses the plan as one baseline for verification.

The format must let Coder act without guessing and let Verifier check the result against concrete expectations.

---

## 1. Design Principles

1. **Plan, not code.** Planner describes what to do, where to do it, and how to approach it. Planner does not write the full implementation. Small illustrative snippets, method signatures, or data shapes are allowed when they clarify intent.

2. **Checkable.** Every planned change should be clear enough for Verifier to compare against the implementation.

3. **Honest about uncertainty.** Any unconfirmed assumption must be called out explicitly.

---

## 2. Implementation Plan Format

````markdown
# Implementation Plan

## 0. Task Summary
- Jira Ticket: {ticket ID and title, or "none"}
- Task Type: {new feature / bug fix / logic change / refactor / CI-CD / documentation / other}
- One-Sentence Goal: {what this task should accomplish}

## 1. Background and Understanding
{Planner's understanding of the task based on Conductor's package and relevant code.
Explain the problem being solved and why the proposed direction fits.}

## 2. Impact Scope
List every expected change location:
- File / class / method: {specific path and name}
- Change type: {add / modify / delete}
- Affected modules and possible knock-on effects

## 3. Implementation Steps
List ordered, executable steps. Each step should include:
- Step number and short title
- Files / classes / methods involved
- Concrete description of the change
- Optional key method signatures, data shapes, or illustrative snippets

Steps should be specific enough that Coder does not need to infer intent.

## 4. Existing Conventions to Follow
Describe the project conventions this work must follow:
- Code style and naming
- Existing framework patterns
- Reusable components, helpers, base classes, or utilities
- Similar local implementations to model

This information should come from Notes, Conductor's task package, and task-relevant code reading.

## 5. Assumptions and Open Questions
- [Assumption] {unconfirmed assumption this plan relies on}
- [Open question] {question that requires user input or further investigation}

If this section is non-empty, Conductor may need to clarify before implementation.

## 6. Testing Focus
Guidance for Test Writer:
- Normal paths to cover
- Boundary conditions
- Error and exception scenarios
- Regression-sensitive existing behaviors

## 7. Risks and Cautions
- Risks introduced by this change
- Areas that require extra care
- Potential side effects for other modules or workflows
````

---

## 3. Insufficient-Information Feedback Format

If Planner cannot create a reliable plan from the available task package and targeted code reading, Planner should not force a speculative plan. Use this format instead:

````markdown
# Insufficient Information Feedback

## Why a Reliable Plan Cannot Be Made
{Explain the missing or unreliable information.}

## Information Needed
- {Specific information Conductor should ask the user for, or ask Analyst to investigate}

## What Is Already Known
{State the useful facts already established, so later work does not repeat them.}
````

Conductor is responsible for asking the user or arranging Analyst investigation, then invoking Planner again.

---

## 4. Important Rules

- Planner is read-only.
- The output is planning text, not file modification.
- The plan should be detailed enough that Coder does not guess intent.
- Any uncertainty must be explicit in section 5.
- When information is insufficient, return feedback instead of a weak plan.
