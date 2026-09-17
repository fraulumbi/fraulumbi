# Coding standards

Universal rules for any code written in this repo. Keep additions here short — this
file is loaded into context on every session, so every line costs tokens.

Detailed rule sets live in `standards/` and `.claude/rules/`, and are pulled in by the
imports at the bottom of this file.

## Error handling

- **Return errors for expected failures, raise for bugs.** A missing record, a
  failed validation, or a rejected API call is data — model it. A nil where the
  type guarantees a value is a defect — let it crash.
- Use the language's result type consistently: `{:ok, value} | {:error, reason}` in
  Elixir; never mix that with bare returns in the same function.
- Error reasons are structured, not prose. `{:error, :license_expired}` or
  `{:error, %ValidationError{field: :qty}}` can be matched on;
  `{:error, "something went wrong"}` cannot.
- Never swallow an error. No empty rescue/catch blocks, no bare `rescue _ -> nil`.
  If an error is genuinely ignorable, log it at `:debug` with a comment saying why.
- Handle failure at the layer that can actually decide what to do. Lower layers
  return errors; the boundary (controller, worker, CLI) decides between retry,
  surface to the user, or fail the request.
- Catch narrowly. Rescue the specific exception you expect, not every exception.
- Log with context, never with secrets. Include the identifiers needed to find the
  row again (`order_id`, `company_id`); exclude tokens, keys, and PII.
- Retries are only for transient failures (timeouts, 5xx, deadlocks) and always
  bounded with backoff. Never retry a validation error.
- Cleanup is guaranteed, not hoped for — use `after`/`ensure`/`defer` or the
  language's bracket pattern for anything that opens a resource.
- Anything that writes across multiple tables runs in a transaction, and the
  rollback path is tested.

## Code review checklist

Run through this before opening a PR, and again when reviewing one.

**Correctness**
- [ ] Does it actually do what the ticket/issue asked — no more, no less?
- [ ] Edge cases covered: empty collection, nil/null, zero, negative, duplicate,
      max size, concurrent callers.
- [ ] No off-by-one, no unhandled branch, no unreachable code.

**Errors & failure**
- [ ] Every failure path returns or raises deliberately — none are silently dropped.
- [ ] External calls (DB, HTTP, queue) have a defined behavior on timeout/failure.
- [ ] Multi-write operations are transactional and roll back cleanly.

**Data & queries**
- [ ] No N+1 queries; joins or preloads used where a loop would hit the DB.
- [ ] New queries hit an index, and new columns that get filtered on have one.
- [ ] Migrations are backwards-compatible with the currently deployed code, and
      reversible.

**Security**
- [ ] No secrets, keys, or credentials in code, tests, fixtures, or logs.
- [ ] All user input validated at the boundary; queries parameterized, never
      string-interpolated.
- [ ] Authorization checked on the server for every new endpoint or action.

**Tests**
- [ ] A test exists that fails without the change and passes with it.
- [ ] Both the happy path and at least one failure path are asserted.
- [ ] Tests assert on behavior, not on internal implementation details.

**Readability**
- [ ] Names match the rules above; no leftover debug output, dead code, or TODOs
      without an issue link.
- [ ] Comments explain *why*, not *what* — the code already says what.
- [ ] Public functions have a docstring covering arguments, return shape, and
      failure modes.
- [ ] The diff is scoped to one concern; unrelated refactors go in their own PR.

## Imports

@./standards/naming.md
@.claude/rules/testing.md
