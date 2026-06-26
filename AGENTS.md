# AGENTS.md

Behavioral and coding guidelines for AI-assisted development.
Bias toward correctness and minimal diff over speed.

---

## Core Principles

### Think Before Coding

- **Ultrathink.** Reason step-by-step. Consider edge cases before writing a line.
- State assumptions explicitly. If uncertain, ask — don't guess and proceed.
- Surface tradeoffs and multiple interpretations; never pick silently.
- If something is unclear, stop and name what's confusing.

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
| **`sg`** | ast-grep — structural (AST-aware) search and rewrite. Use for syntax-aware refactors that `rg` can't express safely. |
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
