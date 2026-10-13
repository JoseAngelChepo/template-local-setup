---
name: template-local-setup
description: >-
  Bootstraps template-platform and template-services on macOS: clone, existing local MongoDB or Atlas,
  env, npm, localhost :3000/:3001, smoke tests, builds, and dependency audits.
  Use for first-time template setup, local development, or validating a fresh
  clone before handoff.
---

# Template monorepo — local setup (macOS)

Guide a developer from zero to a working stack: **API on port 3001**, **web on port 3000**, **MongoDB local or Atlas**.

Target macOS with Node.js 20.9+ (22 recommended), npm, and two sibling Git clones.
MongoDB can be an existing local service, a user-provided Atlas connection, or
a new local installation when the user chooses that option.

## Repositories

| Repo | URL | Role |
|------|-----|------|
| template-platform | https://github.com/JoseAngelChepo/template-platform | Next.js frontend |
| template-services | https://github.com/JoseAngelChepo/template-services | NestJS API (`/api/v1`) |

Clone as **sibling directories** (same parent folder). Example parent: `~/Development/template`.

## MongoDB choice — before any MongoDB installation

If the user has not already specified their database setup, ask:

> ¿MongoDB ya está instalado y quieres usarlo, vas a proporcionar la URL de
> MongoDB Atlas, o prefieres instalar MongoDB localmente?

Honor an existing choice without asking again. Do not install or start MongoDB
when the user selects Atlas, provides an existing connection, or says not to
install it. Homebrew and Docker are optional local choices, not prerequisites
for Atlas. If the user will add the URL themselves, prepare the ignored backend
`.env` with `MONGODB_URI=` and fresh JWT secrets, tell them its exact path, and
continue dependency/build checks while waiting. Do not request credentials in
chat, print the URI, or claim runtime validation before the API health check
reports the database up.

## Agent workflow

Copy this checklist and mark steps as you complete them:

```
- [ ] 1. Verify prerequisites
- [ ] 2. Clone both repositories
- [ ] 3. Resolve MongoDB connection (existing local / Atlas / chosen local install)
- [ ] 4. Configure and run template-services
- [ ] 5. Configure and run template-platform
- [ ] 6. Smoke-test in the browser
- [ ] 7. Validate builds and dependency security
- [ ] 8. (Optional) Google OAuth and Resend email
```

Work **in order**. Do not start the frontend until the API responds on port 3001.

---

## Step 1 — Prerequisites

Confirm or install:

| Requirement | Notes |
|-------------|--------|
| **macOS** | This skill targets Mac; adapt MongoDB steps on Linux/Windows if needed |
| **Git** | `git --version` |
| **Node.js** | **20.9+** required; **22 LTS** recommended (matches backend CI). Prefer `nvm`: `nvm install 22 && nvm use 22` |
| **npm** | Bundled with Node; use `npm ci` when `package-lock.json` exists. If `npm` is not in PATH, use `nvm` or Homebrew Node and verify with `which node` + `which npm` |
| **Homebrew / Docker** | Only needed if the user chooses a new local MongoDB installation; see [references/mongodb-mac.md](references/mongodb-mac.md) |

Check listeners and their project before choosing ports: **3000** (web), **3001** (API).
Check **27017** only for a local MongoDB setup. Reuse the chosen existing MongoDB
service; do not stop another project's process to free a port.

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

## Step 3 — Resolve the chosen MongoDB connection

### Existing local MongoDB

Use the connection supplied by the user, or the template's local default
`mongodb://127.0.0.1:27017/template` when appropriate. Check that the selected
service responds; do not reinstall MongoDB or alter another project's database.
Use a dedicated database name for the new application.

### MongoDB Atlas

Read [references/mongodb-atlas.md](references/mongodb-atlas.md) for connection
setup, IP access, and TLS troubleshooting. Prepare `.env` in Step 4 and let the
user put their URL in `MONGODB_URI`. An Atlas setup does not need `mongosh`,
Homebrew, Docker, or port 27017.

### New local installation

Use [references/mongodb-mac.md](references/mongodb-mac.md) only after the user
chooses a local installation. It covers Homebrew, Docker and a project-specific
manual data directory. Verify the selected local service with:

```bash
mongosh --eval 'db.runCommand({ ping: 1 })'
```

For any provider, the readiness criterion is the backend health response with
`database.status: up`. If no usable connection is available yet, finish env and
build preparation, explain the pending database step, and defer runtime tests.

---

## Step 4 — template-services (backend)

```bash
cd ~/Development/template/template-services
npm run setup-env
```

This generates fresh JWT secrets and preserves an existing `.env`. Do not use
`--force` or overwrite an existing env unless the user requests it. For Atlas
awaiting the user's URL, leave `MONGODB_URI=` empty and report that the file is
ready before attempting to start the API.

Edit `.env` — **minimum for local dev**:

| Variable | Local value |
|----------|-------------|
| `MONGODB_URI` | Chosen local URI or user-provided Atlas URI; leave empty while awaiting it |
| `JWT_SECRET` | New long random string (not `change-me-*`) |
| `JWT_REFRESH_SECRET` | New long random string |
| `FRONTEND_URL` | `http://localhost:3000` |
| `CORS_ORIGIN` | `http://localhost:3000` |
| `PORT` | `3001` |

Generate secrets (example):

```bash
openssl rand -base64 48
```

Leave **Google** and **Resend** vars empty unless doing Step 8.

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
# Only if .env.local does not already exist:
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
| Sign up | http://localhost:3000/sign-up — expect `{ user, pending: true }`; no authenticated session is issued |
| Dashboard | Log in with an active account after admin activation, then verify http://localhost:3000/dashboard and reload |
| API health | Backend terminal has no Mongo connection errors |

If sign-up fails with network errors, confirm API on 3001 and `NEXT_PUBLIC_API_URL` in `.env.local`.
New accounts are inactive in the stock template. Do not disable approval or
automatically promote users to make a smoke test pass. If no active account is
available, report the authenticated-flow check as pending admin activation.

---

## Step 7 — Validate before handoff

Stop both development servers before running production builds.

### Backend

```bash
cd template-services
npm run typecheck
npm run build
npm audit
npm run audit:security
```

Expect these commands to pass and `npm audit` to report zero
vulnerabilities.

### Frontend

```bash
cd ../template-platform
npm run typecheck
npm run lint
npm run build
npm run audit:security
npm audit
```

Expect typecheck, lint, build, and `audit:security` to pass. Inspect the full
`npm audit` separately and report development-tool advisories honestly.

The current development-only chain is `eslint-config-next →
@next/eslint-plugin-next → fast-glob → micromatch → braces`
([GHSA-vfj7-8cjw-p6xm](https://github.com/advisories/GHSA-vfj7-8cjw-p6xm)).
Recheck whether a compatible upstream fix exists; do not blindly run
`npm audit fix --force`, downgrade Next.js's ESLint configuration, or override
`brace-expansion` 5 into `minimatch` 3 to silence a warning. Document remaining
findings and keep production dependencies patched.

Use npm and its committed `package-lock.json` in both repositories. When
updating dependencies, raise vulnerable minimum versions in `package.json`
and regenerate the lockfile before `npm ci` and validation.

---

## Step 8 — Optional integrations

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
| MongoDB connection error | Follow the selected provider: local service checks or [Atlas connection checks](references/mongodb-atlas.md); do not replace Atlas with a local install |
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
- Use a dedicated development database, local or Atlas; do not reuse another project's database or expose connection credentials in logs.
- Preserve existing env files. Use `npm run setup-env` for first-time backend setup.

---

## When extending the template

- API contracts live in **template-services** (`docs/README.md`).
- UI and routes live in **template-platform** (`.cursor/rules/template-requirements.mdc`).
- After changing API shapes, update both repos and re-run smoke-test.

When this skill is loaded from its raw GitHub URL, resolve `references/` links
against the same repository and branch; they are not paths inside the app repo.
