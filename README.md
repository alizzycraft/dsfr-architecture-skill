# DSFR Architecture Skill

This skill encodes **Domain-Scoped Functional Reactivity (DSFR)** as an agent-executable architectural protocol.

It exists to solve a practical problem: agents are good at producing working code, but they drift when architectural placement rules are implicit. DSFR turns those rules into a repeatable decision loop so the agent has to answer the same questions every time:

- What domain owns this?
- Is this state, transformation, orchestration, or effect code?
- What is allowed to mutate here?
- Should this be stored, derived, or routed elsewhere?

The result is not just "follow good architecture." It is a constrained placement model that reduces structural drift across features, fixes, and refactors.

## Why This Skill Exists

This skill is built around a hybrid architectural preference:

- **reactive state** for explicit, inspectable state positions
- **pure transformations** for deterministic business logic
- **class- or domain-owned boundaries** for discoverability and cohesion

It is not strict functional programming, and it is not classic object-oriented programming.

The goal is to combine:

- the predictability of pure functions
- the clarity of explicit reactive state
- the ergonomics of strong domain boundaries

In class-oriented systems, that means the **class itself** is often the preferred domain boundary, and pure transforms may live **inside the domain class** as static or equivalent non-instance members. This is an intentional design choice: ownership and discoverability matter more here than strict FP-style file-level placement.

## What the Skill Enforces

The skill makes agents work through a fixed architectural loop before writing code:

1. Identify the domain
2. Confirm ownership
3. Classify the work
4. Enforce invariants
5. Design state first
6. Extract pure transforms
7. Define entry points
8. Isolate effects
9. Generate code
10. Validate and rewrite if needed

This prevents common failure modes:

- hidden mutation
- business logic buried in effects
- cross-domain coupling
- derived state duplication
- orchestration leaking across boundaries

## Core Model

DSFR separates domain logic into four roles:

- **State**: domain-owned reactive state positions
- **Transform**: pure deterministic logic
- **Orchestration**: entry points coordinating legal state transitions
- **Effect**: external IO and async boundaries

The most important constraint is simple:

> State changes only through domain entry points.

Everything else follows from that.

## Why the Skill Is Opinionated

This skill is intentionally more specific than a generic architecture guide.

A vague principle like "keep things clean" does not help an agent much. The skill instead defines:

- what belongs where
- what is forbidden
- what gets designed first
- how cross-domain interactions are routed
- how to validate the result before returning it

That specificity is the point. The skill is meant to improve consistency over time, not just produce locally reasonable code.

## Angular Compatibility

The skill is not Angular-specific, but modern Angular is a strong fit:

- `signal()` maps naturally to source state
- `computed()` maps naturally to derived state
- classes/services work well as domain boundaries
- static class methods work well for class-local pure transforms

The same model also applies to other class-oriented or reactive systems, as long as the host framework's primitives can be mapped cleanly to DSFR roles.

## File Layout

- [SKILL.md](./src/SKILL.md): the agent-executable protocol
- [references/domain-selection.md](./src/references/domain-selection.md): ownership and routing decisions
- [references/examples.md](./src/references/examples.md): concrete placement examples
- [references/anti-patterns.md](./src/references/anti-patterns.md): common failure modes
- [references/framework-mapping.md](./src/references/framework-mapping.md): mapping DSFR onto real stacks

## Intended Use

Use this skill when working on:

- stateful frontend features
- application-layer domain logic
- reactive architecture
- feature refactors where ownership and boundaries matter

Do not use it for:

- isolated utility functions with no domain ownership
- disposable scripts
- low-level code that does not participate in domain behavior

## In Short

This skill exists to turn a hybrid architectural preference into an operational protocol an agent can actually follow.

It treats architecture less like documentation and more like a placement system:

- determine ownership
- classify behavior
- enforce boundaries
- build in a fixed order
- validate before returning
