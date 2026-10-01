[Docs](./README.md) › **API reference**

# API reference

The Propora API is a JSON REST API with **58 endpoints** across 11 resources.

- **Base URL (local):** `http://localhost:3000`
- **Format:** JSON. Uploads use `multipart/form-data`.
- **Auth:** Session cookie. Send requests with `credentials: "include"`.
- **Access:** Every route needs a session, except the account routes under `/users`.

## Basics

**Responses** are flat objects named after the resource:

```json
{ "property": { "id": "6ab9…", "name": "Ocean View Residences" } }
{ "properties": [ … ] }
{ "message": "Property deleted successfully" }
```

**Errors** always look like `{ "message": "…" }`.

| Status | Meaning |
| --- | --- |
| `200` / `201` | Success / created |
| `400` | Invalid id or missing field |
| `401` | Not signed in, or wrong email/password |
| `404` | Not found, or it belongs to another account (the API never says which) |
| `409` | Username or email already taken |

**Updates** (`PUT`) are partial: only the fields you send are changed.

---

## Users `/users`

| Method | Path | What it does |
| --- | --- | --- |
| `POST` | `/users/register` | Create an account and sign in |
| `POST` | `/users/login` | Sign in with email (or username) and password |
| `POST` | `/users/logout` | Sign out and clear the cookie |
| `GET` | `/users/session` | Return the signed-in user, or `401` |
| `GET` | `/users/:username` | Look up a user by username (no password) |

## Properties `/properties`

| Method | Path | What it does |
| --- | --- | --- |
| `GET` | `/properties?page=1&limit=20` | List properties, paginated (max 100 per page) |
| `POST` | `/properties` | Create a property, with an optional `image` file |
| `GET` | `/properties/:id` | Get one property |
| `PUT` | `/properties/:id` | Update a property, with an optional new `image` |
| `PATCH` | `/properties/:id/status` | Change the status |
| `DELETE` | `/properties/:id` | Delete a property |

## Units `/units`

| Method | Path | What it does |
| --- | --- | --- |
| `GET` | `/units/properties/:propertyId/units` | List a property's units |
| `POST` | `/units/properties/:propertyId/units` | Add a unit (checks you own the property) |
| `GET` | `/units/:id` | Get one unit |
| `PUT` | `/units/:id` | Update a unit |
| `PATCH` | `/units/:id/status` | Change the status |
| `DELETE` | `/units/:id` | Delete a unit |

## Tenants, leases and payments

Each of these has the same five endpoints:

| Method | Path | What it does |
| --- | --- | --- |
| `GET` | `/tenants` · `/leases` · `/payments` | List all |
| `POST` | `/tenants` · `/leases` · `/payments` | Create one |
| `GET` | `/…/:id` | Get one |
| `PUT` | `/…/:id` | Update one |
| `DELETE` | `/…/:id` | Delete one |

Side effects:

- Creating a **tenant** adds a *New tenant* notification.
- Setting a **payment** to `Overdue` adds an *Overdue payment* notification.
- New **leases** always start as `Active`.

## Maintenance `/maintenance`

| Method | Path | What it does |
| --- | --- | --- |
| `GET` | `/maintenance` | List all requests |
| `POST` | `/maintenance` | Create a request (starts `Open`, adds a notification) |
| `GET` | `/maintenance/:id` | Get one request |
| `PUT` | `/maintenance/:id` | Update details |
| `PATCH` | `/maintenance/:id/assignee` | Assign or unassign staff (`assigneeId` or `null`) |
| `PATCH` | `/maintenance/:id/status` | Change the status |
| `POST` | `/maintenance/:id/pause` | Shortcut: pause |
| `POST` | `/maintenance/:id/resume` | Shortcut: back to *In Progress* |
| `POST` | `/maintenance/:id/complete` | Shortcut: complete |
| `DELETE` | `/maintenance/:id` | Delete a request |

Status changes and reassignments are added to the request's `history`. Completing a request sets `completedDate` and, if empty, copies `estimatedCost` into `actualCost`. A scheduled date in the past is rejected.

## Maintenance staff `/maintenance/staff`

| Method | Path | What it does |
| --- | --- | --- |
| `GET` | `/maintenance/staff` | List staff |
| `POST` | `/maintenance/staff` | Add a staff member |
| `GET` | `/maintenance/staff/:id` | Get one staff member |
| `PUT` | `/maintenance/staff/:id` | Update a staff member |
| `PATCH` | `/maintenance/staff/:id/status` | Activate or deactivate |
| `DELETE` | `/maintenance/staff/:id` | Remove a staff member |

## Documents `/documents`

| Method | Path | What it does |
| --- | --- | --- |
| `GET` | `/documents` | List documents |
| `POST` | `/documents` | Upload a document (`file` field) |
| `GET` | `/documents/:id` | Get one document |
| `PUT` | `/documents/:id` | Edit details (not the file) |
| `PATCH` | `/documents/:id/archive` | Archive a document |
| `DELETE` | `/documents/:id` | Delete the record |

The server works out the file size, uploader and upload date by itself. Files are served from `/uploads/documents/<file>`.

## Notifications `/notifications`

| Method | Path | What it does |
| --- | --- | --- |
| `GET` | `/notifications` | List notifications, newest first |
| `PATCH` | `/notifications/:id/read` | Mark one as read |
| `PATCH` | `/notifications/read-all` | Mark all as read |

## Dashboard `/dashboard`

| Method | Path | What it does |
| --- | --- | --- |
| `GET` | `/dashboard` | Live totals: properties, units, and rent collected this month |

```json
{ "totalProp": 4, "totalunits": 15, "monthlyRevenue": 21600 }
```

---

<div align="center">

[← Database design](./05-database-design.md) · [Docs home](./README.md) · [Design system →](./07-design-system.md)

</div>
