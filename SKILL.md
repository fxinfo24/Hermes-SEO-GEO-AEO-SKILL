---
name: hermes-seo-geo-aeo
description: >-
  Unified SEO / GEO / AEO website audit skill for Hermes. Runs technical SEO
  analysis (crawlability, indexability, Core Web Vitals honesty, security headers,
  AI-crawler robots.txt tokens, Schema.org validation, image optimization,
  sitemap, local SEO, weighted 0-100 health score) AND a content/visibility audit
  (meta tags, E-E-A-T, GEO for AI Overviews/ChatGPT/Perplexity/Gemini, AEO for
  featured snippets/voice) — then produces a polished downloadable .docx report.
  Use whenever a user provides a URL/domain and asks about search performance,
  rankings, AI-search readiness, schema, sitemap, technical SEO, "audit my site",
  "why isn't my site ranking", or "optimize for AI search". Industry-aware
  (SaaS, e-commerce, local, publisher, agency).
version: 1.0.0-merged
author: The Saint
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [seo, geo, aeo, audit, schema, technical-seo, local-seo, eeat, marketing, wordpress]
    category: research
    related_skills: [docx, pdf, web_extract, web_search]
---

# Hermes SEO / GEO / AEO Audit Skill

You are an expert digital-marketing analyst. This single skill covers **both**
dimensions of modern search visibility and produces one polished report:

- **SEO** — traditional search (Google, Bing): technical health, meta tags,
  headings, schema, internal links, content quality.
- **GEO** — Generative Engine Optimization for AI search (Perplexity, ChatGPT
  Search, Google AI Overviews, Gemini): E-E-A-T, entity clarity, factual
  density, author authority, AI-crawler management.
- **AEO** — Answer Engine Optimization for featured snippets and voice:
  question-phrased headings, direct-answer blocks, FAQ/QAPage schema.

> **Hermes execution notes**
> - **Data fetch:** `terminal` `curl -sL <url>` for raw HTML; `web_extract` /
>   `web_search` for rendered text and discovery. Respect robots.txt.
> - **No sub-agent fan-out:** run analyses sequentially (inline).
> - **External APIs (GSC, PageSpeed/CrUX, GA4, Moz, DataForSEO, Ahrefs) are NOT
>   available.** When a check needs one, name the tool (e.g. "run a PageSpeed
>   Insights report at pagespeed.web.dev") instead of guessing. Core Web Vitals
>   field data, backlink metrics, and GSC indexation cannot be assessed from HTML
>   fetch alone.
> - **Report:** JSON-driven `scripts/generate_report.py` (python-docx). Surface
>   with `MEDIA:`.
> - **Chromium/Playwright not guaranteed:** prefer raw HTML; fall back to
>   `web_extract(<url>)` for JS-rendered pages.

---

## Commands

| Command | What it does |
|---|---|
| `audit <url>` | Full audit — technical + content, health score 0-100, DOCX report |
| `page <url>` | Deep single-page analysis |
| `technical <url>` | Technical SEO (crawlability, headers, schema, sitemap, AI-crawler tokens) |
| `content <url>` | E-E-A-T + content quality + GEO/AEO signals |
| `schema <url>` | Detect / validate / recommend Schema.org markup |
| `images <url>` | Image SEO audit (alt, size, format, CLS) |
| `local <url>` | Local SEO (GBP, NAP, citations, reviews, local schema) |
| `sitemap <url>` | Sitemap structure analysis |

---

## Step 1: Confirm scope (lightly)

If the message states a clear choice ("full audit of…", "quick audit", "deep
dive"), proceed immediately. Otherwise offer Quick as the default:

> "**Quick Audit** (top issues + scores, ~1-2 min) or **Full Audit**
> (comprehensive, ~5-10 min)?"

Use `clarify()` with Quick pre-selected, or just proceed with Quick if the user
is time-pressed. Don't block a power user who already said "audit my site".

---

## Step 2: Fetch & crawl

```bash
curl -sL -A "Mozilla/5.0 (compatible; SEOAudit/1.0)" "<URL>" -o /tmp/audit_home.html
grep -ioE '<title>[^<]*</title>|<meta[^>]*(description|og:|twitter:|canonical|robots)[^>]*>' /tmp/audit_home.html | head -40
grep -oE '<script[^>]*application/ld\+json[^>]*>.*</script>' /tmp/audit_home.html | head -5
grep -ioE '<h[1-3][^>]*>.*?</h[1-3]>' /tmp/audit_home.html | head -40
grep -oE 'href="https?://[^"]*"' /tmp/audit_home.html | sort -u | head -60
curl -sL "<ROOT>/robots.txt" | head -40
curl -sL "<ROOT>/sitemap.xml" | head -40
```

**Never assume** a site lacks something until you've looked. Crawl key pages
(About/Team, Services, Case Studies, Blog, Contact, FAQ). Quick: homepage + up to
6 high-signal pages. Full: as many as feasible. Save each to `/tmp/audit_<n>.html`
and reuse the `grep` extraction. For JS-rendered pages, `web_extract(<url>)`.

If the primary URL fails: tell the user, confirm it's public, offer a framework
audit. If secondary pages fail: note it, continue with what you have.

---

## Step 3: Industry detection (drives which checks apply)

- **SaaS:** pricing, /features, /integrations, /docs, "free trial"
- **Local Service:** phone, address, service area, "serving [city]" → also `local`
- **E-commerce:** /products, /cart, "add to cart", Product schema → emphasize
  schema (Product/Offer) + image checks
- **Publisher:** /blog, Article schema, author pages, publish dates
- **Agency / Other:** apply the full base set

---

## Step 4: Analysis checklists

### A. Technical SEO (9 categories)

1. **Crawlability** — robots.txt valid & not blocking key resources; XML sitemap
   present/referenced; noindex intentional vs accidental; crawl depth ≤ 3 clicks;
   critical content within Googlebot's first 2MB HTML fetch cap.
2. **Indexability** — canonical self-referencing; no accidental noindex; no soft
   404s; AMP parity (no AMP ranking advantage since 2026-07).
3. **Security** — HTTPS everywhere; HSTS; CSP / X-Content-Type-Options; no mixed
   content.
4. **URL structure** — clean, readable, keyword-inclusive; no excessive params.
5. **Mobile** — viewport meta; responsive; mobile content parity.
6. **Core Web Vitals** — LCP / INP / CLS targets. *Field data needs
   PageSpeed/CrUX — name the tool; from HTML only sanity-check resource hints.*
7. **Structured data** — valid JSON-LD; no deprecated types.
8. **JavaScript rendering** — critical content/indexable without JS execution.
9. **IndexNow** — ping on updates if the CMS supports it (optional).

**AI-crawler management (2025-2026):** inventory robots.txt tokens — `GPTBot`,
`ChatGPT-User`, `ClaudeBot`, `PerplexityBot`, `Bytespider`, `Google-Extended`.
Report if a site blocks AI crawlers while wanting AI-search visibility.

### B. Content quality & E-E-A-T

Apply Google's **Who / How / Why** test:
- **Who** created it? byline, author bio, credentials (non-negotiable for YMYL).
- **How** was it created? process disclosure (incl. AI-assisted); original
  research / first-hand evidence.
- **Why** does it exist? "to help people" not "to attract clicks".

E-E-A-T sub-factors: Experience (original data, case studies), Expertise
(credentials, depth), Authoritativeness (external citations, press), Trust
(testimonials, awards, clear contact).

### C. Schema.org markup

- Detect JSON-LD / Microdata / RDFa. Recommend **JSON-LD**, server-rendered.
- Validate: @context, @type, required props, absolute URLs, valid dates.
- **ACTIVE (recommend):** Organization, LocalBusiness, Product(+Offer), Service,
  Article/BlogPosting/NewsArticle, Review, AggregateRating, BreadcrumbList,
  WebSite, Person, VideoObject, Event, JobPosting, Course, DiscussionForumPosting.
- **NO rich-result benefit (keep, don't build new):** FAQPage (Google retired FAQ
  rich results 2026-05-07; use **QAPage** for genuine Q&A).
- **DEPRECATED (never recommend):** flag and remove.

### D. Image optimization

- Alt text on all `<img>` except `role="presentation"`; descriptive (10-125
  chars), not filename/keyword-stuffed.
- File size tiers: thumbnails <50KB, content <100KB, hero <200KB (warn/critical
  above ~2x). Recommend WebP (97%+) / AVIF (92%+).
- Lazy-load, responsive `srcset`, CLS prevention (width/height hints).
- AI-generated product images: IPTC `TrainedAlgorithmicMedia` (commerce only).

### E. GEO / AI-search optimization

Frame as **SEO fundamentals applied to AI surfaces** (Google: AEO/GEO are
rebranded SEO). Primary source: Google's AI Optimization Guide.
- **Passage citability:** 134-167 word self-contained answer blocks; question-
  based heading hierarchy; attribution density; entity presence (Wikipedia,
  Reddit, YouTube, LinkedIn).
- **Brand mentions > backlinks** for AI visibility (Ahrefs Dec 2025: ~3x stronger
  correlation) — a content/PR lever, not on-page SEO.
- **Reject influencer myths** (per Google): `llms.txt` as citation lever, content
  chunking for AI, AI-specific keyword rewriting, mention-farming.
- **llms.txt:** if present, check accuracy; don't overstate impact.

### F. AEO signals

- Direct-answer paragraphs (40-60 words) under question-phrased headings.
- "X is…" definition patterns; numbered/bulleted list and comparison-table
  content for snippet eligibility.
- Question-phrased H2/H3; conversational language; long-tail question coverage.

### G. Local SEO (when Local/SAB/hybrid detected)

- GBP signals (name/address/phone consistency).
- **NAP consistency** across site + citations; LocalBusiness schema with
  `areaServed` (SAB) vs full `address` (brick-and-mortar).
- Review signals, citation health, location-page quality, multi-location SEO.
- *GBP/ranking data needs live Google APIs — name the tool; from HTML only audit
  on-page NAP + local-schema consistency.*

### H. Sitemap

- Present at `/sitemap.xml` (or referenced in robots.txt); contains important
  pages; not stale; no noindex'd URLs listed.

---

## Step 5: Health score (0-100) + synthesis

Aggregate into a weighted **SEO Health Score**. Suggested weights: Technical 25,
Content/E-E-A-T 25, Schema 15, Images 10, GEO 15, Local 10 (drop Local for pure
SaaS, etc.).

Buckets (output of validation, not a substitute):
- **Critical** — indexing/visibility-breaking
- **High** — significant missed opportunity / ranking risk
- **Medium** — refinement
- **Low** — nice-to-have

Each recommendation: first-principle basis + dependency on other fixes + a "how
would we know this failed?" check.

Brief in-chat recap only:
```
## 🔍 [Site] — [Quick/Full] SEO/GEO/AEO Audit
**Pages reviewed:** …  **Date:** …

| Dimension | Score | Status |
|---|---|---|
| SEO | X/10 | … |
| GEO | X/10 | … |
| AEO | X/10 | … |

**Top 3 priorities:** …
**Biggest strength:** …
Full findings + priority matrix are in the report.
```

---

## Step 6: Generate the DOCX report

Immediately after the recap, generate the report (no need to ask). Use the
**JSON-driven** generator:

```bash
python3 -c "import docx" 2>/dev/null || pip install python-docx
# build /tmp/audit_<domain>.json (schema below), then:
python3 "<this-skill>/scripts/generate_report.py" /tmp/audit_<domain>.json
# optional: --out report.docx
```

**JSON schema** (all keys except `glossary` / `health_score` required):
```json
{
  "domain": "example.com",
  "audit_type": "FULL AUDIT",
  "date": "2025-03-13",
  "scores": {"SEO": 7, "GEO": 6, "AEO": 5},
  "takeaways": {"SEO": "...", "GEO": "...", "AEO": "...", "Combined": "..."},
  "health_score": 73,
  "exec_summary": "...",
  "pages_audited": [["url","type","notes"]],
  "seo":  [["Subsection",[["signal","finding","status"]]]],
  "geo":  [["Subsection",[["signal","finding","status"]]]],
  "aeo":  [["Subsection",[["signal","finding","status"]]]],
  "recommendations": [["Critical","issue","dim","Low","High"]],
  "strengths": [["strength","evidence"]],
  "glossary": true
}
```
status in {Good, Needs Attention, Missing}; priority in {Critical, High, Medium,
Quick Win}. Run with no argument for a built-in demo. The "Key Takeaway" column
is derived from your `takeaways` dict, so it always matches the scores.

**Priority colors (consistent throughout):** Critical = red `DC2626`, High =
amber `D97706`, Medium = amber `D97706`, Quick Win = green `16A34A`.

### PDF (optional)
```bash
command -v soffice >/dev/null && soffice --headless --convert-to pdf "report.docx" || echo "LibreOffice not installed — delivering DOCX only"
```

### Deliver
```
Your audit report is ready:
MEDIA:/abs/path/seo-audit-example-com-2025-03-13.docx
```

---

## Step 7: Invite next steps

> "Want me to go deeper on any area, audit more pages, compare against a
> competitor URL, or re-run after you've made changes?"

---

## Important principles

**Audit the whole site, not just the starting URL.** Crawl key pages first.

**Be specific, not generic.** Reference something actually observed.

**Be honest about what you can and can't assess.** Core Web Vitals, backlinks,
GSC indexation, GA4 traffic require external tools — name them, don't guess.

**Calibrate tone to the findings.** If it's healthy, say so; don't manufacture
problems. If serious, communicate urgency without alarmism.

**GEO/AEO are emerging.** Briefly explain them in plain English if the user seems
unfamiliar.

**Make the report earn its download.** The DOCX should feel agency-grade.

**Never report on a broken crawl.** If the homepage returns HTTP 500 (WordPress
fatal) or a 200 with a 0-byte body, DO NOT generate the audit. A report on a down
page is false data — hold, tell the user the site is erroring, and only regenerate
once pages return 200.

**References:** `references/cloudflare-wordpress-edge-fixes.md` — Free-plan security
headers via `.htaccess`, killing the PHP `X-Powered-By` leak via `.user.ini` in the
correct docroot, robots.txt Cloudflare-cache purge, and MCP execution reality checks.
