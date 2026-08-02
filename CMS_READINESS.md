---
site: rumahwalet.id
deploy: { host: github_pages, cf_account: null, branch: main, deploy_hook: absent }
url_patterns: { artikel: "/{slug}.html", kota: "/kota/jasa-bangun-desain-rumah-walet-di-{kabupaten}-{provinsi}/", page: "/{slug}/", desain: "/desain/{slug}/" }
chrome: { header_lines: "85", footer_lines: "86", byte_identical: "variants:active-state-only" }
analytics: { gtm: "GTM-T5V7HLN", ga4_property_id: "", tiktok: "", fb_verify: "mjcpwxnx5xqytl3akzmoiko58ugine", adsense: "" }
brand: { primary: "#FFB100", font: "Poppins + Open Sans", logo: "/wp-content/uploads/2023/03/logorumahwalet.id.png", favicon: "/wp-content/uploads/2023/03/Favicon.png" }
ecosystem_menu: present
contact: { wa: "+6287725260196", email: "sales@markaswalet.com" }
media: { local_mb: 68, hotlink: "blogspot:1120,tstatic:67,hatena:61,pgimgs:55,burungnya:43,others:~200" }
content: { articles: 172, kota_unique: 489, kota_admin_level: kabupaten }
cleanup: [".nojekyll missing (Jekyll active on Pages)", "canonical & og:url are relative '/' — must be absolute https", "Organization JSON-LD url is http:// not https", "homepage-only hidden footer backlink to saetech.biz.id", "leaked non-prod ref http://rumahwalet-coba.test in source", "no robots.txt", "1120 blogspot image hotlinks (fragile) — rehome to R2"]
blockers: ["write access for CMS not yet provisioned (needs GitHub App w/ Contents:R&W, or repo deploy key — org admin action)", "this session push currently returns 403 (write permission not enabled)"]
---

# rumahwalet.id — CMS Readiness

**What I am:** static HTML export of a WordPress site (WP 6.1.1 / Astra theme / Elementor 3.11.2, exported by Simply Static). **1,687 HTML files, 226 MB.** Not a live WordPress — what is committed is what is served.

**Deploy (Blocker #1 — confirmed):** GitHub Pages served **directly from branch `main`**, **no Actions/CI**, no build step. `CNAME` = rumahwalet.id. Publish = commit to `main`. No deploy hook needed. ⚠️ No `.nojekyll` (Jekyll is active by default — CMS must never emit `_`-prefixed paths; adding `.nojekyll` recommended).

**Write access (Blocker #2):** GitHub-hosted → CMS commits via Git API. Recommend a **GitHub App** (Contents: Read & Write, scoped to this one repo) or a repo **deploy key**. Avoid broad PAT. Requires an org-admin action — cannot be minted from the session.

## URL patterns (exact, with quirks)

- **kota** (489, the SEO engine): `/kota/jasa-bangun-desain-rumah-walet-di-{kabupaten}-{provinsi}/` — folder-style, trailing slash. Kabupaten+provinsi are hyphen-joined in one slug.
- **artikel root** (8): `/{slug}.html` — ⚠️ literal `.html`, no folder, no trailing slash. Different from everything else — do not normalize.
- **blog** (164): `/blog/{slug}/` — folder-style, trailing slash.
- **page**: `/{slug}/` (e.g. `/tentang-kami/`, `/kontak/`, `/kebijakan-privasi/`, `/terms-and-conditions/`) — folder-style.
- **layanan** (6): `/layanan/{bangun|desain|instalasi|perawatan|area-pelayanan}/` + `/layanan/`.
- **desain** (5): `/desain/{slug}/` (ukuran 4x4, 8x8, 10x10, 12x12, 14x14).
- **DEAD legacy dumps** (do not model): `/category/`, `/author/`, `/2023/`, `/wp-json/`, `/feed/`, `/comments/`.

**Hard rule:** slugs are the SEO contract; two URL shapes (`.html` root vs `/dir/`) coexist and both must be preserved. Any change → 301 map.

## Chrome

- Structurally **identical everywhere**: header **85 lines**, footer **86 lines**. Menu is fixed across all pages.
- **NOT purely byte-identical:** WordPress baked **active-menu state into markup per URL** — the `<li>` matching the current path gets `current-menu-item … current_page_item` + `aria-current="page"` (parents get `current-menu-ancestor`). So the CMS must set active-state **by URL match**, not copy verbatim.
- Homepage-only footer difference: a hidden backlink `Powered by rumahwalet` → `https://saetech.biz.id` (see cleanup).
- **Extracted template available** on branch `claude/markaswalet-centralized-cms-2xkqvr` under `cms/`: `layout.html` (with `<!-- CONTENT -->` slot + 9 SEO placeholders), `partials/header.html`, `partials/footer.html`, `partials/head-seo.html`, `partials/head-assets.html`, plus `data/kota-dataset.csv` (489 rows) and `data/media-hotlinks.txt` (29 hosts).

## Config

- **Analytics:** GTM-T5V7HLN (GA4 fires inside GTM; no direct gtag/UA in source). FB domain-verify present. No TikTok/AdSense.
- **Brand:** primary `#FFB100` (walet yellow), secondary `#F9C349`; text `#3A3A3A`/`#000000`. Fonts Poppins (headings) + Open Sans (body).
- **Contact:** WA +62 877 2526 0196 (`wa.me/6287725260196`), email sales@markaswalet.com. Address (JSON-LD): Jl. Gubeng Kertajaya, Surabaya, Jawa Timur.
- **Ecosystem menu present** (outbound network links in chrome): pelatihanwalet.com, ternakwalet.com, suarawalet.id, pemikatwalet.com, pencuciansarangwalet.com — candidate for centralized network-menu control.
- **SEO stack:** Rank Math (title/desc/robots/canonical/OG/Twitter + JSON-LD @graph). Author entity: `alwiishihab`. Sitemaps present (`sitemap_index.xml` → kota-sitemap1/2/3, post/page/desain) — CMS must regenerate on publish.

## Content

- **articles: 172** (blog 164 + root long-form 8). **kota_unique: 489** at **kabupaten** admin level (model as one template × dataset, not 489 manual pages). desain: 5. Extractable fields per page: title, slug, body HTML, meta description, robots, canonical, og:*, published/modified time, JSON-LD (author, category), cover.

## Notes

Cleanest chrome in the network + push-to-deploy simplicity + no CI → **good pilot candidate**. Worst media debt in the network (1,120 blogspot hotlinks) needs rehoming to R2. Scope confirmed additive: existing pages/URLs untouched.
