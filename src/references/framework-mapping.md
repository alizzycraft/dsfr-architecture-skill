# Framework Mapping

DSFR is a placement model, not a requirement to build custom primitives.

## General Rule

If the host framework already provides reactive primitives, use them and map them to DSFR roles:

- signals / refs / stores / atoms / observables -> state positions
- computed / memo / selectors -> derived state
- methods / actions / reducers / commands -> entry points or orchestration
- services / query modules / repository adapters / effect handlers -> effects

## Angular Signals

- `signal()` -> source state
- `computed()` -> derived state
- domain class or service -> ownership boundary
- methods on the domain/service -> entry points
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

Do not force DSFR terminology into a codebase if the existing primitives already express the same ownership and placement cleanly.
