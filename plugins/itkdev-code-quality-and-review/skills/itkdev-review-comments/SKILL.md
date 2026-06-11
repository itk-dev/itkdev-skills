---
name: itkdev-review-comments
user-invocable: true
description: Review and improve inline code comments following ITK Dev conventions. Explains "why" not "what", keeps comments short and focused, removes redundant comments. Use when asked to review comments, clean up comments, or improve code documentation. ONLY modifies comments and docblocks — never changes code.
---

# Code Comments Review

Review the target code and improve its inline comments. This skill touches
**only comments and docblocks** — never code.

## Scope

**Allowed changes:**
- Add a comment or docblock
- Edit an existing comment or docblock
- Remove a redundant comment or docblock

**Forbidden changes — do not touch code:**
- No extracting constants, renaming variables, or refactoring
- No adding, removing, or reordering code statements
- No changing log levels, string literals, or function signatures
- No adding or removing blank lines in code
- No removing `console.log`, logging calls, or any executable statement

Simple test: if a line is not a comment or docblock, leave it exactly as-is.

## Comment Syntax by Language

Use the idiomatic comment syntax for each language:

| Language   | Inline          | Docblock / Block     |
|------------|-----------------|----------------------|
| PHP        | `//`            | `/** */`             |
| JavaScript | `//`            | `/** */` (JSDoc)     |
| Python     | `#`             | `"""docstring"""`    |
| Twig       | `{# #}`         | `{# #}` (multiline) |
| CSS / SCSS | `/* */`         | `/* */`              |
| YAML       | `#`             | `#` (no block form)  |
| Shell      | `#`             | `#` (no block form)  |

For languages not listed, follow the language's standard convention.

## Principles

1. **Explain the "why", not the "what"** — do not restate the code. Explain
   intent, constraints, or non-obvious reasoning.

2. **One comment per concept** — place each comment directly above the thing it
   explains. Do not group multiple explanations into a single block comment.

3. **Keep comments short** — one line preferred, two lines max. If more is
   needed, use a docblock.

4. **Docblocks for non-obvious algorithms** — regex patterns, math formulas,
   and heuristics get a docblock with examples (e.g. input → output).

5. **No redundant comments** — do not comment self-evident code. Remove
   comments that just restate the method name, class name, or next line.
   Keep `@param`/`@return`/`@throws` type annotations needed by static
   analysis tools (e.g. PHPStan).

6. **Language rule** — comments and code in English. Follow the project's own
   conventions for UI text.

## Workflow

1. Read the file(s) the user points to (or the current diff if no file is
   specified).
2. Identify unclear logic, missing context, and redundant comments.
3. Apply the principles above — add, rewrite, or remove comments as needed.
4. **Verify every change is a comment or docblock.** If any line of code would
   change, revert it. Use `git diff` to confirm only comments were modified.
5. Present the changes to the user for review.

## Examples

### Bad — restates the code
```php
// Set timeout to 30
$timeout = 30;
```

### Good — explains the why
```php
// Matches the scanner container's max response time
$timeout = 30;
```

### Bad — code change disguised as comment cleanup
```diff
- console.log("Scan started", { url });
+ console.debug("Scan started", { url });
```
This changes a code statement. The skill must never do this.

### Bad — removing a code statement
```diff
- console.log("Processing update", data);
```
Removing executable code is forbidden, even if it looks like "cleanup".

### Bad — wrong comment syntax for PHP docblock
```php
// Calculate contrast ratio using WCAG 2.1 formula.
// Formula: (L1 + 0.05) / (L2 + 0.05) where L1 >= L2.
private function contrastRatio(float $l1, float $l2): float
```

### Good — proper PHP docblock syntax
```php
/**
 * WCAG 2.1 relative luminance contrast ratio.
 *
 * Formula: (L1 + 0.05) / (L2 + 0.05) where L1 >= L2.
 * Example: white (#fff) vs black (#000) → 21:1.
 */
private function contrastRatio(float $l1, float $l2): float
```

### Bad — redundant docblock restating the method name
```php
/**
 * Find all palettes with their colors loaded.
 *
 * @return ColorPalette[]
 */
public function findAllWithColors(): array
```

### Good — keep only the type annotation
```php
/**
 * @return ColorPalette[]
 */
public function findAllWithColors(): array
```
