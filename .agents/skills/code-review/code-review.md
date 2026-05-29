# Code Review Skill

## Intent
Provide standardized, deterministic code review procedures for the Fast-ASDLC framework. This skill enables a dedicated Code Review Agent (or any authorized agent) to audit AI-generated or human-written source code against architectural specs, hexagonal constraints, and quality standards without modifying production code.

## Pre-conditions
- Source code exists in `/src/` following the hexagonal structure defined in `/.agents/fast-asdlc/METHODOLOGY.md`
- Technical specifications are finalized in `/docs/specs/`
- Domain contracts and acceptance criteria are documented in `/docs/domain/usecases/`
- The Programmer Agent rules are accessible at `/.agents/rules/programmer.md`
- The target feature has an active tracking file in `.backlog/`

## Step-by-Step Logic

### Phase 1: Context Assembly
1. **Ingest Specifications:** Read all relevant specs from `/docs/specs/` for the current feature. Extract class names, method signatures, API contracts, and expected behaviors.
2. **Ingest Domain Contracts:** Parse acceptance criteria from `/docs/domain/usecases/` to map business rules against code implementation.
3. **Load Source Code:** Read the candidate files in `/src/` that correspond to the current feature scope. Respect hexagonal layer boundaries:
   - `domain/` — pure business logic
   - `application/` — use cases, ports, handlers
   - `infrastructure/` — adapters, external integrations
   - `bootstrap/` — wiring and DI

### Phase 2: Static Compliance Audit
4. **Hexagonal Architecture Validation:**
   - Verify dependency direction flows inward: `infrastructure` → `application` → `domain`
   - Confirm `domain/` has zero external framework imports (no ORM, no HTTP, no DB driver dependencies)
   - Confirm ports (interfaces) are defined in `application/ports/` and implemented in `infrastructure/`
   - Flag any leakage: domain layer importing infrastructure or application layer bypassing ports

5. **SOLID & OCP Check:**
   - **Single Responsibility:** Each file/class has one reason to change. Flag god-classes or multi-purpose modules.
   - **Open/Closed:** Prefer additive patterns (new files, new implementations) over heavy mutation of existing legacy files.
   - **Dependency Inversion:** High-level modules depend on abstractions (ports), not concrete adapters.
   - **Interface Segregation:** Ports are focused and client-specific, not bloated.

6. **Spec Traceability:**
   - Every public method in `application/` or `domain/` must trace back to a requirement in `/docs/specs/` or `/docs/domain/usecases/`
   - Every API endpoint in `infrastructure/` must match the contract declared in specs
   - Flag orphaned code with no spec coverage

### Phase 3: Quality & Safety Audit
7. **Test Coverage Review:**
   - Verify unit tests exist for all `domain/` entities and services
   - Verify integration tests exist for `application/` handlers
   - Check that tests are in `/tests/` and mirror the `/src/` structure
   - Flag any production code in `/src/` without corresponding test coverage

8. **Security Hygiene:**
   - No hardcoded secrets, tokens, or credentials
   - Input validation is present at application or infrastructure boundaries
   - No SQL injection risks in persistence adapters
   - Proper error handling without information leakage

9. **Code Style & Consistency:**
   - Naming conventions align with ubiquitous language in `/docs/domain/glossary.md`
   - Language policy from `/.agents/memory-bank/project-brief.md` is respected (comments, logs, variable names)
   - No emojis or decorative symbols in code comments (per architect standards)

### Phase 4: Report Generation
10. **Compile Structured Review Report:**
    - Categorize findings by severity: `BLOCKER`, `CRITICAL`, `WARNING`, `NIT`
    - Cross-reference each finding to the violated rule/spec
    - Provide concrete remediation steps, including exact file paths and suggested refactors

11. **Handoff:**
    - Append the review report to the active `.backlog/` feature tracking file
    - Update `/.agents/memory-bank/active-context.md` with review status and next steps
    - If no blockers found, signal readiness for QA phase

## Quality Gates
- [ ] All specs from `/docs/specs/` are mapped to code artifacts
- [ ] Zero hexagonal leakage detected (domain has no external deps)
- [ ] All public domain/application methods have unit tests
- [ ] No hardcoded secrets or security anti-patterns
- [ ] Review report is structured, actionable, and references exact line numbers where possible

## Output Format

### Review Report Template
```markdown
## Code Review Report: [Feature Name]
- **Reviewer:** Code Review Agent
- **Target Branch/Commit:** [reference]
- **Scope:** [list of files reviewed]

### Summary
- **Status:** [PASSED / PASSED_WITH_WARNINGS / FAILED]
- **Blockers:** [count]
- **Critical:** [count]
- **Warnings:** [count]
- **Nits:** [count]

### Findings

#### [BLOCKER] Hexagonal Leakage
- **File:** `src/[container]/domain/[file]`
- **Violation:** Import of `github.com/lib/pq` (DB driver) in domain layer
- **Rule:** `/.agents/fast-asdlc/METHODODOLOGY.md` Section 3.3
- **Remediation:** Move persistence logic to `infrastructure/persistence/`. Define port in `application/ports/`.

#### [CRITICAL] Missing Test Coverage
- **File:** `src/[container]/domain/service.go`
- **Violation:** `CalculateTotal()` has no unit tests
- **Rule:** `/.agents/rules/programmer.md` — 90%+ coverage requirement
- **Remediation:** Add test cases in `tests/domain/service_test.go`

#### [WARNING] Spec Drift
- **File:** `src/[container]/application/handlers/order_handler.go`
- **Violation:** Method `ProcessRefund()` not found in `/docs/specs/order-spec.md`
- **Remediation:** Either update spec or remove orphaned method

### Sign-off
- **Next Action:** [Route to Programmer / Proceed to QA / Await Human Decision]
```

## Interaction Guidelines
- **Communication:** Use the same language the user employs in the current conversation
- **Code Immutability:** You MUST NOT modify, overwrite, or delete any files inside `/src/`, `/tests/`, or `/docs/`
- **Write Access:** You MAY append review reports to `.backlog/` and update `/.agents/memory-bank/` status files only
- **Git Compliance:** Do not execute `git commit`, `git push`, or branch merges

## Integration Points
- **Input from:** Programmer Agent (build readiness signal), Architect Agent (specs)
- **Output to:** Programmer Agent (defects for rework), QA Agent (green light for testing), Human supervisor (approval gate)
- **Rules Dependency:** `/.agents/rules/programmer.md`, `/.agents/rules/architect.md`, `/.agents/fast-asdlc/METHODOLOGY.md`
