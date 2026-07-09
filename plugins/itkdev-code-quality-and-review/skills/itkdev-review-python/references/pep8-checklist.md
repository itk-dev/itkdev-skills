# PEP 8 & Pythonic-style reference

Load this when doing style-level review. It splits the work the way the skill does: **tools own the
mechanical rules; you own the judgment calls.** Don't hand-check the "tool-owned" column when a linter
is configured — you'll be slower and less consistent than the tool.

Canonical sources: [PEP 8](https://peps.python.org/pep-0008/) (style),
[PEP 257](https://peps.python.org/pep-0257/) (docstrings),
[PEP 484](https://peps.python.org/pep-0484/) / [PEP 585](https://peps.python.org/pep-0585/) (type
hints), [PEP 20](https://peps.python.org/pep-0020/) (the Zen).

## Tool-owned — do NOT hand-review when a linter/formatter is configured

These are 100% deterministic. Run the tool; report its output. Flagging one of these by hand in a
linted repo is noise.

| Rule | Enforced by |
|---|---|
| Line length (79 PEP 8 default; 88 under black/ruff) | `ruff`, `flake8`, `black` |
| Indentation: 4 spaces, no tabs | `ruff`, `black` |
| Blank lines: 2 between top-level defs, 1 between methods | `ruff (E3xx)`, `black` |
| Whitespace around operators / after commas / no trailing | `ruff (E2xx)`, `black` |
| Import ordering & grouping (stdlib / third-party / local) | `ruff (I)`, `isort` |
| Unused imports / variables | `ruff (F401/F841)`, `flake8` |
| Quote style, trailing commas, magic-trailing-comma | `black`, `ruff format` |
| `== None` / `== True` comparisons (E711/E712) | `ruff`, `flake8` |
| Bare `except:` (E722) | `ruff`, `flake8` |
| f-string without placeholders (F541), mutable default not flagged by style (see below) | `ruff` |

**Respect the formatter:** if `black` / `ruff format` runs, its output *is* the style. Never argue
with where it wrapped a line or which quotes it chose.

## You-owned — judgment a linter cannot make

### Naming (semantics + convention)
- Case convention per kind: `snake_case` functions/vars, `PascalCase` classes, `UPPER_SNAKE`
  module constants, `_leading_underscore` for non-public, `__dunder__` only for protocol methods.
- Names that are *accurate*: `data`/`tmp`/`result`/`do_it` say nothing; `user_ids` beats `lst`.
- Avoid `l`, `O`, `I` as single-char names (indistinguishable from digits).
- Booleans read as predicates: `is_active`, `has_perm`, not `active`/`flag`.

### Line breaks & layout (where the *semantic* break belongs)
- Prefer implicit continuation inside `()`/`[]`/`{}` over backslashes.
- Break long boolean/arithmetic expressions **before** the operator, at a meaningful boundary.
- One statement per line; avoid `;`.

### Idioms (PEP 20 — readability counts)
| Prefer | Over |
|---|---|
| `for i, x in enumerate(xs)` | `for i in range(len(xs))` |
| `for a, b in zip(xs, ys)` | index-parallel loops |
| comprehension / generator | `append` in a trivial loop |
| `if x is None` | `if x == None` |
| `isinstance(x, T)` | `type(x) == T` |
| `with open(p) as f:` | manual `open`/`close` |
| `dict.get(k, default)` | `try/except KeyError` for simple lookup |
| `"".join(parts)` | `s += ...` in a loop |
| `pathlib.Path` | `os.path` string surgery (new code) |
| an `Enum` / constant | magic numbers & string literals |
- Comprehensions that span many lines or nest 2+ `for`/`if` are *less* readable — recommend a loop.
- Don't over-apply: a plain loop is fine; the point is readability, not one-liner golf.

### Docstrings (PEP 257) — content, not just presence
- Public modules, classes, and functions have a docstring; complex private ones too.
- First line is a concise imperative summary ("Return the parsed config."), ends with a period.
- Multi-line: summary, blank line, then details; closing `"""` on its own line.
- Document *why*/contract (args, returns, raises), not a restatement of the signature.
- A docstring that contradicts the code is worse than none — flag stale ones.

### Type hints (PEP 484/585)
- Public function signatures annotated where it aids clarity; don't demand hints on trivial locals.
- Modern syntax on 3.10+: `list[str]`, `dict[str, int]`, `X | None` over `List`, `Dict`,
  `Optional[X]`.
- `Any` is an escape hatch — flag it where a real type is knowable.
- Hints must match reality — a wrong annotation is a correctness issue, not style.

### Comments
- Explain *why*, not *what*; delete commented-out code and TODOs without an owner/issue.
- Keep comments in sync with code.

## Severity guidance for style findings
- **🟡 Medium**: misleading names, missing/inaccurate docstrings on public API, unpythonic patterns
  that hurt readability, wrong type annotations.
- **🟢 Low**: idiom nudges, mechanical nits *only* when no linter exists (otherwise route to Step 2).
- **🟠 High** (one case): repo has no linter configured — recommend adopting `ruff` so style stops
  being a manual, drifting concern.
