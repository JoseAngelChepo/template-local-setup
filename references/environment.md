# Environment variables reference

## template-services (`.env`)

Use `npm run setup-env` to generate fresh JWT secrets without overwriting an existing `.env`. **Never commit** `.env`.

| Variable | Required (local) | Description |
|----------|------------------|-------------|
| `MONGODB_URI` | Yes | Chosen local URI or MongoDB Atlas URI with a dedicated app database. If the user will enter it, leave `MONGODB_URI=` blank until supplied. |
| `JWT_SECRET` | Yes | Access token signing — use a strong random value |
| `JWT_REFRESH_SECRET` | Yes | Refresh token signing — different from `JWT_SECRET` |
| `JWT_EXPIRES_IN` | No | Default `7d` |
| `JWT_REFRESH_EXPIRES_IN` | No | Default `30d` |
| `FRONTEND_URL` | Yes | `http://localhost:3000` — OAuth redirects, email links |
| `CORS_ORIGIN` | Yes | `http://localhost:3000` — comma-separated if multiple origins |
| `PORT` | No | Default `3001` |
| `GOOGLE_CLIENT_ID` | No | Google OAuth |
| `GOOGLE_CLIENT_SECRET` | No | Google OAuth |
| `GOOGLE_CALLBACK_URL` | No | Default `http://localhost:3001/api/v1/auth/google/callback` |
| `RESEND_API_KEY` | No | Password-reset email |
| `EMAIL_FROM` | No | Sender for Resend |
| `RESEND_FROM_EMAIL` | No | Alternative to `EMAIL_FROM` |
| `RESEND_FROM_NAME` | No | Display name for emails |
| `AGENT_KEY_PREFIX` | No | Prefix for user API tokens (e.g. `app_`) |

Generate JWT secrets:

```bash
openssl rand -base64 48
```

---

## template-platform (`.env.local`)

Copy from `.env.example`. Next.js loads `.env.local` in development.

| Variable | Required (local) | Description |
|----------|------------------|-------------|
| `NEXT_PUBLIC_ENV` | No | e.g. `development` |
| `NEXT_PUBLIC_APP_URL` | Yes | `http://localhost:3000` |
| `NEXT_PUBLIC_API_URL` | Yes | `http://localhost:3001/api/v1` |
| `NEXT_PUBLIC_API_BASIC_AUTH` | No | `username:password` only if your API layer expects HTTP Basic on unauthenticated client calls |

---

## Cross-repo alignment

| Concern | Backend | Frontend |
|---------|---------|----------|
| Web origin | `FRONTEND_URL`, `CORS_ORIGIN` | `NEXT_PUBLIC_APP_URL` |
| API base | `PORT=3001`, prefix `/api/v1` | `NEXT_PUBLIC_API_URL` |
| OAuth callback | `GOOGLE_CALLBACK_URL` on API host | User lands on `/auth/google/callback` on web |

Mismatch on URLs is the most common local setup bug.
