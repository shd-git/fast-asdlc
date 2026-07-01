# Task Compliance & Traceability Check Skill

## Intent
Verify that implemented code fully satisfies the original intent, acceptance criteria, and technical specifications of the active backlog item. Detect orphaned code (no requirement), missing implementation (no code), and untested acceptance criteria. This skill is strictly stack-agnostic — it validates traceability and completeness, not language-specific idioms.

## Pre-conditions
- Active feature tracking file exists in `.backlog/` (e.g., `.backlog/004-auth-refactor.md`)
- Acceptance criteria are documented in `docs/domain/usecases/[feature].md` or inline in the backlog file
- Technical specifications exist in `docs/specs/[feature].md`
- Source code exists in `/src/`
- Tests exist in `/tests/`
- The Meta-Agent or human has declared which backlog item is active

## Step-by-Step Logic

### Phase 1: Ingest Intent & Contracts
1. **Read Backlog Item:** Load the active `.backlog/[feature].md`. Extract:
   - Feature title and intent
   - Scope boundaries (what is IN scope, what is OUT of scope)
   - Acceptance criteria (if listed inline)
   - Linked use-case and spec file paths

2. **Read Use Cases:** Load `docs/domain/usecases/[feature].md`. Extract structured acceptance criteria:
   - AC identifiers (e.g., AC1, AC2, AC3)
   - Conditions (Given/When/Then or explicit statements)
   - Priority (Must/Should/Could) if present

3. **Read Technical Specs:** Load `docs/specs/[feature].md`. Extract:
   - Public API endpoints, methods, or commands
   - Domain events and handlers
   - Database schema changes
   - File/class names explicitly named in the spec

### Phase 2: Map Implementation
4. **Scan Source Code:** Read all new or modified files under `/src/` for this feature. Map each file to the nearest acceptance criterion based on:
   - Filename matching (e.g., `CancelOrderHandler.cs` → AC2)
   - Spec references (e.g., spec says `OrderRepository.Cancel()` → find `OrderRepository`)
   - Domain language alignment (ubiquitous language from glossary)

5. **Scan Tests:** Read all new or modified files under `/tests/`. Map each test file/class to the AC it validates.

### Phase 3: Build Traceability Matrix
6. **Cross-Reference:** Produce a matrix where each row is one acceptance criterion:

   ```
   AC | Use Case | Spec Reference | Source Files | Test Files | Status
   ```

   Status rules:
   - `IMPLEMENTED` — source file(s) found and appear to cover the AC
   - `TESTED` — test file(s) found and appear to cover the AC
   - `MISSING` — no source file mapped to this AC
   - `UNTESTED` — source exists, but no mapped test file
   - `ORPHANED` — not applicable (used for code, not AC)

### Phase 4: Detect Anomalies
7. **Orphaned Code Detection:** Identify any new files, classes, or public methods in `/src/` that:
   - Are not referenced in the spec
   - Do not map to any acceptance criterion
   - Are outside the declared scope in the backlog item
   Flag as `ORPHANED` with recommendation to remove or document.

8. **Missing Implementation Detection:** Identify any acceptance criterion that:
   - Has no mapped source file
   - Has a mapped source file but the expected method/event/endpoint is not found
   Flag as `MISSING`.

9. **Untested Criteria Detection:** Identify any acceptance criterion that:
   - Is implemented in `/src/`
   - Has no corresponding test coverage in `/tests/`
   Flag as `UNTESTED`.

### Phase 5: Scope Creep Check
10. **Boundary Validation:** Compare the actual changes against the backlog scope. Flag any implementation that:
    - Addresses a different feature number
    - Implements functionality marked as OUT OF SCOPE
    - Introduces new bounded contexts not mentioned in the feature intent

## Quality Gates
- [ ] Every AC from use cases is either IMPLEMENTED or flagged MISSING
- [ ] Every IMPLEMENTED AC has corresponding TESTED or UNTESTED status
- [ ] No ORPHANED code exists without human-approved justification
- [ ] No OUT OF SCOPE functionality is present in `/src/`
- [ ] Spec-declared files/classes exist in `/src/`

## Output Format

### Compliance Report Template
```markdown
## Compliance Report: [Feature ID] — [Feature Title]
- **Backlog:** `.backlog/[feature].md`
- **Use Cases:** `docs/domain/usecases/[feature].md`
- **Specs:** `docs/specs/[feature].md`
- **Reviewer:** Task Compliance Agent
- **Verdict:** [PASS | PARTIAL | FAIL]
- **Routing:** [QA | PROGRAMMER | HUMAN_GATE]

### Summary
- Total ACs: N
- Implemented: N | Missing: N
- Tested: N | Untested: N
- Orphaned code: N files | Scope creep: N items

### Traceability Matrix

| AC | Use Case | Spec Ref | Source Files | Test Files | Status |
|----|----------|----------|--------------|------------|--------|
| AC1 | User cancels own order | `CancelOrderHandler` | `src/.../CancelOrderHandler.cs` | `tests/.../CancelOrderTests.cs` | TESTED |
| AC2 | Refund initiated | `RefundService` | `src/.../RefundService.cs` | — | UNTESTED |
| AC3 | Admin cancels any | `AdminCancelHandler` | — | — | MISSING |

### Findings

#### [MISSING] AC3 — Admin cancels any order
- **Spec:** `docs/specs/auth-refactor.md` §4.2
- **Expected:** `AdminCancelOrderCommand` + `AdminCancelOrderHandler`
- **Found:** No matching files in `src/`
- **Fix:** Implement handler in `src/Orders/Application/Handlers/AdminCancelOrderHandler.cs`

#### [UNTESTED] AC2 — Refund initiated automatically
- **Source:** `src/Orders/Domain/RefundService.cs`
- **Missing:** Unit tests for `RefundService.ProcessRefund()`
- **Fix:** Add `tests/Orders/Domain/RefundServiceTests.cs`

#### [ORPHANED] `LegacyRefundCalculator.cs`
- **File:** `src/Orders/Domain/LegacyRefundCalculator.cs`
- **Issue:** Not referenced in spec or any AC
- **Fix:** Remove or add AC + spec coverage

#### [SCOPE_CREEP] Notification service for SMS
- **File:** `src/Notifications/SmsGateway.cs`
- **Issue:** SMS gateway not in backlog scope (scope = email only)
- **Fix:** Move to separate feature branch or update backlog scope

### Sign-off
- **Next Action:** [Route to Programmer for missing items / Route to QA if only untested / Await Human if scope creep]
```

## Interaction Guidelines
- **Communication:** Use the same language the user employs in the current conversation
- **Code Immutability:** You MUST NOT modify, overwrite, or delete any files inside `/src/`, `/tests/`, or `/docs/`
- **Write Access:** You MAY append compliance reports to `.backlog/` and update `/.agents/memory-bank/` status files only
- **Git Compliance:** Do not execute `git commit`, `git push`, or branch merges

## Integration Points
- **Input from:** Analyst Agent (use cases), Architect Agent (specs), Programmer Agent (build readiness)
- **Output to:** Programmer Agent (missing/orphaned items), QA Agent (green light if fully tested), Human supervisor (scope creep decisions)
- **Runs before:** Code Review Agent (language-specific audit) — compliance check gates the feature before style review
