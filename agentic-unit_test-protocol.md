# Agentic Testing Protocol: Test Architecture and Quality Control
### A standalone companion to the Agentic Coding Control Protocol — governs how tests are structured, placed, and written, independent of how production logic is written.

---

## Why This Is a Separate Document

The main control protocol governs how an agent writes production logic: naming, file boundaries, complexity budgets, review gates. Tests are not production logic. They have a different objective — proving a contract holds, not fulfilling a contract — and they fail in different, specific ways that the main protocol's rules do not catch.

An agent can follow every naming, structure, and simplicity rule in the main protocol perfectly and still produce a test suite that is worthless, because:

- It can write a test that always passes regardless of whether the code is correct.
- It can mock the exact thing it should have left real, and leave real the exact thing it should have mocked.
- It can generate hundreds of tests for implementation details while leaving the feature's actual promise unverified.
- It can dump every test file into one flat directory with no axis of organization, so neither a human nor the next agent session knows where a new test belongs.

This document exists to close those gaps. It applies to any project, any domain, any language with an equivalent of pytest's discovery and fixture model. The examples below use generic placeholder domains (order processing, payment, user accounts) — they are illustrations, not a project template. Replace the nouns; keep the rules.

---

## The 3 Testing Failure Modes

**1. The Mirror Trap.**
The agent writes a test by reading the implementation, mentally executing it, and asserting whatever the implementation produced. The test is not derived from a contract — it is a reflection of the code in a different file. It will pass today and will continue passing after a bug is introduced, because the "expected value" was never independently determined. This is the single most common reason agent-written test suites give false confidence.

**2. The Flat Dump.**
Every test file lands in one directory regardless of what it tests or how it should run. There is no `conftest.py` hierarchy, no separation between fast isolated tests and tests that depend on live external systems, and no consistent file-naming pattern. The result: fixtures get copy-pasted into every file instead of shared, a test that hits a live dependency can get swept into a default run by accident, and neither a developer nor an agent has a deterministic answer to "where does a test for this new function go."

**3. The Granularity Inversion.**
The agent treats "a unit" as "a function" rather than "an observable behavior." Every private helper gets its own dedicated, exhaustive test file — happy path, every branch, every malformed input — regardless of whether that helper is reachable only through one caller. The result is a test count that scales with implementation detail rather than with feature surface area, an inverted test pyramid, and a suite where a harmless internal refactor (renaming a private helper, restructuring how a loop is written) breaks dozens of tests that were never supposed to know that helper existed. When this happens repeatedly, every code change becomes expensive to test — the suite has stopped serving its purpose and started serving as a tax on changing the code at all.

---

## The 6 Testing Principles

**1. Contracts Define Expectations, Not Implementations**
A test's expected value must come from the function's stated contract — its docstring, its requirement, an independently known correct answer — never from running the implementation in your head and writing down what it did. If you cannot state the expected output without looking at the implementation body, you do not yet know what the test should assert.

**2. Mock the Boundary, Not the Logic**
A test double belongs at the edge of your system — network, disk, database, wall-clock time, randomness, a third-party service — never at the boundary of your own pure logic. If you are mocking a function with no side effects and no I/O, you are not testing anything; you are asserting that a function calls another function, which is rarely the thing that matters.

**3. A Unit Is a Behavior, Not a Function**
The thing under test is an observable capability with a defined input and output — not every function that happens to exist in the call chain underneath it. Private helpers reachable only through one public entry point are exercised indirectly, through tests of that entry point, not through a dedicated test file of their own.

**4. Structure Is Discovery**
A developer or agent must be able to answer "where does this test go" and "where does this fixture go" before writing a single line, by following a fixed decision procedure — not by guessing, not by copying whatever the last file did. If the answer requires inspecting the existing mess to infer a pattern, the structure has already failed.

**5. Tests Must Outlive Refactors**
A test that breaks when an internal implementation detail changes — without the function's observable behavior changing — was coupled to the wrong thing. Tests assert on inputs, outputs, and observable side effects. They must not assert on internal call order, private state, or helper function names, unless that internal detail is itself the contract being tested.

**6. Test Suites Have a Budget Too**
Every new function does not need six new test files. Test count must scale with the number of distinct behaviors a feature promises, not with the number of lines or internal helper functions it took to implement that feature. An exploding test count is not thoroughness — it is a signal that granularity (Principle 3) was violated.

---

## Part 1: Test Directory Architecture

### The Mirror Rule

The test tree mirrors the source tree. A test file's location is derivable from the source file it tests, not from when it was written or which developer wrote it.

```
src/
  myapp/
    orders/
      order_validator.py
      order_repository.py
    notifications/
      email_sender.py
tests/
  unit/
    orders/
      test_order_validator.py
      test_order_repository.py
    notifications/
      test_email_sender.py
```

If you cannot point to the single source file a test file is mirroring, the test file's placement has not been decided yet — decide it before creating the file, do not place it provisionally "for now."

### The Category Split

Tests are split by **what they depend on**, not by which sprint they were written in. At minimum, three categories:

| Category | What it depends on | What it verifies | Runs by default? |
|---|---|---|---|
| `unit/` | Nothing external — pure logic, in-memory data, doubles at every true boundary | A single behavior in isolation | Yes — every run |
| `integration/` | Real collaborators within your own system (a real test database, real file I/O, multiple internal components composed together) | That your own components compose correctly | Yes — every run, usually slower |
| `smoke/` (or `e2e/`) | Live external systems — a real third-party API, a deployed environment, network calls that leave the machine | That the whole system is reachable and minimally functional against the real world | No — excluded from default run, triggered explicitly or on a schedule |

Each category gets its own subdirectory under `tests/`. A test's category is decided by what it touches, not by how important it feels. If a test silently depends on network access while sitting in `unit/`, that is a misplacement to fix immediately, not a style nitpick.

### The conftest.py Hierarchy Rule

Fixtures are scoped to the directory they live in and everything beneath it. This is not an implementation detail to work around — it is the mechanism that keeps fixtures from leaking into tests that don't need them.

```
tests/
  conftest.py              <- fixtures needed by EVERY test, regardless of category
  unit/
    conftest.py            <- fixtures needed only by unit tests (e.g. an in-memory fake)
    orders/
      test_order_validator.py
  integration/
    conftest.py             <- fixtures needed only by integration tests (e.g. a real test DB connection)
    test_order_pipeline.py
  smoke/
    conftest.py              <- fixtures needed only by smoke tests (credentials, longer timeouts)
    test_live_dependency.py
```

**The placement rule for a new fixture:**
1. Is it needed by tests in more than one category (unit and integration both use it)? -> the root `tests/conftest.py`.
2. Is it needed only within one category? -> that category's `conftest.py`.
3. Is it needed only within one sub-package inside a category? -> a `conftest.py` nested at that sub-package level.

Never put a fixture in the root `conftest.py` "just in case it's useful later." A fixture lives at the narrowest scope that currently needs it. It moves up only when a second, real caller in a different scope needs it too — this is the same discipline as the Patterns Emerge, They Don't Arrive principle from the main protocol, applied to fixtures.

**Never put test functions inside `conftest.py`.** It exists for fixtures, hooks, and shared setup only.

**Never import a fixture manually.** Pytest (and equivalents) discover and inject fixtures by name automatically from any `conftest.py` in scope. A manual import breaks that mechanism and is a signal the fixture is in the wrong file.

### The Discovery Pattern Rule (the silent-skip footgun)

Every test runner has a default file-naming pattern it uses to discover tests. For pytest, that default is `test_*.py` or `*_test.py` — nothing else. A file that doesn't match this pattern is silently skipped: it does not error, it does not warn, it simply never runs.

**This means:**
- Every test file, with no exceptions, starts with `test_` or ends with `_test`.
- A descriptive prefix like `smoke_`, `live_`, or `manual_` does not replace the discovery prefix — it goes after it: `test_smoke_live_dependency.py`, not `smoke_live_dependency.py`.
- If a non-standard naming pattern is genuinely needed, it must be registered explicitly in the runner's configuration (e.g. pytest's `python_files` setting) — and that registration itself must be visible in the project's central config file, not assumed.

**Enforcement moment:** before creating any test file, the agent must state the exact filename and confirm it matches the discovery pattern the project's configuration uses. This is a one-line check that prevents an entire test file from silently never executing.

### The Test Data Directory Rule

Static fixture data — sample payloads, golden expected-output files, recorded responses — lives in a dedicated, non-test-discovered directory, referenced through a fixture, never through a hardcoded relative path scattered across files.

```
tests/
  data/
    sample_input_record.json
    expected_output_record.json
  unit/
    ...
```

```python
# tests/conftest.py
@pytest.fixture(scope="session")
def test_data_dir(project_root):
    """Directory holding static fixture data shared across the test suite."""
    return project_root / "tests" / "data"
```

A test that opens `"../../data/sample.json"` inline has hardcoded a path — banned under the same rule that bans hardcoded absolute paths in production code.

### The `__init__.py` Rule for Duplicate Names

Adding an `__init__.py` to each test subdirectory allows the same test filename to exist in more than one category without collision — useful once a behavior is tested both in isolation and in composition.

```
tests/
  unit/
    orders/
      __init__.py
      test_order_validator.py
  integration/
    orders/
      __init__.py
      test_order_validator.py     <- same name, different category, no collision
```

### Configuration Lives in One Place

Test runner configuration — discovery patterns, marker registration, default options — belongs in the project's central configuration file (e.g. `pyproject.toml` under `[tool.pytest.ini_options]`), not scattered across ad hoc `.ini` files or assumed defaults.

```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
addopts = ["--strict-markers", "-m", "not smoke"]
markers = [
    "smoke: depends on a live external system, excluded from the default run",
    "slow: takes longer than the rest of the suite, may be deselected locally",
]
```

`--strict-markers` turns an unregistered marker into a hard error instead of a silent typo. Any marker the agent introduces must be registered here in the same change that introduces it.

### The Placement Decision Tree

Before creating any test file, answer this in order:

```
1. What source file does this test exercise?
   -> tests/<category>/<mirrored path to that source file>/test_<source filename>.py

2. What does the behavior under test depend on?
   -> Nothing external, pure logic only            => unit/
   -> Real collaborators within your own system     => integration/
   -> A live external system                        => smoke/  (and mark it accordingly)

3. Does this test need a fixture that doesn't exist yet?
   -> Needed only here                              => this file, or this directory's conftest.py
   -> Needed by other tests in this category only   => this category's conftest.py
   -> Needed across categories                      => the root conftest.py

4. Does the filename start with test_ or end with _test?
   -> If no, stop. Rename before writing a single test inside it.
```

If any answer is unclear, that is the signal to stop and ask, not to place the file provisionally and move it "later."

---

## Part 2: Test Contract Independence (closing the Mirror Trap)

### The Blind Expectation Rule

The expected value in an assertion must be derivable from the function's contract — its docstring, the stated requirement, or an independently verifiable fact — without reading the implementation body. If the only way you can state what a test should assert is by tracing through the function's logic, you are about to write a tautology, not a test.

```python
# WRONG — the expected value was derived by reading the implementation
# and re-stating what it computes. This test will pass even if the
# discount calculation is wrong, as long as it's wrong consistently
# with how it's wrong right now.
def test_apply_discount():
    result = apply_discount_to_price(price=100, discount_percent=10)
    assert result == price - (price * discount_percent / 100)  # mirrors the code

# RIGHT — the expected value comes from the contract: a 10% discount
# on 100 is 90. This is independently true regardless of how the
# function is implemented.
def test_apply_discount_subtracts_percentage_of_price():
    result = apply_discount_to_price(price=100, discount_percent=10)
    assert result == 90
```

### Verifying a Test Suite Isn't Tautological

Line coverage cannot tell you whether a test actually checks anything — it only tells you the line executed. Mutation testing answers the real question: introduce a deliberate fault into the implementation and confirm the test suite fails. A test that keeps passing after the logic it covers has been deliberately broken was never verifying that logic — it was just executing it.

This check is run periodically (end of a feature, not on every single commit — it is expensive), as a quality gate on the test suite itself, separate from running the tests.

---

## Part 3: Test Double Discipline

### The Taxonomy

Not every test double serves the same purpose. Reaching for "mock everything inconvenient" instead of choosing the right tool is how mocking discipline collapses.

| Double | Purpose | Use when |
|---|---|---|
| Dummy | A placeholder passed but never actually used | A parameter is required by the signature but irrelevant to this test case |
| Stub | Returns canned data to drive the test down a specific path | You need a collaborator to return a specific value, and you don't care how it's called |
| Spy | Records how it was called, for inspection after the fact | You need to verify a side-effecting call happened, but don't need to control its return value |
| Mock | Asserts the exact interaction — calls, arguments, order | The *fact that a call happened, with specific arguments* is itself the behavior being tested |
| Fake | A working, simplified real implementation (e.g. an in-memory store standing in for a database) | You want realistic behavior without the cost or flakiness of the real dependency |

### Don't Mock What You Don't Own

Mocking belongs at a boundary your own code defines, not at the boundary of a third-party library's internals. If you find yourself mocking a method deep inside an external SDK, wrap that SDK in a thin adapter you own first, and mock the adapter instead. This keeps your tests stable when the third-party library changes its internal call pattern, since your adapter's contract — not the library's — is what your tests depend on.

### What to Mock, What to Keep Real

| Mock or stub this | Keep this real |
|---|---|
| Network calls, disk I/O, database connections | Pure functions and calculations |
| Wall-clock time, randomness | In-memory data structures |
| Third-party SDK clients (via your own adapter) | Your own domain objects and value types |
| Anything slow, flaky, or nondeterministic | Anything deterministic and in-process |

### The Mock Justification Declaration

Before adding a mock, the agent states:

```
Mocking [collaborator] because it is [network / disk / time / randomness / third-party service].
Everything else in this test uses the real implementation.
```

If the justification doesn't fit one of those boundary categories, the mock is probably hiding logic that should be tested with the real thing.

---

## Part 4: Granularity and the Test Count Budget

### The Decision Tree

```
Is this function or class called from OUTSIDE the module
(it is a public entry point, or it is the feature's observable surface)?

  YES -> Write behavior-level tests directly against it.
         Default budget: 1 happy path + up to 3 edge cases + up to 2 error paths.

  NO (it's a private helper used by exactly one caller) ->
       Does it have complex branching that the public-level tests
       genuinely cannot reach through normal inputs?

         NO  -> No dedicated test file. It is already exercised
                indirectly through the public-entry-point tests.

         YES -> This complexity is a signal the helper should be
                extracted into its own small, named, pure unit.
                Test that unit directly and sparingly — prefer one
                property-based test over a dozen hand-written examples.
```

### The Test Count Budget

Mirrors the main protocol's complexity budget, applied to tests:

| Situation | Budget |
|---|---|
| One public function, straightforward logic | 1 happy path + 2–3 edge cases |
| One public function, several distinct error conditions | add 1 test per distinct error category, not per malformed input variant |
| A private helper with complex branching, extracted and named | covered by Part 4's decision tree, tested directly only when justified |
| An entire new feature | sum of its public-surface budgets — not one test per internal function that happened to get written |

**The budget-exceeded phrase**, used the same way as in the main protocol:

```
TEST BUDGET EXCEEDED: This function would get [N] tests under the default budget.
Reason: [why the behavior genuinely has this many distinct cases].
Proceed at this count, or should the function's contract be simplified instead?
```

A test count climbing because the function itself has too many responsibilities is a signal to simplify the function, not to write more tests around it.

---

## Part 5: The Structure of a Single Test

### Arrange-Act-Assert

Every test follows the same three-part shape, and the parts are not interleaved:

```python
def test_apply_discount_subtracts_percentage_of_price():
    # Arrange
    price = 100
    discount_percent = 10

    # Act
    result = apply_discount_to_price(price, discount_percent)

    # Assert
    assert result == 90
```

### Naming Convention

`test_<unit>_<condition>_<expected_outcome>` — the name alone should tell you what's being verified, without opening the file.

| Correct | Wrong | Why |
|---|---|---|
| `test_apply_discount_subtracts_percentage_of_price` | `test_discount` | Doesn't say what condition or outcome is checked |
| `test_validate_order_raises_when_total_is_negative` | `test_validate_order_2` | Numbered test names hide intent entirely |
| `test_find_user_returns_none_when_not_found` | `test_find_user_edge_case` | "Edge case" doesn't say which edge or what happens |

### One Behavior Per Test

A test verifies one behavior. If a test needs "and" to describe what it checks, it is two tests. This mirrors the main protocol's Single-Concern Rule, applied at the test-function level rather than the file level.

### Parametrize Instead of Duplicating

Several near-identical example-based tests that differ only by input and expected output collapse into one parametrized test plus a data table — this keeps the file's line count, and the agent's token cost to maintain it, proportional to the number of distinct cases, not the number of test function bodies.

```python
@pytest.mark.parametrize(
    "price,discount_percent,expected",
    [
        (100, 10, 90),
        (100, 0, 100),
        (50, 50, 25),
    ],
)
def test_apply_discount_subtracts_percentage_of_price(price, discount_percent, expected):
    assert apply_discount_to_price(price, discount_percent) == expected
```

---

## Part 6: Refactor Resilience and the Economics of Change

A test that asserts only on a function's documented inputs, outputs, and observable side effects survives any internal refactor that does not change that contract. A test that asserts on internal call order, private variable names, or which private helper got called survives nothing — it breaks on every refactor regardless of whether behavior changed, and someone (or some agent, burning tokens) has to re-derive it from scratch every time.

This is the direct payoff of Parts 2–5: a suite built on contract-derived expectations (Part 2), boundary-only mocking (Part 3), behavior-level granularity (Part 4), and clean single-behavior tests (Part 5) does not need to be regenerated when the implementation changes shape underneath it.

### Tooling That Makes This Operational

These are not optional extras — they are how the above principles get verified mechanically instead of by hope:

| Tool category | What it does | When to run it |
|---|---|---|
| Test impact analysis (e.g. `pytest-testmon`) | Tracks which tests actually exercise which lines, so a small change only re-runs the tests affected by it, not the full suite | Every local run |
| Mutation testing (e.g. `mutmut`, `cosmic-ray`) | Deliberately breaks the implementation and confirms tests fail — the mechanical check for the Mirror Trap | End of feature, not every commit |
| Property-based testing (e.g. `hypothesis`) | Replaces a wall of hand-written examples with one invariant checked against generated inputs | Any function with a wide or combinatorial input space |
| Codebase/test dependency graph tools | Lets an agent query "what calls this" or "what tests cover this" without re-reading the whole codebase | Large codebases, or whenever blast-radius of a change needs checking before touching tests |

Run the expensive checks (mutation testing, full dependency graph rebuild) at checkpoints, not on every keystroke. Run the cheap ones (test impact analysis) constantly — that is what they are for.

---

## Part 7: Review Gates for Test Authoring

| Trigger | What the agent does | What you decide |
|---|---|---|
| About to create a new test file | Walks the Placement Decision Tree out loud: source file, category, fixture scope, filename pattern | Confirm placement or redirect it |
| About to add a mock | States the Mock Justification Declaration | Confirm the boundary reasoning or push back |
| A function's test count would exceed the default budget | States the Test Budget Exceeded phrase with itemized reasoning | Approve the higher count or ask for the function to be simplified |
| About to write an assertion's expected value | States where the expected value came from — contract, requirement, or independently known fact, not "what the code returned" | Confirm the expectation is independently derived |
| About to add a test that depends on a live external system | States the category (`smoke/`) and confirms it is marked to be excluded from the default run | Confirm marker and placement before the file is created |
| Several near-identical test functions are about to be written | Proposes parametrization instead | Approve parametrized form or justify keeping them separate |

**The review gate phrase**, identical in spirit to the main protocol:

```
PAUSE: [what decision is needed]
I will not continue until you confirm.
```

---

## Appendix A: System Prompt Addendum

**Paste this alongside the main control protocol's system prompt.**

```
TESTING RULES (separate objective from production-code rules)

PLACEMENT: Before creating any test file, state out loud:
1. Which source file this test exercises, and the mirrored path under tests/.
2. Whether it belongs in unit/, integration/, or smoke/ based on what it
   depends on (nothing external / your own real collaborators / a live
   external system) — not based on how important it feels.
3. Whether a needed fixture already exists at the narrowest applicable
   conftest.py scope (this directory, this category, or the whole suite).
4. That the filename starts with test_ or ends with _test — files that
   don't match the runner's discovery pattern are silently never executed.
Do not write a test body until all four are answered.

CONTRACT INDEPENDENCE: The expected value in any assertion must come from
the function's documented contract or an independently known correct
answer — never from reading the implementation and writing down what it
currently does. If you can't state the expected output without tracing
through the implementation body, stop and find the actual contract first.

MOCKING DISCIPLINE: Mock only at a true boundary — network, disk, database,
time, randomness, or a third-party service wrapped in your own adapter.
Before adding a mock, state: "Mocking [X] because it is [boundary type].
Everything else here uses the real implementation." If no boundary type
fits, do not mock it.

GRANULARITY: A unit under test is an observable behavior, not every
function in the call chain. Public entry points get direct tests
(1 happy path + 2-3 edges + up to 2 error paths, by default). Private
helpers reached only through one caller get no dedicated test file —
they are exercised indirectly. Only test a private helper directly if its
branching genuinely cannot be reached through the public surface, and even
then prefer extracting it into its own named unit first.

TEST BUDGET: If a single function's tests would exceed the default count
above, stop and write: "TEST BUDGET EXCEEDED: [N] tests, because [reason].
Proceed, or simplify the function's contract instead?"

STRUCTURE: One behavior per test, named test_<unit>_<condition>_<outcome>.
Arrange-Act-Assert, not interleaved. Collapse near-identical example tests
into one parametrized test with a data table.

REFACTOR RESILIENCE: Assert only on documented inputs, outputs, and
observable side effects. Never assert on internal call order, private
variable names, or which private helper ran, unless that internal detail
is itself the stated contract.
```

---

## Appendix B: Quick Reference and Red Flags

**Red flags to stop immediately:**
- A new test file was created without walking the Placement Decision Tree first
- A test file's name doesn't start with `test_` or end with `_test` and no discovery-pattern override was registered in the project's central config
- An assertion's expected value can only be explained by re-reading the implementation
- A mock was added with no stated boundary reason
- A private helper used by exactly one caller got its own dedicated test file with no justification
- Several test functions differ only by input/expected-output values and were not parametrized
- A test asserts on internal call order or a private variable name, and that order/name is not itself the contract
- A fixture was placed in the root `conftest.py` "in case it's needed later" rather than where it's currently needed
- A test depending on a live external system is not marked and not placed under a category excluded from the default run
- Test count for one function grew past the default budget with no stated reason

**The one question that catches most of this:**
> "If I deleted this test's assertion and rewrote it from the contract alone, without looking at the implementation, would I write the same expected value?"

If the honest answer is no, the test was derived from the Mirror Trap, not from the contract.
