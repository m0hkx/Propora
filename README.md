<div align="center">
<img src="docs/images/logo.png" alt="Propora" width="88" />

# Propora

### Property management, made legible.

A full-stack property-management platform — a real REST API, a data-driven
operations dashboard, and a marketing site — built end to end as a portfolio
case study in product thinking, interface craft, and backend design.

![React](https://img.shields.io/badge/react-19-0F766E?style=flat-square&logo=react&logoColor=white)
![TypeScript](https://img.shields.io/badge/typescript-6-0F766E?style=flat-square&logo=typescript&logoColor=white)
![Express](https://img.shields.io/badge/express-5-0F766E?style=flat-square&logo=express&logoColor=white)
![MongoDB](https://img.shields.io/badge/mongodb-native_driver-0F766E?style=flat-square&logo=mongodb&logoColor=white)
![Vite](https://img.shields.io/badge/vite-8-0F766E?style=flat-square&logo=vite&logoColor=white)
![Tailwind CSS](https://img.shields.io/badge/tailwind-4-0F766E?style=flat-square&logo=tailwindcss&logoColor=white)

**`FULL-STACK · PRODUCT DESIGN · CASE STUDY`**

</div>

<img src="docs/images/case-study.png" alt="Propora — property management, made legible" width="100%" />

---

## About this project

Property managers live in spreadsheets: units here, arrears there, maintenance
requests in someone's inbox. **Propora** collapses that into one calm operating
picture — properties, tenants, leases, payments, maintenance, and documents,
all in a single screen where the answer to *"how is my portfolio doing?"* is
legible in under five seconds.

It's built as three independent, purpose-built repositories rather than one
monolith — the way a real product team would split frontend, backend, and
marketing concerns:

| Repository | Role | Stack | Source |
| ---------- | ---- | ----- | ------ |
| **[Propora](https://github.com/m0hkx/Propora-Frontend)** | The product — a fully interactive dashboard covering the full property-management workflow | React 19 · TypeScript · Vite · Tailwind CSS v4 · Zustand · React Router | [Propora-Frontend](https://github.com/m0hkx/Propora-Frontend) |
| **[Propora-API](https://github.com/m0hkx/Propora-Backend)** | The backend — a multi-tenant REST API with session auth, file uploads, and MongoDB persistence | Express 5 · TypeScript (ESM) · MongoDB (native driver) · express-session · multer | [Propora-Backend](https://github.com/m0hkx/Propora-Backend) |
| **[ProporaWebsite](https://github.com/m0hkx/Propora-Website)** | The pitch — a marketing/landing site introducing the product | React 19 · TypeScript · Vite · hand-rolled CSS design system | [Propora-Website](https://github.com/m0hkx/Propora-Website) |

Each app is a real, independently-built piece rather than a demo shell: the
API is a working multi-tenant backend with its own auth and persistence, and
the dashboard is a fully interactive, realistic-data build in its own right.
The dashboard currently runs on its own seeded dataset rather than calling the
live API — see [its docs](./docs/frontend-propora) for why that's a deliberate
frontend-only build, not a shortcut.

## Product tour

<img src="docs/images/dashboard-preview.png" alt="Propora dashboard — property management overview" width="100%" />

| Area | Highlights |
| ---- | ---------- |
| **Dashboard** | Animated KPI hero cards, revenue area chart, occupancy donut, property performance, action-required triage |
| **Properties** | Portfolio summary, status/type filters, per-unit breakdown, image upload, create/edit flows |
| **Tenants** | Full roster with search, sorting, pagination, and deep links into leases & payments |
| **Leases** | Status + property filters, tenant-scoped focus mode via shareable URLs |
| **Payments** | Collection KPIs, status/method filters, record-payment flow with automatic overdue notifications |
| **Maintenance** | Work-queue triage, staff assignment, status lifecycle (Open → In Progress → Paused/Completed) with a full history trail |
| **Documents** | Property/tenant-scoped document vault with upload, archive, and expiration tracking |
| **Notifications** | Activity feed for new tenants, overdue payments, new maintenance requests |

## System overview

**Backend (Propora-API)**

- **Auth** — `express-session` cookie, `bcryptjs` password hashing; every document in
  MongoDB carries a `userId`, and every query is scoped to the logged-in user.
- **Uploads** — property images and documents are stored on disk (`multer`) and served
  statically.

**Frontend (Propora)**

- **State** — the dashboard's Zustand store is seeded from `src/data/mock.ts`; every
  write is synchronous, and a subset of slices (units, maintenance, staff) persists
  to `localStorage`. It does not currently call the API above.

## Design system

One teal identity carried through every surface, across all three repos:

- **Palette** — Trust Teal `#0F766E` on a mint wash `#F0FDFA`, deep-teal ink `#134E4A`
  (never pure black), semantic badge tones for every status
- **Type** — Plus Jakarta Sans throughout, tabular numerals so metrics never jitter
- **Shape & depth** — 18px cards, mint-tinted borders, teal-tinted shadows, pill actions
- **Logo** — a single geometric "P" mark (`docs/images/logo.png`), used flat on the
  marketing site and as a white cutout on the teal brand badge in-app

Full token reference: [`docs/design-system/colors.md`](./docs/design-system/colors.md) —
one shared file, since the frontend and the marketing site consume the same tokens verbatim.

## Documentation map

Each repo documents itself in depth; [`docs/`](./docs) mirrors all of it in one
place so you don't have to hop repos to read it:

| Doc | Covers |
| --- | ------ |
| [`docs/`](./docs) | The documentation hub — start here |
| [`docs/api/`](./docs/api) | Backend architecture, auth, data model, and the full REST API reference |
| [`docs/frontend-propora/`](./docs/frontend-propora) | Dashboard architecture, data model, business logic, and user flows |
| [`docs/website/`](./docs/website) | Marketing site's structure and page anatomy |
| [Propora-Frontend/README.md](https://github.com/m0hkx/Propora-Frontend) | Dashboard-specific setup and feature tour |
| [Propora-Backend/README.md](https://github.com/m0hkx/Propora-Backend) | API setup, environment variables, and endpoint summary |
| [Propora-Website/README.md](https://github.com/m0hkx/Propora-Website) | Marketing site setup |

## Running the full stack locally

The three apps are independent git repositories that live side by side, and each
runs standalone — none of the `npm run dev` commands below depend on another:

```bash
# 1. Backend — needs a MongoDB connection string and a session secret
cd Propora-Backend
cp .env.example .env   # fill in MONGODB_URI and SESSION_SECRET
npm install
npm run dev             # http://localhost:3000

# 2. Dashboard — runs on its own seeded dataset, no .env needed
cd ../Propora-Frontend
npm install
npm run dev              # http://localhost:5173

# 3. Marketing site — standalone, no backend dependency
cd ../Propora-Website
npm install
npm run dev              # http://localhost:5174 (or next free port)
```

The API's CORS policy is pre-configured for a credentialed session cookie from
`http://localhost:5173` — the dashboard's dev origin — for when the two are wired
together.

## Why this project

This repo set is a deliberately realistic slice of what shipping a small SaaS
product looks like solo: a documented, multi-tenant API; a polished, fully
interactive frontend demonstrating the entire workflow end to end; and a
marketing surface that sells the same story visually. It's meant to
demonstrate range — product design, frontend craft, backend architecture, and
the judgment to document trade-offs honestly — rather than a single narrow
skill.

---

<div align="center">

Built by [**@m0hkx**](https://github.com/m0hkx) as a full-stack portfolio case study.

</div>
