# Singi Labs — Engineering

The shared engineering harness for every Singi Labs repo (Barazo + Sifa). One
canonical home; product docs and per-repo files link here rather than copying, so
nothing drifts.

| Doc | What it's for |
|---|---|
| [`harness.md`](harness.md) | How we engineer with AI agents — the operating model, what the harness enforces, what you inherit vs. what's maintainer-only. Read this first. |
| [`agent-instructions.md`](agent-instructions.md) | Drop-in `CLAUDE.md` / `AGENTS.md` for your local clone so your coding agent stays in bounds. |
| [`templates/pr-gates.yml`](templates/pr-gates.yml) | Caller workflow a repo adds to run the shared secret scan on every PR. |

See also, at the org root: [`CONTRIBUTING.md`](../CONTRIBUTING.md) ·
[`CODE_OF_CONDUCT.md`](../CODE_OF_CONDUCT.md) · [`CLA.md`](../CLA.md) ·
[`ARCHITECTURE.md`](../ARCHITECTURE.md)

## The harness has two halves

- **Prose** — the docs above and each repo's `AGENTS.md`. Tells you the rules.
- **Executable** — git hooks, CI gates, secret scanning, branch protection, the
  automated reviewer. *Enforces* the rules without depending on anyone remembering.

The executable half is the one that holds. See `harness.md` for why.

## Adding the shared secret scan to a repo

Copy [`templates/pr-gates.yml`](templates/pr-gates.yml) to the repo as
`.github/workflows/pr-gates.yml`. It calls the org's reusable
[`secret-scan`](../.github/workflows/secret-scan.yml) workflow — change the scan
once here and every repo that references it gets the update.
