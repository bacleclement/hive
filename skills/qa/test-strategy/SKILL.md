# test-strategy — Plan Test Approach for New Features

## When to Use
QA Lead uses this when a new feature spec arrives, before implementation begins.

## Inputs
- Feature `spec.md` with user stories and acceptance criteria
- Current architecture (libs, layers, existing test patterns)

## Procedure

1. Read the spec — identify all testable behaviors
2. For each behavior, recommend the test type:
   - **Unit**: Domain aggregates, value objects, domain services
   - **Integration**: Command/query handlers, repository interactions
   - **E2E**: API endpoints, full request-response flows
3. Identify which layers need tests:
   - Domain layer: aggregate invariants, business rules
   - Application layer: handler orchestration, error handling
   - Infrastructure layer: persistence, external service adapters
   - Presentation layer: API contracts, input validation
4. Identify edge cases and error scenarios from the spec
5. Write the test strategy document:

```markdown
## Test Strategy: {feature name}

### Testable Behaviors
1. {behavior} — {test type} — {layer}

### Test Distribution
- Unit: X tests (domain aggregates, value objects)
- Integration: X tests (handlers, repos)
- E2E: X tests (API endpoints)

### Edge Cases
- {scenario}: {how to test}

### Error Scenarios
- {error case}: {expected behavior}

### Notes
- {anything the implementer should know}
```

6. Post to `#daily-standup` for sr-backend reference

## Rules
- Test pyramid: most unit tests, fewer integration, minimal E2E
- Never skip domain layer tests — domain logic must be tested in isolation
- Every aggregate invariant must have a dedicated test
- Strategy is a recommendation, not a mandate — sr-backend can adjust during implementation
