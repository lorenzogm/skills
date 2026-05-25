---
name: setup-vercel
description: Scaffold Terraform-based Vercel project infrastructure and GitHub Actions deploy pipeline (preview → e2e → production). Use when user wants to set up Vercel infra, create infra/vercel, deploy to Vercel with Terraform, or bootstrap a Vercel CI/CD workflow.
---

# Setup Vercel Infrastructure

Generates a complete `infra/vercel/<app>/` directory with Terraform IaC for managing a Vercel project plus a GitHub Actions workflow for automated deployments.

## Required inputs

Collect from the user before generating:

| Input | Example | Required |
|-------|---------|----------|
| App name (directory slug) | `web` | Yes |
| Vercel project name | `my-app-web` | Yes |
| Root directory (monorepo path) | `apps/my-app` | Yes |
| Build command | `npm run build` | Yes |
| Output directory | `dist` or `.next` | Yes |
| Framework | `nextjs` / `astro` / `null` | Yes |
| Production domain(s) | `example.com` | No |
| Redirect domains | `www.example.com → example.com` | No |
| Environment variables | `{ "KEY": "value" }` | No |
| E2E test filter/command | `pnpm --filter my-app run test:e2e` | No |

## Optional inputs (use defaults if not provided)

| Input | Default |
|-------|---------|
| Install command | (Vercel default) |
| Public source | `false` |
| Auto-assign custom domains | `true` |
| Vercel authentication | `{ "deployment_type": "none" }` |
| Package manager | `pnpm` |
| Node version | `24` |
| Terraform version | `1.10.5` |

## Workflow

1. **Collect inputs** — Ask for required values above
2. **Generate infra files** — Create all files in `infra/vercel/<app>/` using templates from [REFERENCE.md](REFERENCE.md)
3. **Write config files** — Populate `config.development.json` and `config.production.json`
4. **Create .env files** — Generate `.env.development` and `.env.production` with env var placeholders
5. **Generate workflow** — Create `.github/workflows/<app>-vercel-deploy.yml`
6. **Instruct user** — Tell them to:
   - Set tokens/secrets in `.env.development` and `.env.production`
   - Run `npx dotenvx encrypt` in the infra directory
   - Add `DOTENV_PRIVATE_KEY` as a GitHub Actions secret (from `.env.keys`)
   - Push to `main` to trigger the pipeline

## File structure produced

```
infra/vercel/<app>/
├── .env.development   # Dev credentials (encrypted with dotenvx)
├── .env.production    # Prod credentials (encrypted with dotenvx)
├── .env.keys          # Decryption keys (gitignored)
├── .gitignore         # Ignores state, keys, tfvars
└── src/
    ├── providers.tf              # Vercel provider
    ├── variables.tf              # Input variable definitions
    ├── main.tf                   # Vercel project + domains + env vars
    ├── config.development.json   # Dev configuration values
    └── config.production.json    # Prod configuration values

.github/workflows/
└── <app>-vercel-deploy.yml  # Full deploy pipeline
```

## Pipeline structure (5 jobs)

1. **check-changes** — Detects infra/app changes, resolves environment
2. **infra-deploy** — Terraform init → import → plan → apply; encrypted state on orphan branch
3. **deploy-preview** — `vercel build` + `vercel deploy --prebuilt`
4. **e2e-tests** — Playwright tests against preview URL
5. **deploy-production** — `vercel deploy --prebuilt --prod`

## Important conventions

- **Encryption**: Always use `dotenvx` for `.env` files — never commit plaintext tokens
- **State management**: Terraform state encrypted with AES-256-CBC, stored on `terraform-state` orphan branch
- **Environment naming**: Projects suffixed with `-dev` / `-prod`
- **Secrets required in .env**: `VERCEL_TOKEN`, `VERCEL_ORG_ID`, `TERRAFORM_STATE_ENCRYPT_KEY`, plus any app env vars
- **GitHub secret**: Only `DOTENV_PRIVATE_KEY` needs to be added as a GitHub Actions secret

## Post-generation checklist

Tell the user:

1. Create a Vercel API token at https://vercel.com/account/tokens
2. Generate a state encryption key: `openssl rand -base64 32`
3. Set secrets in `.env.development` / `.env.production` and run `npx dotenvx encrypt`
4. Add `DOTENV_PRIVATE_KEY` (from `.env.keys`) as a repository secret in GitHub Actions
5. Push to `main` — the workflow auto-detects changes and deploys

See [REFERENCE.md](REFERENCE.md) for complete file templates.
