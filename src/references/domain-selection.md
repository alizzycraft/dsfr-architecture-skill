# Domain Selection

Use this reference when ownership is unclear or when the task touches existing code.

## Ownership Rules

- Extend an existing domain if it already owns the core state and behavior.
- Create a new domain only when adding the behavior to an existing domain would mix unrelated responsibility.
- Prefer one clear owner over splitting closely related state across multiple domains.

## Routing Questions

Answer these in order:

1. What state changes because of this feature or fix?
2. Which domain already owns that state?
3. Is the requested behavior business logic, coordination logic, or IO?
4. Does this require one domain, or coordination between multiple domains?

## Routing Outcomes

### Single-domain outcome

Use the existing domain when:

- one domain owns the source state
- the behavior is a direct extension of that domain's responsibility
- no additional coordinating boundary is needed

### Orchestration outcome

Use or create an orchestration domain when:

- multiple domains participate in one user-facing workflow
- one step in Domain A leads to a reaction in Domain B
- the behavior is coordination, not ownership

The orchestrator may call entry points on multiple domains, but it must not bypass their boundaries.

### New-domain outcome

Propose a new domain when:

- no current domain can own the new state without becoming mixed-purpose
- the feature introduces a new durable business capability
- the existing structure would force cross-domain mutation or hidden ownership

## Existing Codebase Rules

- Prefer extension over creation.
- Reuse existing selectors, entry points, and effects when they already match the needed responsibility.
- Do not add a new layer just to mirror DSFR terminology if the existing code already expresses the same role cleanly.

## Cross-Domain Access

Moderate mode applies by default:

- read-only access is allowed through explicit selectors or public read APIs
- writable signal handles must not cross boundaries
- all writes must go through the target domain's entry points

## Stop Conditions

Stop and ask for clarification when:

- two or more ownership models are equally plausible
- the task implies behavior change but the target domain is disputed
- the existing code hides ownership behind shared mutable structures
