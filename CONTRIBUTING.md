# How We Work – Team Rules

These rules apply to all three repos: `mca-frontend`, `mca-backend`
and `mca-infra-docs`.

## Branches

| Branch | Purpose |
|---|---|
| `main` | Stable, demo-ready code only. Nobody pushes here directly. |
| `develop` | Where finished features are combined and tested. |
| `feature/...` | One branch per task, e.g. `feature/login-page` |
| `bugfix/...` | Fixing a bug in how the app behaves, e.g. `bugfix/wrong-fee-total` |
| `fix/...` | Small corrections to config, values or data, e.g. `fix/compose-port` |
| `docs/...` | Documentation-only changes, e.g. `docs/api-error-format` |
| `chore/...` | Tooling, dependencies, CI or setup, e.g. `chore/update-prisma` |
| `refactor/...` | Restructuring code without changing behaviour, e.g. `refactor/services-helpers` |

**Never push directly to `main` or `develop`.** Always use a Pull Request.

### Branch names
- Lowercase, words separated by hyphens
- Start with `feature/`, `bugfix/`, `fix/`, `docs/`, `chore/` or `refactor/`
- Describe the task: `feature/service-list-page`, not `feature/my-work`

## Daily workflow

**1. Get the latest code before starting:**
```
git checkout develop
git pull
```

**2. Create your branch:**
```
git checkout -b feature/your-task-name
```

**3. Save your work often:**
```
git add .
git commit -m "feat: add login form"
```

**4. Push your branch to GitHub:**
```
git push -u origin feature/your-task-name
```

**5. Open a Pull Request** on GitHub from your branch into `develop`.

## Commit messages

Start with a type, then a short description of what changed:

| Type | Use for | Example |
|---|---|---|
| `feat:` | New feature | `feat: add service list page` |
| `fix:` | Bug fix | `fix: correct login error message` |
| `docs:` | Documentation | `docs: update setup guide` |
| `chore:` | Setup, config, tools | `chore: add prettier config` |
| `refactor:` | Code cleanup, same behaviour | `refactor: split auth controller` |
| `test:` | Adding tests | `test: add login endpoint tests` |

Write it so a teammate understands it without opening the code.

## Pull Requests

- One PR = one task. Keep them small.
- Target branch is `develop`, never `main`.
- Explain **what** you changed and **how to test it**.
- Link the issue, e.g. `Closes #4`.
- At least **1 teammate must approve** before merging.
- The author merges after approval and deletes the branch.

## Reviewing a teammate's PR
- Pull their branch and run it on your laptop.
- Check that it works and the code is readable.
- Comment kindly and specifically: "This could break if the email
  is empty" is better than "this is wrong".
- Approve only when you'd be happy to maintain the code yourself.

## Never commit
- `.env` files (passwords, secrets)
- `node_modules/`
- Personal test files

## Releases
At the end of each sprint, the team lead merges `develop` into `main`
and tags a version, e.g. `v0.1.0`.
