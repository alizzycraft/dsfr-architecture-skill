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

## Selector Dependency Webs

Avoid:

- letting one domain repeatedly pull decision-critical data from several other domains
- creating cross-domain read chains that act like hidden orchestration
- using selectors as a loophole to avoid creating an orchestrator

If a domain's core behavior depends on several external reads, move that coordination into an orchestrator or read-only composition layer.

## Stored Derived State Without Justification

Avoid:

- storing totals, labels, filtered collections, or flags that can be computed cheaply
- duplicating source-derived relationships because it seems convenient

Store derived state only when there is a concrete reason such as performance, external contracts, platform limits, or history semantics.

## Observable Invalid Intermediate State

Avoid:

- sequentially updating several signals when observers can see a broken intermediate combination
- leaking half-applied transitions during optimistic updates, rollback flows, or staged writes

If one user action represents one logical transition, apply it atomically where possible or stage the update so invalid intermediate state is not observable.

## Subscription Leakage

Avoid:

- letting domains subscribe directly to sockets, timers, or external streams
- hiding stream setup or teardown inside unrelated helpers
- mutating domain state from subscription callbacks without explicit entry points

Subscriptions are effects and must route through entry points or orchestrators.

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
