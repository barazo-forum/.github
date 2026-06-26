# Contributing to Singi Labs

Thanks for your interest in contributing. Singi Labs builds open foundations for
networked apps on the AT Protocol — **Barazo** (federated forum) and **Sifa**
(professional identity). This guide covers the shared workflow for every repo in
the org. Product- and repo-specific setup lives in each repo's `README.md` and
`CLAUDE.md`.

## 1. One-time: sign the CLA

Your first pull request triggers the **CLA Assistant** bot. Sign by commenting on
the PR, exactly:

```
I have read the CLA Document and I hereby sign the CLA
```

It records your signature once and covers all future PRs across the org. The CLA
([CLA.md](CLA.md)) confirms contributions are voluntary (no equity, profit, or
ownership) and lets us license the project — read it before signing.

## 2. Access & ground rules

- External contributors fork; invited collaborators get repo access on the
  relevant team.
- **Never push to `main`.** Work on a feature branch and open a PR.
- **You cannot merge your own PRs** — every PR needs a maintainer's review.
- One branch = one feature.

## 3. Local setup

Prereqs are typically Node.js (current LTS or as pinned in the repo), Docker +
Docker Compose, and Git. **Exact setup — package manager, services, env vars — is
in the repo's `README.md` / `CLAUDE.md`.** A common shape:

```bash
git clone https://github.com/singi-labs/<repo>.git
cd <repo>
# install deps (npm ci / pnpm install — see the repo)
# start local services + dev server (see the repo)
# verify before opening a PR:
#   <test>   <lint>   <typecheck>
```

> **AT Protocol auth is OAuth, never app passwords** — org-wide rule, no exceptions.

## 4. Workflow

1. Pick an issue (look for `good first issue`). Comment that you're taking it.
2. Branch from `main`: `git checkout -b feat/short-description`.
3. Build it **with tests**. Run the repo's test/lint/typecheck locally.
4. Open a PR, link the issue (`Closes #NN`), fill the checklist.
5. **Two things review your PR:**
   - an **automated AI reviewer** posts a comment and a **blocking `AI Review`
     check** — if it requests changes, address them and push (it re-runs);
   - a **maintainer** reviews and merges (squash).
6. Push fixups to the same branch during review — no force-push.

**Conventional commits:** `type(scope): description` (`feat`, `fix`, `docs`,
`test`, `refactor`, `chore`, `ci`). Mark breaking changes with `!` or a
`BREAKING CHANGE:` footer.

## 5. Standards

How we engineer — the operating model, what the harness enforces, and what you
inherit — is in [`engineering/harness.md`](engineering/harness.md). If you use a
coding agent, drop [`engineering/agent-instructions.md`](engineering/agent-instructions.md)
into your local clone. The essentials the AI reviewer and maintainers enforce:

- **TypeScript strict** — no `any`, no `@ts-ignore`; `as` casts need a comment.
- **Validate at every boundary** with Zod (inputs, env, AT Protocol records);
  derive types from the schema.
- **Tests required** — co-located `*.test.ts`, aim 80%+ coverage on new code.
- **No `console.*`** — use the project logger.
- **Sanitize user content**; **rate-limit** auth/write/expensive endpoints.
- **Accessibility** (UI) — semantic HTML, keyboard nav, WCAG 2.2 AA.
- For UI work, reuse existing `src/components/ui/` primitives and design tokens
  before adding new ones.

## 6. Getting help

- **Issues / PRs** for anything code-specific.
- A maintainer will share the direct channel for quick questions.
- Stuck more than ~30 minutes? Ask — it's faster for everyone.

## 7. Recognition

Contributors appear in the GitHub contributor graph and release notes. Thank you
for helping build open, user-owned infrastructure.
