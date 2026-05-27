# template-local-setup

Claude **Agent Skill** for bootstrapping the template monorepo on **macOS**: clone [template-platform](https://github.com/JoseAngelChepo/template-platform) and [template-services](https://github.com/JoseAngelChepo/template-services), install local MongoDB, configure env vars, and run both apps on localhost.

## Repositories

| Project | GitHub |
|---------|--------|
| Frontend (Next.js) | https://github.com/JoseAngelChepo/template-platform |
| Backend (NestJS) | https://github.com/JoseAngelChepo/template-services |

## Install the skill in Claude

This repo follows the [Agent Skills](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview) layout: the skill lives at the repository root (`SKILL.md` + `references/`).

### Claude Code

Clone and register the skill path:

```bash
git clone https://github.com/JoseAngelChepo/template-local-setup.git
```

Add the cloned directory to your project's Claude Code skills configuration, or place it where your client loads custom skills (see [Claude Code skills docs](https://docs.anthropic.com/en/docs/claude-code/skills)).

### Claude.ai (custom skills)

1. Zip the repo contents (must include `SKILL.md` at the root of the zip).
2. **Settings → Capabilities → Skills → Upload skill**.
3. Invoke when you ask to set up the template, run localhost, or configure Mongo/env for the monorepo.

### Cursor

Copy or symlink this folder to `.cursor/skills/template-local-setup/` in your workspace, or add to user skills at `~/.cursor/skills/template-local-setup/`.

## What the skill does

- Step-by-step checklist from prerequisites through browser smoke-test
- macOS MongoDB via Homebrew (with Docker fallback in `references/`)
- Env var tables for both repos
- Optional Google OAuth and Resend
- Troubleshooting reference

## Structure

```text
template-local-setup/
├── SKILL.md                 # Main skill (required)
├── references/
│   ├── mongodb-mac.md
│   ├── environment.md
│   └── troubleshooting.md
└── README.md                # This file (humans only)
```

Per Anthropic authoring guidance: keep operational detail in `SKILL.md` under 500 lines; deep reference material lives in `references/`.

## License

MIT — see [LICENSE](LICENSE).
