---
name: setup-github
description: Scaffold Terraform-based GitHub repository infrastructure (repo settings, branch protection, labels, topics, secrets, security). Use when user wants to set up GitHub infra, create infra/github, manage a GitHub repo with Terraform, or bootstrap repository IaC.
---

# Setup GitHub Infrastructure

Generates a complete `infra/github/` directory with Terraform IaC for managing a GitHub repository.

## Required inputs

Collect from the user before generating:

| Input | Example | Required |
|-------|---------|----------|
| `GITHUB_OWNER` | `my-org` | Yes |
| Repository name | `my-app` | Yes |
| Repository description | `My application` | Yes |
| Visibility | `private` / `public` | Yes |
| Default branch | `main` | Yes (default: `main`) |

## Optional inputs (use defaults if not provided)

| Input | Default |
|-------|---------|
| Branch protections | Linear history on `main`, no reviews |
| Labels | Empty list |
| Topics | Empty list |
| Secrets | Empty map |
| Variables | Empty map |
| Security settings | All enabled |

## Workflow

1. **Collect inputs** — Ask for required values above
2. **Generate files** — Create all files in `infra/github/` using templates from [REFERENCE.md](REFERENCE.md)
3. **Write config.json** — Populate `infra/github/src/config.json` with user values
4. **Create .env placeholder** — Generate `.env` with `GITHUB_TOKEN=<placeholder>` and `GITHUB_OWNER=<value>`
5. **Generate workflow** — Create `.github/workflows/github-infra-deploy.yml`
6. **Instruct user** — Tell them to:
   - Replace the token placeholder in `.env`
   - Run `npx dotenvx encrypt` to encrypt
   - Add `DOTENV_PRIVATE_KEY` as a GitHub Actions secret (from `.env.keys`)
   - Run `bash import.sh` to bootstrap

## File structure produced

```
infra/github/
├── .env              # Credentials (encrypted with dotenvx)
├── .env.keys         # Decryption key (gitignored)
├── .gitignore        # Ignores state, keys, tfvars
├── README.md         # Setup and usage docs
├── import.sh         # Bootstrap script (create/import + apply)
└── src/
    ├── providers.tf  # GitHub provider
    ├── variables.tf  # Input variable definitions
    ├── main.tf       # Repository resources
    └── config.json   # Configuration values

.github/workflows/
└── github-infra-deploy.yml  # CI workflow for auto-deploying infra changes
```

## Important conventions

- **Encryption**: Always use `dotenvx` for `.env` — never commit plaintext tokens
- **Token type**: Fine-grained PAT with Administration, Actions, Contents, Issues, Workflows permissions
- **Bootstrap**: Use `import.sh` which handles create-or-import + apply in one step
- **State**: Terraform state is local and gitignored
- **Merge strategy**: Default to squash-only merges with PR title as commit title

## Post-generation checklist

Tell the user:

1. Create a fine-grained PAT at https://github.com/settings/tokens/new
2. Set token in `.env` and run `npx dotenvx encrypt`
3. Add `DOTENV_PRIVATE_KEY` (from `.env.keys`) as a repository secret in GitHub Actions
4. Run `bash infra/github/import.sh` to bootstrap
5. Future changes: edit `config.json`, push to `main` — the workflow auto-applies

See [REFERENCE.md](REFERENCE.md) for complete file templates.
