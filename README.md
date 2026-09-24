# Guards That Scan

Claude Code skill: `guards-that-scan`

## What

Rules for checkers that inspect other code (parity tests, lint rules, conformance scans, audit scripts). A checker with a blind spot is **worse than no checker** — silence is read as a pass. Prove the scan can see; keep negative controls; never confuse AST names with behaviour.

## When to use

Reviewing or writing any checker; whenever a defect slipped past a check written for exactly that defect.

## Install

Copy this folder into your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/guards-that-scan
cp SKILL.md ~/.claude/skills/guards-that-scan/
```

Claude Code loads `SKILL.md` from `~/.claude/skills/<name>/`.

## Sibling skills

`code-that-holds`, `reachability-audit`, `fail-closed-review`, `adversarial-qa`, `verified-delivery`, `contract-and-compat`, `handoff-faber-rigor`, `karpathy-method`

Proposed repos: see [Vorxeo](https://github.com/Vorxeo) `skill-*` packs.

## License

MIT — Copyright (c) 2026 Vorxeo. See [LICENSE](./LICENSE).
