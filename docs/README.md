# Propora — Documentation Hub

Consolidated, read-in-one-place copy of the documentation scattered across the
three Propora repositories. This folder is a **mirror**, not the source of
truth — the original files still live in each repo (and stay there for now,
since each repo's docs are tracked on its own GitHub history). If you're
editing documentation, edit it at the source below and re-sync this copy;
don't edit only here.

| Section | Source repo/folder | Contents |
| --- | --- | --- |
| [frontend-propora/](./frontend-propora) | `Propora-Frontend/docs/` | Dashboard app: architecture, data model, business logic, user flows, interview prep |
| [api/](./api) | `Propora-Backend/docs/` | Backend API: architecture, auth, data model, endpoint reference, error conventions |
| [website/](./website) | `Propora-Website/README.md` | Marketing site's page anatomy, tech stack, and structure |
| [design-system/colors.md](./design-system/colors.md) | shared across all three repos | The single color/visual token reference — frontend and marketing site both consume these tokens verbatim, so there is one copy here, not two |
| [images/](./images) | top-level `README.md` assets | The three screenshots/logo used by the [top-level README](../README.md) |

## Repositories at a glance

| Repo | Role | Docs source |
| --- | --- | --- |
| [Propora-Frontend](../Propora-Frontend) | Frontend dashboard (React + TypeScript, frontend-only) | `Propora-Frontend/docs/` |
| [Propora-Backend](../Propora-Backend) | Backend REST API (Express 5 + MongoDB) | `Propora-Backend/docs/` |
| [Propora-Website](../Propora-Website) | Marketing/landing site | `Propora-Website/README.md` |

## Read in this order

1. [top-level README](../README.md) — the big-picture context: what Propora is and how the three repos fit together
2. [frontend-propora/README.md](./frontend-propora/README.md) → its numbered docs
3. [api/README.md](./api/README.md) → its numbered docs
4. [website/README.md](./website/README.md) — the marketing site's structure
5. [design-system/colors.md](./design-system/colors.md) — the shared color/visual token reference
