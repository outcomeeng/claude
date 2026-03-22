# Testing Methodology

Tests are evidence, not bureaucracy. Every test must answer: "Will the code under test correctly implement the specified outcomes?" If a test doesn't provide evidence about production behavior, delete it.

## Essential principles

- **No mocking. Ever.** Mocking gives you tests that pass while production code fails. If you feel you need to mock, redesign with dependency injection or test at a different level.
- **Reality is the oracle.** Test against real systems whenever possible. A test that passes against a fake proves nothing about production.
- **Test doubles are exceptions, not defaults.** The seven exception cases in Stage 5 are the ONLY legitimate uses.

## Five-stage router

Before writing ANY test, route through all five stages. Do not skip ahead.

| Stage | Outcome                                               | Next Step                                               |
| ----- | ----------------------------------------------------- | ------------------------------------------------------- |
| 1     | Evidence identified                                   | Stage 2                                                 |
| 2     | Level 2 or 3 required                                 | Use real dependencies. DONE.                            |
| 2     | Level 1 appropriate                                   | Stage 3                                                 |
| 3A    | Pure computation                                      | Test directly, no doubles. DONE.                        |
| 3B    | Can extract pure part                                 | Extract, test pure at L1, integration at L2. DONE.      |
| 3C    | Glue/orchestration code                               | Stage 4                                                 |
| 4     | Real system works (reliable, safe, cheap, observable) | Use real at Level 2. DONE.                              |
| 4     | Real system doesn't work for testing                  | Stage 5                                                 |
| 5     | Exception case matches                                | Use appropriate double, document which exception. DONE. |
| 5     | No exception matches                                  | Don't write L1 for this code. Test at L2. DONE.         |

### Stage 1: What evidence do you need?

Before writing any test, answer:

1. **What behavior could be wrong in production?** Not "what code am I testing" but what could actually fail for users?
2. **If this test passes, what does that prove about the real system?** A test that proves nothing about production is waste.
3. **What failure would this test catch that would otherwise reach users?** If you can't name a concrete failure, you don't need the test.

**The Evidence Trap:** Agents often skip this stage. They see code and think "I need to test this." That's backwards.

- **Wrong**: See `OrderProcessor` that calls `repository.save()` → create `InMemoryRepository` fake → write test that passes
- **Right**: Ask "What evidence do I need?" → "Evidence that orders persist correctly" → fake repository proves nothing about persistence → test at Level 2 with real database

### Stage 2: At what level does that evidence live?

Use the five factors to determine where evidence can be proven.

**Factor 1: What does the spec promise?**

| Spec Promise                      | Minimum Level | Why                                |
| --------------------------------- | ------------- | ---------------------------------- |
| "Prices are calculated correctly" | Level 1       | Pure calculation                   |
| "User can export data as CSV"     | Level 1       | File I/O with temp dirs is Level 1 |
| "CLI processes Hugo site"         | Level 2       | Project-specific binary            |
| "Database query returns users"    | Level 2       | Real database required             |
| "User can complete checkout"      | Level 3       | Real payment provider              |
| "Works in Safari"                 | Level 3       | Real browser required              |

**Factor 2: What dependencies are involved?**

| Dependency                             | Minimum Level |
| -------------------------------------- | ------------- |
| None (pure function)                   | Level 1       |
| File system (temp dirs)                | Level 1       |
| Standard dev tools (git, node, curl)   | Level 1       |
| Database                               | Level 2       |
| External HTTP API                      | Level 2       |
| Project-specific binary (Hugo, ffmpeg) | Level 2       |
| Browser API                            | Level 3       |
| Third-party service (live)             | Level 3       |
| Real credentials                       | Level 3       |

**Factor 3: How complex is YOUR code?**

| Code Type                               | Level 1 Value                |
| --------------------------------------- | ---------------------------- |
| Your logic (algorithms, parsers, rules) | HIGH - test thoroughly       |
| Library wiring (argparse, Zod, YAML)    | LOW - trust the library      |
| Simple glue code                        | LOW - covered by integration |

**Factor 4: Debuggability needs**

When a Level 2/3 test fails, will a Level 1 test help find the bug faster?

| Scenario                                        | Add Level 1? | Reason                                 |
| ----------------------------------------------- | ------------ | -------------------------------------- |
| Integration test fails on complex algorithm     | YES          | Level 1 isolates the algorithm         |
| Integration test fails on argparse flag parsing | NO           | Trust argparse; check your usage       |
| E2E test fails on payment flow                  | MAYBE        | If payment calculation is complex, yes |

**Factor 5: Where does achievable confidence live?**

| You Want to Know...           | Achievable At |
| ----------------------------- | ------------- |
| Your math is correct          | Level 1       |
| Your SQL is valid             | Level 2       |
| The API accepts your requests | Level 2       |
| Users can complete the flow   | Level 3       |

**Level selection decision:**

- Evidence lives at Level 3 → Use real environment. DONE.
- Evidence lives at Level 2 → Use real dependencies. DONE.
- Evidence lives at Level 1 → Go to Stage 3.

**If evidence requires Level 2 or 3, stop here.** Use real dependencies. Do not fake what you can test for real.

### Stage 3: What kind of Level 1 code is this?

**3A: Pure computation** — Given inputs, compute outputs. No external state, no side effects. Test directly. No doubles needed. DONE.

**3B: Code with dependencies — can you extract?** Extract the pure computation from the dependency interaction. Test the pure part at Level 1, test integration at Level 2. DONE. The fake repository was never needed.

**3C: Glue/orchestration code — can't extract.** The behavior IS the interaction with the dependency. Go to Stage 4.

### Stage 4: Can the real system produce the behavior?

| Question                                                   | If YES   | If NO         |
| ---------------------------------------------------------- | -------- | ------------- |
| **Reliably?** (deterministic, not flaky)                   | Continue | Go to Stage 5 |
| **Safely?** (won't charge money, send emails, delete data) | Continue | Go to Stage 5 |
| **Cheaply?** (won't cost $$ or take hours)                 | Continue | Go to Stage 5 |
| **Observably?** (can see what you need to assert)          | Continue | Go to Stage 5 |

If YES to all: Use real system at Level 2. DONE.

### Stage 5: Which exception applies?

Now and only now may you consider test doubles. Match a specific exception.

| Exception                | When                                                                                   | Double Type                           |
| ------------------------ | -------------------------------------------------------------------------------------- | ------------------------------------- |
| 1. Failure modes         | Test behavior under specific failures (timeouts, connection resets, throttling)        | Stub returning predetermined errors   |
| 2. Interaction protocols | Correctness depends on conversation pattern (multi-step workflows, pagination, sagas)  | Spy that records calls                |
| 3. Time and concurrency  | Deterministic control over timing (retries with jitter, token refresh races, debounce) | Fake clock, controllable scheduler    |
| 4. Safety                | Real system is destructive (payment providers, email sending, destructive admin APIs)  | Stub that records but doesn't execute |
| 5. Combinatorial cost    | 100+ scenarios AND hours of runtime with real system                                   | Configurable fake                     |
| 6. Observability         | Verify details the real system can't expose (headers, batching, idempotency keys)      | Spy that records request details      |
| 7. Contract testing      | Third-party API you don't control, verify serialization/parsing                        | Contract stub                         |

**If no exception applies:** Don't use test doubles. Re-examine Stage 3B, test at Level 2, or accept no Level 1 test for this code.

## Test double taxonomy

| Double Type | Purpose                           | Use For                                          |
| ----------- | --------------------------------- | ------------------------------------------------ |
| **Stub**    | Returns predetermined responses   | Exceptions 1 (Failure), 4 (Safety), 7 (Contract) |
| **Spy**     | Records calls for verification    | Exceptions 2 (Interaction), 6 (Observability)    |
| **Fake**    | Simplified working implementation | Exceptions 3 (Time), 5 (Combinatorial)           |
| **Mock**    | Strict expectation verification   | Exception 2 (strict sequence only)               |
| **Dummy**   | Placeholder that's never called   | Satisfying type requirements                     |

**Critical distinction:** Mock (BAD default) = framework intercepts method calls on real objects. Test double (OK when exception applies) = you provide an alternative implementation via dependency injection.

## Four-part test progression

| Phase                   | What You're Testing                      | Confidence Level |
| ----------------------- | ---------------------------------------- | ---------------- |
| 1. Typical cases        | Happy paths, common scenarios            | Baseline         |
| 2. Edge/boundary cases  | Limits, special values, error conditions | Robustness       |
| 3. Systematic coverage  | Loops, state transitions, combinations   | Completeness     |
| 4. Property-based tests | Invariants that hold for all inputs      | Deep correctness |

Not every function needs all four phases. Simple utilities: Phase 1 + 2. Complex algorithms: all four. Glue code: Phase 1 only.

**Property-based tests are MANDATORY for:** parsers (parse(format(x)) === x), mathematical operations, serialization/deserialization, complex algorithms where edge cases are hard to enumerate.

## Cardinal rule

**Mocking is always wrong. There is no exception.** The seven exception cases use test doubles (stubs, spies, fakes), not mocks.
