# Barazo - Backend Engineering Standards

**Created:** 2026-02-09
**Status:** Active - enforce from first commit

Standards specific to barazo-api (AppView backend). Read standards/shared.md first.

---

## Security Measures

### Input Validation (Zod)
- **Every API endpoint** validates input with Zod schemas
- **Every firehose record** validated against lexicon schema before indexing
- Reject malformed data early, log the rejection, don't process it
- No raw string interpolation into SQL (use parameterized queries via Drizzle ORM)

### Output Sanitization
- **DOMPurify** for all user-generated HTML/markdown before rendering
- Note: DOMPurify requires jsdom for server-side use. Plan for periodic jsdom window cleanup in long-running processes (AppView) to prevent memory accumulation. Do NOT use happy-dom with DOMPurify (known XSS vectors).
- Sanitize AT Protocol display names and profile data before display
- CSP headers to prevent inline script execution
- **DOMPurify configuration (explicit):** use restrictive ALLOWED_TAGS and ALLOWED_ATTR lists (not permissive defaults). Strip all event handlers, `javascript:` URIs, `data:` URIs for non-image contexts.
- **DOMPurify instance lifecycle:** singleton instance with shared jsdom window. Recreate window every 10,000 sanitizations or every hour (whichever comes first) to prevent memory accumulation.
- **Sanitization order:** markdown -> HTML conversion FIRST, then DOMPurify sanitization on the HTML output (never sanitize raw markdown).
- **Unicode normalization:** NFC normalize all text input before validation AND storage. Strip bidirectional override characters (U+202A-U+202E, U+2066-U+2069) and other control characters.

### HTTP Security (Helmet)
- Helmet middleware on all routes
- Content-Security-Policy (strict, no inline scripts)
- X-Content-Type-Options: nosniff
- X-Frame-Options: DENY
- Strict-Transport-Security (HSTS)
- Referrer-Policy: strict-origin-when-cross-origin

### Rate Limiting
- Per-IP rate limiting on all API endpoints
- Stricter limits on write endpoints (create topic, post reply)
- Auth endpoint rate limiting (prevent brute force)
- Firehose processing rate limiting (prevent resource exhaustion from spam records)
- Per-DID rate limiting for authenticated requests (primary limiter, harder to bypass than per-IP)
- **New account rate limiting:** Accounts with < 7 days of indexed activity in a community get stricter write limits (3 writes/min vs 10 for established accounts). "New" is per-community, not global. See `decisions/content-moderation.md` Bot & Spam Prevention Strategy.
- Search endpoints: stricter limits (20 req/min unauthenticated, 60 req/min authenticated) separate from general API limits
- Auth endpoints: 10 req/min per IP for `/api/auth/login`, 5 req/min for `/api/auth/callback`
- Exponential backoff on repeated rate limit violations
- Per-post mention limit: maximum 10 unique @mentions per topic/reply

### Authentication & Authorization
- AT Protocol OAuth with DPoP token binding (as per spec)
- PKCE required on all OAuth flows (code_verifier + code_challenge with S256)
- State parameter: cryptographically random (minimum 32 bytes), bound to Valkey session, 5-minute expiry
- Role-based authorization for moderation actions
- Verify DID ownership on every authenticated request:
  - Resolve DID document (PLC directory or DNS) with caching (TTL: 1 hour)
  - Fail closed: if DID resolution fails, reject the request (do not serve from stale cache beyond TTL)
  - Monitor PLC audit log for key rotation events; invalidate cached DID documents on rotation
- **Token persistence strategy (hybrid):**
  - Refresh token: HTTP-only, Secure, SameSite=Strict cookie (never accessible to JavaScript)
  - Access token: short-lived (15 minutes), held in memory only (React state/context)
  - Token refresh: silent refresh via `/api/auth/refresh` using HTTP-only cookie
  - Never store access tokens in localStorage or sessionStorage
- Session management: Valkey-backed sessions, 7-day refresh token expiry (configurable)
- CSRF: Bearer tokens in Authorization header are inherently CSRF-safe. The refresh endpoint (cookie-based) uses SameSite=Strict. If SameSite is insufficient for any future flow, add CSRF tokens.
- Session invalidation: on account deletion (`DELETE /api/users/me` or firehose `#account` deletion), immediately delete ALL Valkey sessions for that DID

### Data Deletion (Right to Erasure)
- Process firehose deletion events per `specs/prd-api.md` Firehose Consumer section (`#commit` delete + `#account` deletion; do NOT implement deprecated `#tombstone`)
- Provide direct deletion request mechanism (contact form / AT Protocol notification) independent of firehose
- Delete from live/production systems immediately upon valid request
- Backup data: apply deletions within 30 days (data "beyond use" pending natural expiration)
- If backup is restored, re-apply all deletions that occurred since backup creation
- Response to direct requests within one month (GDPR Art. 12(3))

### Dependency Security
- Dependabot enabled with weekly schedule (migrate to Renovate if moving off GitHub)
- `pnpm audit` in CI pipeline (fail on high/critical vulnerabilities)
- Lock file committed and integrity-checked
- Minimal dependency footprint (prefer standard library where possible)

### Secret Rotation
- Document rotation procedures for every secret type: database passwords, Valkey password, OAuth client credentials, AI_ENCRYPTION_KEY, GlitchTip DSN
- Zero-downtime rotation: accept both old and new credentials during rotation window
- Managed hosting: automate rotation on a schedule (minimum quarterly)
- Self-hosters: document manual rotation in security hardening guide
- Monitor logs for credential exposure patterns (connection strings, API keys in Pino output)

### CORS
- Explicitly configured allowed origins (no wildcard `*` in production)
- Credentials mode properly configured for OAuth flow

### Age Requirements
- Terms of Service require minimum age of 16 (Dutch UAVG GDPR threshold)
- Age self-declaration on first attempt to access Mature content or post in Mature categories (not required for SFW posting)
- See `decisions/legal.md` Children's Data & Age Verification section for full requirements
- Implementation: `specs/prd-api.md` M10 age-declaration endpoint

---

## Fastify API Patterns

### Response Schema Serialization

Fastify uses `fast-json-stringify` to serialize responses. This **strips any field not declared in the response JSON schema**. This is a silent data loss issue -- your handler returns the right data, but the client never sees it.

**Every error response schema must explicitly include:**
```typescript
const errorJsonSchema = {
  type: "object" as const,
  properties: {
    error: { type: "string" as const },
    message: { type: "string" as const },
    statusCode: { type: "integer" as const },
    // ... plus any domain-specific fields
  },
};
```

If you omit `message` or `statusCode`, the client receives `{}` even though Fastify's error handler sets those fields.

### Nullable Fields in OpenAPI Schemas

For fields that can be explicitly set to `null` (e.g., clearing a description, moving a category to root):

- **Zod:** `.nullable().optional()` -- allows `null` (clear value), `undefined` (don't change), or the actual type
- **OpenAPI/Fastify JSON schema:** `type: ["string", "null"]` (not just `type: "string"`)

```typescript
// Zod
description: z.string().max(500).nullable().optional(),

// Fastify route schema
description: { type: ["string", "null"], maxLength: 500 },
```

### Gate Consistency Across Endpoint Variants

When adding access control or filtering to a **list endpoint** (e.g., `GET /api/topics`), always check these related endpoints for the same gate:

1. **Single-resource GET** (e.g., `GET /api/topics/:uri`) -- often overlooked, allows bypassing filters via direct URL
2. **Write endpoints** (e.g., `POST /api/topics`) -- users shouldn't create resources they can't view
3. **Nested resource endpoints** (e.g., `GET /api/topics/:uri/replies`) -- inherit parent's access rules

This is especially critical for maturity filtering, community membership, and role-based access.

---

## Database Practices

### PostgreSQL
- All queries via Drizzle ORM (type-safe, parameterized)
- Migrations tracked in version control (Drizzle Kit)
- Indexes on all frequently queried columns
- Connection pooling (pg-pool or Drizzle's built-in)
- No raw SQL strings unless in migration files
- **Multi-tenant query safety:** Every query against shared tables (topics, categories, replies, reactions) MUST include `communityDid` in the WHERE clause. This applies to counts, existence checks, and joins -- not just primary queries. Forgetting this causes cross-community data leakage in global aggregator mode.
- **ID generation:** Use `crypto.randomUUID()` (Node.js built-in) for all server-generated identifiers (category IDs, session tokens, etc.). Never use `Math.random()` -- it is not cryptographically secure and produces predictable values. Format: `prefix-${randomUUID()}` (e.g., `cat-`, `rpt-`).
- **Database role separation (mandatory):**
  - `barazo_migrator` -- owns schema, runs DDL (used only by migration scripts via `MIGRATION_DATABASE_URL`)
  - `barazo_app` -- INSERT/UPDATE/DELETE/SELECT on data tables (used by the API server via `DATABASE_URL`)
  - `barazo_readonly` -- SELECT only (used for search, public read endpoints, reporting)
  - Plugin database access: restricted role with explicit grants on plugin-specific tables only (PostgreSQL-level enforcement, not just application-level)
  - Migrations run with `barazo_migrator` via separate `MIGRATION_DATABASE_URL`, never the application role
- **Connection pooling configuration (explicit):**
  - `max`: 20 (single-community), 50 (global aggregator)
  - `idleTimeoutMillis`: 30000
  - `connectionTimeoutMillis`: 5000
- **Query safety:**
  - `statement_timeout`: 30 seconds on `barazo_app` role (kills runaway queries)
  - `idle_in_transaction_session_timeout`: 60 seconds
  - Search queries: separate 5-second `statement_timeout` via `SET LOCAL` before vector/full-text queries
- **PostgreSQL logging (production):**
  - `log_statement = 'ddl'` (only log schema changes, not data queries with PII)
  - `log_min_duration_statement = 1000` (log slow queries over 1 second)
- **Indexes for security-sensitive operations:**
  - `author_did` / `did` columns across all tables (required for GDPR deletion performance)
  - `reporter_did` in reports table
  - `community_did + status` in filter tables
- **Audit logging:**
  - `admin_audit_log` table: append-only (no UPDATE/DELETE for application role)
  - Log all admin actions: settings changes, category CRUD, maturity changes, threshold changes, plugin install/enable/disable, global filter changes, user deletion requests
  - Retain for GDPR accountability period

### Valkey
- **Authentication required:** configure `requirepass` in Valkey; connect via `redis://:${VALKEY_PASSWORD}@valkey:6379`
- **Dangerous commands disabled:** rename `FLUSHALL`, `FLUSHDB`, `CONFIG`, `DEBUG`, `KEYS` via `rename-command` in Valkey config
- **Memory policy:** `maxmemory-policy volatile-lru` (evict only keys with TTL set)
- Used for caching and sessions (not as primary data store for content)
- TTL on all cache entries (no unbounded cache growth)
- **Cache key namespacing:** `barazo:{entity}:{id}` (e.g., `barazo:session:{token_hash}`, `barazo:reputation:{did}`, `barazo:summary:{topic_id}`)
- **Multi-tenant (P3):** prefix all keys with `barazo:{community_id}:{entity}:{id}` to prevent cross-tenant cache leakage
- Cache invalidation strategy documented per cached entity
- Connection error handling (graceful degradation if Valkey is down)
- Never cache raw Bearer tokens or OAuth refresh tokens. Cache only a hash or reference.
- **TLS:** Required for managed hosting (P3). Optional but recommended for self-hosted production.

### Encryption at Rest

**Sensitive Data Classification:**
The following data requires application-level encryption before storage:
- BYOK API keys (AI providers: OpenAI, Anthropic, OpenRouter, etc.)
- OAuth refresh tokens (stored in Valkey sessions)
- Any future PII stored beyond AT Protocol public data

**Encryption Standard:**
- Algorithm: AES-256-GCM (authenticated encryption)
- Key derivation: HKDF from master secret with per-community salt
- Master secret: environment variable (`AI_ENCRYPTION_KEY`), never stored in database
- Per-community data encryption key (DEK): generated per community, encrypted with master key (KEK), stored in database
- Memory handling: zero sensitive values from memory after use

**Key Rotation:**
- Master key rotation: re-encrypt all DEKs with new master key (no re-encryption of data needed)
- DEK rotation: re-encrypt all data encrypted with old DEK
- Document rotation procedure in operations runbook
- Log key access events (not values) for breach detection

**Disk Encryption:**
- Managed hosting: rely on Hetzner's disk encryption
- Self-hosters: recommend LUKS full-disk encryption in security hardening guide
- Backups: encrypt with GPG or `age` before storing off-server (see `prd-deploy.md` backup section)

---

## Error Monitoring & Observability

### GlitchTip (self-hosted)
- Self-hosted on Hetzner infrastructure (Sentry SDK-compatible, same `@sentry/node` client, different DSN)
- Integrate from first deployment
- Capture all unhandled exceptions and rejections
- Performance monitoring for API endpoint latency
- Source maps uploaded during build
- Environment separation (development, staging, production)
- Alert on new error types
- No third-party data transfer -- all error data stays on our infrastructure

### Structured Logging (Pino)
- Pino logger (native Fastify integration)
- JSON structured logs in production
- Pretty-print in development
- Log levels: error, warn, info, debug
- Request ID correlation across log entries
- No sensitive data in logs (no tokens, no PII)

### Health Checks
- `/health` endpoint: basic liveness (is the server running?)
- `/health/ready` endpoint: readiness (database connected, Valkey connected, firehose subscribed)
- Used by Docker HEALTHCHECK and load balancer

---

## Monitoring & Alerting

### Metrics to Track
- API response times (p50, p95, p99)
- Error rates by endpoint
- Firehose processing lag (time between record creation and indexing)
- Database query latency
- Cache hit/miss ratio
- Active WebSocket connections (for future real-time features, P5+)

### Alerts (via GlitchTip for P1-2, Grafana + Prometheus from P3)
- Error rate spike (> 5% of requests)
- API latency spike (p95 > 2s)
- Firehose subscription disconnect
- Database connection failures
- Disk/memory threshold warnings

### Data Breach Response

Breach notification procedures and timelines defined in `decisions/legal.md` Data Breach Notification section. Implementation requirements:

- Breach detection and severity classification built into monitoring (GlitchTip alerts)
- Breach notification templates prepared and tested before launch
- Breach notification timeline specified in DPA with managed hosting customers
- Breach log maintained per GDPR Art. 33(5)

---

## Analytics (P3)

### Umami (self-hosted)
- Privacy-first web analytics for the marketing site and public-facing pages
- Self-hosted on Hetzner (no third-party data transfer)
- No cookies, no fingerprinting, no visitor profiling
- GDPR compliant by design -- no consent banner needed

### PostHog (self-hosted, P3)
- Track admin actions and business metrics ONLY
- Self-hosted on Hetzner (all data stays on our infrastructure)
- NO end-user tracking on the forum (no cookies, no pixels, no fingerprinting)
- Forum usage stats (posts, users, activity) come from the AppView database, not analytics
- Respect Do Not Track headers
- GDPR compliant by design

### What we track (admin/business only)
- Admin dashboard feature usage
- Onboarding funnel (forum setup flow)
- Billing conversion metrics
- MRR, churn, growth

### What we never track
- Forum end-user browsing behavior
- User reading patterns
- Personal identification of forum visitors
- Any data that would conflict with "no tracking, no ads" principle

---

## References

- Fastify Security: https://fastify.dev/docs/latest/Guides/Recommendations/
- GlitchTip: https://glitchtip.com/documentation (uses Sentry SDK: https://docs.sentry.io/platforms/javascript/guides/node/)
- Drizzle ORM: https://orm.drizzle.team/
- PostHog: https://posthog.com/docs/self-host
- Stripe Billing: https://stripe.com/docs/billing
- Valkey: https://valkey.io/
