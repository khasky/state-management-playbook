# State Management Playbook

[![License: MIT](https://img.shields.io/badge/license-MIT-green)](LICENSE) [![Web Reactions](https://api.webreactions.app/badge/github/khasky/state-management-playbook.svg)](https://webreactions.app/?utm_source=github&utm_channel=repository&utm_medium=state-management-playbook)

Practical guide to client state architecture in React: state taxonomy, MobX, store design, and choosing the right state layer.

> *If I were starting a React app today, what would I standardize about state before reaching for any state library at all?*

This playbook is about:
- classifying state before choosing tools;
- keeping server state out of client stores;
- local-first defaults that delay the store decision;
- MobX as a domain-state engine, with its sharp edges named;
- store design that survives growth;
- the URL as a first-class state layer;
- testing state without rendering anything.

---

## Table of Contents

- [State Management Playbook](#state-management-playbook)
  - [Table of Contents](#table-of-contents)
  - [Companion playbooks](#companion-playbooks)
  - [Philosophy](#philosophy)
  - [The state taxonomy](#the-state-taxonomy)
    - [Server state](#server-state)
    - [UI state](#ui-state)
    - [Session state](#session-state)
    - [Domain state](#domain-state)
  - [The defaults I'd reach for first](#the-defaults-id-reach-for-first)
  - [The decision model](#the-decision-model)
  - [MobX in depth](#mobx-in-depth)
    - [Observable domain models](#observable-domain-models)
    - [Computed is the workhorse](#computed-is-the-workhorse)
    - [Actions and strict mode](#actions-and-strict-mode)
    - [Reactions, sparingly](#reactions-sparingly)
    - [When MobX shines](#when-mobx-shines)
    - [MobX pitfalls](#mobx-pitfalls)
  - [Store design rules](#store-design-rules)
  - [State and the URL](#state-and-the-url)
  - [State in monorepos and cross-platform](#state-in-monorepos-and-cross-platform)
  - [Testing state](#testing-state)
  - [Things to avoid](#things-to-avoid)
  - [Worth reading](#worth-reading)
  - [License](#license)

---

## Companion playbooks

These repositories form one playbook suite:

- [AI-Assisted Engineering Playbook](https://github.com/khasky/ai-assisted-engineering-playbook) — agent workflows, guardrails, and quality control for AI-heavy teams
- [API Design Playbook](https://github.com/khasky/api-design-playbook) — versioning, pagination, idempotency, error contracts, and webhooks
- [Auth & Identity Playbook](https://github.com/khasky/auth-identity-playbook) — sessions, tokens, OAuth, and identity boundaries across the stack
- [Backend Architecture Playbook](https://github.com/khasky/backend-architecture-playbook) — APIs, boundaries, OpenAPI, persistence, and errors
- [Best of JavaScript](https://github.com/khasky/best-of-javascript) — curated JS/TS tooling and stack defaults
- [Caching Playbook](https://github.com/khasky/caching-playbook) — HTTP, CDN, and application caches; consistency and invalidation
- [Code Review Playbook](https://github.com/khasky/code-review-playbook) — PR quality, ownership, and review culture
- [CTO Playbook](https://github.com/khasky/cto-playbook) — org design, hiring, technical strategy, budgets, and due diligence
- [DevOps Delivery Playbook](https://github.com/khasky/devops-delivery-playbook) — CI/CD, environments, rollout safety, and observability
- [Engineering Lead Playbook](https://github.com/khasky/engineering-lead-playbook) — standards, RFCs, and technical leadership habits
- [Frontend Architecture Playbook](https://github.com/khasky/frontend-architecture-playbook) — React structure, performance, and consuming API contracts
- [Git Collaboration Playbook](https://github.com/khasky/git-collaboration-playbook) — branching, stacked PRs, merge queues, and CI collaboration at scale
- [Marketing and SEO Playbook](https://github.com/khasky/marketing-and-seo-playbook) — growth, SEO, experimentation, and marketing surfaces
- [Messaging & Async Playbook](https://github.com/khasky/messaging-and-async-playbook) — queues, events, outbox, idempotent consumers, and retries
- [Monorepo Architecture Playbook](https://github.com/khasky/monorepo-architecture-playbook) — workspaces, package boundaries, and shared code at scale
- [Node.js Runtime & Performance Playbook](https://github.com/khasky/nodejs-runtime-performance-playbook) — event loop, streams, memory, and production Node performance
- [Observability Playbook](https://github.com/khasky/observability-playbook) — logs, traces, metrics, SLOs, and production visibility
- [React Cross-Platform Playbook](https://github.com/khasky/react-cross-platform-playbook) — shared React UI and logic across web and native with TypeScript
- [Software Design Playbook](https://github.com/khasky/software-design-playbook) — separation of concerns, composition, and module boundaries
- **State Management Playbook** — client state architecture, MobX, and choosing a state layer
- [Styling Architecture Playbook](https://github.com/khasky/styling-architecture-playbook) — type-safe styling, design tokens, and theming at scale
- [Sysadmin Operations Playbook](https://github.com/khasky/sysadmin-operations-playbook) — servers, backups, DNS, TLS, identity, and the self-hosted ops stack
- [Testing Strategy Playbook](https://github.com/khasky/testing-strategy-playbook) — unit, integration, contract, E2E, and CI-friendly test layers

---

## Philosophy

Most "state management problems" are not state management problems.

They are:
- server data copied into a client store and drifting out of date;
- UI state hoisted globally because lifting felt tedious once;
- derived values stored instead of computed;
- two sources of truth being synced by effects.

The library matters less than the taxonomy.
Classify the state first; the tool choice usually falls out on its own.

---

## The state taxonomy

Every piece of state in a React app is one of four kinds. Mixing them is where the pain starts.

### Server state

Data owned by the backend: entities, lists, search results. The client only ever holds a **cache** of it.

- [TanStack Query](https://tanstack.com/query/latest) owns this layer: fetching, caching, invalidation, deduplication, retries;
- never copy query results into a client store — the copy is a second source of truth with no invalidation story;
- "stale" is a feature, not a bug: model freshness explicitly with `staleTime`.

How the data-fetching layer plugs into the app is covered in the [Frontend Architecture Playbook](https://github.com/khasky/frontend-architecture-playbook).

### UI state

Open/closed, hover, focus, input drafts, active tab.

- local first: `useState` in the component that owns it;
- most UI state never needs to leave the component that created it;
- promotion to a parent is fine; promotion to a global store is almost never needed.

### Session state

Current user, permissions, feature flags, locale, theme.

Small, read-mostly, changes at login/logout boundaries. One tiny context or store is enough. It rarely grows, so do not design for growth here.

### Domain state

Client-owned models that outlive any single component: an editor document, a cart, a multi-step wizard, a canvas scene, an offline draft.

This is the only category that earns a real store. If an app has no domain state, it does not need a state management library.

---

## The defaults I'd reach for first

If I were building a React app today, I would escalate in this order and stop at the first level that holds:

- **`useState` + derived values** — compute during render, no extra state
- **Lift state** — to the nearest common parent when two components genuinely share it
- **TanStack Query** — for anything that came from the network, no exceptions
- **URL as state** — for anything a user would want to share or refresh into
- **A small store** — only when sharing is real, lifting hurts, and the state is domain state

Most apps that follow this order end up with TanStack Query, a router, and one or two small stores. That is a healthy endpoint, not a missing architecture.

---

## The decision model

When a store is actually warranted:

| Layer | Good when | Watch out for |
| --- | --- | --- |
| `useState` + context | Session-shaped, read-mostly values; small apps | Context re-renders every consumer on change; hot values do not belong here |
| Zustand | Small shared stores; minimal API; selector-based subscriptions | Everything-in-one-store drift; logic scattered across components instead of the store |
| MobX | Rich interconnected domain models; OOP-style logic; editors and complex forms | Implicit tracking surprises; over-observing; discipline required (strict mode) |
| Redux Toolkit | Large teams wanting one enforced pattern; serializable actions; time-travel debugging | Ceremony for small apps; boilerplate gravity; server state accidentally living in slices |
| Jotai / signals | Fine-grained reactive graphs of small independent atoms | Atom sprawl; dependency graphs that no one can read after a year |

My preference: TanStack Query plus Zustand covers most apps; MobX earns its place when the domain logic itself is the hard part.

### Where the default came from

Flux was Facebook's answer to a specific bug: the unread-message counter kept going wrong because several parts of the app could write the same state. Redux (Dan Abramov, 2015) was then written for a React Europe talk on hot reloading and time-travel debugging — the strict single store and serializable actions exist because *those* are what make a state history replayable.

Dan Abramov published "You Might Not Need Redux" the following year. The library's own author, within a year of it becoming the industry default, arguing that most apps adopting it did not have the problem it solved.

Two things follow. The constraints in the Redux Toolkit row are not ceremony for its own sake — they buy the debugging property, and if you do not use that property you are paying the price without collecting. And a library becoming a default is not evidence it fits your app; Redux got there by being an excellent answer to a question most teams were not asking.

---

## MobX in depth

MobX is a reactive dependency graph, not a dictionary you put things in. Used with discipline, it is the best tool I know for rich client-side domain models.

### Observable domain models

- model domain concepts as classes with `makeAutoObservable(this)` in the constructor;
- keep behavior next to data: `cart.addItem(sku)` beats a reducer three files away;
- references between models are fine — MobX tracks through them.

### Computed is the workhorse

- everything derivable is a `computed`: totals, validity, visible subsets, selection summaries;
- computeds are cached and only recalculate when tracked dependencies change;
- a store with many computeds and few observables is usually a well-designed store.

### Actions and strict mode

- turn on `enforceActions: "always"` from day one;
- every mutation goes through an action; strict mode makes the write paths findable;
- async flows: mutate only inside actions or `runInAction`, never in a bare `await` continuation.

### Reactions, sparingly

- `reaction` and `autorun` are for side effects at the edge: persistence, analytics, syncing to storage;
- deriving state with a reaction is a smell — that is what `computed` is for;
- every reaction is a hidden subscription; each one needs an owner and a disposal path.

### When MobX shines

- rich interconnected domain logic where changing one thing legitimately affects five others;
- OOP-style models with invariants and behavior;
- editors, canvas tools, spreadsheets, complex multi-step forms;
- code where "recompute exactly what changed" matters for both correctness and performance.

### MobX pitfalls

- **Implicit tracking surprises** — dereference an observable outside an `observer` component or tracked context and nothing updates; the failure is silent;
- **Over-observing** — making huge arrays or deep structures observable when only a summary is read; use `observable.shallow` or plain data plus one computed;
- **Giant root stores** — one `RootStore` that knows everything becomes the god object; keep stores per domain;
- **Mutating in render** — render must be a pure read; strict mode catches this, which is another reason to enable it.

---

## Store design rules

These rules keep stores boring, which is the goal:

1. **Normalize or keep small.** Nested duplicated entities rot; either normalize by id or keep the store small enough that it cannot rot.
2. **Derive, don't duplicate.** If a value can be computed from other state, compute it. Stored derivations desynchronize; computed ones cannot.
3. **Unidirectional flow, even in MobX.** Components call actions; actions mutate; observation flows back down. No component writes an observable directly.
4. **Stores per domain, not one god store.** `CartStore`, `EditorStore`, `SessionStore` — composed at the root, owned separately.
5. **No server cache inside client stores.** TanStack Query already is that cache. A store holding fetched entities is a bug waiting for an invalidation.

Store boundaries are module boundaries; the reasoning behind them is the [Software Design Playbook](https://github.com/khasky/software-design-playbook)'s territory.

---

## State and the URL

The URL is a state layer, and it is the only one users can see, share, and bookmark.

Belongs in the URL:
- filters and search terms;
- active tab and sort order;
- current selection or open record;
- pagination position.

The test is simple: if a user refreshes or sends the link to a colleague, should the view survive? If yes, the state lives in the URL and the router owns it — not a store, and not both.

---

## State in monorepos and cross-platform

When web and native share logic, the store is exactly the thing worth sharing.

- keep stores **headless**: plain TypeScript classes or vanilla stores in a shared package, zero React imports;
- UI bindings live per platform: `observer` components, hooks, and selectors stay in the app packages;
- session and domain logic written once this way runs unchanged on web and native.

The package layout and binding patterns are covered in the [React Cross-Platform Playbook](https://github.com/khasky/react-cross-platform-playbook).

---

## Testing state

The payoff of headless stores is that state logic tests without a DOM.

- test stores as plain classes and functions: construct, call actions, assert on state and computeds;
- no rendering, no providers, no `act()` — these tests are fast and do not flake;
- render tests exist only to prove the wiring: one test that the component observes the store, not fifty that re-test store logic through the UI.

If store logic can only be tested by rendering, the store is coupled to the view — fix the store, not the test.

---

## Things to avoid

- context as a state manager for hot, frequently-changing values;
- prop-drilling phobia — passing a prop two levels is fine and honest;
- a global store as the default home for every new piece of state;
- syncing two sources of truth instead of deleting one of them;
- effects that mirror state (`useEffect` copying props or query data into `useState`);
- storing derived values that a `computed` or a render-time expression would keep correct for free.

---

## Worth reading

- [The gist of MobX](https://mobx.js.org/the-gist-of-mobx.html)
- [Defining data stores (MobX)](https://mobx.js.org/defining-data-stores.html)
- [TanStack Query overview](https://tanstack.com/query/latest/docs/framework/react/overview)
- [Zustand documentation](https://zustand.docs.pmnd.rs)
- [Redux Toolkit documentation](https://redux-toolkit.js.org)
- [Jotai documentation](https://jotai.org)

---

## License

MIT is a sensible default for a repository like this, but choose the license that fits how you want others to reuse the material.
