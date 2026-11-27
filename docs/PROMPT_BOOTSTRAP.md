# Congelator — Premier prompt Codex (BOOT-001)

Read [AGENTS.md](../AGENTS.md), [CONTRIBUTING.md](../CONTRIBUTING.md), and docs/PRODUCT.md, ARCHITECTURE.md, TESTING.md, INFRASTRUCTURE.md, SECURITY.md, ROADMAP.md and the ADR index.

Implement BOOT-001 only. Name the workspace `congelator` and use `@congelator` as the internal package scope. Start by inspecting the repository and proposing a concise plan, including compatible stable Node/pnpm/Nx/NestJS/React/Vite/Vitest versions verified against current official documentation. Initialize a pnpm + Nx TypeScript strict monorepo with a minimal NestJS API and React/Vite web app, basic Nx architectural lint boundaries, minimal tests, lint/typecheck/build and GitHub Actions CI. Establish clear local startup instructions and commit-ready lockfile. Keep the existing documentation. No auth, business modules, database migrations, AI calls, Redis, worker implementation, live deployment, cloud provisioning or paid services in this ticket. Do not overwrite existing user files.

Verify the actual commands and report results, failures, version decisions, compatibility issues, changed files and remaining risks. Do not claim unrun tests pass. Follow the approval rules in the contribution guide. If a substantial architecture deviation is needed, stop and explain the options instead of silently changing the decision.
