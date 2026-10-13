# MongoDB on macOS (template-services)

Use this reference only when the user chooses an existing or new **local** MongoDB setup. Resolve that choice in `SKILL.md` first; never install MongoDB as a fallback for Atlas without the user choosing it.

Default URI from `.env.example`:

```text
mongodb://127.0.0.1:27017/template
```

Database name `template` is created on first write. No manual DB creation required.

---

## Option A — Homebrew (recommended)

### Install

```bash
brew tap mongodb/brew
brew install mongodb-community@7.0
```

If `@7.0` is unavailable, use `brew search mongodb-community` and pick the latest stable formula shown.

### Start / stop

```bash
# Start at login (background service)
brew services start mongodb-community@7.0

# Stop
brew services stop mongodb-community@7.0

# One-off foreground (debugging)
mongod --config /opt/homebrew/etc/mongod.conf
```

Intel Macs may use `/usr/local/etc/mongod.conf` instead of `/opt/homebrew/etc/`.

### Verify

```bash
mongosh --eval 'db.runCommand({ ping: 1 })'
```

Expected: `{ ok: 1 }`.

### Shell access (optional)

```bash
mongosh
show dbs
use template
show collections
```

---

## Option B — Docker

When Homebrew is not desired:

```bash
docker run -d --name template-mongo \
  -p 127.0.0.1:27017:27017 \
  -v template-mongo-data:/data/db \
  mongo:7
```

Use the same URI: `mongodb://127.0.0.1:27017/template`.

Stop: `docker stop template-mongo`. Start again: `docker start template-mongo`.

---

## Option B2 — Manual local config (when Homebrew service fails)

Use this when Homebrew Mongo cannot start because an existing global data dir is incompatible (common after major upgrades, e.g. FCV mismatch).

```bash
mkdir -p ~/Development/template/.mongo-data
mongod --dbpath ~/Development/template/.mongo-data --bind_ip 127.0.0.1 --port 27017
```

Then verify in another terminal:

```bash
mongosh --eval 'db.runCommand({ ping: 1 })'
```

This keeps project data isolated from Homebrew's default global data directory.

---

## MongoDB Atlas

Atlas is an equal setup option. Use [mongodb-atlas.md](mongodb-atlas.md) and skip
local installation, service startup, and port 27017 checks.

---

## Common issues

| Issue | Fix |
|-------|-----|
| `Connection refused` on 27017 | Service not running — `brew services start mongodb-community@7.0` |
| FCV/version mismatch after Mongo upgrade | Use project-specific `mongod --dbpath` (Option B2) or Docker, then retry |
| Wrong architecture (Apple Silicon) | Reinstall via brew; avoid mixing Intel and ARM binaries |
| Port 27017 busy | `lsof -i :27017` — stop duplicate `mongod` or Docker container |
| Auth errors with local URI | Stock template uses **no auth** on localhost; do not add username/password unless you configured Mongo auth yourself |
