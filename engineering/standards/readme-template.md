# README Template for Barazo Repositories

All Barazo repos (`barazo-api`, `barazo-web`, `barazo-lexicons`, `barazo-deploy`, `barazo-website`, `barazo-docs`) follow this template. When updating any README, use this as the reference. When changing the template itself, update ALL repo READMEs to match.

## Conventions

1. **Logo**: All repos use the theme-aware `<picture>` element. Never omit it.
2. **Badges**: Standard set in this order: Status, License, CI (if workflow exists), Node.js (code repos), TypeScript (code repos). Use shields.io format.
3. **Section separators**: `---` between every major section.
4. **Related Repositories**: Always a table. Exclude the current repo from the list.
5. **Community**: Always present, identical across repos (except issues link).
6. **License**: Bold license name + link to LICENSE file. No rationale or justification -- that belongs in decision docs, not public READMEs.
7. **Copyright**: `(c) 2026 Barazo` at the bottom of every README.
8. **Section order**: Follow the ordering below exactly. Omit sections that don't apply (e.g., no "Tech Stack" for deploy). Never reorder.
9. **Headings**: Use `##` for all sections. No `###` at the top level.
10. **Tables**: Prefer tables over bullet lists for structured data (tech stack, features, routes).

## Template

```markdown
<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/barazo-forum/.github/main/assets/logo-dark.svg">
  <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/barazo-forum/.github/main/assets/logo-light.svg">
  <img alt="Barazo Logo" src="https://raw.githubusercontent.com/barazo-forum/.github/main/assets/logo-dark.svg" width="120">
</picture>

# {Repo Name}

**{One-line tagline. Consistent across docs.}**

[![Status: Alpha](https://img.shields.io/badge/status-alpha-orange)]()
[![License: {LICENSE}]({badge_url})]({license_url})
[![CI]({ci_badge_url})]({ci_actions_url})
[![Node.js](https://img.shields.io/badge/node-24%20LTS-brightgreen)](https://nodejs.org/)
[![TypeScript](https://img.shields.io/badge/typescript-5.x-blue)](https://www.typescriptlang.org/)

</div>

---

## Overview

{2-4 sentences: what this repo is, what role it plays in the Barazo ecosystem,
and who the audience is (developers, admins, self-hosters, etc.)}

---

## {Repo-specific sections}

{Content unique to this repo: tech stack, features, API routes, schemas,
deployment modes, etc. Use tables for structured data. Keep sections
focused and scannable.}

---

## Quick Start

{Clone, install, configure, run. Keep it copy-pasteable. Include
prerequisites inline rather than as a separate section.}

---

## Development

{Test/lint/build commands as a code block. Link to CONTRIBUTING.md.
List 3-5 key standards as bullet points.}

---

## Related Repositories

{Table format. Exclude the current repo.}

| Repository | Description | License |
|------------|-------------|---------|
| ... | ... | ... |

---

## Community

- **Website:** [{product-domain}]({product-url})
- **Bluesky:** [@{product-bsky-handle}](https://bsky.app/profile/{product-bsky-handle})
- **Issues:** [Report bugs](https://github.com/singi-labs/{repo}/issues)

---

## License

**{License Name}** -- {One sentence: why this license.}

See [LICENSE](LICENSE) for full terms.

---

(c) 2026 Barazo
```

## Badge Reference

| Badge | Markdown | When to include |
|-------|----------|-----------------|
| Status | `[![Status: Alpha](https://img.shields.io/badge/status-alpha-orange)]()` | Always |
| AGPL-3.0 | `[![License: AGPL-3.0](https://img.shields.io/badge/License-AGPL%203.0-blue.svg)](https://opensource.org/licenses/AGPL-3.0)` | barazo-api only |
| MIT | `[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)` | All other repos |
| CI | `[![CI](https://github.com/singi-labs/{repo}/actions/workflows/ci.yml/badge.svg)](https://github.com/singi-labs/{repo}/actions/workflows/ci.yml)` | Repos with CI workflow |
| Node.js | `[![Node.js](https://img.shields.io/badge/node-24%20LTS-brightgreen)](https://nodejs.org/)` | Code repos (api, web, lexicons) |
| TypeScript | `[![TypeScript](https://img.shields.io/badge/typescript-5.x-blue)](https://www.typescriptlang.org/)` | Code repos (api, web, lexicons) |

## Section Order

Mandatory sections (all repos):

1. Header (logo, name, tagline, badges)
2. Overview
3. Quick Start
4. Development
5. Related Repositories
6. Community
7. License

Optional sections (between Overview and Quick Start, repo-specific):

- Tech Stack
- Features / Implemented Features
- Route Modules, Database Schema, etc.
- Planned Features
- API Documentation
- Deployment modes, scripts, environment variables
- Schemas, package exports, installation

## Updating

When this template changes:

1. Update this file
2. Regenerate all 6 repo READMEs to match
3. Commit template change to workspace
4. Create PRs for each repo README
