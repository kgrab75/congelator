# Contributing to Congelator

Congelator uses a public repository and currently has no license.

## Branches and pull requests

Use trunk-based development. Start a short-lived branch from `main` for each ticket. `main` must be protected, and all changes must go through a pull request. Only the maintainer may merge pull requests, using **Squash and merge**. While the project has a single maintainer, zero external approving reviews are required. Enable required CI checks once the CI workflows exist.

Branch naming examples

- `feat/household-creation`
- `fix/stock-quantity-rounding`
- `chore/boot-001`
- `docs/inventory-model`
- `refactor/quantity-value-object`
- `test/household-isolation`

Use Conventional Commits for commit messages and pull request titles. For example, `docs: align project documentation and development workflow`.

## Review and authorization

Coding agents must obtain explicit human approval before committing, pushing, or creating a pull request. Report the changes, decisions, checks and actual results, and any remaining issues for review. Only the maintainer may authorize production deployments.

Keep secrets, personal information, and production-specific infrastructure details out of the repository. See [AGENTS.md](AGENTS.md) for coding-agent rules and the [roadmap](docs/ROADMAP.md) for ticket scope and delivery gates.
