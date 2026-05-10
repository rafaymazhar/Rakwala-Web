# RAKHWALA — integration notes (auth, API, maps)

## New / modified surface area

- **Server** (`server/`): Express + `bcryptjs` + `jsonwebtoken`, with **MongoDB (Mongoose) when `MONGODB_URI` is set**, or a **JSON file store** (`server/data/users.json`, override with `FILE_DB_PATH`) when Mongo is unset or unreachable.
- **Routes**: `POST /api/auth/signup`, `POST /api/auth/login`, `GET /api/user/profile` (Bearer JWT).
- **Frontend**: `/signup` page, JWT persistence (`localStorage` key `rakhwala_jwt`), dashboard session card + Google Maps panel, `ShellNav`, theme cycle (dark / light / system), Sonner toasts, Vite proxy to API.

## Environment variables

### Server (`server/.env` — copy from `server/.env.example`)

| Variable | Purpose |
|----------|---------|
| `MONGODB_URI` | Optional Mongo connection string; empty or failed connect uses the file store |
| `JWT_SECRET` | HMAC secret for JWT |
| `PORT` | API port (default `4000`) |
| `CLIENT_ORIGIN` | Allowed CORS origin(s), comma-separated |
| `JWT_EXPIRES_IN` | JWT expiry (default `7d`) |
| `FILE_DB_PATH` | Optional path for `users.json` (default under `server/data/`) |

### Web (`.env` at repo root for Vite)

| Variable | Purpose |
|----------|---------|
| `VITE_GOOGLE_MAPS_API_KEY` | Browser Maps JavaScript API key (Maps + clustering) |
| `VITE_API_BASE_URL` | Optional absolute API base; leave empty to use same-origin `/api` (proxy in dev). |
| `VITE_ALLOW_DEMO_LOGIN` | Set to `true` only for offline demos: allows guest session when API fails or Firebase is off. Default (unset) requires a real API login. |

Firebase env vars remain optional for Google sign-in.

## NPM packages added

### Root (`package.json`)

- `sonner` — toast host
- `@react-google-maps/api`, `@googlemaps/markerclusterer` — map + clustering
- `@types/google.maps` — types
- `concurrently` — run Vite + API together

### Server (`server/package.json`)

- `express`, `mongoose`, `bcryptjs`, `jsonwebtoken`, `cors`, `dotenv`, `tsx` (+ typings)

## How to run locally

```bash
# 1) API deps
cd server
npm install
cp .env.example .env   # set JWT_SECRET; MONGODB_URI optional (file store if empty)
npm run dev

# 2) Web (second terminal) — or from repo root after server deps:
cd ..
npm install
npm run dev:web          # frontend only — proxies /api → http://127.0.0.1:4000

# Or both:
npm run dev
```

Signup/login work without Mongo: the API falls back to the file store. Use Mongo when you want a shared database across instances.

## Production reminders

- Point `CLIENT_ORIGIN` at your HTTPS web origin(s).
- Use a strong `JWT_SECRET`; rotate JWTs deliberately if leaked.
- Never expose Mongo connection strings or `JWT_SECRET` in the frontend bundle (`VITE_*` is public by design).

## Behavioral notes

- **Login**: uses `/api/auth/login` (JWT). If the API is unreachable, the app only falls back to guest/demo mode when `VITE_ALLOW_DEMO_LOGIN=true`; otherwise the user sees an error (no silent “everyone in” session).
- **Signup**: optional **emergency contact** (name + phone) is stored on the user and shown on the dashboard session card for API accounts.
- **Dashboard profile**: JWT users hydrate IMEI/model (and emergency contact when set) from the API; failures show the modal + local random device values + retry.
- **Maps**: without `VITE_GOOGLE_MAPS_API_KEY`, a glass placeholder appears (by design).
