# refactor — Apply Code Refactoring

## When to Use
Sr Backend uses this when code smells are identified during review, implementation, or dedicated cleanup. Applies disciplined refactoring with safety guarantees.

## Inputs
- Identified code smell (duplication, long method, feature envy, god class, wrong layer, poor naming, etc.)
- Affected file(s) and function(s)
- Existing test coverage for the affected code

## Procedure

1. **Identify the smell** — name it specifically (e.g., "duplication between X and Y", "feature envy in Z accessing A's data", "domain logic in infrastructure layer")
2. **Verify test coverage** — run tests for the affected code. If tests do not exist or are insufficient:
   - Write characterization tests that capture current behavior
   - Ensure all tests pass before proceeding
3. **Plan the refactoring** — choose the appropriate technique:
   - Extract Method / Extract Class
   - Move to Correct Layer (domain / application / infrastructure)
   - Rename for Clarity
   - Replace Conditional with Polymorphism
   - Inline unnecessary abstraction
   - Consolidate duplicated logic
4. **Apply the refactoring** — make the structural change without altering behavior
5. **Run all tests** — every test that passed before must still pass
6. **Verify behavior unchanged** — no new test failures, no different outputs, no changed APIs
7. **Commit** with format: `refactor(scope): description`

```bash
# Example commit messages
refactor(domain): extract follow-up scheduling into dedicated service
refactor(infrastructure): move email template logic to application layer
refactor(api): rename getUserData to findProfessionalProfile
```

## Output Format
One commit per refactoring. Commit message follows `refactor(scope): description` format. If multiple smells are found, each gets its own cycle through this procedure.

## Rules
- **Never refactor without tests** — if tests don't exist, write them first (separate commit)
- **Never change behavior during refactoring** — if a bug is found, fix it in a separate commit
- **One refactoring per commit** — atomic changes are reviewable and revertable
- **Run tests after every change** — not at the end, after every single change
- **Do not refactor code you don't understand** — read it, trace it, understand it, then refactor
- **Respect layer boundaries** — refactoring must not violate DDD / Clean Architecture constraints (check ADRs)
