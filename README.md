# WordPress Performance Checklist

A structured, opinionated checklist for achieving **90+ PageSpeed scores** on WordPress sites — built from 10+ years of real-world frontend and team lead experience across international client projects.

This is not a collection of generic tips. Every item here has been validated on production sites, and every category reflects a decision I make when auditing a new WordPress environment.

> **Who this is for:** WordPress developers, frontend leads, and agency teams who want a repeatable process for performance — not a one-time fix.

---

## Table of Contents

- [How to use this checklist](#how-to-use-this-checklist)
- [Phase 1 — Hosting & Server](#phase-1--hosting--server)
- [Phase 2 — Theme & Code](#phase-2--theme--code)
- [Phase 3 — Images & Media](#phase-3--images--media)
- [Phase 4 — JavaScript & CSS](#phase-4--javascript--css)
- [Phase 5 — Caching & Delivery](#phase-5--caching--delivery)
- [Phase 6 — Plugin Hygiene](#phase-6--plugin-hygiene)
- [Phase 7 — Monitoring & Maintenance](#phase-7--monitoring--maintenance)
- [Tools I use](#tools-i-use)
- [Before & After template](#before--after-template)
- [Contributing](#contributing)

---

## How to use this checklist

Work through each phase in order — the phases are sequenced by impact and dependency. Don't jump to Phase 4 (JS/CSS) before Phase 1 (Hosting) is solid; you'll be polishing a weak foundation.

Each item is marked with a priority level:

- 🔴 **Critical** — do this first, highest impact
- 🟡 **Important** — significant gain, do this in your sprint
- 🟢 **Nice to have** — marginal gain, do when time allows

After completing an audit, use the [Before & After template](#before--after-template) to document your results without revealing client details.

---

## Phase 1 — Hosting & Server

The fastest WordPress site on bad hosting is still a slow WordPress site. Infrastructure is where most teams underinvest.

### Server environment
- [ ] 🔴 Use a managed WordPress host (Kinsta, WP Engine, Cloudways, or similar) rather than generic shared hosting
- [ ] 🔴 PHP 8.1+ enabled — PHP 8.x is measurably faster than PHP 7.x for WordPress
- [ ] 🔴 Server located in the same region as your primary audience
- [ ] 🟡 HTTP/2 or HTTP/3 enabled — verify with [https://tools.keycdn.com/http2-test](https://tools.keycdn.com/http2-test)
- [ ] 🟡 GZIP or Brotli compression enabled at the server level
- [ ] 🟡 Keep-Alive connections enabled
- [ ] 🟢 OPcache enabled and configured for your PHP version

### Database
- [ ] 🔴 MySQL 8.0+ or MariaDB 10.4+
- [ ] 🟡 Autoloaded options under 800KB — check with `SELECT SUM(LENGTH(option_value)) FROM wp_options WHERE autoload = 'yes'`
- [ ] 🟡 Post revisions limited (set `WP_POST_REVISIONS` to 5 or fewer in `wp-config.php`)
- [ ] 🟡 Scheduled database cleanup running (revisions, trashed posts, expired transients, spam comments)
- [ ] 🟢 Object cache configured (Redis or Memcached) if your host supports it

### SSL & Security headers
- [ ] 🔴 HTTPS enforced with valid SSL certificate
- [ ] 🟡 HSTS header set
- [ ] 🟢 Security headers configured (X-Frame-Options, X-Content-Type-Options, Referrer-Policy)

---

## Phase 2 — Theme & Code

Theme quality has more impact on performance than most developers expect — especially page builders.

### Theme selection & architecture
- [ ] 🔴 Theme uses minimal third-party dependencies (avoid themes that load 10+ scripts globally)
- [ ] 🔴 No unused page builder markup on pages that don't use a page builder
- [ ] 🟡 Child theme in use if using a parent theme (avoids losing custom code on updates)
- [ ] 🟡 Theme does not register unnecessary `wp_enqueue_scripts` globally — load assets per-page where possible
- [ ] 🟡 Custom fonts loaded with `font-display: swap` and subset where possible
- [ ] 🟢 Theme code follows WordPress coding standards — avoids `SELECT *` queries, uses `WP_Query` correctly

### Core Web Vitals targets
- [ ] 🔴 LCP (Largest Contentful Paint) under 2.5s — identify LCP element and prioritise its load
- [ ] 🔴 CLS (Cumulative Layout Shift) under 0.1 — set explicit width/height on all images and embeds
- [ ] 🔴 INP (Interaction to Next Paint) under 200ms — audit long JavaScript tasks
- [ ] 🟡 TTFB (Time to First Byte) under 600ms — primarily a hosting/caching problem
- [ ] 🟡 FCP (First Contentful Paint) under 1.8s

### Render-blocking resources
- [ ] 🔴 No render-blocking CSS in `<head>` that isn't needed for above-the-fold content
- [ ] 🔴 No render-blocking JS — all non-critical scripts use `defer` or `async`
- [ ] 🟡 Critical CSS inlined for above-the-fold content (use Autoptimize or manual extraction)
- [ ] 🟢 `preconnect` hints added for third-party origins (Google Fonts, analytics, CDN)

---

## Phase 3 — Images & Media

Images are consistently the largest performance problem on WordPress sites. This phase alone can move PageSpeed scores by 20–30 points.

### Format & compression
- [ ] 🔴 All images served in WebP format (use Imagify, ShortPixel, or Cloudflare's Polish)
- [ ] 🔴 No images over 200KB on the front end without a strong reason
- [ ] 🟡 AVIF support offered where host/CDN supports it (better compression than WebP)
- [ ] 🟡 SVGs used for logos and icons instead of PNG/JPG
- [ ] 🟡 Image compression configured at upload level — don't rely only on caching layer
- [ ] 🟢 Responsive image `srcset` attributes present on all `<img>` tags

### Lazy loading & prioritisation
- [ ] 🔴 `loading="lazy"` on all images below the fold
- [ ] 🔴 `loading="eager"` and `fetchpriority="high"` on the LCP image — do NOT lazy-load it
- [ ] 🟡 Hero/banner images preloaded with `<link rel="preload">` in `<head>`
- [ ] 🟡 No off-screen images loaded on initial page load

### Embeds & media
- [ ] 🟡 YouTube embeds replaced with a facade (lite-youtube-embed or similar) — a single YouTube embed can add 500KB+
- [ ] 🟡 Google Maps embeds replaced with a static image + click-to-load pattern
- [ ] 🟢 Video hosted externally (YouTube/Vimeo) rather than served from WordPress media library

---

## Phase 4 — JavaScript & CSS

This phase requires the most technical judgement. The goal is not to eliminate JS — it's to eliminate JS that blocks rendering or isn't needed.

### JavaScript
- [ ] 🔴 Total JS payload under 200KB (compressed) for a typical page
- [ ] 🔴 No jQuery loaded on pages that don't need it — or replaced with vanilla JS where feasible
- [ ] 🔴 All non-critical scripts deferred (`defer` or `async` attribute)
- [ ] 🟡 Third-party scripts audited — each one adds latency; remove or delay any that aren't essential
- [ ] 🟡 JavaScript loaded conditionally per page/post type where possible (use `is_page()`, `is_single()` etc.)
- [ ] 🟡 Unused JS removed — use Chrome DevTools Coverage tab to identify dead code
- [ ] 🟢 Code splitting implemented if using a React/block-based theme

### CSS
- [ ] 🔴 Total CSS payload under 50KB (compressed) for a typical page
- [ ] 🟡 Unused CSS removed — use PurgeCSS or the Coverage tab in DevTools
- [ ] 🟡 CSS minified
- [ ] 🟡 No CSS `@import` inside stylesheets (blocks parallel loading) — use `<link>` tags instead
- [ ] 🟢 CSS loaded conditionally per template where possible

### Third-party scripts
- [ ] 🔴 Google Tag Manager audited — it's often a container for unoptimised tags
- [ ] 🔴 Chat widgets, marketing scripts, and A/B testing tools loaded after user interaction where possible
- [ ] 🟡 Analytics script loaded with `async` and only on production
- [ ] 🟡 Facebook Pixel and other tracking pixels deferred or loaded via GTM with triggers
- [ ] 🟢 Self-host Google Fonts instead of loading from `fonts.googleapis.com`

---

## Phase 5 — Caching & Delivery

A well-cached site can serve most visitors without touching PHP or the database at all.

### Page caching
- [ ] 🔴 Full-page cache active (WP Rocket, W3 Total Cache, LiteSpeed Cache, or host-level cache)
- [ ] 🔴 Cache correctly excluded for: logged-in users, WooCommerce cart/checkout, search results
- [ ] 🟡 Cache preloading configured — warm the cache after purge rather than waiting for visitors
- [ ] 🟡 Cache lifetime appropriate — static pages can cache for 24h+; frequently updated content shorter

### CDN
- [ ] 🔴 CDN configured for static assets (images, JS, CSS) — Cloudflare, BunnyCDN, KeyCDN, or similar
- [ ] 🟡 Cloudflare (or CDN) caching rules configured correctly — don't cache admin or WooCommerce pages
- [ ] 🟡 CDN serving WebP automatically where supported (Cloudflare Polish, BunnyCDN Image Optimizer)
- [ ] 🟢 CDN edge locations verified for your primary audience geography

### Browser caching
- [ ] 🟡 Cache-Control headers set for static assets — at minimum 1 year for versioned assets
- [ ] 🟡 ETags configured correctly
- [ ] 🟢 Service worker implemented for offline/repeat visit performance (advanced — not always appropriate)

---

## Phase 6 — Plugin Hygiene

Plugins are WordPress's biggest performance liability. Each one can add database queries, scripts, and styles to every page load.

### Audit process
- [ ] 🔴 List all active plugins and their purpose — if you can't justify a plugin in one sentence, deactivate it
- [ ] 🔴 Check each plugin's impact with Query Monitor — identify plugins adding excessive DB queries
- [ ] 🔴 Deactivate and delete (not just deactivate) all plugins not in active use
- [ ] 🟡 Check each plugin loads scripts/styles globally vs. conditionally
- [ ] 🟡 Replace bloated multi-feature plugins with focused single-purpose alternatives where possible

### Common offenders
- [ ] 🟡 Contact form plugins (Gravity Forms, CF7) — load scripts globally by default; restrict to pages with forms
- [ ] 🟡 Slider plugins — almost always replaceable with CSS or lightweight alternatives
- [ ] 🟡 Social sharing plugins — often load multiple external scripts; replace with simple share links
- [ ] 🟡 SEO plugins — Yoast and RankMath are fine, but audit what they load on the front end
- [ ] 🟢 WooCommerce — if used, audit cart fragments and ensure AJAX cart doesn't break caching

### Plugin loading control
- [ ] 🟡 Use a plugin like Asset CleanUp or WP Rocket's asset manager to disable plugins per page/post type
- [ ] 🟢 Consider `mu-plugins` for site-critical code that doesn't need a plugin UI

---

## Phase 7 — Monitoring & Maintenance

Performance isn't a one-time fix. Scores degrade as content, plugins, and code accumulate.

### Ongoing monitoring
- [ ] 🔴 Set up Google Search Console — monitors Core Web Vitals from real user data (CrUX)
- [ ] 🔴 Run PageSpeed Insights (field data, not just lab data) on key pages monthly
- [ ] 🟡 Set up UptimeRobot or Better Uptime for availability monitoring
- [ ] 🟡 Schedule monthly database cleanup (post revisions, expired transients, spam)
- [ ] 🟡 Review plugin list quarterly — remove anything no longer needed
- [ ] 🟢 Set up automated Lighthouse CI in your deployment pipeline (GitHub Actions + Lighthouse CI)

### After major changes
- [ ] 🔴 Run full PageSpeed audit after any major plugin update, theme update, or new feature launch
- [ ] 🟡 Check Core Web Vitals in Chrome DevTools after significant layout changes
- [ ] 🟡 Purge all caches and re-test after server or PHP version upgrades
- [ ] 🟢 Run a monthly WebPageTest report on your primary landing page and archive the results

---

## Tools I use

These are the tools I reach for regularly — not a comprehensive list, just what I've found reliable in production.

| Category | Tool | Notes |
|---|---|---|
| Auditing | [PageSpeed Insights](https://pagespeed.web.dev) | Start here — field + lab data |
| Auditing | [WebPageTest](https://www.webpagetest.org) | Deeper waterfall analysis |
| Auditing | [GTmetrix](https://gtmetrix.com) | Good for historical comparison |
| Debugging | [Query Monitor](https://wordpress.org/plugins/query-monitor/) | Essential — DB queries, hooks, slow scripts |
| Debugging | Chrome DevTools (Coverage tab) | Identify unused JS/CSS |
| Images | [Imagify](https://imagify.io) | WebP + compression at upload |
| Images | [Squoosh](https://squoosh.app) | Manual optimisation for key images |
| Caching | [WP Rocket](https://wp-rocket.me) | My go-to paid caching plugin |
| Caching | [LiteSpeed Cache](https://wordpress.org/plugins/litespeed-cache/) | Best option on LiteSpeed servers (free) |
| CDN | [Cloudflare](https://cloudflare.com) | Free tier covers most sites |
| CDN | [BunnyCDN](https://bunny.net) | Better performance, affordable |
| Monitoring | [Google Search Console](https://search.google.com/search-console) | Real user CWV data |
| Monitoring | [UptimeRobot](https://uptimerobot.com) | Free uptime monitoring |
| Fonts | [google-webfonts-helper](https://gwfh.mranftl.com) | Self-host Google Fonts |

---

## Before & After template

Use this to document performance work without revealing client details. Useful for your own records, team retrospectives, or portfolio evidence.

```
## Project: [internal codename or generic description e.g. "E-commerce site, WooCommerce, ~200 products"]
**Date:** [Month Year]
**Stack:** [e.g. WordPress 6.x, Elementor, WooCommerce, Cloudflare, Kinsta]

### Before
- PageSpeed Mobile: XX
- PageSpeed Desktop: XX
- LCP: X.Xs
- CLS: X.XX
- TTFB: XXXms
- Total page weight: X.XMB
- Notable issues: [e.g. "3 render-blocking scripts, no image compression, 47 plugins active"]

### After
- PageSpeed Mobile: XX
- PageSpeed Desktop: XX
- LCP: X.Xs
- CLS: X.XX
- TTFB: XXXms
- Total page weight: X.XMB

### What changed
- [Change 1]
- [Change 2]
- [Change 3]

### Time invested
[X hours]

### Key learning
[One sentence on what surprised you or what you'd do differently]
```

---

## Contributing

Found something missing, outdated, or wrong? PRs are welcome.

If you have a tool recommendation or a checklist item from real production experience, open an issue with a brief explanation of the context — I'll review and merge anything that's genuinely battle-tested.

---

*Built and maintained by [Hamidreza (Hoomaan) Sheikholeslami](https://hoomaan.dev) — Frontend Engineering Lead with 10+ years of WordPress performance work across US, UK, and European clients.*
