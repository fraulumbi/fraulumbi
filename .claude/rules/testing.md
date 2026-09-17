# Testing conventions

## Test naming

- A test name states the **condition and the expected outcome**, not the function
  name. The reader should understand the failure from the test output alone,
  without opening the file.
  - Good: `test "returns :error when the license expired before the order date"`
  - Bad: `test "validate_license"` / `test "works"` / `test "test 2"`
- Use the form *"<verb phrase> when <condition>"*. Present tense, no "should".
- Group by the unit under test with `describe "function_name/arity"`, so failures
  print as `MyModule.validate/2: returns :error when ...`.
- Test files mirror the source tree exactly: `lib/orders/pricing.ex` →
  `test/orders/pricing_test.exs`. One test module per source module.
- A test written to lock in a bug fix names the bug:
  `test "does not double-count split shipments (bug #1423)"`.

## Assertion style

- **One behavior per test.** Multiple assertions are fine when they describe a
  single outcome; a test asserting two unrelated behaviors is two tests.
- Assert on the exact expected value, not on a property of it. `assert result ==
  {:ok, %{qty: 3}}` beats `assert match?({:ok, _}, result)`, which passes on the
  wrong quantity.
- Pattern-match in the assertion when only part of the shape is relevant:
  `assert {:ok, %Order{status: :fulfilled}} = create_order(attrs)`.
- Never assert on a truthy value alone. `assert result` passes for `1`, `"no"`,
  and `%{}` — say what you actually expect.
- Expected value on the right, actual on the left, consistently — diff output
  depends on it.
- Assert errors precisely: the exact error tuple or exception type and message,
  never just "it raised something".
- No conditional logic in tests. An `if` in a test means it's two test cases, and
  a branch that never runs silently asserts nothing.
- No assertions on log output, timing, or internal call counts unless that *is*
  the behavior under test.
- A test with no assertion is a bug. Tests that only check "it didn't crash" must
  say so explicitly in the name.

## Fixtures and setup

- Prefer **factories over static fixtures**. A factory builds the minimum valid
  record and takes overrides for the fields the test cares about:
  `insert(:order, status: :draft)`.
- The test body shows every value it depends on. If an assertion turns on
  `status: :draft`, that must be visible in the test, not buried in `setup`.
- `setup` holds only what is genuinely shared and incidental — a checked-out DB
  connection, a started process, a frozen clock. Not the data under test.
- Each test creates its own data. Never depend on rows left behind by another
  test, and never depend on execution order.
- Tests run in a transaction that rolls back. Anything that can't (external HTTP,
  file writes) is stubbed at the boundary.
- No shared mutable state across tests. Async tests that touch a global must be
  marked `async: false`, with a comment saying which global.
- Time is injected, never read from the system clock in assertions. A test that
  fails on the last day of the month is a broken test.
- External services are stubbed with a recorded, realistic payload — including
  their error responses. Every external call needs at least one failure test.
- Fixture data is realistic but obviously fake: `"Test Dispensary LLC"`,
  `"C11-0000001-TEMP"`. Never real customer names, license numbers, or PII.

## Coverage expectations

- Every bug fix ships with a test that fails without the fix.
- Every new branch of business logic has a test. Coverage percentage is a smell
  detector, not a goal.
- Test behavior through the public interface. Reaching into private functions
  couples tests to implementation and blocks refactoring.
- The happy path and at least one failure path are both asserted.
