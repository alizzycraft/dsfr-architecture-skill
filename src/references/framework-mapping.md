# Framework Mapping

DSFR is a placement model, not a requirement to build custom primitives.

## General Rule

If the host framework already provides reactive primitives, use them and map them to DSFR roles:

- signals / refs / stores / atoms / observables -> state positions
- computed / memo / selectors -> derived state
- methods / actions / reducers / commands -> entry points or orchestration
- services / query modules / repository adapters / effect handlers -> effects

When the host language is class-oriented and supports static or equivalent non-instance members:

- prefer the class as the domain boundary when it can cleanly own state and behavior
- prefer class-local static pure transforms over file-level helpers when that improves ownership and discoverability
- do not treat file-level co-location as the primary domain boundary if the architecture uses explicit domain classes

## Angular Signals

- `signal()` -> source state
- `computed()` -> derived state
- domain class or service -> ownership boundary
- methods on the domain/service -> entry points
- `private static` methods on the domain/service -> preferred pure transforms when they remain free of instance coupling
- HTTP services or adapter services -> effects

## Solid

- `createSignal()` -> source state
- `createMemo()` -> derived state
- domain module/class -> ownership boundary
- exported actions/functions -> entry points

## Vue

- `ref()` / `reactive()` -> source state
- `computed()` -> derived state
- composable or domain service -> ownership boundary
- explicit actions in composables/services -> entry points

Prefer caution with `reactive()` structures that invite in-place mutation; preserve immutable update discipline at the domain boundary.

## React Ecosystem

- state libraries such as Zustand, Jotai, Redux Toolkit, or custom signal/store layers can map to DSFR roles
- selectors and derived hooks map to derived state
- actions, store methods, or domain services map to entry points
- async thunks, query adapters, or service modules map to effects

In ecosystems that do not rely on classes as the primary ownership boundary, keep using the nearest equivalent domain container. The class-local static transform preference applies only where the language and architecture make that boundary explicit.

## Streaming and Subscription-Heavy Systems

- websocket handlers, event-source listeners, timer loops, and subscription setup belong to effects
- effect callbacks should route into result entry points or orchestrators
- do not let subscription-capable framework primitives bypass the effect boundary

Do not force DSFR terminology into a codebase if the existing primitives already express the same ownership and placement cleanly.
