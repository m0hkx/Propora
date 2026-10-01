[Docs](./README.md) › **Case study**

# Case study

<img src="./images/case-study-cover.jpg" alt="Propora case study cover" width="100%" />

| Role | Focus | Stack | Year |
| --- | --- | --- | --- |
| Solo: design + full-stack development | UI system · Backend | React · Express · MongoDB | 2026 |

## The problem

Property managers juggle many tools. Units live in one spreadsheet, rent in another, and repair requests sit in someone's inbox. Answering a simple question like *"How is my portfolio doing?"* means opening five files.

That causes real problems:

- **Late rent is noticed late**, because nobody tracks due dates by hand every day.
- **Repairs get lost**, because there is no single list with an owner and a status.
- **Numbers disagree**, because the same fact is typed into several places.

## The goal

1. Answer *"How is my portfolio doing?"* in under five seconds.
2. Automate the boring, error-prone work, starting with monthly rent.
3. Keep every account's data fully private.
4. Make it feel calm: one look, one set of rules, on every page.

## The solution

Propora is one dashboard plus seven focused pages: **Properties, Tenants, Leases, Payments, Maintenance, Documents** and a **Profile**. The dashboard shows the health of the portfolio at a glance. Each page handles one job well.

<img src="./images/desktop-preview.png" alt="Propora dashboard on a desktop monitor" width="100%" />

Behind the screens is a real backend. The React app talks to an Express REST API, and the API stores everything in MongoDB. See [System design](./04-system-design.md) for how they fit together.

## Key challenges

### 1. Billing rent automatically, but never twice

**Challenge:** Rent is due on the 1st of every month. Creating those bills by hand is slow and easy to get wrong.

**Solution:** A scheduled job on the server runs every day at 00:05 UTC, and once when the server starts. It creates each month's rent for every active lease and marks unpaid rent as *Overdue* one day after the due date. A unique database index on *(lease, month)* makes a duplicate bill impossible, even if two runs overlap.

### 2. Keeping every account private

**Challenge:** Many managers share one database. No one should ever see another person's data.

**Solution:** Every record stores the owner's `userId`, taken from the login session, never from the request. Every query filters by it. Asking for someone else's record returns *404 Not found*, so you can't even tell it exists.

### 3. Repairs that target a building, some units, or some tenants

**Challenge:** A broken lift affects the whole building. A leaking tap affects one unit. A noise complaint is about specific tenants.

**Solution:** Each maintenance request has a `scope` (*property*, *units* or *tenants*) plus the matching list of ids. The form only offers units and tenants from the chosen property, and the rules are checked again before saving. No fake placeholder records are needed.

### 4. Trusting uploaded files

**Challenge:** Uploads are where untrusted files enter the system. A renamed `.exe` should never pass as a PDF.

**Solution:** The app checks each file's real bytes (the "magic number" at the start of the file) and, for Word and Excel files, looks inside the zip for the right folders. Files must be under 10 MB. The server then saves each file under a random generated name.

## Results

- **8 main screens** covering the full property-management workflow, all backed by live data.
- **58 REST endpoints** across 11 resources, with session authentication.
- **10 MongoDB collections** with per-user data isolation.
- **Hands-free rent:** monthly bills and overdue flags with no manual work.
- **One design system** carried through every screen, chart and form.

## What I learned

- **Design the data first.** Getting the maintenance `scope` model right early saved many UI bugs later.
- **Think about system design early.** Defining clear boundaries between components, data flow, and responsibilities early made the system easier to extend.
- **Validate twice.** The form checks for a fast, friendly response. The server checks again because the client can't be trusted.
- **Document the trade-offs.** Knowing what I left out, and why, matters as much as what I built. See [Engineering decisions](./09-engineering-decisions.md).

---

<div align="center">

[← Docs home](./README.md) · [Screenshots →](./02-screenshots.md)

</div>
