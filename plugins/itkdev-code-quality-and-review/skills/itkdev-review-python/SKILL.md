---
name: itkdev-review-python
user-invocable: true
description: Python code review for ITK Dev projects (Django, Flask, FastAPI, scripts, data pipelines). Use when reviewing, auditing, critiquing, or giving feedback on Python code — including snippets, files, or pull requests. Covers a dedicated security review (injection, broken auth, secrets, weak crypto, insecure deserialization, SSRF, path traversal, misconfiguration) plus correctness, performance, type safety, and PEP 8 style. Triggers for "review my Python", "security review", "what's wrong with this", or when Python code is pasted for feedback.
---

# Python Code Review

Produce structured, prioritized code reviews for Python codebases (Django, Flask, FastAPI,
scripts, data pipelines). Reviews should be developer-friendly: specific, actionable, and ranked
so the author knows what to fix first.

## Review Process

### Step 1 — Gather context

Before diving in, identify:

- **Framework**: Django, Flask, FastAPI, plain script, or data pipeline.
- **Purpose**: Web endpoint? CLI tool? Library? Batch job?
- **Scope**: Quick feedback, security focus, full audit, or pre-merge PR review?
- **Standards**: Apply [PEP 8](https://peps.python.org/pep-0008/) (style),
  [PEP 257](https://peps.python.org/pep-0257/) (docstrings), and
  [PEP 484](https://peps.python.org/pep-0484/) (type hints), plus any project conventions the user
  mentions; otherwise use community defaults.
- **Tooling**: does the repo configure a linter/formatter? Look for `ruff`, `flake8`, `black`,
  `isort`, `mypy` in `pyproject.toml`, `ruff.toml`, `setup.cfg`, `.flake8`, or `.pre-commit-config.yaml`.
  This decides how much mechanical style you check by hand (see Step 2).

If the user hasn't shared the code yet, ask for it.

### Step 2 — Run the tooling first (deterministic style)

Mechanical PEP 8 — line length, import order, whitespace, blank lines, quote style, trailing
commas — is **owned by tooling, not by you**. Linters catch it perfectly and instantly; hand-eyeballing
it is slower, inconsistent, and erodes trust when you miss one or invent one. So:

- **If the repo configures a linter/formatter** (found in Step 1), run it — or, when you can't execute
  commands, tell the author the exact command — and treat its output as ground truth for mechanical
  style:
  - `ruff check .` — lint (import order, unused names, many PEP 8 rules)
  - `ruff format --check .` (or `black --check .`) — formatting
  - `isort --check-only .` — import ordering (if not already handled by ruff)
  - `mypy .` (or `pyright`) — type checking
  Fold the findings into the report; **do not** re-report by hand what the tool already flags.
- **Respect the formatter.** When the project uses `black` / `ruff format`, do **not** raise findings
  the formatter owns — line length (88 by default), quote style, blank-line counts, trailing commas.
  The formatter is the authority; fighting it is noise. Flagging line length in a black-formatted repo
  is a bug in the review, not a finding.
- **If no linter is configured**, apply PEP 8 by hand using `references/pep8-checklist.md`, and
  **recommend adopting `ruff`** as a High-priority finding — a one-time `pyproject.toml` config beats
  every future manual style review.

Either way, spend your own judgment on the style questions tools *can't* decide — see the "Style &
Pythonic idioms" checks below and the reference file.

### Step 3 — Run the review checklist

Work through each category. Skip irrelevant ones and say so briefly.

#### 🔴 Critical (must fix before shipping)

**Security**

- [ ] SQL injection: f-strings/`%`-formatting in queries with user input? Use parameterized
      queries or the ORM.
- [ ] Command injection: `subprocess` with `shell=True`, `os.system()` on user input.
- [ ] Insecure deserialization: `pickle.loads()`, `yaml.load()` (use `safe_load`), `eval`/`exec`
      on untrusted input.
- [ ] Authentication/authorization gaps: missing access controls, insecure session handling,
      insecure direct object references.
- [ ] Sensitive data exposure: hardcoded API keys, passwords, tokens; secrets in logs or errors.
- [ ] Path traversal: user-controlled file paths without sanitization.
- [ ] SSRF: user-controlled URLs passed to `requests`/`urllib` without allowlisting.

**Correctness**

- [ ] Logic errors that produce wrong results or silent failures.
- [ ] Off-by-one errors, wrong comparison operators.
- [ ] Missing `None` checks before dereferencing.
- [ ] Race conditions (file I/O, shared state, threading).
- [ ] Uncaught exceptions that crash the process.

#### 🟠 High (should fix — impacts reliability or maintainability)

- **Error handling**: bare `except:` or `except Exception: pass`, missing error logging, missing
  validation of external inputs; no rollback on partial DB transactions; use exception chaining
  (`raise NewError() from err`).
- **Performance**: N+1 queries; missing indexes implied by query patterns; unbounded loops over
  large datasets without pagination/batching; repeated computation inside loops.
- **Resource management**: files opened without `with`, connections/sessions not closed, large
  objects held in long-lived references.
- **Type safety**: missing type annotations where complexity warrants; incorrect type usage.

#### 🟡 Medium (fix soon — reduces tech debt)

- **Code quality**: functions doing more than one thing; functions longer than ~40–50 lines without
  good reason; deep nesting (>3–4 levels); magic numbers/strings; duplicate code; dead code.
- **Naming** (judgment — linters can't check *semantics*): misleading or inaccurate names; obscure
  abbreviations; single-letter names outside short loops; wrong PEP 8 *case convention* for the kind
  of name (`snake_case` functions/vars, `PascalCase` classes, `UPPER_SNAKE` constants, `_leading`
  for non-public, avoiding `l`/`O`/`I` as single chars).
- **Docstrings & comments**: missing docstrings on public functions/classes (especially complex
  ones); comments describing *what* rather than *why*; outdated comments contradicting the code.

#### 🟢 Low / Suggestions (nice to have)

- **Style & Pythonic idioms** (only what tools *don't* own — mechanical PEP 8 is Step 2): a long line
  that should wrap at a *semantic* boundary rather than where the formatter broke it; `l`/`O`/`I`
  ambiguous names; `== None` instead of `is None`; `type(x) == T` instead of `isinstance`; unpythonic
  loops (`range(len(...))`, manual index tracking) where a comprehension/`enumerate`/`zip` reads
  better; over-dense comprehensions that should be a loop; string concatenation in a loop. See
  `references/pep8-checklist.md`. If the repo has no linter, this is where "adopt `ruff`" belongs.
- **Design**: opportunities to apply a pattern more clearly; tight coupling that could be injected;
  testability improvements (hard-coded dependencies, global state).
- **Testing**: missing unit/integration tests for critical paths; tests without assertions; no test
  for edge cases identified in this review.

### Step 4 — Python-specific deep checks

- **`eval()` / `exec()`**: almost always a red flag.
- **Mutable default arguments**: `def f(x=[]):` is a classic gotcha.
- **`global` / `nonlocal` overuse**: sign of poor state design.
- **`__all__`**: defined if the module is a public API?
- **Context managers**: `with` used for files, locks, DB sessions, HTTP clients?
- **Django specifics**: raw SQL via `raw()` / `cursor.execute()` vs ORM; `select_related()` /
  `prefetch_related()` for N+1; `@login_required` / permissions on views; no business logic in
  signals.
- **Flask/FastAPI specifics**: input validation (Pydantic, WTForms, marshmallow); thin route
  handlers (business logic in services); proper HTTP status codes.
- **Data pipelines / scripts**: large files streamed/chunked rather than loaded fully into memory;
  pandas `apply()` over vectorized ops is slow — flag it; logging configured (not `print()`) for
  long-running scripts.

### Step 5 — Dedicated security review

When the user asks for a **security review** (or a full audit), act as a senior application
security engineer and cover at minimum these categories:

1. **Injection** — SQL, command, LDAP, template injection.
2. **Authentication & authorization** — broken auth, missing access controls, insecure sessions.
3. **Secrets & credentials** — hardcoded keys/passwords/tokens, insecure env-variable usage.
4. **Cryptography** — weak algorithms, hardcoded salts/IVs, improper key management.
5. **Input validation** — missing sanitization, type confusion, path traversal.
6. **Dependency risks** — known-vulnerable libraries (flag any imports worth checking).
7. **Insecure deserialization** — `pickle`, `yaml.load`, `eval`/`exec` on untrusted input.
8. **Error handling & logging** — sensitive data in logs, verbose errors exposed to users.
9. **Race conditions & TOCTOU** — filesystem or shared-state issues.
10. **Security misconfigurations** — debug mode on, permissive CORS, weak TLS settings.

For each security finding, provide:

- **Severity**: Critical / High / Medium / Low / Informational
- **Location**: file and line number(s)
- **Vulnerability type**: e.g. SQLi, XSS, SSRF, hardcoded secret
- **Description**: what the issue is and why it's dangerous
- **Proof of concept**: a brief example of how it could be exploited
- **Remediation**: a specific code fix or mitigation

After the findings, provide a **prioritized remediation plan** — what to fix first and why.

### Step 6 — Format the output

```
## Code Review: [filename or description]

### Summary
One paragraph: what the code does, overall impression, most important theme.

### 🧰 Tooling
Which linter/formatter/type-checker was run (or the command the author should run), and a one-line
digest of its output. "No linter configured — recommend adopting ruff (see High)." if none.

### 🔴 Critical Issues
[numbered list, each with: location, problem, why it matters, concrete fix]

### 🟠 High Priority
[same format]

### 🟡 Medium Priority
[same format]

### 🟢 Suggestions
[brief bullets — optional, keep it short]

### ✅ What's Working Well
[2–5 genuine positives]
```

For a security-focused review, use the per-finding shape from Step 5 and end with the prioritized
remediation plan.

## Tone Guidelines

- **Be direct, not harsh.** Explain the fix, don't just condemn the code.
- **Always explain why**, not just what to change.
- **Distinguish opinion from fact.** "This *could* be extracted" vs "This *is* vulnerable."
- **Prioritize ruthlessly.** Group issues that share a root cause.
- **Acknowledge good work.** "What's Working Well" is not optional filler.

## Quick Reference: Common Python Vulnerabilities

| Issue | Bad | Good |
|---|---|---|
| SQL injection | `cursor.execute(f"SELECT * FROM users WHERE id={id}")` | `cursor.execute("SELECT * FROM users WHERE id=%s", (id,))` |
| Mutable default | `def f(items=[]):` | `def f(items=None): items = items or []` |
| Bare except | `except:` | `except SpecificError as e:` |
| Subprocess injection | `subprocess.run(f"ls {user_input}", shell=True)` | `subprocess.run(["ls", user_input])` |
| Insecure deserialization | `pickle.loads(user_data)` | Use JSON, or validate the source |
| Unsafe YAML | `yaml.load(data)` | `yaml.safe_load(data)` |
