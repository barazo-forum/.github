# Barazo - Frontend Engineering Standards

**Created:** 2026-02-09
**Status:** Active - enforce from first commit

Standards specific to barazo-web (forum frontend). Read standards/shared.md first.

---

## Frontend Standards

### shadcn/ui
- Used for admin dashboard, settings, billing pages
- Components copied into project (no npm dependency lock-in)
- Follows Tailwind design system
- Built on Radix UI Primitives (ARIA attributes, keyboard navigation, focus management handled automatically)
- Developers must still provide: labels, color contrast verification, meaningful alt text

### Design System: Colors

Barazo uses **Radix Colors** as the structural color system with **Flexoki** accent colors for visual identity.

- Use Radix Colors' 12-step semantic scales for all UI chrome: backgrounds, surfaces, borders, text, focus rings, interactive states
- Seed Radix custom palette with Flexoki accent hues to generate the full scale
- Light/dark mode via Radix's `.light` / `.dark` class toggle, respecting `prefers-color-scheme`
- All custom color usage must reference design tokens (CSS variables), never raw hex values
- Verify contrast ratios against WCAG 2.2 AA: 4.5:1 for normal text, 3:1 for large text and UI components

**Active accent palette (5 colors from Flexoki):**

| Role | CSS variable | Light | Dark | Use for |
|------|-------------|-------|------|---------|
| Primary | `--accent-primary` | `#24837B` | `#3AA99F` | Links, primary buttons, active nav, focus rings |
| Info | `--accent-info` | `#5E409D` | `#8B7EC8` | Notification badges, tips, informational callouts |
| Success | `--accent-success` | `#66800B` | `#879A39` | Confirmations, solved markers, online status |
| Warning | `--accent-warning` | `#BC5215` | `#DA702C` | Pending moderation, limits, flagged content |
| Error | `--accent-error` | `#AF3029` | `#D14D41` | Validation errors, destructive actions, reports |

- Yellow, blue, and magenta are **not** in the active palette. Do not use them for UI elements.
- The extended Flexoki scale remains available for non-UI uses (syntax highlighting, data visualization) if needed.
- Each accent must be defined as a Radix 12-step scale (generated via Radix custom palette tool) for hover, active, border, and solid variants.

**References:** Radix Colors (https://www.radix-ui.com/colors), Flexoki (https://stephango.com/flexoki)

### Design System: Icons

Barazo uses **Phosphor Icons** as the sole icon library (replacing shadcn/ui's default Lucide).

- Package: `@phosphor-icons/react` (tree-shakeable, only imported icons affect bundle)
- Default weight: `regular` | Default size: `24px`
- Weight usage convention:
  - `thin` -- decorative, background, low-emphasis
  - `light` -- secondary actions, metadata
  - `regular` -- standard UI (buttons, navigation, form elements)
  - `bold` -- emphasis, primary actions, active states
  - `fill` -- selected/active state indicators
  - `duotone` -- feature highlights, onboarding, empty states
- Create a project `<Icon>` wrapper component that enforces default size/weight and accepts overrides
- When adapting shadcn/ui components, replace all Lucide imports with Phosphor equivalents
- No mixing with other icon libraries -- Phosphor only

**Reference:** Phosphor Icons (https://phosphoricons.com)

### Design System: Typography

Barazo uses the **Source** font family, self-hosted via `next/font` (zero external DNS calls).

| Role | Font | Usage |
|------|------|-------|
| UI + body | Source Sans 3 | All interface text, post body, headings, navigation |
| Monospace | Source Code Pro | Code blocks, inline code, technical content |
| Serif (optional) | Source Serif 4 | Available in system but not used by default |

- Load via `next/font/local` or `next/font/google` (both download at build time, serve from own domain)
- Font weights to load: 400 (regular), 600 (semibold), 700 (bold). Do not load extra weights.
- Set `font-display: swap` for all faces (handled automatically by `next/font`)
- Define as CSS custom properties: `--font-sans`, `--font-mono`, `--font-serif`
- Apply `--font-sans` to `<body>`, `--font-mono` to `<code>`, `<pre>`, `<kbd>` elements
- Never reference font names directly in components -- always use the CSS variables

**References:** Source Sans 3 (https://github.com/adobe-fonts/source-sans), Source Code Pro (https://github.com/adobe-fonts/source-code-pro)

### Design System: Syntax Highlighting

Code blocks in forum posts are highlighted with **Shiki** using the **Flexoki** theme, server-side rendered.

- Package: `shiki` (MIT)
- Render code blocks on the server -- no client-side JS for highlighting
- Use Flexoki dark theme when `.dark` is active, Flexoki light theme when `.light` is active
- Dual theme rendering: Shiki outputs both themes' styles in a single pass, CSS toggles visibility
- Markdown code fences (` ```language `) trigger highlighting; inline code (single backticks) gets background tint only
- Support all major languages at minimum: JavaScript, TypeScript, Python, Rust, Go, HTML, CSS, JSON, YAML, Bash, SQL, Markdown
- Line numbers: off by default, opt-in via ` ```language showLineNumbers `
- Copy-to-clipboard button on all code blocks (top-right corner, Phosphor `Copy` icon)

**References:** Shiki (https://shiki.style), Flexoki theme (https://github.com/kepano/flexoki)

### Forum UI
- Custom components for forum-specific patterns (topic lists, thread views, editors)
- Forum components follow Radix-style composable primitive patterns (same ARIA rigor as shadcn/ui)
- Consistent with shadcn/ui design tokens but purpose-built for forum UX
- Mobile-first responsive design (see Mobile-First Development below)
- Keyboard navigation support on all interactive elements

### Mobile-First Development

Mobile is the primary viewport. Desktop is the enhancement.

**Development discipline:**
- Build and visually verify every component at 375px width first, then scale up to tablet (768px) and desktop (1440px)
- Tailwind utilities without a breakpoint prefix define the mobile layout. Use `md:` and `lg:` prefixes to adapt upward. Never design desktop-first and then "fix mobile later."
- If a component looks broken at 375px, that's the bug. If it looks broken at 1440px, that's also a bug -- but mobile gets fixed first.

**Tailwind lint enforcement:**
- `eslint-plugin-tailwindcss` in the ESLint config to catch class ordering issues and invalid utilities
- No automated rule can fully enforce "mobile-first thinking," but the plugin catches structural Tailwind mistakes that correlate with responsive issues

**QA review:**
- Review all UI changes at 375px, 768px, and 1440px viewports (Responsively App or equivalent multi-viewport tool)
- Mobile viewport (375px) is the primary review target -- check it first

---

## Rendering Strategy: Server Components First

Every component is a React Server Component by default. Client Components (`"use client"`) are the exception, used only when genuinely required. This mirrors Astro's islands architecture: the page is static HTML with zero client JS, except for explicitly interactive elements.

### Rules

1. **Server Components are the default.** Do not add `"use client"` unless the component genuinely needs it.
2. **Push `"use client"` boundaries as deep as possible.** If a page needs one interactive element, only that element is a Client Component -- not the page, not the section, not the wrapper.
3. **Never mark layout or page components as Client Components.** `app/layout.tsx` and `app/*/page.tsx` must always be Server Components. Interactive parts go in separate Client Component leaf files.
4. **Every `"use client"` directive requires a comment explaining why.** Enforced in PR review.

### Valid reasons for `"use client"`

- Uses `useState`, `useReducer`, `useEffect`, `useRef` with DOM manipulation
- Attaches event handlers (`onClick`, `onChange`, `onSubmit`, etc.)
- Uses browser-only APIs (`localStorage`, `window`, `IntersectionObserver`)
- Uses React context that reads/writes client state
- Uses third-party libraries that require browser environment

### NOT valid reasons for `"use client"`

- Importing a Client Component (importing from a Server Component is fine -- the import creates the boundary automatically)
- Using `async/await` (Server Components support this natively)
- Fetching data (Server Components fetch directly from database/API/cache)
- Reading cookies or headers (use `next/headers`)
- Conditional rendering based on server-side data

### Expected Client Components

Maintain an explicit list of Client Components and their justification. Any new `"use client"` directive in a PR must be reviewed against this list -- if it's not here, it needs discussion.

| Component | Reason |
|-----------|--------|
| ReactionButton | `onClick`, `useState` for optimistic update |
| ReplyEditor | Browser textarea API, `useState`, event handlers |
| ThemeToggle | `localStorage`, `useState` |
| NotificationBadge | Real-time updates, `useState` |
| SearchCombobox | Keyboard events, `useState`, focus management |
| CommandPalette (cmdk) | Keyboard events, `useState`, portal |
| InfiniteScrollLoader | `IntersectionObserver` (opt-in only, never default) |
| TopicSortDropdown | `useState`, event handlers |
| ModerationActions | `onClick`, confirmation dialogs |
| CopyCodeButton | `navigator.clipboard` API |

This list will grow as features ship, but growth should be gradual and justified. If it exceeds ~30 entries, audit whether any can be consolidated or pushed back to server.

### Client JS Budget (per route, gzipped, excluding shared React runtime)

| Route | Budget |
|-------|--------|
| Topic listing (category/homepage) | 40KB |
| Topic thread view | 50KB |
| User profile | 30KB |
| Admin panel | 80KB |
| Auth/login | 20KB |

The React runtime (~85KB gzipped) is shared across all routes via the framework chunk. This is the cost of using Next.js -- it's justified by the forum's genuine interactivity needs. Everything above that baseline must earn its place.

### Streaming SSR

Use Next.js streaming with `<Suspense>` boundaries for progressive page delivery:

- **Shell** (nav, sidebar, footer) renders immediately as static Server Components
- **Main content** streams as data resolves from database/API
- **Skeleton components** as `<Suspense>` fallbacks (matching final layout, not spinners)
- **Below-fold content** (e.g., replies beyond the initial viewport) deferred with separate `<Suspense>` boundaries
- **Non-critical UI** (related topics sidebar, user badges) in their own `<Suspense>` boundaries so they don't block the main content stream

### Data Fetching Pattern

- **Server Components** fetch data directly (database queries via Drizzle, API calls, Valkey cache reads) -- no client-side fetch for initial page render
- **React Query** (`@tanstack/react-query`) used only in Client Components for: mutations, optimistic updates, polling for real-time data, and cache invalidation after user actions
- **Server Actions** for form submissions (works with and without JS -- progressive enhancement)
- **No `useEffect` for data loading.** If a component needs data on mount, it should be a Server Component or receive data as props from a Server Component parent.

### Component File Convention

```
src/components/
  topic-card.tsx              # Server Component (no directive needed)
  topic-card-actions.tsx      # "use client" -- reactions, bookmark, share
  reply-editor.tsx            # "use client" -- textarea, formatting toolbar
  category-header.tsx         # Server Component
  notification-badge.tsx      # "use client" -- real-time count
```

Server Components have no directive. Client Components start with `"use client"` on line 1 followed by a comment explaining why. Separate files enforce the boundary -- never mix server and client logic in one file.

### Skills

Two Claude Code skills enforce this strategy:

- **`barazo-component`** -- invoke when creating new components. Guides RSC-first structure and determines whether `"use client"` is needed.
- **`barazo-bundle-audit`** -- invoke to audit existing code for unnecessary client JS, misplaced `"use client"` boundaries, and bundle budget compliance.

---

### UX Architecture (see decisions/ux-architecture.md for full strategy)

**Every feature must meet these UX standards:**

1. **Time to Value < 60 seconds** — from landing page to first post created
2. **90%+ Completion Rate** — track signup, first post, first reply funnels
3. **Keyboard-first navigation** — every action has a shortcut (documented via `?` help)
4. **Loading states** — skeleton screens (not spinners), match final layout
5. **Empty states** — helpful prompts with call-to-action ("No topics yet — create the first one!")
6. **Error states** — specific fix instructions ("Topic title too short (min 3 chars)"), never generic
7. **Progressive enhancement** — core functionality works without JavaScript
8. **Undo for destructive actions** — soft deletes with 30-second undo window
9. **Micro-interactions** — immediate feedback (button ripple, optimistic updates, toast notifications)
10. **Mobile-first** — design for 375px width, scale up to desktop

**Built-in patterns (use from day 1):**
- Form validation: Zod schemas with user-friendly errors (client + server)
- Command palette: Cmd+K search for topics, categories, actions (cmdk library)
- Keyboard shortcuts: c (create), r (reply), j/k (navigate), Esc (close modals), / (search)
- Smart defaults: 80/20 rule (works for 80%, experts can customize)

**Quality gates (CI/CD):**
- Lighthouse score ≥90 (all categories)
- Signup completion rate ≥90% (PostHog funnel)
- Keyboard-only walkthrough passes (manual testing before release)
- WCAG 2.2 AA compliance (eslint-plugin-jsx-a11y strict + vitest-axe + @axe-core/playwright)

---

## SEO & Indexability

Follow the SEO decisions in `decisions/frontend.md` and implement per the milestones in `specs/prd-web.md`.

### Content maturity indexing rules

Per `decisions/frontend.md` "SEO and Content Maturity":

- **SFW pages:** full SEO (meta tags, JSON-LD, sitemaps, OG tags). No special treatment.
- **Mature pages:** add `<meta name="rating" content="mature">`. Include in sitemaps. Include JSON-LD. OG image must fall back to community default (no user-uploaded images).
- **Adult pages:** add `<meta name="robots" content="noindex, nofollow">`. Exclude from all sitemaps. Omit JSON-LD. Omit OG tags.
- **Maturity inheritance:** category maturity inherits from community default. Post maturity inherits from category. Individual ratings can be higher but not lower than parent.
- **Global aggregator:** SSR must render the age gate component (not content) for Mature pages when the request has no authenticated session. This ensures crawlers see the gate, not the content.

### Key coding standards
- Use `generateMetadata` per page type (Next.js App Router)
- All URLs must be absolute in canonical and OpenGraph tags
- JSON-LD preferred over Microdata (Google-recommended)
- Test structured data with Google Rich Results Test in CI
- Default OG image: 1200x630 with forum branding (fallback when no image in topic)
- Dynamically generate robots.txt via Next.js `app/robots.ts`
- See `decisions/frontend.md` for URL structure, robots.txt rules, blocked bots, sitemap structure, and structured data types

---

## Accessibility (WCAG 2.2 AA)

### Target: WCAG 2.2 Level AA + selective AAA

Barazo must be accessible from the first commit. This is both a legal requirement (European Accessibility Act, enforceable since June 2025) and a competitive advantage (Discourse has well-documented, persistent accessibility failures).

### Semantic HTML (foundation)
- Proper landmarks on every page: `<header>`, `<nav>`, `<main>`, `<footer>`
- Heading hierarchy: `<h1>` page title, `<h2>` topic title, `<h3>` individual replies
- `<article>` for each reply with `aria-labelledby`
- `<button>` for actions, `<a>` for navigation (never `<div>` with click handlers)
- `<nav aria-label="...">` for all navigation regions
- Breadcrumbs with `<nav aria-label="Breadcrumb">` and `aria-current="page"`

### Keyboard Navigation
- Every feature usable without a mouse
- Visible focus indicators using Tailwind `focus-visible:ring-*` (TailwindCSS resets remove default outlines)
- Skip links: "Skip to main content" + "Skip to reply editor" (first child of body, visible on focus)
- Custom `useFocusOnNavigate` hook to move focus after client-side route changes (Next.js does NOT handle this)
- Escape key dismisses all overlays, popovers, and hover cards
- Tab enters/exits editor toolbar, Arrow keys navigate within (roving tabindex)

### Content Loading
- **Pagination by default** for topic lists and thread views
- Infinite scroll only as opt-in enhancement (never default)
- New content announced via `aria-live="polite"` status messages

### Dynamic Content & Notifications
- `role="status"` with `aria-live="polite"` for non-critical updates ("3 new replies")
- `role="alert"` with `aria-live="assertive"` only for errors
- `role="log"` for thread/reply streams
- Live region containers must exist in DOM at page load (not dynamically created)
- Batch notifications instead of per-item announcements
- Dedicated notifications page as alternative to real-time popups

### Interactive Elements
- Every button and control has an accessible name (`aria-label` or visible text)
- Reaction buttons: `aria-pressed` for toggle state, `aria-label` includes count
- Minimum target size: 24x24 CSS px (WCAG 2.2 AA), prefer 44x44 (AAA)
- Profile hover cards: keyboard-triggerable, Escape-dismissible (SC 1.4.13)
- Search: WAI-ARIA Combobox pattern with result count announcement

### Form Fields
- Every form field must use `<FormLabel>` from `@/components/ui/form-label`
- Every field gets either the `required` or `optional` prop -- no unmarked fields
- Asterisks render with `<span aria-hidden="true">` (screen readers use the HTML `required` attribute instead)
- Required inputs must have the HTML `required` attribute
- `<fieldset>/<legend>` groups: add `<span aria-hidden="true" className="ml-1 text-destructive">*</span>` inline to `<legend>` text, not via `FormLabel`
- Checkbox labels: pass `block={false}` to `FormLabel`
- Description/hint text stays as separate `<p>` elements below the label (not absorbed into `FormLabel`)

### Theme Mode (Dark / Light)
- **Dark mode is the default.** Light mode available via toggle.
- Toggle control in main navigation bar (Phosphor `Moon` / `Sun` icon)
- Initial state: respect `prefers-color-scheme` for first-time visitors; fall back to dark if no OS preference
- Persist choice in `localStorage` (anonymous) or user preference record (authenticated)
- Apply `.dark` / `.light` class on `<html>` element (Radix Colors convention)
- Prevent FOUC: inline `<script>` in `<head>` reads preference before React hydration
- All components must be tested in both modes -- never hardcode colors outside design tokens
- Logo: render dark variant in dark mode, light variant in light mode (swap via CSS or conditional import)
- Applies to both the forum frontend (barazo-web) and the marketing website (barazo.forum)

### Color & Visual
- Contrast ratios: 4.5:1 for normal text, 3:1 for large text and UI components
- No information conveyed by color alone (always include text/icon alternative)
- Respect `prefers-reduced-motion` media query
- Respect `prefers-color-scheme` for initial theme mode (see Theme Mode section above)

### Editor
- Markdown textarea with fixed toolbar above (not floating)
- WAI-ARIA Toolbar pattern for formatting buttons
- Keyboard shortcuts for common formatting (Ctrl+B, Ctrl+I, etc.) with visible documentation
- Escape key to exit textarea and move to next focusable element
- Preview pane separate from editing area

### Accessibility Testing (CI/CD)

**Every PR (Tier 1):**
- `eslint-plugin-jsx-a11y` in **strict** mode (not default "recommended")
- `vitest-axe` for component-level tests (requires JSDOM, not Happy DOM)
- `@axe-core/playwright` for page-level tests against rendered pages

**Nightly (Tier 2):**
- `pa11y-ci` crawling all page types against staging
- Lighthouse CI with minimum accessibility score of 95

**Before release (Tier 3 - manual):**
- VoiceOver + Safari testing
- Full keyboard-only navigation walkthrough
- axe DevTools browser extension review

**Note:** Automated tools catch only 30-50% of accessibility issues. Manual testing with assistive technology is required for every release.

### Legal Compliance
- Accessibility statement published at launch
- Feedback mechanism (contact form) for reporting accessibility barriers
- VPAT / Accessibility Conformance Report documenting conformance level
- Self-hosted documentation includes guidance on maintaining accessibility

---

## References

- shadcn/ui: https://ui.shadcn.com/
- WCAG 2.2: https://www.w3.org/TR/WCAG22/
- WAI-ARIA Authoring Practices: https://www.w3.org/WAI/ARIA/apg/
- European Accessibility Act: https://accessible-eu-centre.ec.europa.eu/
- axe-core: https://www.deque.com/axe/axe-core/
- Radix UI Accessibility: https://www.radix-ui.com/primitives/docs/overview/accessibility
- Google Structured Data (Forums): https://developers.google.com/search/docs/appearance/structured-data/discussion-forum
- Schema.org DiscussionForumPosting: https://schema.org/DiscussionForumPosting
