---
name: template-local-setup
description: >-
  Bootstraps template-platform and template-services on macOS: clone, MongoDB,
  env, npm, localhost :3000/:3001. Use for first-time template setup or local dev.
compatibility: >-
  macOS. Node.js 20+ (22 recommended). Homebrew MongoDB Community or Docker.
  Two sibling git clones. Network for npm install and git clone.
metadata:
  author: JoseAngelChepo
  version: 1.0.0
  repos:
    platform: https://github.com/JoseAngelChepo/template-platform
    services: https://github.com/JoseAngelChepo/template-services
---

# Template monorepo — local setup (macOS)

Guide a developer from zero to a working stack: **API on port 3001**, **web on port 3000**, **MongoDB on localhost**.

## Repositories

| Repo | URL | Role |
|------|-----|------|
| template-platform | https://github.com/JoseAngelChepo/template-platform | Next.js frontend |
| template-services | https://github.com/JoseAngelChepo/template-services | NestJS API (`/api/v1`) |

Clone as **sibling directories** (same parent folder). Example parent: `~/Development/template`.

## Agent workflow

Copy this checklist and mark steps as you complete them:

```
- [ ] 1. Verify prerequisites
- [ ] 2. Clone both repositories
- [ ] 3. Install and start MongoDB (macOS)
- [ ] 4. Configure and run template-services
- [ ] 5. Configure and run template-platform
- [ ] 6. Smoke-test in the browser
- [ ] 7. (Optional) Google OAuth and Resend email
```

Work **in order**. Do not start the frontend until the API responds on port 3001.

---

## Step 1 — Prerequisites

Confirm or install:

| Requirement | Notes |
|-------------|--------|
| **macOS** | This skill targets Mac; adapt MongoDB steps on Linux/Windows if needed |
| **Git** | `git --version` |
| **Node.js** | **20+** required; **22 LTS** recommended (matches backend CI). Prefer `nvm`: `nvm install 22 && nvm use 22` |
| **npm** | Bundled with Node; use `npm ci` when `package-lock.json` exists. If `npm` is not in PATH, use `nvm` or Homebrew Node and verify with `which node` + `which npm` |
| **Homebrew** | For MongoDB via brew (or use Docker in [references/mongodb-mac.md](references/mongodb-mac.md)) |

Ports must be free: **3000** (web), **3001** (API), **27017** (MongoDB).

---

## Step 2 — Clone both repositories

```bash
mkdir -p ~/Development/template
cd ~/Development/template

git clone https://github.com/JoseAngelChepo/template-services.git
git clone https://github.com/JoseAngelChepo/template-platform.git
```

Expected layout:

```text
template/
├── template-services/
└── template-platform/
```

If the user already has one repo, clone only the missing sibling into the same parent.

---

## Step 3 — MongoDB on macOS (local)

Default connection used by the backend template:

`mongodb://127.0.0.1:27017/template`

**Preferred (Homebrew):** full install and service commands are in [references/mongodb-mac.md](references/mongodb-mac.md).

Quick path:

```bash
brew tap mongodb/brew
brew install mongodb-community@7.0
brew services start mongodb-community@7.0
mongosh --eval 'db.runCommand({ ping: 1 })'
```

If Homebrew Mongo fails (for example FCV mismatch with an existing global data dir), use a **manual project-specific Mongo setup** from `references/mongodb-mac.md` (custom `--dbpath`) or switch to Docker.

If `mongosh` ping succeeds, proceed. If MongoDB is not running, **stop** — the API will fail on startup.

---

## Step 4 — template-services (backend)

```bash
cd ~/Development/template/template-services
cp .env.example .env
```

Edit `.env` — **minimum for local dev**:

| Variable | Local value |
|----------|-------------|
| `MONGODB_URI` | `mongodb://127.0.0.1:27017/template` |
| `JWT_SECRET` | New long random string (not `change-me-*`) |
| `JWT_REFRESH_SECRET` | New long random string |
| `FRONTEND_URL` | `http://localhost:3000` |
| `CORS_ORIGIN` | `http://localhost:3000` |
| `PORT` | `3001` |

Generate secrets (example):

```bash
openssl rand -base64 48
```

Leave **Google** and **Resend** vars empty unless doing Step 7.

```bash
npm ci
npm run dev
```

**Success:** console shows `template-services running on port 3001`.

Verify (new terminal):

```bash
curl -s 'http://localhost:3001/api/v1/health'
```

Expect **200** with `"status":"ok"` and `"database":{"status":"up"}` (not connection refused).

Full env reference: [references/environment.md](references/environment.md).

---

## Step 5 — template-platform (frontend)

```bash
cd ~/Development/template/template-platform
cp .env.example .env.local
```

`.env.local` defaults are correct for local dev if the API runs on 3001:

| Variable | Value |
|----------|--------|
| `NEXT_PUBLIC_APP_URL` | `http://localhost:3000` |
| `NEXT_PUBLIC_API_URL` | `http://localhost:3001/api/v1` |

Only set `NEXT_PUBLIC_API_BASIC_AUTH` if the deployment adds HTTP Basic on public routes (not required for the stock template).

```bash
npm ci
npm run dev
```

**Success:** Next.js ready on **http://localhost:3000**.

---

## Step 6 — Smoke-test

| Check | How |
|-------|-----|
| Landing | Open http://localhost:3000 — public landing loads |
| Sign up | http://localhost:3000/sign-up — create a user (hits API) |
| Dashboard | After login, http://localhost:3000/dashboard — session works |
| API health | Backend terminal has no Mongo connection errors |

If sign-up fails with network errors, confirm API on 3001 and `NEXT_PUBLIC_API_URL` in `.env.local`.

---

## Step 7 — Optional integrations

### Google OAuth

1. Google Cloud Console → OAuth client (Web).
2. Authorized redirect URI: `http://localhost:3001/api/v1/auth/google/callback`
3. Set in `template-services/.env`:

   - `GOOGLE_CLIENT_ID`
   - `GOOGLE_CLIENT_SECRET`
   - `GOOGLE_CALLBACK_URL=http://localhost:3001/api/v1/auth/google/callback`

4. Restart `npm run dev` in template-services.

### Password reset (Resend)

1. Resend API key and verified sender domain.
2. Set `RESEND_API_KEY`, `EMAIL_FROM` (or `RESEND_FROM_EMAIL` + `RESEND_FROM_NAME`).
3. Restart backend.

---

## Running day-to-day

Use **two terminals**:

```bash
# Terminal A
cd template-services && npm run dev

# Terminal B
cd template-platform && npm run dev
```

Start **services first**, then **platform**.

---

## Troubleshooting

| Symptom | Likely fix |
|---------|------------|
| `ECONNREFUSED` on API calls | Start template-services; check port 3001 |
| MongoDB connection error | Start service; if Homebrew data dir is incompatible, use manual `mongod --dbpath` setup or Docker (see `references/mongodb-mac.md`) |
| CORS errors in browser | `CORS_ORIGIN=http://localhost:3000` in services `.env` |
| OAuth redirect mismatch | `GOOGLE_CALLBACK_URL` and Google Console URI must match exactly |
| `npm` command not found | Normalize PATH using `nvm` or Homebrew Node (`/opt/homebrew/bin`), then restart terminal |
| Stale Next cache | `npm run dev:clean` in template-platform |
| Port in use | `lsof -i :3000` / `lsof -i :3001` and stop conflicting process |

More detail: [references/troubleshooting.md](references/troubleshooting.md).

---

## Security reminders (tell the user)

- Never commit `.env` or `.env.local`.
- Replace placeholder JWT secrets before any shared or production deploy.
- Use a dedicated local DB name (`template`) — not a production cluster URI.

---

## When extending the template

- API contracts live in **template-services** (`docs/README.md`).
- UI and routes live in **template-platform** (`.cursor/rules/template-requirements.mdc`).
- After changing API shapes, update both repos and re-run smoke-test.
