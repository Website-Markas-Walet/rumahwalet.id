# rumahwalet.id — CMS Integration Profile & Brief Response

`site_id: rumahwalet.id` · Pilot candidate · Prepared for cms.markaswalet.id (Next.js + Supabase / Vercel, media on Cloudflare R2 → cdn.markaswalet.id)

All facts below are **verified directly from the repo**, not assumed. Scope is understood as **additive**: CMS generates/edits content and commits to this repo; navbar/menu/footer and existing URLs are **not** changed.

---

## ★ BLOCKER #1 — Deploy mechanism (CONFIRMED)

| Item | Value |
|---|---|
| **How repo → live** | **GitHub Pages, served directly from a branch** (classic Pages). Verified: `CNAME` = `rumahwalet.id` present at repo root; **no `.github/workflows/`** (no Actions build); no server, no build step. What is committed is what is served, verbatim. |
| **Production branch** | **`main`** (default). Live site = `main` HEAD. |
| **Publish flow** | `git commit` + `git push origin main` → GitHub Pages redeploys automatically. **No build hook needed.** |
| **⚠️ Jekyll trap** | **No `.nojekyll` file.** GitHub Pages runs Jekyll by default, which **ignores files/dirs starting with `_`**. We currently have none, so nothing breaks *today* — but the CMS must **never** emit `_`-prefixed paths, or add a `.nojekyll` (recommended, one-time, zero-risk). |

**→ The "Publish" button target = commit to `main` of `Website-Markas-Walet/rumahwalet.id`.**

---

## ★ BLOCKER #2 — Write access for CMS (mechanism, needs human action)

GitHub-hosted, so CMS commits via the Git API. **Recommended: GitHub App** (finest scope, no personal token, revocable, higher rate limits).

- **Preferred:** a **GitHub App** installed on `Website-Markas-Walet/rumahwalet.id` with **Contents: Read & Write** only. CMS authenticates as an installation and commits to `main`.
- **Acceptable fallback:** a **deploy key** (SSH, write-enabled) scoped to this single repo — simplest if the CMS commits over SSH.
- **Avoid:** a broad personal PAT (org-wide blast radius).

**I cannot mint credentials from here** — this is an admin action on the GitHub org. Tell me which mechanism you choose and I'll confirm the repo is ready (branch protection, `.nojekyll`, path conventions). *Do not paste tokens in chat.*

---

## Point 3 — Chrome captured as a layout template ✅ delivered

Files in this `cms/` dir:

| File | What it is |
|---|---|
| `layout.html` | **Full page skeleton** with a `<!-- CONTENT -->` slot and 9 `{{PLACEHOLDERS}}` (TITLE, META_DESCRIPTION, CANONICAL_ABS_URL, OG_TYPE, PUBLISHED_ISO, MODIFIED_ISO, JSON_LD, PAGE_CSS, BODY_CLASS). Wrap new pages with this; existing pages untouched. |
| `partials/header.html` | **Canonical chrome header**, byte-exact (85 lines) — extracted from a kota page (the 900+-page common case, **no active-state baked in**). |
| `partials/footer.html` | **Canonical chrome footer** + trailing script manifest (86 lines + scripts to `</html>`). |
| `partials/head-seo.html` | Head top: FB domain-verify, **GTM-T5V7HLN**, charset/viewport, + per-page SEO block as placeholders. |
| `partials/head-assets.html` | **Verbatim** theme/plugin CSS+JS manifest (25 asset tags: Astra, Elementor, Essential Addons, WPForms). Inject as-is. |

**⚠️ Chrome is NOT purely byte-identical — one nuance the CMS must handle:**
The header structure is identical everywhere (same 85 lines), but WordPress baked the **active-menu state into the markup per URL**: the nav `<li>` matching the current path gets `current-menu-item … current_page_item` + `aria-current="page"` (and parent gets `current-menu-ancestor…`). The canonical partials I extracted have **no** active item (safe base). **The CMS must set active-state by matching the current URL against the menu** when wrapping a page — it's deterministic, not free-form.

**Menu structure (source of truth for the CMS menu model):**
```
Home              /
Tentang Kami      /tentang-kami
Layanan           /layanan   ▸ Bangun Rumah Walet     /layanan/bangun-rumah-walet
                             ▸ Desain Rumah Walet     /layanan/desain-rumah-walet
                             ▸ Instalasi Rumah Walet  /layanan/instalasi-rumah-walet
                             ▸ Perawatan Rumah Walet  /layanan/perawatan-rumah-walet
                             ▸ Area Pelayanan         /layanan/area-pelayanan
Kontak            /kontak
Blog              /blog
Footer links:     Kontak · Kebijakan Privasi
```

---

## Point 4 — URL patterns (exact, with the quirks)

| Content type | Pattern | Count | Quirk |
|---|---|---|---|
| **kota** (SEO engine) | `/kota/jasa-bangun-desain-rumah-walet-di-{kabupaten}-{provinsi}/` | **489** | Folder-style (served via `index.html`), **trailing slash**. City+province are hyphen-joined in one slug (see `data/kota-dataset.csv`). |
| **blog** | `/blog/{slug}/` | **164** | Folder-style, trailing slash. |
| **artikel (root long-form)** | `/{slug}.html` | **8** | ⚠️ **literal `.html` at root, NO folder, NO trailing slash** — different pattern from everything else. Do not "normalize" these. |
| **layanan** | `/layanan/{bangun\|desain\|instalasi\|perawatan\|area-pelayanan}/` + `/layanan/` | 6 | Folder-style. |
| **desain** | `/desain/{slug}/` (ukuran: 4x4, 8x8, 10x10, 12x12, 14x14) | 5 | Folder-style. |
| **static pages** | `/`, `/tentang-kami/`, `/kontak/`, `/kebijakan-privasi/`, `/terms-and-conditions/` | 5 | Folder-style. |
| **legacy dumps (DEAD)** | `/category/`, `/author/`, `/2023/`, `/wp-json/`, `/feed/`, `/comments/` | — | Frozen WordPress residue. **Not content types** — do not model. |

**Hard rule:** slugs are the SEO contract. Two URL shapes coexist (`.html` root vs `/dir/`) and **must both be preserved**. Any slug change → ship a 301 map.

---

## Point 5 — Site config

- **Analytics:** **GTM-T5V7HLN** (Google Tag Manager, in `<head>` + `<noscript>`). GA4 fires *inside* GTM (no direct `gtag`/UA in source). **Facebook domain-verification:** `mjcpwxnx5xqytl3akzmoiko58ugine`. No TikTok/AdSense/UA tags found.
- **Brand:** primary **`#FFB100`** (walet yellow), secondary **`#F9C349`**; text `#3A3A3A`/`#000000`. Fonts **Poppins** (headings 500/600) + **Open Sans** (body) via Google Fonts; Astra also pulls Roboto/Roboto Slab.
- **Logo:** `/wp-content/uploads/2023/03/logorumahwalet.id.png` (180×30 in header). **Favicon:** `/wp-content/uploads/2023/03/Favicon.png`. Schema logo: `/wp-content/uploads/2023/03/1677138874027.png`.
- **Contact:** WhatsApp **+62 877 2526 0196** (`wa.me/6287725260196`) · email **sales@markaswalet.com** · address (JSON-LD): *Jl. Gubeng Kertajaya, Surabaya, Jawa Timur*.
- **Ecosystem/network menu** (outbound links found in chrome): pelatihanwalet.com, ternakwalet.com, suarawalet.id, pemikatwalet.com, pencuciansarangwalet.com — **candidate for centralized network-menu control**.
- **SEO stack:** **Rank Math** (title/desc/robots/canonical/OG/Twitter + JSON-LD `@graph`: Organization, WebSite, WebPage, Article, Person, Place). Author entity: `alwiishihab`.

---

## Point 6 — Structured content export

- **kota → dataset, NOT N pages:** `data/kota-dataset.csv` (489 rows: `slug, url_path, location_raw`). Model as **one template × dataset**. `location_raw` is `{kabupaten}-{provinsi}` hyphen-joined — needs a split pass (province is the trailing token group). This is the master pattern to reuse network-wide.
- **articles/blog:** fields available per page for extraction — `title, slug, body HTML, meta_description, robots, canonical, og:*, published_time, modified_time, JSON-LD (author, category), cover image`. Body lives in `entry-content` / Elementor container inside the `#content` wrapper. (Full structured JSON export can follow once the CMS content schema is fixed.)

---

## Point 7 — Media inventory

- **Local (safe):** `wp-content/uploads` = **68 MB**, committed to git (2019/2020/2023 + elementor). Referenced mixed relative + absolute `https://rumahwalet.id/...`.
- **⚠️ External hotlinks to REHOME to R2 → `data/media-hotlinks.txt`** (29 external hosts). rumahwalet.id is the **worst offender in the network**: **1,120 refs to `1.bp.blogspot.com`**, plus tstatic, hatena, pgimgs, burungnya.com, newshanter, radarsulbar, borneonews, jambiupdate, etc. Fragile + copyright risk → CMS should rehost these to `cdn.markaswalet.id`.
- **Fonts:** runtime from `fonts.googleapis.com` (fine for static).

---

## Point 8 — URL/slug stability & other flags

- **Do not change URLs/slugs.** Preserve both URL shapes. Provide 301 map on any change.
- **Sitemaps present:** `sitemap_index.xml` → `kota-sitemap1/2/3.xml`, `post-sitemap.xml`, `page-sitemap.xml`, `desain-sitemap.xml`. **CMS must regenerate these** when it adds/edits content.
- **⚠️ No `robots.txt`** — recommend adding one (+ sitemap ref).
- **⚠️ SEO bug to fix while templatizing:** legacy pages set **`<link rel=canonical href="/">`** and `og:url="/"` (relative) and JSON-LD `Organization.url="http://rumahwalet.id"` (http). The `layout.html` placeholder `{{CANONICAL_ABS_URL}}` expects an **absolute https** URL — CMS should emit absolute canonicals going forward.
- **⚠️ Stray backlink:** homepage footer only contains a hidden link `Powered by rumahwalet` → **`https://saetech.biz.id`** (not on other pages). Flagging — likely unwanted; decide keep/remove.
- **⚠️ Leaked non-prod refs:** a `http://rumahwalet-coba.test` reference exists in source — should be scrubbed.

---

## TL;DR for the CMS session

Static site on **GitHub Pages, branch `main`, no CI** → publish = commit to `main` (need **GitHub App / deploy key**, Blocker #2). Additive only. Chrome is **one canonical template** (`cms/layout.html` + partials) — but **active-menu state is per-URL**, so the CMS must set it by URL match, not copy verbatim. Content = **4 live types**: **kota (489, dataset-driven — biggest asset)**, **blog (164)**, **root articles (8, `.html` pattern)**, **layanan/desain/pages**. **URLs frozen** (two shapes coexist). Media: **1,120 blogspot hotlinks to rehome to R2** (worst in network). SEO = Rank Math; regenerate sitemaps; add robots.txt; fix relative→absolute canonicals. GTM-T5V7HLN; brand `#FFB100`. Clean chrome + push-to-deploy = **good pilot.**
