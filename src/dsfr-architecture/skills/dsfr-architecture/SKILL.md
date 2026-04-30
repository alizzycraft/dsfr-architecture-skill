---
name: dsfr-architecture
description: Use when designing, modifying, or refactoring domain logic around reactive state, pure transformations, explicit entry points, and isolated IO boundaries. Applies to stateful frontend and application-layer architecture using signals, stores, atoms, observables, or equivalent reactive primitives.
---

# DSFR Architecture

Use this skill when the task involves domain logic, reactive state, feature structure, or architectural placement decisions. Do not use it for isolated utilities, one-off scripts, or low-level helpers that do not participate in domain behavior.

## Mandatory Decision Loop

Complete this sequence before writing code:

```text
1. Identify or confirm domain
2. Check for existing domain ownership
3. Classify work (state / transform / orchestration / effect)
4. Enforce Tier 1 invariants at design level
5. Design or extend signals (source vs derived)
5.5 Confirm the state model is complete before proceeding
6. Extract or define pure transforms
7. Define entry points (state transitions)
8. Isolate or integrate effects
9. Generate code
10. Validate code against Tier rules
11. If validation fails, rewrite before returning
```

No code may be written before steps 1-4 are complete.
The agent must internally classify all relevant logic before writing code, even if that classification is not shown in the final output.

## Domain Selection Rules

- If an existing domain already owns the core state, extend it.
- Do not create a new domain unless responsibility would otherwise become mixed.
- If one domain owns the core state, route the work there.
- If multiple domains are involved, use or create an orchestration domain.
- If no domain can own the behavior cleanly, stop and propose a new domain boundary before coding.
- In class-oriented systems, prefer the class itself as the domain boundary unless the host architecture clearly requires another container.

For detailed routing rules, read [references/domain-selection.md](references/domain-selection.md).

## Required Work Classification

Every relevant piece of logic must be classified as exactly one of:

- `STATE`: domain-owned reactive state
- `TRANSFORM`: pure deterministic function from input to output
- `ORCHESTRATION`: entry point coordinating state transitions
- `EFFECT`: external IO, timers, persistence, network, storage, navigation, or framework-bound side effects

If logic appears to fit multiple classifications, split it until each unit has one classification only.

## Tier 1: System Invariants

These must not be violated.

- A domain entry point is any method or action owned by the domain that is allowed to mutate domain state.
- Entry points are split into:
  - intent entry points: handle user or system requests that initiate behavior
  - result entry points: reconcile success, failure, rollback, or externally resolved outcomes
- No function other than a domain entry point may mutate domain state.
- `.set()` or equivalent mutation is allowed only inside domain-owned entry points.
- Domain-owned mutable application state must live in signals or the host framework's equivalent reactive primitive.
- Signals or equivalent primitives must not be exposed in a writable form across domain boundaries.
- Domains must not mutate other domains.
- Cross-domain reads are allowed only through explicit selectors or read-only APIs, never through raw writable state handles.
- If a domain's core decisions repeatedly depend on another domain's state, prefer orchestration over ad hoc selector dependency chains.
- Effects must not contain business logic.
- Effects may cause state changes only by invoking domain entry points or orchestrators that invoke domain entry points.
- Effect outputs must re-enter the system through entry points or orchestrators.
- State transitions must be explicit and traceable.
- No hidden mutation or implicit side effects.

## Tier 2: Structural Rules

These are strong defaults. Deviation requires clear justification.

- Business logic should be implemented as pure functions.
- Pure functions may be co-located inside the domain container when the language or framework supports doing so without coupling them to instance state.
- In class-oriented systems, prefer class-local static pure transforms over file-level helpers when that preserves explicit domain ownership and discoverability.
- Pure functions must not access instance state, `this`, signals, injected collaborators, or mutable shared state.
- Any function that accesses instance state, `this`, signals, injected collaborators, or writable shared state is not a pure transform and must be classified as orchestration or effect code.
- Source state is stored in signals.
- Derived state is computed from source state and should not be stored unless justified.
- Valid reasons to store derived state:
  - performance constraints
  - required persistence or external contract
  - reactive platform limitation
  - snapshot or history semantics
- Signal values should be replaced, not mutated in place.
- Arrays and objects should be updated immutably.
- Pure functions must not mutate inputs.
- Host-framework mutation affordances do not override DSFR mutation discipline.
- Async logic belongs in effects only.
- Domains receive resolved values and apply them via entry points.
- Long-lived subscriptions, streaming sources, and callback-driven external feeds are effects.
- Subscription callbacks must route through entry points or orchestrators.
- Ownership of subscription setup and teardown must be explicit.
- Entry points coordinate behavior; complex business logic belongs in pure transforms.
- If multiple state updates form one logical transition, apply them atomically where the platform supports it, or stage them so invalid intermediate state is not observable.
- Prefer framework-native reactive primitives only when they clearly map to DSFR roles such as state, derived state, entry points, and effects.
- Do not introduce a new domain, effect, or transform layer unless the task introduces a distinct responsibility.
- Repeated cross-domain read composition without mutation may be implemented as a read-only composition layer that owns no mutable state and does not bypass domain boundaries.

## Tier 3: Conventions

These are advisory unless the project defines stricter local standards.

- Group behavior by domain.
- Use action-based names for entry points.
- Keep naming and structure consistent.
- Prefer discoverability over fragmented placement.

## State Design Procedure

Before implementing behavior:

1. Identify all required state.
2. Classify each value as `source state` or `derived state`.
3. Define signals for source state only.
4. Define derived relationships before implementing orchestration.

No behavior should be implemented before this step is resolved.

## Effect Routing

Effects must follow this path:

```text
external IO -> effect -> domain entry point -> signal update
```

Effects may map transport formats, but they must not contain domain business logic and may cause state changes only by invoking domain entry points or orchestrators.

## Bug Fix and Refactor Rules

- Bug fix: preserve existing domain ownership unless incorrect ownership is the root cause.
- Bug fix: prefer minimal structural change.
- Bug fix in legacy mixed-ownership code: isolate the smallest boundary that prevents further architectural violation, but do not force full DSFR normalization unless requested.
- Refactor: preserve observable behavior unless the task explicitly allows semantic change.
- Refactor: improve structure without silently changing domain meaning.

## Validation Checklist

Before finishing, check:

### Tier 1

- no direct state mutation outside entry points
- no cross-domain mutation
- no writable state leaked across domain boundaries
- no effects causing state changes except through entry points or orchestrators
- no hidden mutation paths

### Tier 2

- pure functions are isolated
- class-local pure transforms do not depend on instance state or injected collaborators
- derived state is not stored without reason
- async is isolated to effects
- no in-place mutation
- multi-step transitions do not leak invalid intermediate state
- repeated cross-domain selector use has not become hidden orchestration

### Tier 3

- naming is consistent
- structure is readable
- domain ownership is obvious

If Tier 1 fails, stop and refactor before returning the result.
If Tier 2 fails, correct it unless there is an explicit justification.
If ambiguity remains after reasonable inspection, ask for clarification instead of guessing.

When storing derived state, explicitly justify:

- why computed or selector-based derivation is insufficient
- what invalidation path keeps the stored value correct
- why the duplication risk is acceptable

## Supporting References

Read these only as needed:

- [references/domain-selection.md](references/domain-selection.md): routing and ownership decisions
- [references/examples.md](references/examples.md): canonical DSFR examples
- [references/anti-patterns.md](references/anti-patterns.md): invalid placements and common failures
- [references/framework-mapping.md](references/framework-mapping.md): mapping DSFR to common reactive stacks
