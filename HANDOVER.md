# HANDOVER — chernteh.github.io

A handover for any agent (especially a future **designer agent**) working on Chern's personal site.
Read this end-to-end before editing. It describes the live state of the repo, the design system, the
custom pieces, and the gotchas that are not obvious from the code.

- **Live site:** https://chernteh.github.io
- **Repo:** https://github.com/chernteh/chernteh.github.io
- **Default & deploy branch:** `main` (pushing to `main` triggers deploy)
- **Owner / git author:** chernteh (chernteh@gmail.com)

---

## 1. What this is

A **personal portfolio + blog** for Chern Teh — final-year NUS Mathematics undergrad (2nd major
Quantitative Finance, pursuing SOA actuarial ASA). Three surfaces:

1. **About Me** — homepage at `/` ([index.html](index.html))
2. **Projects** — at `/projects/` ([projects.html](projects.html))
3. **Blog posts** — at `/posts/<title>/` (currently one: the Market-News Bot writeup)

The visual identity is a **dark, GitHub-flavored developer aesthetic**: near-black background,
slate sidebar, blue accent. Clean, technical, calm.

---

## 2. Tech stack

- **Jekyll 4.3** (Ruby) — static site generator. **No theme.** Layouts are fully custom and hand-written.
  This is NOT Chirpy, despite some leftover Chirpy scaffolding (see §9 Gotchas).
- **Kramdown + Rouge** for Markdown → HTML and server-side syntax highlighting.
- **Plain CSS**, single file: [assets/css/style.css](assets/css/style.css). No Sass pipeline, no Tailwind,
  no build step for CSS. Edit the file directly.
- **Vanilla JS only**, inlined in [_layouts/post.html](_layouts/post.html) (the TOC generator). No frameworks, no npm.
- **Font Awesome 6.5** via CDN (social icons only), loaded in [_layouts/default.html](_layouts/default.html).
- **GitHub Actions** builds and deploys to GitHub Pages on every push to `main`.

### Deploy pipeline — [.github/workflows/pages-deploy.yml](.github/workflows/pages-deploy.yml)
1. Checkout → Setup Ruby 3.4 (`bundler-cache: true`) → `bundle exec jekyll build` (production env)
2. **html-proofer** runs against `_site` (internal links/images must resolve, or the build FAILS —
   external links are disabled in the check). Broken internal links will block a deploy.
3. Upload artifact → deploy to Pages.

Deploy takes ~1–2 min after push. Watch with `gh run watch` or the Actions tab.

---

## 3. File structure

```
_layouts/
  default.html   Main shell: <body class="layout-{{ page.layout }}"> → banner + .page-body + fixed sidebar
  post.html      layout: default. Wraps content in .post-grid (article + TOC) and holds the TOC-builder JS
  page.html      layout: default passthrough for generic pages (exists; not heavily used)
_posts/
  2026-06-27-markets_bot.md   The only blog post. Front matter sets title/subtitle/excerpt/date
_plugins/
  posts-lastmod-hook.rb       Jekyll generator hook (last-modified dates). Leave alone unless needed
assets/
  css/style.css   ALL styles live here (one file). This is the file a designer edits 95% of the time
  img/avatar.png  Sidebar profile photo (square ~800px, cropped to a circle via center/cover)
  lib/            Git submodule (Chirpy static assets) — effectively UNUSED, see Gotchas
index.html        About Me (layout: default, title: About Me)
projects.html     Projects (layout: default, permalink: /projects/)
_config.yml       Site config: title, description, kramdown/rouge, posts permalink, exclude list
Gemfile           jekyll ~> 4.3, html-proofer, tzinfo/wdm for Windows
.github/workflows/pages-deploy.yml   CI build + deploy
README.md, LICENSE, tools/   Excluded from build (see _config.yml exclude)
```

`_config.yml` highlights:
- `posts` default → `layout: post`, `permalink: /posts/:title/`
- Rouge highlighter with `css_class: highlight`, line numbers off
- `timezone: Asia/Kuala_Lumpur`

---

## 4. The design system (read before changing visuals)

### Theme system — DUAL (dark + light)

The site ships with a **full token-based dual-theme**. All colours are CSS custom properties. Dark is the
default; light (`data-theme="light"` on `<html>`) is the Apple Cupertino Light palette.

**Dark theme tokens (`:root`, GitHub Dark aesthetic):**
```
--bg:            #0d1117   page background
--sidebar-bg:    #161b22   sidebar / card surface
--card-bg:       #161b22   project cards
--nav-active:    #1d3a5f   active nav link background
--text:          #e6edf3   primary text / headings / bold
--body-text:     #d0d7de   normal paragraph/list text (slightly dimmer than --text so bold is visible)
--text-muted:    #8b949e   secondary / muted text
--accent:        #58a6ff   links, highlights, hover borders (GitHub blue)
--accent-soft:   …         softer accent tint for hover states
--accent-faint:  …         very faint accent for blockquote backgrounds
--border:        #21262d   hairline borders / dividers
--hover-bg:      …         subtle hover background on nav/cards
--banner:        …         banner gradient start colour
--banner-glow:   …         banner gradient end colour
--code-bg:       #080c13   code block background
--code-base:     #d4d4d4   code default text
--sidebar-width: 280px     drives BOTH sidebar width AND .main-content left margin
```

**Syntax tokens (dark, VS Code Dark+):**
`--sx-keyword #569cd6` · `--sx-type #4ec9b0` · `--sx-string #ce9178` · `--sx-comment #6a9955`
`--sx-number #b5cea8` · `--sx-func #dcdcaa` · `--sx-builtin #569cd6` · `--sx-var #9cdcfe`
`--sx-error #f44747`

**Light theme overrides (`:root[data-theme="light"]`, Apple Cupertino):**
```
--bg:            #f5f5f7   off-white page background
--sidebar-bg:    #ffffff   white sidebar
--card-bg:       #ffffff   white cards
--nav-active:    #e8e8ed   subtle grey active link
--text:          #1d1d1f   near-black headings
--body-text:     #424245   dark grey body copy
--text-muted:    #86868b   Apple secondary grey
--accent:        #0071e3   Apple blue
--border:        #d2d2d7   light dividers
--code-bg:       #ececf0   light code surface
--code-base:     #1d1d1f
```

**Syntax tokens (light, Xcode Default Light):**
`--sx-keyword #9b2393` · `--sx-string #c41a16` · `--sx-comment #707f8c` · `--sx-number #1c00cf`
`--sx-func #3900a0` · `--sx-type #0d5ae5` · `--sx-builtin #aa0d91`

**Rule:** never hardcode hex values inline — always reference the token. Both themes update automatically
at zero extra effort when you do this.

### Theme toggle

A `<button class="theme-toggle" id="themeToggle">` sits inside `.banner` (top-right, absolute-positioned).
It shows a moon icon (dark mode) or sun icon (light mode). JS at the bottom of `default.html` handles
click — flips `data-theme` on `<html>` and writes `localStorage('theme', 'dark'|'light')`.

**No-flash script** (inline in `<head>` of `default.html`): reads `localStorage` → OS
`prefers-color-scheme` → defaults dark. Sets `data-theme` before first paint so there is no colour flash.

### Layout model

- `.wrapper` is `display:flex`. `.main-content` is `flex:1` with **`margin-left: var(--sidebar-width)`**.
- `.sidebar` is `position: fixed; left:0; top:0; bottom:0` — a full-height fixed **LEFT** rail (280px).
  ⚠️ The sidebar was moved from the right to the left during the 2026-06-28 session. Update any notes
  that say "right rail" — they are stale.
- `.banner` is a 280px gradient header strip at the top of the content column.
- `.page-body` holds the page content: `max-width: 900px; margin: 0 auto; padding: 2.5rem 3rem`.
  On wide screens this leaves whitespace on both sides of the 900px block.
- **Scrollbar gutter is reserved** (`html { overflow-y: scroll }`) so short and long pages share the
  same content width and the fixed sidebar never shifts. Don't remove this — it fixes a real bug.

### Typography

- System font stack (`-apple-system, Segoe UI, Roboto…`), base `line-height: 1.6` (1.75 for paragraphs).
- Type scale: `h1 2.25rem · h2 1.6rem · h3 1.2rem · post-title 2.25rem`.
- Normal paragraph/list copy uses `var(--body-text)` (slightly dimmer than `--text`) so bold (`var(--text)`)
  is visually distinct. Don't collapse these two tokens back into one.
- Headings, lists, blockquotes, tables, inline code, and code blocks are all styled under `.page-body …`.
- Code: inline code and `pre` use `var(--code-bg)` / `var(--code-base)` + full Rouge syntax token palette
  (all syntax colours are CSS vars now, so both themes get correct highlighting automatically).

### Heading semantics (fixed)

- Sidebar title "Chern Teh" → `<p class="site-title">` (not `<h1>`; visual style preserved via CSS).
- Every content page now has its own real `<h1>`:
  - `index.html` → `<h1>Welcome!</h1>`
  - `projects.html` → `<h1>Projects</h1>`
  - Blog posts inherit their `<h1>` from `post.html` (post title).

### Components

- **Project cards** (`.project-card`): bordered rounded cards, lift + accent border on hover. Markup is a
  plain `<a>` wrapping `<h3>`, `.project-subtitle`, `<p>`, and `.project-tags` pills. See [projects.html](projects.html).
- **Theme toggle button** (`.theme-toggle`): circular, blurred backdrop, absolute top-right of `.banner`.
  Moon icon = dark mode; sun icon = light mode.
- **Sidebar**: avatar circle (150px, `center/cover`), site title `<p>` (1.5rem), tagline, vertical nav
  links, social row (GitHub / LinkedIn / email Font Awesome icons).

---

## 5. The sidebar (current values)

In [_layouts/default.html](_layouts/default.html) `<aside class="sidebar">`:
- **Position:** fixed **LEFT** rail (`left: 0; top: 0; bottom: 0`). `border-right` separates it from content.
  `.main-content` has `margin-left: var(--sidebar-width)` (not margin-right — was changed 2026-06-28).
- Avatar circle → `assets/img/avatar.png`, **150px**, 3px border.
- Title "Chern Teh" → `<p class="site-title">` 1.5rem (deliberately `<p>`, not `<h1>` — heading semantics).
- Tagline "Quant Finance · Actuarial / Data Engineering".
- Nav: **About Me → `/`**, **Projects → `/projects/`**. Active state via Liquid:
  `{% if page.url == '/' %}active{% endif %}` and `{% if page.url contains 'projects' %}active{% endif %}`.
- Social links: GitHub `github.com/chernteh`, LinkedIn `linkedin.com/in/chern-teh-59068325a/`,
  email `chernteh@gmail.com`. All with `rel="noopener noreferrer"` and `aria-hidden="true"` on icons.
- `--sidebar-width` currently **280px**.

These were iteratively dialed in with the owner; they care about proportion and comfortable spacing.
When changing one sidebar dimension, eyeball the others (avatar bottom margin, title size, nav padding)
so it stays balanced.

---

## 6. The custom Table of Contents (the most non-obvious feature)

Blog posts show an **auto-generated, scroll-synced TOC** in a LEFT column next to the article.
This was built from scratch (no plugin) and went through several iterations — understand it before touching.

**Where:**
- Markup + JS: [_layouts/post.html](_layouts/post.html). The post content and `<nav class="toc-panel">` are
  wrapped in a `.post-grid` container (article first in DOM, TOC second).
- CSS: the `Table of Contents` section in [style.css](assets/css/style.css).

**How it works:**
- JS (`querySelectorAll('article h2, article h3')`) builds the list, slugifies heading text into `id`s,
  and uses an `IntersectionObserver` (`rootMargin: '0px 0px -70% 0px'`) to toggle `.toc-active` on the
  link for the section currently in view.
- DOM order stays article-first (so heading IDs & the observer work); the TOC is moved to the **left**
  visually via grid + `order: -1` on `.toc-panel`.

**Layout / responsive logic (important):**
- Below **1280px**: `.post-grid` is a plain block and `.toc-panel` is `display:none` — single-column,
  no TOC. (Mobile ≤900px is unaffected; TOC never shows there.)
- At **≥1280px**: post `.page-body` widens to **1320px**; `.post-grid` becomes
  `grid-template-columns: 280px minmax(0, 1fr)` with a 3.5rem gap; `.toc-panel` becomes
  `position: sticky; top: 2rem`. The TOC sits in a real grid column rather than fighting for whitespace.
- **Why the grid, not fixed-positioning:** the original attempt used `position: fixed` with a `calc()`
  offset and only worked above ~1650px (the math: 900px centered content leaves no room for a 200px TOC
  until the viewport is ~1640px wide). The grid approach makes content + TOC one centered unit, so it
  works from 1280px up. Don't regress to fixed positioning.
- Anchor jumps land with breathing room via `article h2, article h3 { scroll-margin-top: 3.5rem }`, and
  `html { scroll-behavior: smooth }` animates the scroll.

**Known tension:** the TOC column (280px) currently equals the sidebar width, so a very wide post page
can read like it has two side rails. If asked to make posts feel lighter/more consistent with the rest of
the site, the levers are: narrow the TOC column (e.g. 220px), reduce post `page-body` max-width, or
increase the gap. (The owner previously asked to *enlarge* the TOC, so confirm direction before shrinking.)

---

## 7. Responsive behavior

- **≤900px**: `.wrapper` switches to a column. The sidebar becomes a horizontal **top bar** (avatar 50px,
  nav in a row, social pushed right), `.main-content` loses its right margin, banner shrinks to 160px,
  `.page-body` padding tightens. The TOC is hidden. This collapse works — verify it still does after
  sidebar changes (the mobile block hard-overrides desktop sizes, so desktop tweaks don't leak to mobile).
- **900–1280px**: desktop sidebar, single-column post, no TOC.
- **≥1280px**: full layout with the post TOC.

---

## 8. Working on this repo — practical notes

- **No local Jekyll/Bundler on the owner's machine** (Windows). You generally **cannot `bundle exec jekyll
  build` locally** — rely on the GitHub Actions build to validate. Keep changes template/CSS-only when you
  can, since those carry near-zero build risk.
- **Validate before pushing** when you add links/images: html-proofer will fail the deploy on a broken
  *internal* link or missing image. Double-check `href`/`src` paths (root-relative, e.g. `/projects/`,
  `/assets/img/avatar.png`).
- **Line endings:** Git warns `LF will be replaced by CRLF` on Windows. Harmless — ignore.
- **Commit/push discipline:** the owner wants changes shipped — commit and push to `main` after each
  agreed change. End commit messages with the Co-Authored-By trailer used in this repo's history.
- **Images can't be created from chat.** If the owner pastes an image, ask them to save it into the repo
  (they saved the avatar as a file in the repo root, then it was moved to `assets/img/`). You move/rename
  and commit it; you cannot write binary image bytes yourself.
- **Editing recipes:**
  - New blog post → add `_posts/YYYY-MM-DD-slug.md` with front matter (`title`, `subtitle`, `excerpt`,
    `date`). It auto-gets `layout: post` and `/posts/<title>/` from `_config.yml`. The TOC builds itself
    from `##`/`###` headings.
  - New project → add another `<a class="project-card">…</a>` block in [projects.html](projects.html).
  - New nav item → add a `.nav-link` in [_layouts/default.html](_layouts/default.html) and an active-state check.
  - Palette change → edit the `:root` tokens only.

---

## 9. Gotchas / leftover scaffolding

- **Not Chirpy.** The repo was bootstrapped from Chirpy-flavored scaffolding but the theme was removed and
  layouts are bespoke. Ignore Chirpy conventions.
- **`assets/lib` git submodule** (`.gitmodules` → `cotes2020/chirpy-static-assets`) is **unused** — the
  deploy workflow checks out with submodules *commented out*, and nothing references `assets/lib`. Don't
  rely on it; don't spend time on it.
- **`.nojekyll` file exists** at root, but the site is built by the **Actions Jekyll job**, not Pages'
  built-in Jekyll. The Actions workflow is the source of truth for builds.
- **`_plugins/posts-lastmod-hook.rb`** runs at build (custom plugins work because we use the Actions build,
  not Pages' restricted plugin sandbox). Leave it unless you specifically need lastmod behavior.
- **`tools/`, `README.md`, `LICENSE`, `Gemfile*`** are in `_config.yml`'s `exclude` — not published.

---

## 10. Recent change history (most recent first, for context)

### 2026-07-03 — Pluang card clickable (stretched link)

- **Pluang Experience card is now whole-card clickable (commit a838d1e)** → Chern's LinkedIn post about
  the internship. Implemented as `.exp-card--linked` modifier + `.exp-card-link` absolute overlay
  (`inset:0; z-index:1`); the logo image and company name sit at `z-index:2` and still → pluang.com.
  Hover = accent border + 2px lift. **The ASE card stays inert** — the "inert card body" rule now applies
  only to cards *without* the modifier. Tracking params were stripped from the LinkedIn URL on purpose.

### 2026-07-03 — NDA-safe Pluang blurb + security review

- **Pluang blurb rewritten (commit e87a2eb):** removed internal specifics (Looker→Tableau migration,
  "rebuilt end-to-end" claim, BigQuery/Mixpanel/Figma-via-MCP architecture). Now generic-safe, casual
  first-person; final wording is Chern's own. **Do not re-add internal Pluang project details.**
- **Security review:** standard GitHub Pages baseline confirmed; no vulnerabilities. Optional hardening
  (SRI hash on the Font Awesome CDN `<link>`, self-hosting FA, CSP meta tag, Dependabot, branch
  protection) reviewed and declined by Chern — treat as optional backlog, not defects.

### 2026-06-29 — Experience cards + content (final state)

**Content finalised (commits 931bb79 → bd476a8):**
- Pluang description: first-person prose summary (Chern's own wording) covering Looker→Tableau
  migration, dbt+BigQuery pipelines, and production agentic AI system via MCP.
- ASE description: "Will update soon!" placeholder — to be replaced when the role matures.
- Date ranges: ASE "June 2026 – Present"; Pluang "Dec 2025 – May 2026".
- Company name (`<a class="exp-company">`) styled as a link (hover underline) pointing to company site.

### 2026-06-29 — Experience cards on About Me

- Replaced the plain Experience `<ul>` in [index.html](index.html) with **Apple-style "Wide Media"
  cards** (chosen from a 5-option preview round): 190px image column on the left + content on the right.
- **Only the image is clickable** (`<a class="exp-logo">` → company site, new tab); the card body is
  inert by design so the card can't be clicked by accident. Image fills the column edge-to-edge
  (`object-fit: cover`); hover gives a gentle zoom + a ↗ badge overlay.
- New styles in [style.css](assets/css/style.css) under "Experience cards"; `--exp-shadow` token added to
  **both** themes; `≤900px` media rule narrows the image column to 110px.
- **Placeholders are live**: logo slots are styled `<div class="exp-logo-ph">` (ASE / PL), `href="#"`,
  and empty `<p class="exp-blurb">` — all marked with TODO comments. **Pending from Chern:** real logo
  images (→ `assets/img/`), company URLs, date ranges, descriptions. Swap per the comments in `index.html`.

### 2026-06-28 — Full design + SEO + a11y session

**Theme system (dark/light toggle)**
- Rewrote `assets/css/style.css` to a full CSS-variable token system: all interface colours, banner, code
  background, and all Rouge syntax tokens are now custom properties.
- Dark theme (`:root`): GitHub Dark aesthetic (unchanged in character from before).
- Light theme (`:root[data-theme="light"]`): Apple Cupertino Light palette (off-white BG, Apple blue accent,
  Xcode Default Light syntax colours).
- Added no-flash inline script in `<head>` of `default.html` (reads `localStorage` → OS `prefers-color-scheme`
  → default dark; sets `data-theme` before paint).
- Added `.theme-toggle` button in `.banner` (moon icon = dark, sun = light; toggle JS at body bottom;
  persists via `localStorage`).
- Introduced `--body-text` token (dark: `#d0d7de`, light: `#424245`) so normal paragraph text is visibly
  dimmer than `--text` (headings/bold), restoring bold contrast.

**Sidebar moved left**
- `.sidebar`: changed from `right: 0` to `left: 0`; added `border-right` (removed `border-left`).
- `.main-content`: changed from `margin-right` to `margin-left: var(--sidebar-width)`.
- DOM order: `<aside>` before `<main>` (already was; confirmed correct for left positioning).

**SEO / meta**
- Added Open Graph (`og:title/description/url/image/type`), Twitter Card (`summary_large_image`), and
  `<link rel="canonical">` to `default.html`.
- Added JSON-LD `Person` schema on homepage and `Article` schema in `post.html`.
- Added `jekyll-sitemap` plugin (`_config.yml` + `Gemfile`) for auto-generated `/sitemap.xml`.
- OG cover image referenced at `/assets/img/og-cover.png` — **not yet created** (pending).

**Accessibility**
- `aria-hidden="true"` on all decorative Font Awesome icons.
- `rel="noopener noreferrer"` on all external links.
- `aria-label` on social icon links and theme toggle button.
- TOC focus ring: switched from `outline` to `box-shadow: inset 0 0 0 2px var(--accent)` (outline
  overflowed the TOC panel; inset box-shadow is physically contained).
- Removed skip-link entirely (Chern chose to remove after confirming it didn't serve his use case).

**Heading semantics**
- Sidebar title `<h1 class="site-title">` → `<p class="site-title">` (branding, not document heading).
- `index.html`: added `<h1>Welcome!</h1>`.
- `projects.html`: `<p class="section-title">` → `<h1>Projects</h1>`.

**Typography**
- Type scale: h1 2.25rem, h2 1.6rem, h3 1.2rem, post-title 2.25rem.

**Content**
- `projects.html` subtitle: "Things I've built." → "Small projects that I've been working on for fun
  aside from my studies!" (friendlier tone, Chern's wording).
- `index.html`: "National University of Singapore (NUS)" linked to `nus.edu.sg`; "Society of Actuaries
  (SOA)" linked to `soa.org`.

---

**Earlier history (pre-2026-06-28)**
- Reserve scrollbar gutter (`overflow-y: scroll`) so the sidebar doesn't shift between short/long pages.
- Sidebar tuning: width 280px, avatar 150px, social icons 1.2rem, larger title/nav, extra avatar margin.
- Added profile photo `assets/img/avatar.png` and pointed the avatar CSS at it.
- Fixed LinkedIn URL to `/in/chern-teh-59068325a/`.
- Smooth scrolling + `scroll-margin-top: 3.5rem` for comfortable TOC anchor landings.
- Moved the TOC to the **left** column (`order: -1`).
- Widened TOC column/fonts; widened post body to 1320px.
- **Rebuilt the TOC as a CSS-grid sticky left column** (replacing the broken `position:fixed`/1650px
  approach) — works from 1280px up.
- Blog post copy edits (headings/prose polish).

---

## 11. Pending items (deferred, not done)

| Item | What's needed | Notes |
|---|---|---|
| **Favicon** | Go to favicon.io → Text → "CT", bg `#0d1117`, text `#58a6ff` → download ZIP → place files in `assets/img/` → add `<link rel="icon">` tags to `default.html` | Deliberately deferred by Chern |
| **OG cover image** | Create 1200×630px `og-cover.png` → `assets/img/og-cover.png` | OG `<meta>` already wired, just missing the file |
| **Google Search Console** | Get verification `<meta name="google-site-verification">` from GSC and add to `default.html` | Not started |
| **Banner content** | Currently an empty gradient strip | Chern deferred redesign |
| **Homepage body** | 3 paragraphs + 2-item list — thin | Content expansion deferred |

---

## 12. For the future designer agent — start here

Your sandbox is almost entirely **[assets/css/style.css](assets/css/style.css)** plus the three layout
files. The mental model:

1. **Palette** lives in `:root` (dark) and `:root[data-theme="light"]` (light). Edit both when changing
   any colour — they are separate token sets. The syntax tokens (`--sx-*`) also need both versions.
2. **Layout** is a fixed **LEFT** sidebar + centered 900px content column, with a special wide grid on posts.
3. **The TOC is custom and fragile-ish** — re-read §6 before restyling it; keep the grid + DOM-order +
   IntersectionObserver contract intact.
4. **You can't build locally** — make tasteful, low-risk CSS changes, push, and check the live site after
   Actions deploys (~1–2 min). Keep internal links valid so html-proofer doesn't block the deploy.
5. **The owner iterates on proportion and comfort** (spacing, sizes, balance). Make one coherent change at
   a time, commit + push, and describe what you changed and why so they can react. Confirm direction before
   shrinking things they previously asked to enlarge (e.g. the TOC, the avatar).
6. **Theme toggle is live** — don't remove or break the `themeToggle` button, the no-flash `<head>` script,
   or the `[data-theme="light"]` token block. All three must stay in sync.

When in doubt about a design decision that changes the site's character (palette, removing the sidebar,
banner imagery), ask the owner rather than guessing.
