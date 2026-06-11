---
name: itkdev-review-php
user-invocable: true
description: PHP code review for ITK Dev projects (Symfony, Drupal, Laravel, plain PHP). Use when reviewing, auditing, critiquing, or giving feedback on PHP code — including snippets, files, or pull requests. Covers a dedicated security review (SQL injection, XSS, mass assignment, file inclusion, command injection, weak crypto, insecure deserialization, misconfiguration) plus correctness, performance (N+1 queries), type safety, and PSR-12 style. Triggers for "review my PHP", "security review", "what's wrong with this", or when PHP code is pasted for feedback.
---

# PHP Code Review

Produce structured, prioritized code reviews for PHP codebases (Symfony, Drupal, Laravel, plain
PHP). Reviews should be developer-friendly: specific, actionable, and ranked so the author knows
what to fix first.

## Review Process

### Step 1 — Gather context

Before diving in, identify:

- **Framework**: Symfony, Drupal, Laravel, or raw PHP.
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
- **Symfony specifics**:
  - Services autowired correctly?
  - Doctrine entities: fetch type appropriate (LAZY vs EAGER)?
  - Forms: CSRF protection enabled?
- **Drupal specifics**:
  - `\Drupal::` static calls inside services? Inject dependencies instead.
  - Access checks on routes and entity operations (`_permission` requirements, `$entity->access()`)?
  - Output escaped in render arrays? `t()` placeholders (`@`, `%`, `:`) used instead of string
    concatenation?
  - Database API with placeholders — no string-concatenated SQL?
  - APIs deprecated in Drupal 10/11?
  - Business logic in hooks (prefer services)?
  - For deeper Drupal checks, use the `itkdev-drupal` skill (from the
    `itkdev-scaffolding-and-templates` plugin) when it is installed.
- **Laravel specifics**:
  - Mass assignment protection (`$fillable` / `$guarded` on models).
  - Policies and Gates used for authorization?
  - Jobs/Queues idempotent? Handle retries?
  - Eloquent N+1: using `with()` for relationships?
  - Config via `config()` in app code (not `env()` outside config files).
- **PSR compliance**: PSR-4 autoloading, PSR-12 code style.

### Step 4 — Dedicated security review

When the user asks for a **security review** (or a full audit), act as a senior application
security engineer and cover at minimum these categories:

1. **Injection** — SQL, command, LDAP, template injection.
2. **Authentication & authorization** — broken auth, missing access controls, insecure sessions.
3. **Secrets & credentials** — hardcoded keys/passwords/tokens, insecure env-variable usage.
4. **Cryptography** — weak algorithms, hardcoded salts/IVs, improper key management.
5. **Input validation** — missing sanitization, type juggling, path traversal, file inclusion.
6. **Dependency risks** — known-vulnerable Composer packages (flag any worth checking).
7. **Insecure deserialization** — `unserialize()` on untrusted input, Phar deserialization.
8. **Error handling & logging** — sensitive data in logs, verbose errors exposed to users.
9. **Race conditions & TOCTOU** — filesystem or shared-state issues.
10. **Security misconfigurations** — `display_errors` in production, debug mode on, permissive
    CORS, weak TLS settings.

For each security finding, provide:

- **Severity**: Critical / High / Medium / Low / Informational
- **Location**: file and line number(s)
- **Vulnerability type**: e.g. SQLi, XSS, SSRF, hardcoded secret
- **Description**: what the issue is and why it's dangerous
- **Proof of concept**: a brief example of how it could be exploited
- **Remediation**: a specific code fix or mitigation

After the findings, provide a **prioritized remediation plan** — what to fix first and why.

### Step 5 — Format the output

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

For a security-focused review, use the per-finding shape from Step 4 and end with the prioritized
remediation plan.

## Tone Guidelines

- **Be direct, not harsh.** Explain the fix, don't just condemn the code.
- **Always explain why**, not just what to change.
- **Distinguish opinion from fact.** "This *could* be extracted" vs "This *is* vulnerable."
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
