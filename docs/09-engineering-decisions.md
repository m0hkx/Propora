[Docs](./README.md) › **Engineering decisions**

# Engineering decisions

The main choices behind Propora: what I picked, why, and what it costs. Good engineering is mostly trade-offs, so the costs are listed too.

## Key decisions

### Session cookies instead of JWT

- **Why:** The cookie is `httpOnly`, so page scripts can't read it, and signing out truly ends the session on the server. There are no tokens to store or refresh.
- **Trade-off:** Sessions live in the server's memory. A restart signs everyone out, and it won't work across several servers without a shared session store.

### One database, every record tagged with its owner

- **Why:** Every document carries a `userId` from the session, and every query filters by it. It's simple, and asking for someone else's record returns *404*, so nothing leaks, not even whether it exists.
- **Trade-off:** Safety depends on every query remembering the filter. A shared data-access helper would make that automatic.

### Rent automation on the server

- **Why:** Rent must be billed even when nobody has the app open. A `node-cron` job runs daily at 00:05 UTC and once on start-up, to catch up on any missed days.
- **How it stays correct:** The rules are pure functions that take *today's date* as an input, so they're easy to test. A unique index on *(lease, month)* blocks duplicate bills, even if two runs overlap.
- **Trade-off:** The job only runs while the API is awake. The start-up run covers the gap.

### Simple API layers: route → controller → database

- **Why:** Each endpoint reads top to bottom in one file. No service or repository layers to jump between.
- **Trade-off:** Some code repeats, like the `userId` filter and response shaping. At 11 resources that's fine. It's the first thing I'd extract as the API grows.

### The server is the source of truth

- **Why:** The web app never invents ids or saves data locally. It sends a request, waits for the saved record, and merges that into the store. What you see is what's in the database.
- **Trade-off:** The app loads every resource on sign-in. That's fast at portfolio size, but large accounts would need paging and caching for every list.

### Validate twice

- **Why:** The browser checks forms for instant feedback. The API checks again, because a browser can always be bypassed.
- **Trade-off:** The server's checks are lighter than the browser's. A schema library such as Zod would close that gap.

### Model repairs with a `scope`

- **Why:** A request targets the whole property, some units, or some tenants. A `scope` field plus id lists says exactly that. No fake "whole building" unit is needed, and reports stay honest.

### Work out occupancy, don't store it

- **Why:** A unit counts as *Occupied* when a current tenant or lease points to it. Storing it would mean updating it on every move-in, move-out and lease end, and each of those could forget.

### Hand-drawn SVG charts

- **Why:** The dashboard needs just a few chart types. Drawing them in SVG keeps the bundle small and keeps full control over the brand look.

## Known limitations

| Limitation | Why it matters |
| --- | --- |
| No automated tests yet | The rent rules and validators are pure functions, ready for tests, but none are written |
| Delete protection is in the UI only | The app blocks deleting a unit or staff member that's still in use, but the API doesn't check this itself |
| Deleting a document keeps the file | The database record is removed, but the file stays in `uploads/` |
| Sessions are in memory | A restart signs everyone out, and the API can't run as several instances |
| Inbox uses demo data | The messages panel isn't connected to the API yet |

## What's next

1. **Tests first:** Vitest for the rent rules, validators and sorting.
2. **Schema validation** on every API endpoint with Zod.
3. **Server-side delete rules**, so the API enforces the same blockers as the UI.
4. **Persistent sessions** with a MongoDB session store.
5. **Cloud file storage** for uploads, and file clean-up on delete.
6. **Real messaging** between managers and tenants.

---

<div align="center">

[← Getting started](./08-getting-started.md) · [Docs home](./README.md)

</div>
