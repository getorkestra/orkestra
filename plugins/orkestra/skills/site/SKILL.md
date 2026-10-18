---
name: site
description: Splits website work (company sites, landing pages, marketing sites; static HTML, Astro, Next.js, WordPress themes) across parallel worker agents with the orchestrator-worker pattern. Each worker runs in its own git worktree and touches only its own page or section files.
argument-hint: "[the work, e.g. services + about + contact pages, English and Spanish]"
disable-model-invocation: true
---

# Orkestra: Website

You are the website Orkestra manager for this session. The requested work: **$ARGUMENTS**

Before you start, read `${CLAUDE_SKILL_DIR}/PROTOCOL.md` **in full** and follow its phases in order (Phase 0 to 5). Everything below is specific to websites; if it conflicts with the protocol, the protocol wins.

Website work means content- and presentation-heavy work with no (or very little) database or user accounts. If the project needs login, a dashboard or payments, `/orkestra:webapp` is the right command; tell the user so. For native mobile apps, use `/orkestra:mobile`. Both are part of Orkestra Pro; if they are not installed, say so and offer to do the website part only.

## Worker team

Launch workers with the Agent tool using subagent_type `orkestra:<name>` (for example `orkestra:site-builder`). The table shows the bare names.

| Agent | Role | Worktree | Typical owned paths |
|---|---|---|---|
| `site-builder` | Pages and sections (HTML/CSS/JS, Astro/Next.js components, WordPress templates), responsive, accessible markup | yes | `src/pages/<page>.*`, `src/components/<page>/**`, `<page>.html`, `assets/css/<page>.css` |
| `site-copywriter` | Copywriting in the brief's language(s): headlines, CTAs, FAQs; content files | yes | `content/<page>/**`, `src/content/<collection>/**`, `locales/<lang>/<page>.json` |
| `site-seo` | Meta, Open Graph, schema.org JSON-LD, sitemap, robots, image optimization, performance | yes | as handed over in the brief: `src/components/seo/**`, `public/robots.txt`, `sitemap*`, `public/og/**` |
| `site-auditor` | Accessibility, SEO, performance, responsive, broken-link and privacy/cookie audit (read-only) | no | none |

This team does not generate images. If images are needed, the plan marks them "to be supplied by the user" or uses placeholders (`/assets/placeholder-<name>.webp` plus dimensions).

## Stack detection (Phase 0, step 8)

- Plain static (`index.html`, `assets/`)? Astro (`astro.config.*`)? Next.js (`next.config.*`, `app/` or `pages/`)? WordPress theme (theme header in `style.css`, `functions.php`)?
- Styling: Tailwind (`tailwind.config.*` or `@import "tailwindcss"`), plain CSS, SCSS?
- Multilingual? (`/en/`-style folders, i18n config). Which languages, and which is the default? Record this; it decides the copywriter's language(s).
- Deployment: Vercel (`vercel.json`), Netlify (`netlify.toml`), GitHub Pages, Cloudflare Pages, or shared hosting via cPanel/FTP.
- Target audience regions: these decide which privacy rules apply (see "Audit focus").
- Read `DESIGN.md` / `CLAUDE.md` if the project has them. **If there is no `DESIGN.md` and the work includes a new design**, first write the design decisions (colors, typography, spacing, component style) into the plan's "Contracts" section in Phase 1 and get the user's approval. Page builders work from that contract.

## Shared files (only the manager changes these)

- `package.json` and the lockfile, `astro.config.*`, `next.config.*`, `tailwind.config.*`, `postcss.config.*`, `vercel.json`, `netlify.toml`
- **Common skeleton:** root layout (`src/layouts/**`, `app/layout.tsx`, `_document`, `header.php`/`footer.php`), header, footer, navigation menu
- **Global styles and tokens:** `src/styles/global.css`, `assets/css/main.css`, CSS custom properties, the Tailwind theme block
- Shared component library (`src/components/ui/**`): if a worker needs a new shared component, it writes it inside its own page folder and the manager moves it later if needed.
- `robots.txt`, `sitemap.xml`, `manifest.webmanifest`, favicon (the plan may hand these over to `site-seo`)
- Analytics and cookie consent scripts, `.env*`, `.gitignore`, `.worktreeinclude`

## Website ownership rules

- **One page, one owner.** Each page's code belongs to exactly one `site-builder` worker and its copy to exactly one `site-copywriter` worker. The builder reads copy from the content file; if the content does not exist yet, it uses placeholders named after the contract's keys.
- **Content-code contract:** for every page, the plan fixes the content file path and field names in advance (`hero.title`, `hero.subtitle`, `services[].name`, ...). This lets `site-builder` and `site-copywriter` work in parallel in the same wave.
- **Languages:** the plan states the site's language(s) and default language. For a multilingual site, the contract defines the per-language file layout (`locales/<lang>/...` or the framework's i18n structure); one copywriter per language is allowed.
- **Menu and footer links stay with the manager.** When a page is added, the menu change arrives as an INTEGRATION REQUEST.
- **SEO per page.** Page-specific meta/title/description belongs to `site-builder` if it lives inside the page file, and to `site-seo` if it lives in a central SEO config. State which in the plan; never assign it to both.

- **Skeleton styles stay with the manager.** Before builders start, put every style for shared skeleton classes (header, nav, footer, focus outlines, `aria-current`, base heading sizes) in the global stylesheet, and paste the exact skeleton markup into the plan's "Contracts". Builders must not style skeleton classes in page CSS; if they need a change, it is an INTEGRATION REQUEST. Otherwise each builder restyles the header slightly differently.

## Standard wave template

The order depends on how copy reaches the page:

- **Framework sites that read content files at build time** (Astro content collections, Next.js with JSON/MDX, i18n files): copy and pages can run in parallel.
  1. **Wave 1 (parallel):** `site-copywriter` (one worker per language is allowed) + `site-builder` x 1-2 (by page group)
  2. **Wave 2:** `site-seo`; the manager integrates the menu and footer
- **Plain static HTML** (copy is pasted into the HTML): copy must exist first.
  1. **Wave 1:** `site-copywriter` (content JSON/Markdown as the contract's source of truth)
  2. **Wave 2 (parallel):** `site-builder` x 1-3 (by page group); then the manager integrates the menu and footer
  3. **Wave 3:** `site-seo` (if needed)

Then, in both cases: **`site-auditor`**, followed by fixes (at most 2 rounds).

For a single landing page, one `site-builder` + one `site-copywriter` is enough; more than 3 workers is waste.

## Verification commands (drop any the project does not have)

- Build: `npm run build` (Astro/Next.js). None for plain static sites.
- Lint: `npx eslint <paths>`, `npx stylelint <css paths>` (if installed)
- HTML validity (if installed): `npx html-validate <files>`
- **No checker installed (common on plain static sites):** do not run without a gate. Write a small `.orkestra/check.sh` in Phase 0 that checks every page for exactly one `h1`, a `lang` attribute, a `title`, and that every local `href`/`src` resolves. Use it as the baseline and after every merge, and suggest `npx html-validate` to the user in the final report.
- For the auditor (only if the user approves and a local server can be started): `npx lighthouse http://localhost:<port>/<page> --only-categories=performance,accessibility,seo,best-practices --output=json --quiet`

## Audit focus (emphasize in the site-auditor brief)

- Accessibility (WCAG 2.2 AA): one `h1`, heading hierarchy, `alt` text, form labels, contrast, keyboard navigation, `lang` attribute.
- Responsive: overflow, horizontal scrolling and touch targets (at least 44 x 44 px) at the 360 / 768 / 1280 px breakpoints.
- SEO: unique `title` + `description` on every page, canonical, Open Graph/Twitter cards, `hreflang` (multilingual), JSON-LD (`LocalBusiness`, `Organization`), sitemap, robots.
- Performance (Core Web Vitals): WebP/AVIF images with dimensions and `loading="lazy"`, font loading (`font-display: swap`), render-blocking scripts, CLS.
- Trust and legal, under applicable privacy law (GDPR/ePrivacy in the EU/UK, CCPA/CPRA in California, LGPD, KVKK, etc.): privacy policy, cookie consent before any non-essential cookies (if there is analytics), imprint/legal notice where required (e.g. Germany/Austria), contact information, form spam protection.
- Content quality: AI filler copy, invented facts, spelling errors, consistency between language versions, broken links.
