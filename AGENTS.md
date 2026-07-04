# AGENTS.md

Behavioral and coding guidelines for AI-assisted development.
Bias toward correctness and minimal diff over speed.

---

## Model Contract

Binding rules for which models run what. Where tooling allows a choice, these are
not suggestions.

| Role | Rule |
|---|---|
| **Orchestrator** | **Fable Medium** if available; otherwise **Opus 4.8 High**. The orchestrator plans, decomposes, delegates, and reviews — it does not grind through bulk implementation itself. |
| **Implementor** | Chosen **by the orchestrator, per task**. Optimize for *good over fast*. **Fable Low is the ceiling** for implementor models — never assign Fable Medium/High to implementation work. |
| **When** | The orchestrator/implementor split is **mandatory for any complex task** (multi-step, multi-file, or requiring independent verification). Simple, single-step edits may be done directly by the orchestrator. |

Implementor selection guidance: match the model to the task's difficulty — subtle
refactors, tricky debugging, or security-sensitive code get the strongest permitted
model (Fable Low); mechanical or well-specified changes may use a faster model
(e.g., Sonnet or Haiku). The orchestrator reviews all delegated output before
accepting it.

---

## Core Principles

### Think Before Coding

- **Ultrathink.** Reason step-by-step. Consider edge cases before writing a line.
- State assumptions explicitly. For decisions only the user can make (product behavior,
  destructive actions, scope changes), ask before implementing. For everything else,
  state your assumption and proceed — don't stall on questions you can answer by
  reading the code.
- Surface tradeoffs and multiple interpretations; never pick silently.
- If something is unclear, stop and name what's confusing.

### Understand Before Changing

- Read the code you're about to modify — and its callers — before editing.
  Never edit from memory of what a file "probably" contains.
- Search for existing helpers, patterns, and conventions before writing new ones.
  Duplicating an existing utility is a bug.
- Debug to root cause. Reproduce the failure first, then fix the cause, not the
  symptom. A fix you can't explain is not a fix.
- When evidence contradicts your hypothesis, revise the hypothesis — don't force
  the fix through with retries or workarounds.

### Simplicity First

- Write the minimum code that solves the problem. Nothing speculative.
- No unrequested features, abstractions, or future-proofing.
- No error handling for impossible scenarios.
- If you write 200 lines and 50 would do: rewrite it.

> Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### Surgical Changes

- Touch only what the task requires. Match existing style, even if you'd do it differently.
- Don't "improve" adjacent code, comments, or formatting.
- Remove only the imports/variables/functions **your** changes made unused.
- If you notice unrelated dead code, mention it — don't delete it.

### Goal-Driven Execution

For multi-step tasks, state a brief verifiable plan before implementing:

```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Clarifying questions come **before** implementation, not after mistakes.

### Verify Before Claiming

"Done" means **demonstrated working**, not "code written that should work."

- Run the relevant build, tests, linter, and type checker before declaring success.
  If you can't run them, say so explicitly — never imply verification that didn't happen.
- Report results faithfully: failing tests get quoted output, not a summary that
  hides them. A partial success is reported as partial.
- **Never game the check.** Do not weaken assertions, delete failing tests, add
  lint/type suppressions, mock away the behavior under test, or special-case the
  test's inputs to get green. If a check seems wrong, say why and stop.
- Distinguish failures you introduced from pre-existing ones: check whether the
  failure exists on a clean baseline before assuming your change caused it — and
  never "fix" a pre-existing failure by silencing it.
- For bug fixes: write a test that reproduces the bug first, then confirm it passes
  after the fix.

### Finish the Job

- Don't stop at the first plausible answer — check the edge cases against the code
  before concluding. The second look is where wrong assumptions die.
- If a step fails, diagnose and retry with a changed approach; don't return a
  half-result with "you could try...". Escalate to the user only when genuinely
  blocked on something only they can decide or provide.
- Leave no debris: no commented-out experiments, stray debug prints, or scratch
  files in the diff.

---

## General Standards

| Concern | Rule |
|---|---|
| **Security** | Apply coding and security best practices throughout. Least privilege always. |
| **Privilege escalation** | Only `sudo` when strictly required. Never by default. |
| **Dryrun** | Implement `--dryrun` on any script with side effects (writes, deletes, API calls). |
| **Verbose** | Implement `--verbose` / `-v` on any non-trivial script. Silent by default; verbose emits step-level progress and key variable state. |
| **Typing** | Use the strongest static typing the language supports. No untyped escape hatches. |
| **Testing** | Test extensively. Cover edge cases, error paths, and boundary conditions. |
| **Script headers** | Every script opens with a comment block: intended usage + all CLI switches documented. |
| **Commits** | Sign commits (`git commit -S -s`). One logical change per commit. Imperative present tense. |

---

## Tooling

Prefer these over their slower/legacy equivalents when assisting with agentic
coding in this repo:

| Tool | Use for |
|---|---|
| **`bat`** | Viewing files with syntax highlighting + line numbers (a `cat` replacement). |
| **`biome`** | Linting and formatting JS/TS/JSX/TSX. Fast; the canonical formatter/linter here. |
| **`bun`** | JS/TS package manager + runtime. Prioritize over `npm` for installs, scripts, and running TS. |
| **`rg`** | Ripgrep — fast recursive text/code search. Default over `grep`/`find`. |
| **`sg`** | ast-grep — structural (AST-aware) search and rewrite. Use for syntax-aware refactors that `rg` can't express safely. Use "outline" subcommand to get quick summary and steering |
| **`ty`** | Astral's fast Python type checker. |
| **`ruff`** | Python linting + formatting. |
| **`rtk`** | Rust Token Killer — token-optimized CLI proxy for dev operations (transparent via hook). |
| **`uv`** | Python package + standalone-script manager (PEP 723). The only Python package manager — never `pip`. |
| **`gh`** | GitHub CLI — PRs, issues, releases, API access. |

---

## Language Guidance

### Python (3.14+)

- **Package manager:** `uv` exclusively. No `pip`, no standalone `requirements.txt`.
- **PEP 723:** All standalone scripts must include inline metadata:
  ```python
  # /// script
  # requires-python = ">=3.14"
  # dependencies = ["httpx"]
  # ///
  ```
- Strict typing: annotate all functions and class members. No bare `Any`.
- Prefer `dataclasses(slots=True)` or `msgspec` over plain dicts for structured data.
- Use `asyncio.TaskGroup` for concurrent tasks. Free-threaded mode where applicable.
- Leverage `match`, `TypeAlias`, `ParamSpec`, `typing.Self`, and `type X = ...` syntax.
- `subprocess.run` with explicit `check=True`/`capture_output=True`. No `shell=True`.

### Go (1.26+)

- `any` over `interface{}`. Use `slices`, `maps`, `cmp` stdlib packages.
- Handle all errors explicitly. Use anonymous closures for `defer body.Close()` checks.
- No ignored return values — code must be `errcheck`-clean.
- Flat package structure; only add layers when complexity demands it.
- Range-over-func iterators where they simplify collection traversal.

### Rust (1.96+)

- `?` for error propagation. No `.unwrap()` in non-test code.
- `thiserror` for library errors; `anyhow` for binaries.
- `clippy` (all warnings as errors) and `rustfmt` before every commit.
- `#[must_use]` on result-returning functions where ignoring is a likely mistake.
- Prefer `impl Trait` in arguments; use const generics and `std::sync::LazyLock` for static.

### TypeScript

- Strict mode on. No `any` — type all props, state, and API boundaries.
- `bun` for package management and running scripts; `biome` for lint/format.
- React with `shadcn/ui` + Tailwind; prefer server state (React Query / SWR) over local state.
- Recharts for data viz; colocate chart config with the component.
- Externalize user-facing strings (i18n-ready); use locale-aware date/number formatting.

### Shell / Bash

- `set -euo pipefail` at the top of every script.
- Ensure scripts run on both Linux (priority) and macOS
- Quote all variable expansions. No word-splitting bugs.
- `[[ ]]` over `[ ]`. `$(...)` over backticks.
- Color-coded output for user-facing scripts (ANSI codes; check `$NO_COLOR`).
- `--dryrun` flag for any script that mutates state.
- `--verbose` flag to log more detail.

---

## Web App Standards

### Theme & Design

- **[`AESTHETIC_CONTRACT.md`](AESTHETIC_CONTRACT.md) is the binding design contract for all UI work in this repo.**
  Read it before writing or changing any UI/chart code. It fixes the palette, fonts, the HeroInsight
  results pattern, and the Tufte chart conventions (tick contrast, direct labeling,
  reference-line treatment, range-area bands), plus per-app token locations and a
  compliance checklist. Where this section and the contract disagree, the contract wins.
- **Light/dark mode toggle, defaulting to dark.**
- Clean, minimalistic, and intuitive. Follow [Apple HCI guidelines](https://developer.apple.com/design/human-interface-guidelines/).
- No decorative chrome. Every element earns its place.

### Stack

- **Components:** `shadcn/ui` preferred. Tailwind CSS for utility styling.
- **Language:** TypeScript strict mode. No `any`. Type all props, state, and API boundaries.
- **State:** Prefer server state (React Query / SWR) for remote data; minimize local state.
- **Storage:** Prefer `localStorage` for client-side persistence in SPAs (theme, preferences, cached data). Wrap access in a helper to guard against SSR and storage-quota errors.
- **Charts/data viz:** Recharts for React; keep chart configs colocated with the component.

### Quality

- Semantic HTML. ARIA labels where needed. Fully keyboard navigable.
- Mobile-first responsive. Verify at 390 px, 768 px, and 1280 px.
- **Cross-browser:** Layout and code must work on Chrome, Safari, and Firefox.
- No magic numbers. Extract constants; use CSS variables / Tailwind tokens for colors and spacing.
- Dark mode must be intentional: check contrast ratios, not just `dark:` class application.
- **i18n-ready:** All user-facing strings must be externalized (e.g. `react-i18next`). No hardcoded copy in JSX. Use locale-aware formatting for dates, numbers, and currencies from the start.
- **Testing:** Build unit tests for components and logic; actively look for opportunities to increase test coverage.

---

## Checklist Before Submitting

- [ ] Assumptions stated or questions asked before implementation
- [ ] Only changed lines required by the task
- [ ] Build/tests/lint/type check run, with results reported honestly (or "not run" stated)
- [ ] No suppressed warnings, weakened tests, or debug leftovers in the diff
- [ ] git commits are signed and use SSH keys for signing
