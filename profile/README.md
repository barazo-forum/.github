![Singi Labs](https://raw.githubusercontent.com/singi-labs/.github/main/assets/banner-singi.svg)

**Open source foundations for the parts of your online life you shouldn't have to rent.**

We build in the open on [AT Protocol](https://atproto.com). Portable identity, user-owned data, no ads.

[![GitHub Org Stars](https://img.shields.io/github/stars/singi-labs?style=flat&label=total%20org%20stars)](https://github.com/singi-labs)
[![Website](https://img.shields.io/badge/singi.dev-website-DA702C)](https://singi.dev)

---

## Products

![Sifa](https://raw.githubusercontent.com/singi-labs/.github/main/assets/banner-sifa.svg)

Professional identity and career network on the AT Protocol. Portable profiles, verifiable track record from real community contributions, no vendor lock-in. The LinkedIn alternative for the open web.

**Status:** Alpha (P1 MVP complete)
**License:** MIT (SDK, lexicons, page renderer, page)

| Repository | Description | CI | Updated |
|------------|-------------|----|---------|
| [sifa-sdk](https://github.com/singi-labs/sifa-sdk) | Public TypeScript client for the Sifa AppView (on npm) | [![CI](https://github.com/singi-labs/sifa-sdk/actions/workflows/ci.yml/badge.svg)](https://github.com/singi-labs/sifa-sdk/actions/workflows/ci.yml) | ![Updated](https://img.shields.io/github/last-commit/singi-labs/sifa-sdk?label=updated) |
| [sifa-lexicons](https://github.com/singi-labs/sifa-lexicons) | AT Protocol professional profile schemas (MIT) | [![Validate](https://github.com/singi-labs/sifa-lexicons/actions/workflows/validate.yml/badge.svg)](https://github.com/singi-labs/sifa-lexicons/actions/workflows/validate.yml) | ![Updated](https://img.shields.io/github/last-commit/singi-labs/sifa-lexicons?label=updated) |
| [sifa-page-renderer](https://github.com/singi-labs/sifa-page-renderer) | Pure HTML renderer for academicpages-style personal sites, driven by Sifa profile data (on npm) | | ![Updated](https://img.shields.io/github/last-commit/singi-labs/sifa-page-renderer?label=updated) |
| [sifa-page](https://github.com/singi-labs/sifa-page) | Self-hostable static-site scaffold for a personal academic site, driven by your Sifa profile | [![Deploy demo site](https://github.com/singi-labs/sifa-page/actions/workflows/deploy.yml/badge.svg)](https://github.com/singi-labs/sifa-page/actions/workflows/deploy.yml) | ![Updated](https://img.shields.io/github/last-commit/singi-labs/sifa-page?label=updated) |
| [sifa-docs](https://github.com/singi-labs/sifa-docs) | Documentation site (Fumadocs) | | ![Updated](https://img.shields.io/github/last-commit/singi-labs/sifa-docs?label=updated) |

[sifa.id](https://sifa.id) | [Documentation](https://docs.sifa.id)

![Barazo](https://raw.githubusercontent.com/singi-labs/.github/main/assets/banner-barazo.svg)

Federated forums on the AT Protocol. Self-hostable. One account works across every Barazo forum and Bluesky. Your posts live on your Personal Data Server, not locked inside any single platform.

**Status:** Alpha (Phase 2 complete)
**License:** AGPL-3.0 (backend) + MIT (frontend, lexicons, deploy)

| Repository | Description | CI | Updated |
|------------|-------------|----|---------|
| [barazo-api](https://github.com/singi-labs/barazo-api) | AppView backend (Fastify, PostgreSQL, AT Protocol) | [![CI](https://github.com/singi-labs/barazo-api/actions/workflows/ci.yml/badge.svg)](https://github.com/singi-labs/barazo-api/actions/workflows/ci.yml) | ![Updated](https://img.shields.io/github/last-commit/singi-labs/barazo-api?label=updated) |
| [barazo-web](https://github.com/singi-labs/barazo-web) | Forum frontend (Next.js, React, TailwindCSS) | [![CI](https://github.com/singi-labs/barazo-web/actions/workflows/ci.yml/badge.svg)](https://github.com/singi-labs/barazo-web/actions/workflows/ci.yml) | ![Updated](https://img.shields.io/github/last-commit/singi-labs/barazo-web?label=updated) |
| [barazo-lexicons](https://github.com/singi-labs/barazo-lexicons) | AT Protocol schema definitions | [![CI](https://github.com/singi-labs/barazo-lexicons/actions/workflows/ci.yml/badge.svg)](https://github.com/singi-labs/barazo-lexicons/actions/workflows/ci.yml) | ![Updated](https://img.shields.io/github/last-commit/singi-labs/barazo-lexicons?label=updated) |
| [barazo-deploy](https://github.com/singi-labs/barazo-deploy) | Docker Compose templates for self-hosting | [![Validate](https://github.com/singi-labs/barazo-deploy/actions/workflows/validate-compose.yml/badge.svg)](https://github.com/singi-labs/barazo-deploy/actions/workflows/validate-compose.yml) | ![Updated](https://img.shields.io/github/last-commit/singi-labs/barazo-deploy?label=updated) |
| [barazo-plugins](https://github.com/singi-labs/barazo-plugins) | Official plugins monorepo + starter template | [![CI](https://github.com/singi-labs/barazo-plugins/actions/workflows/ci.yml/badge.svg)](https://github.com/singi-labs/barazo-plugins/actions/workflows/ci.yml) | ![Updated](https://img.shields.io/github/last-commit/singi-labs/barazo-plugins?label=updated) |
| [barazo-docs](https://github.com/singi-labs/barazo-docs) | Documentation site (Fumadocs) | | ![Updated](https://img.shields.io/github/last-commit/singi-labs/barazo-docs?label=updated) |
| [barazo-website](https://github.com/singi-labs/barazo-website) | Marketing site | | ![Updated](https://img.shields.io/github/last-commit/singi-labs/barazo-website?label=updated) |

[barazo.forum](https://barazo.forum) | [Documentation](https://docs.barazo.forum)

---

## Principles

- **Portable identity** -- One account, everywhere
- **User-owned data** -- Your content lives on your PDS
- **Transparent by design** -- Open protocol, open code
- **No ads, ever** -- Your attention isn't the product

---

## Tech stack

TypeScript / Node.js / Fastify / PostgreSQL / Next.js / React / TailwindCSS / AT Protocol

---

## Contributing

Contributions are welcome. See [CONTRIBUTING.md](https://github.com/singi-labs/.github/blob/main/CONTRIBUTING.md) for guidelines.

### Contributors

<!-- CONTRIBUTORS:START -->
<a href="https://github.com/gxjansen"><img src="https://avatars.githubusercontent.com/u/487722?v=4&s=80" width="80" alt="@gxjansen" /></a>
<a href="https://github.com/StevenLangbroek"><img src="https://avatars.githubusercontent.com/u/296796?v=4&s=80" width="80" alt="@StevenLangbroek" /></a>
<a href="https://github.com/nmokkenstorm"><img src="https://avatars.githubusercontent.com/u/33529698?v=4&s=80" width="80" alt="@nmokkenstorm" /></a>
<!-- CONTRIBUTORS:END -->

---

## Links

- [singi.dev](https://singi.dev) -- Organization homepage
- [sifa.id](https://sifa.id) -- Sifa professional network
- [docs.sifa.id](https://docs.sifa.id) -- Sifa documentation
- [barazo.forum](https://barazo.forum) -- Barazo marketing site
- [docs.barazo.forum](https://docs.barazo.forum) -- Barazo documentation
- 🦋 [Bluesky](https://bsky.app/profile/singi.dev) -- @singi.dev

---

Made with ♥ in 🇪🇺
