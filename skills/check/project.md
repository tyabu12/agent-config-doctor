# Standard Project Diagnostic Procedure

This file defines the diagnostic procedure for AGENTS.md and SKILL.md health checks.
It is read by the main SKILL.md after Before Starting.
Valid value lists and severity definitions are in reference.md.
Safety Constraints (including Bash usage restrictions) are defined in the main SKILL.md and apply to all sections below.

## AGENTS.md Best Practices Thresholds

Based on GitHub Blog analysis of 2,500+ repositories and community guidelines:

- **Line count**: WARN if over 150 lines (per agents.md official guidance: "Aim for ≤ 150 lines"). Note if over 80 lines (consider splitting into nested files).
- **Commands placement**: Commands (build, test, lint) should appear in the first 30% of the file.
- **Boundary definitions**: Should include explicit "do" and "don't" sections or equivalent.
- **Code examples**: At least one concrete code example preferred over prose-only style guides.
- **Nested file depth**: WARN if AGENTS.md nesting exceeds 3 directory levels.

## Mode Selection

Check the command argument: `$ARGUMENTS`

1. Trim leading and trailing whitespace from `$ARGUMENTS`. If the trimmed value contains spaces, treat as unrecognized.
2. If the result is `light`: **light mode** — execute Sections 0-4, skip Section 5, then proceed to Review Loop. Add `**Mode**: Light` to the report header.
3. If the result is `full` or empty/unset: **full mode** — execute all sections 0-5. Add `**Mode**: Full` to the report header.
4. If the result is anything else: stop immediately and output:
   > "Unrecognized argument. Valid options: `light` (skip best practices section), `full` (all sections), or omit for full review."

## Procedure

Run the following reviews sequentially and collect findings as you go.
If a section's target files do not exist, mark it SKIPPED and move on.

**Scalability rule**: If any section involves more than 30 files, sample up to 30 representative files (prioritizing recently modified) and note: "Sampled N of M files."

### 0. AGENTS.md Structure

If root AGENTS.md does not exist, mark SKIPPED.

- [ ] **Existence**: Confirm AGENTS.md exists at the repo root.
- [ ] **Encoding**: Verify via Bash `file AGENTS.md` that encoding is UTF-8 (without BOM). Flag other encodings.
- [ ] **Line count**: Report total lines. WARN if over 150 lines (see AGENTS.md Best Practices Thresholds above). Note if over 80 lines.
- [ ] **Section structure**: Check for presence of key sections. Recommended sections per AGENTS.md best practices: build/test commands, coding conventions, project structure overview, boundary definitions (do/don't). Note missing categories as informational.
- [ ] **Commands placement**: Check whether build/test/lint commands appear in the first 30% of the file. Agents reference commands frequently — burying them reduces effectiveness. Note as informational if commands are placed late.
- [ ] **AGENTS.override.md**: Check if `AGENTS.override.md` exists at the repo root. If so, note it — overrides take precedence and may cause unexpected behavior if forgotten.

### 1. AGENTS.md Content

If root AGENTS.md does not exist, mark SKIPPED.

Read root AGENTS.md and check:

- [ ] **Command accuracy**: Every executable command mentioned (e.g., `npm test`, `pnpm build`, `pytest`) — verify the corresponding tool exists in the project. Cross-reference with `package.json` scripts, `Makefile` targets, `Cargo.toml`, etc. Flag commands that reference non-existent scripts or tools.
- [ ] **Path accuracy**: Every file path or directory path mentioned — verify it exists on disk via Glob. Flag broken references.
- [ ] **Tech stack accuracy**: If AGENTS.md describes the tech stack, cross-reference against actual dependency files (`package.json`, `Cargo.toml`, `pyproject.toml`, `go.mod`, etc.). Flag major frameworks listed in AGENTS.md that don't appear in dependencies, and core dependencies absent from AGENTS.md.
- [ ] **Ambiguous instructions**: Flag instructions that are too vague for an agent to act on (e.g., "follow best practices", "be careful", "use common sense"). These waste context tokens without providing actionable guidance. Rate as WARN. In the report, briefly note why the instruction is not actionable (e.g., "no concrete command or threshold specified").
- [ ] **Contradictory instructions**: Flag instructions within the same file that contradict each other (e.g., "always use tabs" vs "indent with 2 spaces").
- [ ] **Stale references**: Flag sections that reference file paths, directory structures, or tools that no longer exist. Do not flag high-level concept references — only concrete, verifiable references.

### 2. Nested Consistency

Use the nested AGENTS.md files discovered in "Before Starting" (see SKILL.md).
If only the root AGENTS.md exists (no nested files), mark SKIPPED.

For each nested AGENTS.md:

- [ ] **Internal validity**: Apply the same Structure checks from Section 0 (encoding, line count) and Content checks from Section 1 (command accuracy, path accuracy) to each nested file.
- [ ] **Contradiction detection**: Compare each nested AGENTS.md against the root AGENTS.md. Flag direct contradictions:
  - Different package managers specified (e.g., root says `pnpm`, nested says `npm`)
  - Conflicting code style rules (e.g., root says "use tabs", nested says "use spaces" without scoping)
  - Conflicting boundary definitions (e.g., root says "never modify tests", nested says "always update tests")
- [ ] **Precedence awareness**: Note that the nearest AGENTS.md takes precedence. A nested file intentionally overriding root instructions is acceptable if scoped appropriately — only flag unscoped contradictions.
- [ ] **Nesting depth**: WARN if any nested AGENTS.md exceeds 3 directory levels deep from the repo root. Deep nesting makes precedence resolution harder for agents to follow.
- [ ] **Override files**: If any `AGENTS.override.md` exists in subdirectories, note them and check for contradictions with both the nearest `AGENTS.md` and the root.

### 3. Skills

Use the SKILL.md files discovered in "Before Starting" (see SKILL.md).
If no SKILL.md files exist, mark SKIPPED.

**Skip this plugin's own skill** — do not review agent-config-doctor's own SKILL.md.

For each other SKILL.md:

- [ ] **Frontmatter syntax**: Verify YAML frontmatter is well-formed and parseable.
- [ ] **Required fields**: Verify `name` and `description` are present. Check `name` is lowercase with hyphens only. Check `description` is not empty.
- [ ] **Field validation**: For each recognized field present, validate against the valid values list in [reference.md](./reference.md). Flag unrecognized fields as WARN.
- [ ] **Tool permissions**: If `allowed-tools` is present, validate tool names against the valid tools list in [reference.md](./reference.md).
- [ ] **Description quality**: WARN if `description` is under 20 characters (too short for reliable auto-triggering) or over 1024 characters (may be truncated).
- [ ] **Content accuracy**: Verify that files and paths referenced in the skill body actually exist on disk.
- [ ] **Supporting files**: If the skill references supporting files (e.g., `reference.md`, `examples/`, scripts), verify they exist within the skill directory.
- [ ] **Placement path**: Note which skill directory the SKILL.md is in. If placed in a tool-specific path (e.g., `.claude/skills/`), note it is not cross-tool accessible. Suggest `.agents/skills/` for maximum compatibility if appropriate.

### 4. Cross-file Consistency

This section checks consistency between AGENTS.md and SKILL.md files. AGENTS.md-to-AGENTS.md consistency is covered in Section 2.

If only one of AGENTS.md or SKILL.md files exists (not both), mark SKIPPED.

- [ ] **AGENTS.md ↔ Skills alignment**: Check that skills don't contradict instructions in AGENTS.md. Examples:
  - AGENTS.md says "never use rm -rf" but a skill references it in its procedure
  - AGENTS.md specifies a testing framework but a skill assumes a different one
  - AGENTS.md defines code style rules but a skill generates code in a conflicting style
- [ ] **Skill-to-skill consistency**: If multiple skills exist, check for contradictions between them (e.g., two skills defining conflicting conventions for the same file types).
- [ ] **Nested AGENTS.md ↔ Skills**: If skills are placed in subdirectories alongside nested AGENTS.md files, verify alignment between the local AGENTS.md and the local skills.

### 5. Best Practices (Advisory)

Use WebSearch (or equivalent web search tool) to check for current AGENTS.md and SKILL.md best practices. Restrict searches to the allowlist in [reference.md](./reference.md).

If no web search capability is available, mark this section SKIPPED with note: "Web search not available in this environment. Skipping best practices comparison."

Do NOT guess or construct URLs for WebFetch without a prior search result.

If all searches return zero results, note: "Documentation domains may have changed. Skipping best practices comparison." and mark SKIPPED.

Compare the project's configuration against documented recommendations. Areas to check:

- [ ] **AGENTS.md structure**: Does the file follow recommended patterns (commands early, examples over prose, clear boundaries)?
- [ ] **SKILL.md specification compliance**: Are skills following the current Agent Skills specification?
- [ ] **New features**: Are there recently documented features or patterns the project could adopt?
- [ ] **Cross-tool compatibility**: Are configuration files placed for maximum cross-tool compatibility?

For search results needing more detail, use WebFetch to read the full page. Limit to at most 3 page fetches.

**This section is advisory only.** Label all findings as informational, never FAIL. Include source URLs so the user can verify independently.
