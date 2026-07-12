# Contributor agent instructions

Generic instruction file for contributors using Claude Code or a similar coding
agent. **Copy this into the root of your local clone** as `CLAUDE.md` (or
`AGENTS.md`) so your agent stays within the house rules. It references no private
Singi Labs context — safe to use anywhere.

> Do not commit this file to the repo. If a maintainer `AGENTS.md` already exists
> in the repo, that one wins for repo-specific detail — this file adds the
> behavioural guardrails.

## Project

You are contributing to a **Singi Labs** repo — open foundations for your online life
on AT Protocol. Products: **Barazo** (federated forum) and **Sifa**
(professional identity). The repo's own `README.md` / `AGENTS.md` has the exact
stack, package manager, and setup. General shape: TypeScript (strict), Node current
LTS, npm or pnpm per repo, PostgreSQL + Drizzle and Valkey on the backends, Next.js
/ React / Tailwind on the frontends.

## Hard rules (never violate)

- **Never push to `main`.** Branch (`feat/…`, `fix/…`) and open a PR.
- **Never commit secrets** — no `.env`, keys, tokens, JWKS, or credentials. A secret
  scan blocks the PR if you do.
- **AT Protocol auth = OAuth only.** Never use app passwords in code.
- **No AI attribution** in commit messages, PR titles/bodies, or comments. No
  `Co-Authored-By`, no "Generated with…" lines. Write neutral messages.
- **Conventional commits**: `type(scope): description` (`feat`, `fix`, `docs`,
  `test`, `refactor`, `chore`, `ci`).

## Engineering standards

- TypeScript strict — no `any`, no `@ts-ignore`; an `as` cast needs a comment.
- Write tests with the change; aim 80%+ coverage on new code.
- Validate input with Zod at every boundary; derive types from the schema.
- Sanitize user-generated output; rate-limit auth/write/expensive endpoints.
- Accessibility (UI): semantic HTML, keyboard nav, WCAG 2.2 AA.
- Use the project logger, not `console.*`.
- UI: reuse existing `src/components/ui/` primitives and design tokens before
  creating new ones.

## Before claiming a task is done

Run and confirm all pass — show the output, never assert "fixed"/"passing" without it:

```bash
npm test        # or the repo's test command
npm run lint
npm run typecheck
```

## Workflow

1. Read the linked issue. Confirm scope before large changes.
2. Branch from `main`, one feature per branch.
3. Implement with tests. Keep diffs focused.
4. Open a PR, `Closes #NN`, fill the checklist. An automated reviewer posts a
   blocking check (address what it requests); a maintainer reviews before merge —
   you cannot self-merge.

See [`harness.md`](harness.md) for the full operating model and what the harness
enforces.
