# English Draft Tracking Checklist

Last updated: 2026-06-01; Chinese files were synchronized back from the deployed English version.

## Target Directory Structure

```text
~/.copilot/
    agents/
        conductor.agent.md      Done
        analyst.agent.md        Done
        planner.agent.md        Done
        coder.agent.md          Done
        verifier.agent.md       Done
        test-writer.agent.md    Done
        note-keeper.agent.md    Done
    schemas/
        notes.md                Done
        planner-output.md       Done
```

## Files

| Current file | Intended target path | Status |
| --- | --- | --- |
| notes.md | schemas/notes.md | Done |
| planner-output.md | schemas/planner-output.md | Done |
| conductor.agent.md | agents/conductor.agent.md | Done |
| analyst.agent.md | agents/analyst.agent.md | Done |
| planner.agent.md | agents/planner.agent.md | Done |
| coder.agent.md | agents/coder.agent.md | Done |
| verifier.agent.md | agents/verifier.agent.md | Done |
| test-writer.agent.md | agents/test-writer.agent.md | Done |
| note-keeper.agent.md | agents/note-keeper.agent.md | Done |

## Agent Profile Format Decisions

- Agent profiles use Markdown files with `.agent.md` extensions and YAML frontmatter.
- Each agent has a required `description` field.
- `Conductor` is `user-invocable: true`.
- All specialist agents are `user-invocable: false`; they are intended to be invoked by Conductor as subagents, not manually selected as the user-facing entry point.
- No `tools` allowlist is configured. The original design intentionally avoids a tools allowlist because tool and MCP server names may vary across environments. Role boundaries are enforced by the prompts.
- `Conductor` stores the subagent roster as `metadata.agents: "['Analyst', 'Planner', 'Coder', 'Verifier', 'Test Writer', 'Note Keeper']"` and in the body. The non-official top-level `agents:` field is intentionally not used, for GitHub configuration compatibility, and the metadata value is a string as GitHub's configuration reference requires.
- `tools` is intentionally omitted. Under GitHub's custom-agent configuration rules, omitted `tools` means all available tools are enabled, including custom-agent invocation where supported.
- Agents with write permissions (`Coder`, `Test Writer`, `Note Keeper`) explicitly forbid `git commit`, `git push`, and other version-control submission actions.
- Iteration is stateless: when asking an agent to revise prior work, Conductor must include the prior output plus the new feedback in the task package.

## Cross-File Consistency

- `Conductor` and `Verifier` agree on two-layer verification and regression review.
- `Conductor`, `Analyst`, `Note Keeper`, and `schemas/notes.md` agree on Notes reliability labels and the incremental Notes-growth workflow.
- `Planner` and `schemas/planner-output.md` agree on the implementation-plan and insufficient-information-feedback formats.
- `Coder`, `Verifier`, and `Test Writer` agree that implementation, logical review, and test writing are separate responsibilities.
- Multi-project work is handled by Conductor, which must identify the target project and any reference project in every task package.

## Bilingual Synchronization Rule

- Any substantive change to an agent, schema, or progress document must update both `copilot-agents-cn` and `copilot-agents-en` in the same change.
- The English version is the deployed compatibility baseline. If a new design is discussed first in Chinese, the same change must also be written into the English files before the work is considered complete.
- If a one-sided change is temporarily unavoidable, both `_progress.md` files must record the reason, risk, and remaining sync task. Normal maintenance should not leave one-sided differences.
- Synchronization does not require word-for-word translation, but responsibilities, constraints, frontmatter configuration, paths, schema fields, and output formats must remain equivalent.

## Enhancement: New Documents, Diagrams, and Skills as Notes Inputs

The English version now supports a new use case: the user may provide Confluence pages, diagrams, screenshots/PDFs, design docs, chat excerpts, fragments, prewritten skills, or other uncertain-format artifacts to improve Notes. Updates:

- `Conductor` recognizes this as continuous Notes improvement, checks accessibility and target project, and routes large/mixed material to Analyst for digestion and code verification.
- `Analyst` has a document-digestion investigation mode: extract candidate knowledge, classify it, verify implementation claims, and recommend Notes sections and labels. Skill-like material is read as knowledge by default, not installed or executed.
- `Note Keeper` merges extracted knowledge rather than dumping raw source material, preserving only useful source context.
- `schemas/notes.md` includes the document-intake flow in the targeted-update protocol.

## Status

The English version is the deployed baseline and is ready to be copied into a GitHub Copilot custom-agent setup. The Chinese version is maintained as the synchronized source for Chinese discussion and review.
