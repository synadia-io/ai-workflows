# `/security-audit` — Claude Code Security Audit Command

**For single use, you can simply paste the `security-audit` contents as claude prompt**

A comprehensive, automated security audit slash command for [Claude Code](https://claude.com/claude-code). Runs a 5-phase penetration-test-grade audit on any codebase and produces a detailed `SECURITY_AUDIT_REPORT.md`.

## What it does

| Phase | Description |
|-------|-------------|
| 1. Planning | Discovers project structure, identifies trust boundaries, builds a threat model |
| 2. Investigation | Launches up to 8 parallel subagents covering: protocol parsing, TLS/crypto, auth, DoS, input validation, dependencies, git history, concurrency |
| 3. Triage | Deduplicates, classifies, and ranks all findings by severity × confidence |
| 4. Verification | Confirms each Medium+ finding, writes PoC tests, proposes fixes |
| 5. Report | Writes a full `SECURITY_AUDIT_REPORT.md` with executive summary, detailed findings, and prioritized recommendations |

## Installation

### Global (available in every project)

```bash
mkdir -p ~/.claude/commands
cp security-audit.md ~/.claude/commands/security-audit.md
```

### Project-specific (available only in one repo)

```bash
mkdir -p .claude/commands
cp security-audit.md .claude/commands/security-audit.md
```

> Project-level commands live in `.claude/commands/` at the repo root and are available to anyone who clones the repo.

## Usage

Open Claude Code in any project directory and run:

```
/security-audit
```

That's it. The command handles everything automatically.

## Requirements

- **Claude Code** — the CLI tool from Anthropic
- **Opus model recommended** — the audit is context-heavy with many parallel subagents; Opus produces the most thorough results
- **Permissions** — the audit reads files, runs bash commands (`git log`, `cargo audit`, `npm audit`, etc.), and spawns subagents. For unattended runs, use a permissive mode or pre-approve tool calls

## Output

The audit writes `SECURITY_AUDIT_REPORT.md` in the current working directory with:

- Executive summary
- Threat model
- Findings table (ID, severity, confidence, category, location, verification status)
- Detailed write-up for each finding: description, root cause, exploitation scenario, impact, PoC test, proposed fix
- Prioritized recommendations
- Scope limitations

## Language support

The command adapts to any language/framework. It has specific guidance for:

- **Rust** — `unwrap()`/`expect()` in parsing paths, `unsafe` blocks, `as usize` casts
- **Go** — unchecked slice indexing, `unsafe.Pointer`, race conditions
- **C/C++** — buffer overflows, format strings, use-after-free
- **Python/JS/Java** — regex DoS, unsafe deserialization, type confusion
- **Web apps** — XSS, CSRF, SSTI, SQL injection

It also skips irrelevant domains automatically (e.g., no TLS/crypto audit for a pure CLI tool).

## Tips

- **Large codebases**: The audit handles them well thanks to parallel subagents, but expect it to take several minutes
- **NATS ecosystem**: If NATS MCP tools are available, the audit will cross-reference findings against other NATS client implementations
- **False positives**: The command is designed to minimize noise — findings are ranked by severity × confidence, and theoretical/informational items are filtered out
- **Re-running**: Safe to run multiple times; it overwrites the previous `SECURITY_AUDIT_REPORT.md`
