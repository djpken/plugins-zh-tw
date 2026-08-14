# Skills

[繁體中文](./README.md) · English

A collection of agent skills (slash commands and behaviors) for Claude Code and other Agent-Skills-standard harnesses.

## Structure

Skills live under `skills/<bucket>/<skill-name>/SKILL.md`. Buckets:

- `engineering/` — daily code work
- `productivity/` — daily non-code workflow tools
- `misc/` — kept around but rarely used, not promoted
- `personal/` — tied to one's own setup, not promoted
- `in-progress/` — drafts not yet ready to ship
- `deprecated/` — no longer used

`engineering/` and `productivity/` are the **promoted** buckets — every skill in them is registered in this README and in `.claude-plugin/plugin.json`. See `CLAUDE.md` for the full set of conventions.

## Local dev

```bash
scripts/link-skills.sh
```

Symlinks every skill in `skills/` into `~/.claude/skills` and `~/.agents/skills` for local testing. Re-run after adding, removing, or renaming a skill.

## Reference

Skills split on one axis — who can invoke them. **User-invoked** skills are reachable only when you type them; their job is to orchestrate. **Model-invoked** skills can be invoked by you _or_ reached for automatically by the agent when the task fits. A user-invoked skill may invoke model-invoked skills, but never another user-invoked one.

## User-invoked skills

- [`/git-fork-remotes`](./skills/engineering/git-fork-remotes/SKILL.md) — Configure fork/upstream remotes and branch tracking.
- [`/translating-skill-docs`](./skills/productivity/translating-skill-docs/SKILL.md) — Add translated `SKILL.<locale>.md` siblings without changing runtime behavior.
- [`/writing-router-skill`](./skills/engineering/writing-router-skill/SKILL.md) — Ask which skill in this repo fits your situation. A router over the whole skill set.

## Model-invoked skills

- [`/hv-analysis`](./skills/productivity/hv-analysis/SKILL.md) — Run the Horizontal-Vertical Analysis method — a deep-research framework tracing a subject's full history against a same-period competitive comparison — and produce a polished PDF report.
- [`/storage-analyzer`](./skills/productivity/storage-analyzer/SKILL.md) — Read-only macOS/Windows storage analyzer that scans disk usage, tiers cleanup candidates by risk, and generates an interactive HTML report with one-click cleanup.
- [`/neat-freak`](./skills/engineering/neat-freak/SKILL.md) — Knowledge and governance closeout: reconcile project docs, rule files, authorized agent memory, and workspace residue with what the code and runtime actually do.
