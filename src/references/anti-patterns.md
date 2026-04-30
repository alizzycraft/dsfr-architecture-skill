# Anti-Patterns

These patterns violate DSFR or usually indicate poor placement.

## Hidden State Mutation

Avoid:

- mutating signal-backed arrays or objects in place
- exposing writable references that allow callers to mutate state indirectly
- updating domain state from helper functions that are not entry points

## Business Logic Inside Effects

Avoid:

- embedding pricing, eligibility, validation, or policy rules in API modules
- mixing domain decisions with transport concerns
- letting async handlers decide business outcomes that should be pure transforms

## Cross-Domain Mutation

Avoid:

- Domain A calling `.set()` on Domain B state
- passing writable signal handles across domain boundaries
- bypassing Domain B entry points because "it is simpler"

## Stored Derived State Without Justification

Avoid:

- storing totals, labels, filtered collections, or flags that can be computed cheaply
- duplicating source-derived relationships because it seems convenient

Store derived state only when there is a concrete reason such as performance, external contracts, platform limits, or history semantics.

## Utility Graveyards

Avoid:

- moving domain-specific business rules into generic helpers just to keep classes smaller
- creating "utils" files that erase ownership and make discovery harder

If the logic belongs to one domain, keep it in that domain's boundary, typically as pure transforms in the same module or domain folder.

## Orchestration Bloat

Avoid:

- stuffing business math into entry points
- turning orchestrators into second domains
- combining ownership, coordination, and IO in one method

Entry points should coordinate transitions. Heavy business logic belongs in pure transforms.
