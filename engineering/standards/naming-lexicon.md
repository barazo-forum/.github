# Naming Lexicon

Internal terminology for Barazo and Sifa. Use these terms in all docs, diagrams, specs, and conversations to eliminate ambiguity.

**Rule: Never use "Barazo" or "Sifa" alone.** Always qualify which entity you mean.

**Status:** Active
**Created:** 2026-03-05

---

## Entities

### Barazo

| Term | Definition |
|---|---|
| **Barazo Inc** | The company. The people, the business, the decision-maker. |
| **Barazo SaaS** | The hosted forum service at barazo.forum, run by Barazo Inc. |
| **Barazo software** | The open-source codebase: barazo-api, barazo-web, barazo-deploy. |
| **Barazo instance** | One running deployment of Barazo software. Use when the hosting model (SaaS or self-hosted) is irrelevant. |
| **Barazo Discover** | The cross-community discovery surface. A Barazo instance without a community DID filter, sorted by popularity. Runs on a subdomain of barazo.forum. |

### Sifa

| Term | Definition |
|---|---|
| **Sifa Inc** | The company behind sifa.id. Separate entity from Barazo Inc, same owner. |
| **Sifa app** | The product at sifa.id. Professional profiles, feed, and endorsements. |

### Shared

| Term | Definition |
|---|---|
| **AT Protocol user** | Any person identified by a DID, regardless of which AT Protocol app they use. |

---

## Roles

| Term | Definition |
|---|---|
| **member** | A person who posts, replies, or votes in a Barazo forum. Identified by their DID. |
| **operator** | A person who runs a self-hosted Barazo instance. Responsible for the server. |
| **SaaS subscriber** | A person who pays Barazo Inc for hosted forums. |
| **community admin** | A person who administrates a specific community: categories, settings, branding. |
| **moderator** | A person who moderates content within a community. |
| **Barazo Inc team** | Barazo Inc employees or contributors. |
| **profile holder** | A person who has a professional profile on Sifa app. |
| **Sifa client** | A person or organization that pays Sifa Inc. Business model TBD. |
| **Sifa Inc team** | Sifa Inc employees or contributors. |

---

## Overlap Rules

One person can hold multiple roles simultaneously. A single DID might belong to a member of three forums, a profile holder on Sifa app, and a SaaS subscriber. The terms stack; they do not conflict.

When writing about a person, use the role relevant to the context. A SaaS subscriber who posts in their own forum is a "SaaS subscriber" in billing discussions and a "member" in content discussions.

---

## Scope

This lexicon covers internal usage: workspace docs, CLAUDE.md files, decision docs, diagrams, specs, and conversations with AI agents. Public-facing copy (marketing site, docs site, README files) may use simplified terms, but the internal terms remain the source of truth.
