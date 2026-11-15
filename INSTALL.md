# Installing this skill

This repository is both a C style guide (`CODING_RULES.md`) and a
distributable Claude Code plugin that teaches an agent to follow it and
enforce formatting with `clang-format`. Pick the path that matches your
setup.

> **Fork notice:** This is [nexnpvnk's personal fork](https://github.com/nexnpvnk/c-code-style)
> of [Tilen MAJERLE's style guide](https://github.com/MaJerle/c-code-style).
> See [Fork-specific changes](CODING_RULES.md#fork-specific-changes) for links
> to the customized rules. The commands below install this fork.

The formatter configuration requires LLVM 23 or newer. Use the same LLVM
version for `clang-format` and `clangd` when both are installed.

## Option 1: Claude Code plugin (marketplace)

From inside Claude Code, in any project:

```
/plugin marketplace add nexnpvnk/c-code-style
/plugin install c-code-style
```

This adds the `c-code-style` skill, which auto-loads whenever you ask Claude
to write, edit, or review C code, and it will check for / run `clang-format`
as part of that flow.

> The plugin bundles the whole repository — `CODING_RULES.md`, the
> `template.c`/`template.h` references, and `scripts/check-format.sh` all
> travel with it, so it works the same whether you're inside this repo or in
> your own project. This distribution path (`.claude-plugin/marketplace.json`
> + `plugin.json`) was authored against Claude Code's documented plugin
> conventions but not exercised end-to-end with a live `/plugin install` in
> this session — verify the two commands above work before pointing others
> at it, and open an issue on this repo if they don't.

## Option 2: Pinned submodule (any agent, incl. Codex)

Keep the rules and formatter in one versioned dependency:

```sh
git submodule add https://github.com/nexnpvnk/c-code-style.git tools/c-code-style
```

Use this in the project's root `.clang-format`:

```yaml
BasedOnStyle: InheritParentConfig=tools/c-code-style
```

Keep the project's own `AGENTS.md` and link it to
`tools/c-code-style/CODING_RULES.md`; its workflow and validation restrictions
remain authoritative. For Codex skill discovery, add a small project-local
`.agents/skills/c-code-style/SKILL.md` adapter that links to
`../../../tools/c-code-style/.claude/skills/c-code-style/SKILL.md` and adapts
commands to the project's shell and explicitly selected files. Do not copy the
canonical skill into the project as a second source of rules.

Commit `.gitmodules`, the submodule pointer, the root formatter redirect, and
the project instructions together. Other clones restore the recorded revision
with `git submodule update --init -- tools/c-code-style`. Review and adopt newer
style commits explicitly; ordinary submodule initialization keeps the pinned
revision. Format only project-authored files; imported and adapted files retain
their original style as described in the coding rules.

## Option 3: Copy a snapshot into your project

Copy these into your repository:

- `CODING_RULES.md` — the rules (do **not** copy this repo's `README.md`;
  it's just a symlink to `CODING_RULES.md` so it renders on this repo's
  GitHub front page, and copying it would overwrite your own README)
- `.clang-format` — the formatting rules
- `AGENTS.md` — picked up automatically by Codex and other AGENTS.md-aware
  agents; merge it with your existing `AGENTS.md` if you already have one
- `scripts/check-format.sh` — the install-check + format/verify script both
  `AGENTS.md` and the Claude skill call out to
- `template.c` / `template.h` — reference examples for exact formatting and
  doxygen alignment

If you're using Claude Code specifically and don't want the marketplace
plugin, copy `.claude/skills/c-code-style/` into your project instead (or in
addition) for project-scoped auto-discovery.

## Option 4: CI only, no agent

Just want the formatting gate on pull requests, no AI agent involved? Copy
`.github/workflows/clang-format.yml`, `.clang-format`, and
`scripts/check-format.sh`, then follow the branch-protection steps in
`CODING_RULES.md`'s "Continuous integration (GitHub Actions)" section. Replace the
workflow's reference-template file list with an explicit list of project-authored
files; preserve vendor, generated-file, and submodule exclusions.
