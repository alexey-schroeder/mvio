# Agent instructions — mvio

Read this file when starting or restoring project context.


## DeepSeek Harness: mandatory multi-agent prompt and context rule (2026-10-25)

This is an **operating preference for future prompts**, not a claim that every run already uses Harness. Read this section at the start of each session, on context restoration/handoff, and **before drafting any prompt** for this repository. Preserve all existing repository-specific rules and acceptance gates.

- Default development orchestration: **DeepSeek Harness Desktop native Workflow Engine**, with **Claude Code as Supervisor/independent Reviewer** and **Codex as implementing Worker**. The primary Harness agent coordinates; do not replace the native Workflow with a standalone orchestration script.
- When the user asks to "write a prompt", "make a Codex prompt", or requests autonomous work, **default to a ready-to-paste prompt for Harness** explicitly naming both subagents and their roles, unless the user specifically requests a single-agent prompt or a different tool. Include the project/workspace path only if verified; do not invent local paths.
- Typical sequence: Claude defines scope and acceptance criteria -> Codex implements and runs focused checks -> Claude independently reviews code and evidence -> on `CONTINUE`, Codex fixes and reruns -> on `DONE`, stop; on `NEEDS_HUMAN`, stop and explain the blocker. Set a finite cycle limit (default 5), time budget, no-progress detection, and final factual report. Do not claim success from model assertions alone; require actual tool/test evidence.
- **Codex model preference:** select the **third most recently released Codex-compatible model** among models actually available to the user's Codex subscription at execution time (newest = position 1, second-newest = 2, desired = 3). Rank distinct model families/versions by verified release chronology, **not** alphabetically or by an assumed fixed model ID; do not mistake CLI/runtime version for model version. Before invoking Codex, enumerate available models and confirm the ordering. If release ordering cannot be verified, fewer than three candidates exist, or Harness cannot select the requested model for the official subagent, **do not silently substitute** the latest/default model: report the limitation and request a decision. Never hard-code a model name that will become outdated.
- Codex should continue using its official Harness subagent and ChatGPT sign-in; Claude Code should continue using its official subagent and Claude Pro sign-in. Main Harness model/API billing is separate. Do not copy credentials into repos or prompts.
- Preserve repository-specific safety rules, branches, tests, and approval/merge policy. Do not assume model-generated `DONE` authorizes push or merge.
- When producing a context handoff/summary, **explicitly include this Harness/Codex/Claude workflow and third-newest Codex model preference**, plus a pointer to this file; do not rely on chat memory alone.
- Shared router and recovery docs: `alexey-schroeder/CI-Infrastructure` -> `docs/harness-workflow-codex-claude.md`. Current router supports `agent(prompt, {provider:'codex'})` and `agent(prompt, {provider:'claude-code'})` inside native Harness Workflow. A fully generic persistent development workflow and dynamic Codex model selector are **not yet proven installed**, so prompts must not claim these are already guaranteed.


## Mandatory Harness slash command for development (2026-10-25)

**Critical context-restoration and prompt-authoring rule.** This section must be read whenever starting or resuming work on this project and before drafting any prompt for autonomous coding. Carry the exact command syntax into every handoff summary.

For repository changes, bug fixing, implementation, refactoring, performance optimization, or audits that may lead to edits, the preferred **copy-paste-ready DeepSeek Harness Desktop Development Workflow input** is:

```text
/development-workflow <describe the concrete development task, acceptance criteria, project constraints and test expectations>
```

The **literal `/development-workflow` prefix is mandatory in prompts intended for the low-cost direct-entry workflow**; do not return only a natural-language request such as "Use Harness workflow" or "Run Claude and Codex". When the user asks "write a prompt" for this project, default to producing a ready-to-paste prompt starting with that prefix (unless the user explicitly wants a Codex-only prompt, a different tool, or explanatory text).

This slash command is provided by the custom Development Workflow preset in `alexey-schroeder/CI-Infrastructure` (`harness/development-workflow-command/`). It calls native `workflowEngine.start()` directly, rather than sending the development-task turn through GPT-6 Luna. The program invokes **Claude Code Supervisor → Codex Worker → independent Claude Reviewer**; Claude `CONTINUE` repeats Codex/review, `DONE` terminates immediately, and `NEEDS_HUMAN` stops with an explanation. Defaults: maximum five cycles, approximately 30-minute workflow deadline, detect no progress; honor stricter repository rules.

**Cost distinction:** In Harness `0.2.1-rc.3`, sending an ordinary prose request (even while Development Workflow preset is selected) may invoke the GPT-6 Luna main OpenAI API model. Using the registered `/development-workflow <task>` command avoids the primary-model turn for that fixed workflow, while Claude Code/Codex use their official subscription authorization. Do not promise zero total costs or assume the command exists until the custom preset/command has been installed and enabled.

Codex model preference remains the third-newest available Codex-compatible model **when verifiably selectable**; the present official provider supports a manually configured model but not dynamic third-newest selection. Do not claim dynamic selection exists.

Context recovery: Read root `AGENTS.md` and existing project-specific mandatory documents. Explicitly repeat both the slash command and the Claude/Codex roles in any context handoff. Do not override repository safety, test, branch, or merge requirements.
