---
name: trace-flow
description: Traces one feature or execution flow (e.g. "login", "checkout", "file upload", "password reset") through a codebase from its entry point to its final effects, and produces a curated, verified Mermaid diagram plus a step table with file:line references. Use when the user asks for a diagram of a flow, wants to understand how a feature works end to end, or asks "what happens when...", "trace the X flow", "how does X work", "diagrama del login", "flujo de X", "cómo funciona X".
---

# Trace Flow

Produce a **curated** map of one flow — the steps that matter, not every function call. Clarity beats exhaustiveness: a 200-node call graph is as confusing as the code itself.

If arguments were passed (`$ARGUMENTS`), treat them as the flow to trace and any options (see *Options*).

## 1. Scope

- Identify the flow and its trigger. Ask the user only if it is genuinely ambiguous (e.g. the repo has both a web login and an API token login); otherwise pick the main one and say which.
- Find the **entry point**: HTTP route/controller, CLI command, UI event handler, message/queue consumer, scheduled job, or public API function.
- Search strategies: route definitions, handler names, UI labels and strings ("Sign in"), test names (`test_login*`), and framework conventions (Spring `@PostMapping`, Express `router.post`, Django `urls.py`, Rails `routes.rb`, FastAPI decorators, etc.).

## 2. Trace

Follow calls from the entry point, **reading the actual code** at each hop. Resolve indirection explicitly — this is where flows hide:

- Interfaces / abstract classes → the concrete implementation actually wired in
- Dependency injection containers and providers
- Middlewares, filters, interceptors, guards, decorators, AOP
- Events, signals, observers, pub/sub listeners
- ORM hooks, DB triggers, background jobs / queues enqueued by the flow
- Config-driven routing or feature flags
- **Reactive frameworks** (React, Vue, Svelte, Angular signals): state updates do not run in-line. Follow the chain *state change → re-render → effects/watchers/computed that depend on it → further state changes*, and check what each intermediate render shows (e.g. derived data that is briefly empty or stale before an effect corrects it). Also note dev-only behavior such as React `StrictMode` double-running effects.
- **Cross-repo / client → server**: when the flow crosses into another repository or service you have access to, follow the call into it (match the client URL/method to the server route) instead of stopping at the HTTP boundary.

If a hop cannot be resolved statically (reflection, runtime-built names, plugin loading), mark it as **unresolved** and state why. Never guess.

## 3. Filter

Keep a step only if it:

- **crosses a boundary**: HTTP, DB, cache, queue, filesystem, external service, another module/layer
- **changes state**: writes, sessions, tokens, cookies, counters
- **decides**: a branch that changes the outcome (invalid credentials, locked account, MFA required)
- **is security-relevant**: authentication, authorization, hashing, token signing/validation, rate limiting. For these, read the comparison logic itself (prefix/`startsWith` matching, unanchored regexes, fallbacks that default to allow) and confirm the check is actually wired in, not just defined.

Drop logging, trivial getters, formatting, DTO mapping, and generic helpers unless they matter for the flow.

Target **8–15 steps** at the top level. If there are more, group them into named sub-flows and offer to drill into each one.

## 4. Verify

Before writing any output, check every step and every arrow:

- Each step has a `file:line` where the code actually runs or the call is made. Re-open the file to confirm the reference; ranges must start and end on the relevant code, not approximately around it.
- A step you cannot anchor to a location is either removed or listed under *Gaps* as unverified.
- The order matches real execution, including middleware that runs *before* the handler.

## 5. Output

Use exactly this structure:

1. **Summary**: one or two sentences describing the flow and its entry point.
2. **Diagram** in a fenced ```mermaid block:
   - `sequenceDiagram` when several components/services interact (the usual case)
   - `flowchart TD` when branching logic dominates
   - Participants are **components** (controller, service, repository, DB, external API), not individual functions
   - Show error paths with `alt` / `else` blocks (or decision nodes in flowcharts)
   - Keep labels short; quote any label containing `()`, `:`, `/` or `#`
3. **Steps** table:

   | # | Step | Location | Notes |
   |---|------|----------|-------|
   | 1 | Receive POST /login | `src/routes/auth.ts:42` | Rate-limited by middleware |

   Step numbers should match the order in the diagram.
4. **Gaps & assumptions**: unresolved hops, dynamic dispatch, and anything inferred rather than read.
5. **Drill-down**: offer to expand specific steps or sub-flows.

## Options

The user may ask for any of these in natural language or via arguments:

- `happy-path`: omit error branches
- `detailed`: allow up to ~30 steps and function-level participants
- `flowchart` / `sequence`: force the diagram type
- `from <X>` / `to <Y>`: trace only a segment of the flow

## Rules

- Never copy secrets, credentials, or real config values found in code or env files into the output. Refer to them by name only (e.g. `JWT_SECRET`).
- Do not modify any files; this skill is read-only.
- Prefer the code over comments and docs when they disagree, and note the disagreement.
