# Troubleshooting — template local stack

## Diagnostic order

1. MongoDB responding on `27017`
2. template-services listening on `3001`
3. template-platform on `3000` with correct `NEXT_PUBLIC_API_URL`
4. Browser network tab for failed requests

---

## Backend

### Nest fails at startup with Mongo error

- Run `mongosh --eval 'db.runCommand({ ping: 1 })'`
- Confirm `MONGODB_URI` host is `127.0.0.1` and port `27017`
- Restart Mongo: `brew services restart mongodb-community@7.0`
- If Homebrew Mongo still fails to boot (FCV/version mismatch), run manual Mongo with a project db path:

```bash
mkdir -p ~/Development/template/.mongo-data
mongod --dbpath ~/Development/template/.mongo-data --port 27017
```

### Port 3001 already in use

```bash
lsof -i :3001
kill <PID>
```

Or set `PORT=3002` in `.env` **and** update platform `NEXT_PUBLIC_API_URL` to match.

### `npm` not found in PATH

- Run `which node` and `which npm`
- If Node comes from app tooling but npm is missing, normalize via `nvm` (`nvm install 22 && nvm use 22`) or Homebrew Node
- For Homebrew on Apple Silicon, ensure `/opt/homebrew/bin` is in PATH and restart terminal

### JWT / auth oddities after copying `.env` from another project

Generate new `JWT_SECRET` and `JWT_REFRESH_SECRET`; old tokens will be invalid (expected).

---

## Frontend

### API calls fail with CORS

- `CORS_ORIGIN` must include `http://localhost:3000` (no trailing slash mismatch)
- API must be running before testing sign-up

### `NEXT_PUBLIC_*` changes ignored

Restart `npm run dev` — Next.js inlines public env at build/dev start.

### zsh breaks URLs with `?` in curl

Always quote URLs containing query params:

```bash
curl 'http://localhost:3001/api/v1/auth/username/availability?username=testuser'
```

### Blank or stale UI

```bash
npm run dev:clean
```

---

## Auth flows

### Sign-up returns 400

Read response body — often validation (username format, duplicate email). Username rules: lowercase `a-z`, digits, underscore, 3–30 chars.

### Google OAuth redirect error

- Google Console redirect URI must equal `GOOGLE_CALLBACK_URL`
- `FRONTEND_URL` must be `http://localhost:3000`
- Restart backend after env changes

### Password reset email never arrives

Resend vars must be set; check Resend dashboard logs. Optional for initial local setup.

---

## Git / security

- `.env` and `.env.local` must not appear in `git status`
- If `.env` was ever committed, rotate all secrets before pushing
