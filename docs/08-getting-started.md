[Docs](./README.md) › **Getting started**

# Getting started

Run Propora on your own machine in about five minutes.

## You'll need

- **Node.js** 20.19+ or 22.12+
- **MongoDB**: a free [MongoDB Atlas](https://www.mongodb.com/atlas) cluster, or MongoDB running locally
- **Git**

## 1. Get the code

Propora is two repositories. Clone them side by side:

```bash
git clone https://github.com/m0hkx/Propora-Backend.git
git clone https://github.com/m0hkx/Propora-Frontend.git
```

## 2. Start the API

```bash
cd Propora-Backend
npm install
cp .env.example .env    # then fill in the values below
npm run seed            # creates the demo account and sample data
npm run dev             # http://localhost:3000
```

Your `.env` file:

```bash
MONGODB_URI="mongodb+srv://<user>:<password>@<cluster>/"   # your MongoDB connection string
SESSION_SECRET="any-long-random-string"                    # signs the login cookie
CORS_ORIGIN="http://localhost:5173"                        # where the web app runs
```

## 3. Start the web app

In a second terminal:

```bash
cd Propora-Frontend
npm install
cp .env.example .env    # set VITE_API_URL=http://localhost:3000
npm run dev             # http://localhost:5173
```

## 4. Sign in

Open <http://localhost:5173> and use the demo account:

| Email | Password |
| --- | --- |
| `demo@propora.dev` | `Demo1234!` |

> [!NOTE]
> Want a clean slate? Run `npm run seed -- --reset` in the API folder to wipe the demo data and create it again.

## Environment variables

**API** (`Propora-Backend/.env`)

| Variable | Required | What it does |
| --- | :-: | --- |
| `MONGODB_URI` | ✅ | MongoDB connection string. The data goes into the `property_management` database. |
| `SESSION_SECRET` | ✅ | Secret used to sign the session cookie |
| `CORS_ORIGIN` | ✅ | Comma-separated list of web app addresses allowed to call the API |
| `PORT` | | Port to listen on. Defaults to `3000`. |
| `NODE_ENV` | | Set to `production` when deployed: turns on secure, cross-site cookies |

**Web app** (`Propora-Frontend/.env`)

| Variable | Required | What it does |
| --- | :-: | --- |
| `VITE_API_URL` | ✅ | Address of the API. Defaults to `http://localhost:3000`. |

## Useful scripts

| Where | Command | What it does |
| --- | --- | --- |
| API | `npm run dev` | Start with auto-reload |
| API | `npm run seed` | Create the demo account (add `-- --reset` to recreate it) |
| API | `npm run build` then `npm run start:prod` | Compile to JavaScript and run it |
| Web app | `npm run dev` | Start the dev server with hot reload |
| Web app | `npm run build` | Type-check, then build for production |
| Web app | `npm run lint` | Run ESLint |
| Web app | `npm run preview` | Preview the production build |

## Common problems

| Problem | Fix |
| --- | --- |
| **"MONGODB_URI is not defined"** | The API can't find its `.env` file. Create it in the API folder. |
| **CORS error in the browser** | `CORS_ORIGIN` must match the web app's address exactly, including the port. |
| **Signed out after restarting the API** | Expected. Sessions are kept in memory, so a restart signs everyone out. |
| **Images or documents don't load** | Check that `VITE_API_URL` points to the running API. Files are served from its `/uploads` folder. |

---

<div align="center">

[← Design system](./07-design-system.md) · [Docs home](./README.md) · [Engineering decisions →](./09-engineering-decisions.md)

</div>
