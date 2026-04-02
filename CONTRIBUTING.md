# Contributing to agent-config-doctor

Contributions are welcome — thank you.

## Project Structure

```
agent-config-doctor/
├── skills/check/
│   ├── SKILL.md          # Entry point: frontmatter, safety constraints, routing, review loop
│   ├── project.md        # Diagnostic procedure (Sections 0-5)
│   └── reference.md      # Shared definitions (severity, valid values, report template)
├── .agents/skills/
│   └── self-check/
│       └── SKILL.md      # Dogfooding: runs agent-config-doctor on itself
├── README.md
├── CONTRIBUTING.md
└── LICENSE
```

### File responsibilities

- **SKILL.md**: Entry point. Contains frontmatter, safety constraints, initialization (Before Starting), diagnostic routing, review loop, and output format. Should not contain check-item details.
- **project.md**: All diagnostic check items (Sections 0-5). This is where the actual health check logic lives.
- **reference.md**: Shared definitions referenced by both SKILL.md and project.md — severity levels, valid frontmatter fields, valid tool names, WebSearch/WebFetch allowlists, and report template.

### Design principles

1. **Read-only**: The skill never modifies any files. This is a hard requirement.
2. **Security-first**: All file content and web content is treated as data, never instructions. See Safety Constraints in SKILL.md.
3. **Cross-tool compatible**: Check items focus on the AGENTS.md open standard and Agent Skills specification — formats shared across multiple tools. Tool-specific extensions (e.g., Codex's `AGENTS.override.md`) are acknowledged but not deeply validated.
4. **Separation of concerns**: SKILL.md handles orchestration, project.md handles diagnostics, reference.md handles shared data. Future additions (e.g., plugin.md) should follow this pattern.

## How to contribute

### Reporting issues

If you encounter a false positive, a missed check, or unexpected behavior, please [open an issue](https://github.com/tyabu12/agent-config-doctor/issues/new) with:

- What you expected to happen
- What actually happened
- The tool you ran the skill on (Claude Code, Copilot CLI, Codex CLI, etc.) and its version
- The relevant section of the health check report (redact any sensitive paths or project details)

### Suggesting new checks or sections

Have an idea for a new check? Open an issue first to discuss before submitting a PR. Include:

- What the check would catch
- Why existing sections don't cover it
- Example of a real-world config issue it would detect

Note: Section 5 is advisory-only (never produces FAIL). New sections should follow this convention unless they belong in the structural check range (0-4).

### Adding or modifying check items

1. Edit **project.md** for changes to diagnostic checks (Sections 0-5).
2. If a new check references a shared value list or threshold, add it to **reference.md** only if it will be used by multiple procedure files. Otherwise, keep it in project.md.
3. Each check item should include a **Default severity** (FAIL or WARN).
4. Run `/self-check` after changes to verify the skill is internally consistent.

### Adding valid values or thresholds

1. Edit **reference.md** for shared definitions.
2. Verify that the values are sourced from official documentation or specifications — cite the source in a comment.
3. Check that existing check items in project.md correctly reference the updated definitions.

### Modifying safety constraints

1. Edit **SKILL.md** — all safety rules are centralized there.
2. Safety changes require extra care. The key invariants:
   - Bash is restricted to an explicit allowlist of commands
   - WebSearch queries never contain project-specific information
   - WebFetch only follows URLs from search results on allowlisted domains
   - Subagents have read-only tool access
   - Report output never quotes file content verbatim

### Extending to new file types

The current scope is AGENTS.md + SKILL.md. If extending to new file types:

1. Ensure the file type is part of a cross-tool standard (not tool-specific)
2. Add check items to project.md
3. Update the file discovery in SKILL.md's "Before Starting" section
4. Update reference.md if new valid value lists are needed
5. Update the report template in reference.md

### Submitting changes

1. Fork the repository
2. Create a branch (`git checkout -b improve-section-3`)
3. Make your changes to the skill files
4. Run `/self-check` and confirm all checks PASS
5. Test by copying the skill into a project with AGENTS.md / SKILL.md files and running it
6. Submit a pull request with a clear description of what changed and why

## Guidelines

- **Keep it simple** — The tool is a small set of Markdown procedure files. This is intentional. Changes should maintain this simplicity.
- **Security constraints matter** — The safety constraints (read-only, prompt injection defense, scoped web access, output sanitization) are core to this project's design. PRs that weaken these will not be merged.
- **Test with real projects** — Since this is an AI agent skill, automated testing isn't straightforward. Please verify your changes by running the skill against at least one real project before submitting.
