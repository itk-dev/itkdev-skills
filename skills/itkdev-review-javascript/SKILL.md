---
name: itkdev-review-javascript
user-invocable: true
description: JavaScript/TypeScript code review for ITK Dev projects (browser, Node.js, frontend frameworks). Use when reviewing, auditing, critiquing, or giving feedback on JS/TS code — including snippets, files, or pull requests. Covers security (XSS, prototype pollution, eval, injection), correctness, async/promise error handling, type safety (any usage), and style (var vs const/let). Triggers for "review my JS/TS", "what's wrong with this", "security review", or when JavaScript/TypeScript code is pasted for feedback.
---

# JavaScript / TypeScript Code Review

Produce structured, prioritized code reviews for JavaScript and TypeScript codebases (browser
code, Node.js services, frontend frameworks). Reviews should be developer-friendly: specific,
actionable, and ranked so the author knows what to fix first.

## Review Process

### Step 1 — Gather context

Before diving in, identify:

- **Runtime/framework**: browser, Node.js, React/Vue/Svelte, or a build tool.
- **Language**: plain JavaScript or TypeScript.
- **Purpose**: UI component? API handler? CLI? Library?
- **Scope**: Quick feedback, security focus, full audit, or pre-merge PR review?
- **Standards**: Apply ESLint/Prettier config and any project conventions the user mentions;
  otherwise use community defaults.

If the user hasn't shared the code yet, ask for it.

### Step 2 — Run the review checklist

Work through each category. Skip irrelevant ones and say so briefly.

#### 🔴 Critical (must fix before shipping)

**Security**

- [ ] XSS: unescaped user input written to the DOM (`innerHTML`, `dangerouslySetInnerHTML`,
      `document.write`)? Sanitize or use text APIs.
- [ ] `eval()` / `new Function()` / `setTimeout("string")`: code execution on dynamic input.
- [ ] Prototype pollution: merging untrusted objects into `__proto__` / `constructor`.
- [ ] Injection: building shell commands, SQL, or template strings from user input.
- [ ] Sensitive data exposure: secrets/tokens hardcoded or committed to client bundles.
- [ ] Insecure `postMessage` / CORS / `window.open` targets without origin checks.
- [ ] SSRF in Node.js: user-controlled URLs passed to `fetch`/`http` without allowlisting.

**Correctness**

- [ ] Logic errors that produce wrong results or silent failures.
- [ ] `==` vs `===` (loose equality bugs); truthiness pitfalls (`0`, `""`, `NaN`).
- [ ] Missing null/undefined checks before dereferencing.
- [ ] Unhandled promise rejections; missing `await`; floating promises.
- [ ] Race conditions in concurrent async flows.

#### 🟠 High (should fix — impacts reliability or maintainability)

- **Error handling**: missing `try/catch` (or `.catch()`) on async/await and promises; errors
  swallowed silently; missing validation of external inputs (API responses, user data).
- **Performance**: unnecessary re-renders (React); work inside render/hot loops; unbounded data
  loaded without pagination; missing memoization where it clearly matters; large synchronous work
  blocking the event loop (Node.js).
- **Resource management**: event listeners/intervals/subscriptions not cleaned up; open handles or
  streams not closed (Node.js).
- **Type safety (TypeScript)**: `any` usage where a real type is feasible; unsafe casts (`as`);
  non-null assertions (`!`) hiding real nullability.

#### 🟡 Medium (fix soon — reduces tech debt)

- **Code quality**: functions doing more than one thing; very long functions; deep nesting
  (prefer early returns); magic numbers/strings; duplicate code; dead code.
- **Naming**: misleading names; obscure abbreviations; inconsistent casing (camelCase for
  functions/vars, PascalCase for classes/components).
- **Comments**: comments describing *what* rather than *why*; outdated comments contradicting the
  code.

#### 🟢 Low / Suggestions (nice to have)

- **Style**: `var` instead of `const`/`let`; inconsistent formatting; missing semicolons where the
  project requires them; import ordering.
- **Design**: opportunities to apply a pattern more clearly; tight coupling that could be injected;
  testability improvements (hard-coded dependencies, global state).
- **Testing**: missing unit/integration tests for critical paths; tests without assertions; no test
  for edge cases identified in this review.

### Step 3 — JS/TS-specific deep checks

- **`var` usage**: should be `const` or `let`.
- **`any` type (TypeScript)**: flag where a concrete type is feasible; prefer `unknown` for truly
  dynamic values.
- **Async error handling**: every `await` and promise chain has a failure path.
- **Equality**: `===` / `!==` over `==` / `!=` unless intentional.
- **Immutability**: avoid mutating props/state directly (React/Vue).
- **`innerHTML` and friends**: justified and sanitized?
- **Dependency risks**: known-vulnerable packages; flag any imports worth checking.

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

- **Be direct, not harsh.** Explain the fix, don't just condemn the code.
- **Always explain why**, not just what to change.
- **Distinguish opinion from fact.** "This *could* be memoized" vs "This *is* an XSS sink."
- **Prioritize ruthlessly.** Group issues that share a root cause.
- **Acknowledge good work.** "What's Working Well" is not optional filler.

## Quick Reference: Common JS/TS Vulnerabilities

| Issue | Bad | Good |
|---|---|---|
| XSS | `el.innerHTML = userInput` | `el.textContent = userInput` (or sanitize) |
| eval | `eval(userInput)` | Parse explicitly; avoid dynamic code execution |
| Prototype pollution | `Object.assign({}, untrusted)` into shared objects | Validate keys; reject `__proto__`/`constructor` |
| Loose equality | `if (x == null)` mixing types | `if (x === null || x === undefined)` |
| Floating promise | `doAsync()` | `await doAsync()` with `try/catch` |
| any type | `const data: any = resp` | `const data: unknown = resp` then narrow |
