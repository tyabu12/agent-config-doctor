# 🩺 agent-config-doctor

A [skill](https://agents.md/) that health-checks your AGENTS.md and SKILL.md configuration files across AI coding agents.

It goes beyond structural linting — detecting semantic conflicts, cross-file contradictions, and best practices gaps across tools like Copilot CLI, Claude Code, Codex, Cursor, and more.

## Why agent-config-doctor

Typical linters check file existence, frontmatter syntax, and naming conventions.
agent-config-doctor uses an AI coding agent skill to inspect what they can't.

|  | Linters | agent-config-doctor |
| --- | --- | --- |
| Frontmatter & YAML syntax | ✅ | ✅ |
| File/directory existence | ✅ | ✅ |
| **Semantic consistency** (e.g. contradictory instructions within AGENTS.md) | ❌ | ✅ |
| **Cross-file contradiction detection** (AGENTS.md ↔ SKILL.md) | ❌ | ✅ |
| **Nested AGENTS.md consistency** (root ↔ subdirectory conflicts) | ❌ | ✅ |
| **Best practices sync** (fetches latest official docs at runtime) | ❌ | ✅ |

### Cross-tool compatible

Unlike tool-specific linters, agent-config-doctor focuses on the [AGENTS.md](https://agents.md/) open standard and the [Agent Skills specification](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills) — formats shared across 60+ AI coding tools. The skill itself runs on any agent that supports SKILL.md (Copilot CLI, Claude Code, etc.).

---

For security reasons, this skill is strictly read-only — it only outputs a report and never modifies any files.

If the findings look reasonable, just ask your agent to "update my config based on these findings" and it will optimize your configuration for you.

### Sample output

```
> /agent-config-doctor

Running agent config health check. Let me gather the initial context.

...

# Agent Config Health Check

**Date**: 2026-04-02
**Project**: my-app
**Reviewer**: /agent-config-doctor
**Review iterations**: 0
**Mode**: Full

## Summary

| Section                    | Status       | Issues |
|----------------------------|--------------|--------|
| 0. AGENTS.md Structure     | ✅ PASS      | 0      |
| 1. AGENTS.md Content       | ⚠️ WARN      | 2      |
| 2. Nested Consistency      | ❌ FAIL      | 1      |
| 3. Skills                  | ✅ PASS      | 0      |
| 4. Cross-file              | ❌ FAIL      | 1      |
| 5. Best Practices          | ℹ️ ADVISORY  | 2      |

## Recommended Actions
1. [❌ FAIL] Section 2: Root AGENTS.md says "use pnpm" but packages/api/AGENTS.md says "use npm"
2. [❌ FAIL] Section 4: AGENTS.md says "never use rm -rf" but skill deploy-helper references it
3. [⚠️ WARN] Section 1: AGENTS.md is 183 lines — consider splitting into nested files
4. [⚠️ WARN] Section 1: Instruction "follow best practices" is not actionable (no concrete command or threshold specified)
```

---

## Installation

Copy the skill files into your project. Place them in `.agents/skills/` for maximum cross-tool compatibility:

```
mkdir -p .agents/skills/agent-config-doctor
for f in SKILL.md project.md reference.md; do
  curl -fsSL "https://raw.githubusercontent.com/tyabu12/agent-config-doctor/main/skills/check/$f" \
    -o ".agents/skills/agent-config-doctor/$f"
done
```

## Usage

```
# Recommended: full check (includes best practices search)
/agent-config-doctor

# Light mode: structural checks only (faster, no web access)
/agent-config-doctor light
```

## What it checks

The skill consists of the following 6 review sections.

| Section | What it checks |
| --- | --- |
| 0. AGENTS.md Structure | Existence, encoding, line count, section structure, override files |
| 1. AGENTS.md Content | Command accuracy, path references, tech stack, ambiguous/contradictory instructions, staleness |
| 2. Nested Consistency | Root ↔ subdirectory AGENTS.md contradictions, precedence issues, nesting depth |
| 3. Skills | SKILL.md frontmatter validation, description quality, tool permissions, placement paths |
| 4. Cross-file | AGENTS.md ↔ SKILL.md contradictions, skill-to-skill consistency |
| 5. Best Practices | Fetches latest AGENTS.md and Agent Skills docs at runtime and compares against your config |

FAIL items are cross-reviewed by a sandboxed subagent (max 1 iteration) before the final report.

## Security

This skill is designed with security in mind while maintaining practical utility.

* **Strictly read-only** — never modifies any files
* **Prompt injection defense** — all config file content and web content is treated as data to analyze, never as instructions to follow
* **Scoped web access** — WebSearch and WebFetch are restricted to a hardcoded allowlist of official documentation domains only. No project-specific information ever leaves your machine via search queries
* **Output sanitization** — findings reference file paths and line numbers, never quoting content verbatim, preventing second-order injection if the report is shared
* **Subagent sandboxing** — cross-review subagents are limited to `Read, Glob, Grep` only

## Compatibility

| Agent / Tool | Supported | Recommended Model | Notes |
| --- | --- | --- | --- |
| Claude Code | ✅ | Opus (auto) | Full support (all features including `model: opus`) |
| Copilot CLI | ✅ | Opus | `model` field ignored; tool pre-approval may prompt |
| VS Code (Copilot) | ✅ | Opus | Skills auto-discovered from `.agents/skills/` |
| Codex CLI | ✅ | GPT-5.4 | SKILL.md auto-discovered from `.agents/skills/` |
| Cursor | ✅ | Composer 2 | Skills auto-discovered from `.agents/skills/`, `.cursor/skills/` and others |

## Requirements / Limitations

* **Model**: The skill specifies `model: opus` for best accuracy. Claude Code uses Opus automatically. Other tools ignore this field and use their default model — structural checks (Sections 0-3) work reliably across models, but semantic analysis (contradiction detection, ambiguity flagging) and best practices comparison (Section 5) may be less accurate with smaller models. See the Compatibility table for recommended models per tool
* Full mode uses `WebSearch`, `WebFetch`, and `Agent` tools for best practices comparison and cross-review. `light` mode uses only `Read`, `Glob`, `Grep`, `Bash`, and `Agent`
* **Read-only** — never modifies any configuration files. No auto-fix functionality
* **Execution time**: Full review takes approximately 3-5 minutes depending on project size. `light` mode is faster but skips best practices search

## Related

* **[claude-config-doctor](https://github.com/tyabu12/claude-config-doctor)** — Health check for Claude Code-specific configuration (CLAUDE.md, rules, hooks, settings). By the same author.
* **[AGENTS.md](https://agents.md/)** — The open standard for guiding AI coding agents
* **[Agent Skills specification](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills)** — The shared skill format

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

[MIT](LICENSE)
