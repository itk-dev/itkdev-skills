---
name: itkdev-review-php
user-invocable: true
description: PHP code review for ITK Dev projects (Laravel, Symfony, plain PHP). Use when reviewing, auditing, critiquing, or giving feedback on PHP code — including snippets, files, or pull requests. Covers security (SQL injection, XSS, mass assignment, file inclusion, command injection), correctness, performance (N+1 queries), type safety, and PSR-12 style. Triggers for "review my PHP", "what's wrong with this", "security review", or when PHP code is pasted for feedback.
---

# PHP Code Review

Produce structured, prioritized code reviews for PHP codebases (Laravel, Symfony, plain PHP).
Reviews should be developer-friendly: specific, actionable, and ranked so the author knows what
to fix first.

## Review Process

### Step 1 — Gather context

Before diving in, identify:

- **Framework**: Laravel, Symfony, or raw PHP.
- **Purpose**: Web endpoint? CLI tool? Library? Queue worker?
- **Scope**: Quick feedback, security focus, full audit, or pre-merge PR review?
- **Standards**: Apply PSR-12 and any project conventions the user mentions; otherwise use
  community defaults.

If the user hasn't shared the code yet, ask for it.

### Step 2 — Run the review checklist

Work through each category. Skip irrelevant ones and say so briefly.

#### 🔴 Critical (must fix before shipping)

**Security**

- [ ] SQL injection: raw queries with user input? Use prepared statements / the ORM.
- [ ] XSS: unescaped output to HTML? (`echo $_GET[...]`, unescaped Twig/Blade).
- [ ] Authentication/authorization gaps: missing auth checks, insecure direct object references.
- [ ] Sensitive data exposure: secrets/passwords in code, logs, or error messages.
- [ ] Path traversal: user-controlled file paths without sanitization.
- [ ] Command injection: `exec()`, `shell_exec()`, `system()`, `proc_open()` with user input.
- [ ] Mass assignment: unguarded `$request->all()` (Laravel) or unfiltered model hydration.
- [ ] Insecure deserialization: `unserialize()` on untrusted data.
- [ ] File inclusion: `include`/`require` with user-controlled variables.
- [ ] Dependency vulnerabilities: obviously outdated packages with known CVEs.

**Correctness**

- [ ] Logic errors that produce wrong results or silent failures.
- [ ] Off-by-one errors, wrong comparison operators (`==` vs `===`).
- [ ] Missing null checks before dereferencing.
- [ ] Race conditions (especially around file I/O or shared state).
- [ ] Uncaught exceptions that crash the request/process.

#### 🟠 High (should fix — impacts reliability or maintainability)

- **Error handling**: bare `die()`, empty `catch` blocks, errors swallowed silently; missing
  validation of external inputs (API responses, user data, file contents); no rollback on partial
  database transactions.
- **Performance**: N+1 query problems (loop calling the DB per iteration — use eager loading);
  missing indexes implied by query patterns; unbounded loops over large datasets without
  pagination/batching; repeated computation inside loops.
- **Resource management**: DB connections not closed, file handles left open, large objects held
  in long-lived references.
- **Type safety**: missing type hints on function signatures (PHP 7.4+ / 8.x), incorrect type usage.

#### 🟡 Medium (fix soon — reduces tech debt)

- **Code quality**: functions doing more than one thing; functions longer than ~40–50 lines
  without good reason; deep nesting (>3–4 levels — prefer early returns or extraction); magic
  numbers/strings (extract to named constants); duplicate code; dead code.
- **Naming**: misleading names; obscure abbreviations; not following camelCase (methods) /
  snake_case-or-camelCase (variables, per project convention).
- **Docblocks & comments**: missing docblocks on public functions/classes (especially complex
  ones); comments describing *what* rather than *why*; outdated comments contradicting the code.

#### 🟢 Low / Suggestions (nice to have)

- **Style**: PSR-12 violations (spacing, brace style, line length); inconsistent style within a file.
- **Design**: opportunities to apply a pattern more clearly; tight coupling that could be injected;
  testability improvements (hard-coded dependencies, global state).
- **Testing**: missing unit/integration tests for critical paths; tests without assertions; no test
  for edge cases identified in this review.

### Step 3 — PHP-specific deep checks

- **Superglobals**: `$_GET`, `$_POST`, `$_COOKIE`, `$_REQUEST` used directly without sanitization?
- **`eval()`**: almost always a red flag.
- **`include`/`require` with variables**: potential file inclusion vulnerability.
- **`isset()` vs `empty()` vs `??`**: correct null-handling idiom used?
- **Laravel specifics**:
  - Mass assignment protection (`$fillable` / `$guarded` on models).
  - Policies and Gates used for authorization?
  - Jobs/Queues idempotent? Handle retries?
  - Eloquent N+1: using `with()` for relationships?
  - Config via `config()` in app code (not `env()` outside config files).
- **Symfony specifics**:
  - Services autowired correctly?
  - Doctrine entities: fetch type appropriate (LAZY vs EAGER)?
  - Forms: CSRF protection enabled?
- **PSR compliance**: PSR-4 autoloading, PSR-12 code style.

### Step 4 — Format the output

```
## Code Review: [filename or description]

### Summary
One paragraph: what the code does, overall impression, most important theme.

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

## Tone Guidelines

- **Be direct, not harsh.** "This query is vulnerable to SQL injection — use a prepared statement
  instead", not "This is terrible code."
- **Always explain why**, not just what to change.
- **Distinguish opinion from fact.** "This *could* be extracted into a helper" vs "This query *is*
  vulnerable."
- **Prioritize ruthlessly.** Group issues that share a root cause.
- **Acknowledge good work.** "What's Working Well" is not optional filler.

## Quick Reference: Common PHP Vulnerabilities

| Issue | Bad | Good |
|---|---|---|
| SQL injection | `"SELECT * FROM users WHERE id = $_GET['id']"` | `$pdo->prepare("SELECT * FROM users WHERE id = ?")` |
| XSS | `echo $_GET['name']` | `echo htmlspecialchars($_GET['name'], ENT_QUOTES)` |
| Mass assignment | `User::create($request->all())` | `User::create($request->only(['name','email']))` |
| File inclusion | `include($_GET['page'] . '.php')` | Use a whitelist / route map |
| Command injection | `shell_exec("ping " . $_GET['host'])` | `escapeshellarg()` + fixed command, or a library |
