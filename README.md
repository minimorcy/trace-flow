# trace-flow

A [Claude Code](https://claude.com/claude-code) skill that traces one feature flow through your codebase — from its entry point to its final effects — and returns a **curated, verified** diagram.

> "Trace the login flow" → Mermaid sequence diagram + step table with `file:line` for every step.

## Why

Full call graphs are unreadable. `trace-flow` keeps only the steps that matter:

- boundary crossings (HTTP, DB, cache, queues, external APIs)
- state changes (sessions, tokens, writes)
- decisions (error branches that change the outcome)
- security-relevant steps (auth, hashing, token validation)

It also resolves the hidden parts that static tools miss, such as middlewares, dependency injection, events, and interface → implementation. Every step is anchored to a real `file:line`, and anything it can't verify is listed explicitly instead of guessed.

## Install

```
/plugin marketplace add <your-github-user>/trace-flow
/plugin install trace-flow@trace-flow
```

Or copy `plugins/trace-flow/skills/trace-flow/` into `~/.claude/skills/` for a personal install.

## Usage

Just ask in natural language:

```
trace the login flow
what happens when a user uploads a file?
diagrama del flujo de reset de contraseña
```

Or invoke it directly:

```
/trace-flow:trace-flow checkout happy-path
```

Options: `happy-path`, `detailed`, `flowchart` / `sequence`, `from <X>` / `to <Y>`.

## Output

1. Summary
2. Mermaid diagram (sequence or flowchart, with error branches)
3. Step table (`# | Step | Location | Notes`)
4. Gaps & assumptions
5. Drill-down offers

The skill is read-only: it never modifies your code or copies secret values into its output.

## License

MIT
