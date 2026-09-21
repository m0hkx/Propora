# 05 · Technical Interview Prep

Questions an interviewer is likely to ask about this project, with answers grounded in the
actual code. Where a weakness exists, the honest answer is written out — those tend to score
better than a defensive one.

---

## The 90-second project pitch

> "Propora is a property-management dashboard — properties, units, tenants, leases, payments,
> maintenance and documents. React 19 + TypeScript + Vite, Tailwind v4 for styling, Zustand
> for state, React Router for navigation.
>
> The interesting constraint is that it's frontend-only: no backend, no database. That pushed
> two decisions. First, the Zustand store acts as the service layer — every important write
> goes through a guard function that can reject it, so validation isn't only cosmetic form
> checking. Second, anything that would normally be a scheduled backend job — rent generation,
> overdue transitions — is written as pure planner functions that take an explicit `today`,
> with idempotent event-driven triggers on app boot and page mount. If a backend arrived
> tomorrow, those planners would move server-side unchanged."

---

## Architecture

**Q: Why Zustand instead of Redux Toolkit or Context?**

The app is one user, one session, synchronous writes, no server cache. Redux's middleware,
action/reducer indirection and devtools ceremony would buy nothing here. Context was rejected
for a different reason: a single context holding the whole domain re-renders every consumer
on any write — with ~128 tenants and ~67 maintenance rows in live tables, that matters.
Zustand gives per-selector subscriptions, which is the one thing actually needed. If the app
grew real server state I'd add React Query alongside it, not replace it — they solve different
problems (server cache vs. client state).

**Q: Why individual selectors instead of destructuring the store?**

```ts
const tenants = useStore((s) => s.tenants);   // ✅ re-renders only when tenants changes
const { tenants } = useStore();               // ❌ re-renders on every store write
```

`useStore()` with no selector subscribes to the whole state object, and since every mutator
returns a new object, every component would re-render on every toast. It's a project-wide
convention, not a per-case optimisation.

**Q: How did you decide what to persist?**

`partialize` persists exactly three slices: `units`, `maintenance`, `staff` — the ones holding
records a user creates or changes. The rest is demo seed data that intentionally resets, so
the demo can't drift into a broken state with no reset button. A nice property of the choice:
because `partialize` only ever *added* keys, no version migration was needed — older persisted
blobs merge in and simply leave the new slices at seed values.

If this were real, I'd persist nothing and let the server own it; localStorage here is standing
in for a database.

**Q: Why does `validateMaintenanceTarget` live in `lib/` rather than next to the page?**

Layering. `state/store.ts` imports from `lib/` and `data/` only — never from `pages/`. The
store needs that guard, so it belongs in `lib/`. The display-only helpers for the same feature
(`scopeLabel`, `tenantsLabel`, the search haystack) stayed in
`pages/Maintenance/maintenanceUtils.ts`. The split is "who needs to call it", not "what feature
is it about".

**Q: Why are the six create modals in `App.tsx` instead of their pages?**

They're opened from the shared header action button, whose label is derived from the route.
The pattern: the modal collects a **draft** and hands it back; `App.tsx` generates the id,
stamps dates, seeds history and calls the store, then toasts and (sometimes) pushes a
notification. The modal stays free of store access and id logic, which is why the same
`AddPropertyModal` serves both add and edit with just a `mode` prop.

Page-local modals (unit form, staff form, details) deliberately call the store directly —
they're not part of the header flow, and routing them through `App.tsx` would be indirection
for its own sake.

---

## Data modelling

**Q: Walk me through the maintenance scope design.**

A request can target the whole property, specific units, or specific tenants. I modelled that
as a discriminant plus two arrays:

```ts
scope: 'property' | 'units' | 'tenants';
unitIds: string[];    // only when scope === 'units'
tenantIds: string[];  // only when scope === 'tenants'
```

The alternative — nullable `unitId`/`tenantId` fields — can't express "multiple", and an
entire-property request would have had to invent a placeholder unit row, which is exactly the
kind of fake data that poisons reporting later. With a discriminant, the *absence* of ids is
meaningful rather than missing, and invalid combinations (a unit from another property, a
request with both units and tenants) are a type-plus-validation concern rather than a
convention nobody enforces.

It also made the UI honest: the form is a three-way tab control, and switching property or
scope clears both arrays, so a Property-A unit can't survive into Property B.

**Q: What's the `unit` + `unitId?` pair about?**

Legacy tolerance. Tenants, leases and documents predate the Unit registry, so they carry a
free-text label *and* an optional FK. Every matcher honours the id first and falls back to the
label (`tenantOccupiesUnit`). It's the migration pattern you'd use against a real database
too: add the FK, backfill what you can, keep reading the old column until the backfill is
complete. The cost is that every matcher has two branches, which I documented in one place
(`lib/units.ts` header) rather than re-explaining per call site.

**Q: Unit type is "2 BR" and there's also a `bedrooms: number`. Isn't that redundant?**

It was, and it let the two disagree. The fix was to make `type` authoritative:
`bedroomsForType()` maps `Studio → 0`, `1 BR → 1` … and returns `undefined` for `'Other'` —
the one type that genuinely doesn't imply a count. The form derives and disables the Bedrooms
input for every other type, with a caption explaining why. `bedrooms` stays on the entity
because it's the numeric value other code reads; it just stopped being a second source of
truth.

The alternative — deleting `bedrooms` entirely and parsing the label — would have made every
consumer do string parsing, and `'Other'` would have had nowhere to store a count.

**Q: Why is unit occupancy computed instead of stored?**

`status` on a unit is a stored *fallback*; `resolveUnitStatus(unit, tenants, leases)` overrides
it to `Occupied` whenever a live tenant or lease links to the unit. Storing it would mean
keeping it in sync on every tenant move-in, move-out, lease expiry and deletion — five write
paths that can each forget. Deriving costs an O(n) scan on render, which at this data size is
free, and the value can never be stale.

**Q: How do you handle deletes?**

Blocked, never cascaded. `getUnitBlockers` and `getStaffBlockers` return a list of human-
readable reasons ("Assigned to tenant Amelia Hart", "Has open maintenance request M-204"), and
the UI shows the blocking records instead of a confirm dialog. Silent cascade deletes are how
you lose audit history; making the user resolve dependents first keeps referential integrity
without a database enforcing it.

---

## Business logic

**Q: How does the rent automation work without a backend?**

Three pieces:

1. **Pure planners** (`planMonthlyPayments`, `planOverdueTransitions`, `planAutomationRun`)
   that take an explicit `today` and touch nothing — no clock, no DOM, no store.
2. **One applier** in the store that takes a plan and does at most one `set()` — and none at
   all when there's nothing to do.
3. **Event-driven triggers** instead of timers: app boot and Payments page mount. A browser
   tab can't run a cron job, so the model is "reconcile whenever the app is opened", which is
   how offline-first apps handle this.

Because every run is idempotent, running it twice on boot and mount is harmless.

**Q: How is idempotency guaranteed?**

Two layers. Generated payments get a deterministic id — `PAY-<leaseId>-<YYYY-MM>` — so the
same lease/period can only exist once. On top of that, generation treats *any* same-tenant
payment whose date falls in the period as covering it, so a manually recorded payment also
blocks generation. Within a single run, later periods see earlier periods' creations before
deciding duplicates. There's no database to put a unique constraint on, so the deterministic
key plus the pre-insert check inside the store updater is the enforcement.

**Q: Why not `new Date().toISOString().slice(0,10)` for "today"?**

That gives the UTC calendar day, not the business day — a user at UTC-7 at 6pm would get
tomorrow's date. The app defines `APP_TIMEZONE` and derives today via
`Intl.DateTimeFormat(…, { timeZone })` in `zonedToday`. Day arithmetic then runs on
`YYYY-MM-DD` strings through UTC millis (`addDaysIso`), which is DST-proof — adding one day is
always +86,400,000 ms because the strings carry no time component.

(Worth flagging honestly: `RecordPaymentModal` still uses the naive
`toISOString().slice(0,10)` at module scope for its "no backdating" rule — a known
inconsistency that a code review caught. The fix is to route it through the same helper.)

**Q: Explain pause/resume on a maintenance request.**

`Paused` was added to the status union, and the transition logic already had the right shape:
`completedDate` and `actualCost` are only stamped when the target status is `'Completed'`,
`scheduledDate` is preserved with `m.scheduledDate ?? …`, and `history` is append-only. So
pausing preserves everything by construction — resuming just sets the status back to
`In Progress` with another history entry. Pausing is the one transition behind a
`ConfirmDialog`, because it's the one that stalls work.

The design point: nothing is *reset* and nothing is *hidden* — a paused request is still in
the list, still counted in its own tab, and keeps its full audit trail.

**Q: A payment is due Oct 1. When does it become Overdue?**

Oct 2. `planOverdueTransitions` computes `cutoff = today - OVERDUE_GRACE_DAYS` (1 day) and
flips every `Pending` payment with `date <= cutoff`. On Oct 1 the cutoff is Sep 30, so it
stays Pending; on Oct 2 the cutoff is Oct 1, so it flips. `Paid` rows are never touched, and
the store **re-checks** `status === 'Pending'` at apply time so a payment marked Paid between
planning and applying can't be clobbered by a stale plan.

**Q: How is file upload secured with no server?**

Extension and MIME are treated as hints, never proof. `verifyDocFileContent` reads actual
bytes: magic numbers for PDF (`%PDF`), legacy Office (`D0CF11E0`) and OOXML (`PK`), NUL-byte
sampling for text — and for docx/xlsx it walks the zip central directory and requires a
`word/` or `xl/` entry, so a renamed `.zip` or `.png` fails. Filenames are sanitised (directory
components stripped, control characters removed, length capped) before storage. The same
`validateDocRecord` runs in the form *and* in `store.addDocument`, so the two gates can't drift.

Caveat I'd state out loud: client-side sniffing is a UX and data-hygiene measure. With a real
backend, the server must repeat all of it — a determined user can always bypass client code.

---

## React & TypeScript

**Q: Where did you need `useMemo`, and where did you deliberately not?**

Used for list derivations that run over hundreds of rows on every keystroke — the filter/sort
pipelines in `Maintenance.tsx`, `Payments.tsx`, `Leases.tsx`, `Tenants.tsx` — and for the
`indexOptions` normalisation inside `SearchSelect`, which pre-lowercases labels once per option
list rather than once per keystroke.

Not used for cheap scalar derivations (`selected`, `editing`, a `.find()` on six properties).
Memoising those costs more in comparisons and cognitive overhead than it saves.

**Q: You have a component that adjusts state during render. Why isn't that an effect?**

`PhoneInput` needs the dial code to follow the selected property while the field is still
untouched. Written as an effect, that's a `setState` in `useEffect` — a cascading render, and
ESLint's `react-hooks/set-state-in-effect` rule flags it. The React-documented alternative is
to adjust state during render, guarded by a comparison with the previous prop value:

```ts
if (!locked && defaultCountry !== prevDefaultCountry) {
  setPrevDefaultCountry(defaultCountry);
  setCountry(defaultCountry);
}
```

React re-runs the component immediately without committing the intermediate render, so the
value is correct before paint instead of flashing the old one.

**Q: How do you handle a controlled `<input type="date">` with invalid input?**

This is a trap in the codebase worth naming: the browser lets the year segment exceed four
digits, producing values like `99999-01-01`. The current guard is
`onChange={(e) => { if (isValidIsoDate(e.target.value)) setDate(e.target.value); }}`.

The bug in that pattern — which a review caught — is that *rejecting without setting state*
produces no re-render, so React never reconciles the DOM back, and the input visibly shows the
rejected text while state holds the old value. The correct fix is to always store what was
typed and surface a validation error, or force a re-render. I'd fix it by accepting the value
into state and letting the error path handle it.

**Q: What do the strict TS flags buy you?**

* `verbatimModuleSyntax` — type-only imports must say `import type`, so the emitted JS has no
  phantom imports and bundlers can drop them reliably.
* `erasableSyntaxOnly` — no enums, no namespaces. Enums emit runtime objects and have
  surprising bidirectional mapping; the codebase uses string-literal unions plus `satisfies`
  instead, which are erasable and narrow better.
* `noUnusedLocals`/`noUnusedParameters` — dead imports fail the build rather than rotting.

An example of the union approach paying off:

```ts
export const MAINTENANCE_STATUS_ORDER =
  ['Open', 'In Progress', 'Paused', 'Scheduled', 'Completed'] as const
  satisfies readonly MaintenanceStatus[];
```

`satisfies` checks every member is a real status without widening the tuple to `string[]`, so
`byRank` still gets literal types. And `bedroomsForType` uses an exhaustive `switch` over the
union — add a new unit type and the compiler flags the missing case.

**Q: Any performance concerns with the tables?**

At current volumes (128 tenants, 67 requests, 48 documents) client-side filter + sort + slice
is well under a frame. The tables paginate or cap rendered rows (`SearchSelect` renders at most
100 options with a "keep typing to narrow" hint). Past a few thousand rows this stops being
viable and I'd move to virtualisation (`react-virtual`) plus server-side search with debounced
queries — which is also the point where the search haystack strings should become a server
concern rather than a per-render `join()`.

---

## Styling

**Q: How is Tailwind v4 set up here, and why no config file?**

v4 moves design tokens into CSS. `src/index.css` declares the layer order
(`@layer theme, base, components, utilities`) and defines every token in `@theme` —
colours, radii, shadows, easings, fonts and a custom `--breakpoint-compact: 1100px` that
generates `max-compact:`/`min-compact:` variants. No `tailwind.config.js` exists because
nothing needs JS-side configuration.

Repeated multi-utility patterns are `@apply` composites in `styles/components.css`
(`.card`, `.btn-teal`, `.badge`, `.field`, `.combo-*`, `.modal-*`), which sit in the
`components` layer — so they always beat base styles and always lose to utilities. That
ordering is what lets a one-off `className="mt-2"` override a composite without `!important`.

**Q: How do you keep status colours consistent?**

`lib/tone.ts` is the only place a status maps to a colour. One `Tone` vocabulary
(`success | warn | info | danger | neutral`), one function per status union. Components call
`<Badge tone={paymentTone(p.status)}>`; they never pick a colour themselves. When `Paused` was
added, exactly one function changed.

---

## Testing & quality

**Q: There are no tests. Defend that, then tell me what you'd add first.**

I won't defend it as a good state — it's the biggest gap. What I *did* do is keep the logic
testable: every domain rule lives in a pure module under `lib/` with no React, no store and no
clock. `planAutomationRun(snapshot, '2026-10-02')` is a pure function of its arguments.

First tests I'd write, in order:

1. `paymentAutomation` — idempotency (run twice, one row), the grace-day boundary (Oct 1 vs
   Oct 2), timezone handling, and the ineligibility classifications. Highest value: it's the
   only logic that mutates data without a user pressing anything.
2. `lib/files.ts` sniffers — `sniffDocHead`/`sniffOoxmlTail` are pure over `Uint8Array`, so a
   renamed-zip fixture is a three-line test.
3. `validateUnit` / `validateStaff` / `validateMaintenanceTarget` — cheap table-driven tests
   covering the cross-property cases.
4. `lib/sort.ts` — the missing-values-last-in-both-directions rule is subtle and easy to
   regress.

Vitest, since Vite is already the build tool. Then React Testing Library for the two flows
worth integration coverage: create-maintenance (scope switching clears selections) and the
phone field (typing formats, invalid blocks submit).

**Q: How do you catch regressions today?**

`npm run build` runs `tsc -b` first, so type errors block the build, and ESLint runs the
`react-hooks` rules including `set-state-in-effect` and exhaustive-deps. That catches shape
errors and hook misuse but nothing behavioural — which is precisely why the automation logic
being pure matters.

---

## Known issues (have these ready — being candid scores well)

A recent review surfaced real problems. Worth naming a couple unprompted if asked "what would
you fix next":

| Issue | Impact |
| --- | --- |
| `.modal` is `overflow-hidden` with a scrolling `.modal-body`, so `SearchSelect`/`MultiSearchSelect` dropdowns are clipped inside dialogs | Highest-severity UI bug; affects every searchable picker in a modal |
| Focus trap in `Modal` queries the whole subtree, so stacked modals fight over focus and one Escape closes both | Keyboard users can't stay in a nested dialog |
| `onActivateKey` on a row `preventDefault()`s Enter/Space bubbling up from nested buttons | Keyboard activation of row actions triggers the wrong handler |
| `role="button"` on `<tr>` overrides the implicit `row` role | Destroys table semantics for screen readers |
| Date inputs reject invalid values without setting state | DOM and state can diverge |
| `RecordPaymentModal`'s module-scope UTC `today` | Blocks legitimate same-day entry west of UTC |

The pattern I'd point to: the ones that hurt most are **accessibility and layout** regressions,
not domain logic — because the domain logic has guards and the UI doesn't have tests.

---

## Behavioural angles

**"What was the hardest technical decision?"**
Replacing `MaintenanceRequest`'s `unit`/`unitId`/`tenantId`/`assignee` fields with
`scope`/`unitIds`/`tenantIds`/`assigneeId`. It touched ~15 files including seed data, and the
safe-looking option was to add new fields alongside the old ones. I chose the clean break
because `assignee` was a free-text string that was never a real relationship — keeping it would
have meant two ways to express the same fact forever. I de-risked it by migrating the seed data
in the same change and writing a throwaway script to assert every generated row's unit/tenant
ids actually belong to its property.

**"Tell me about a bug you prevented rather than fixed."**
The overdue transition re-checks `status === 'Pending'` at apply time, not just at plan time.
Planning and applying are separate steps, so a payment marked Paid in between would otherwise
be flipped to Overdue by a stale plan. It's a one-line guard that only matters in a race that
is rare today — but would be routine the moment this runs against a server.

**"What would you do differently starting over?"**
Write the tests for `lib/` from the start — they're the cheapest possible tests given the
purity, and their absence is why UI regressions land unnoticed. I'd also put the scope
discriminant on maintenance from day one instead of migrating to it; the original
single-`unitId` shape was never going to survive the first "can I report on the whole
building?" request.

**"Where does this app break at scale?"**
Three places, in order: (1) tables filter and sort the whole array on every keystroke — needs
virtualisation and debounced server search past a few thousand rows; (2) `localStorage` has a
~5 MB budget and is synchronous — persisted slices would need IndexedDB; (3) every "resolve id
→ name" call is an O(n) `.find()` inside a render loop, which is fine for six properties and
quadratic for six thousand — those become `Map` lookups built once per render.
