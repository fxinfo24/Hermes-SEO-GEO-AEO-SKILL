# Hermes SEO/GEO/AEO Skill

**A unified SEO, GEO, and AEO website-audit skill for [Hermes Agent](https://github.com/NousResearch/hermes).**
Runs technical SEO analysis (crawlability, indexability, Core Web Vitals honesty,
security headers, AI-crawler robots.txt tokens, Schema.org validation, image
optimization, sitemap, local SEO, a weighted 0–100 health score) **and** a
content/visibility audit (meta tags, E-E-A-T, GEO for AI Overviews / ChatGPT /
Perplexity / Gemini, AEO for featured snippets / voice) — then produces a polished,
downloadable Word report (`.docx`).

> **Two skills, one lineage.**
>
> * 🌐 **This repo** → `Hermes-SEO-GEO-AEO-SKILL`: the Hermes-native port, MIT, no
>   external dependencies beyond `python-docx`. This is the one to use with Hermes.
> * 🔧 **Upstream inspiration** → [`AgriciDaniel/claude-seo`](https://github.com/AgriciDaniel/claude-seo):
>   a Claude Code plugin (25 sub-skills + 18 sub-agents, Playwright, parallel
>   execution). We ported its *methodology* into a single Hermes skill and
>   replaced its Claude-only tooling with Hermes equivalents.

---

## Why this skill

- **AI-search first.** Aligned with [Google's AI Optimization Guide](https://developers.google.com/search/docs/fundamentals/ai-optimization-guide).
  Scores passage citability (134–167 word self-contained answer blocks), question-based
  heading hierarchy, attribution density, and entity presence (Wikipedia, Reddit,
  YouTube, LinkedIn).
- **One skill, no collision.** A single trigger covers both technical and content
  audits — no competing skills to disambiguate.
- **Honest about limits.** It never fabricates Core Web Vitals, backlinks, or GSC
  indexation. When a check needs an external API, it names the tool instead of guessing.
- **Agency-grade report.** The `.docx` generator uses a fixed navy design system,
  color-coded scores, and a prioritized action matrix.

---

## Who this is for

- **SEO freelancers / consultants** — anchor a scope with a real audit + 0–100 score.
- **In-house marketers** — second pair of eyes before executive reviews; catches schema
  gaps and AI-citability weaknesses that GSC hides.
- **Hermes users** who want a repeatable, scripted SEO audit they can run on any URL.

---

## Installation (Hermes)

Clone this repo and copy the skill into your active Hermes profile's `skills/` directory:

```bash
git clone https://github.com/fxinfo24/Hermes-SEO-GEO-AEO-SKILL.git
mkdir -p ~/.hermes/profiles/web-craft/skills/hermes-seo-geo-aeo
cp -r Hermes-SEO-GEO-AEO-SKILL/. ~/.hermes/profiles/web-craft/skills/hermes-seo-geo-aeo/
```

Then restart Hermes so it rebuilds its skill trigger index. The skill appears as
`hermes-seo-geo-aeo` and is enabled automatically.

> **Report dependency:** the DOCX generator needs `python-docx`. Install once if missing:
> ```bash
> python3 -c "import docx" 2>/dev/null || pip install python-docx
> ```

---

## Quick Start

In a Hermes chat, point it at a site:

```
audit https://pathosbay.com/
```

The skill will fetch the homepage + key pages, analyze technical and content signals,
and deliver a `.docx` audit report via a `MEDIA:` link.

---

## Commands

| Command | Description |
| --- | --- |
| `audit <url>` | Full audit — technical + content, 0–100 health score, DOCX report |
| `page <url>` | Deep single-page analysis |
| `technical <url>` | Technical SEO (crawlability, headers, schema, sitemap, AI-crawler tokens) |
| `content <url>` | E-E-A-T + content quality + GEO/AEO signals |
| `schema <url>` | Detect / validate / recommend Schema.org markup |
| `images <url>` | Image SEO audit (alt, size, format, CLS) |
| `local <url>` | Local SEO (GBP, NAP, citations, reviews, local schema) |
| `sitemap <url>` | Sitemap structure analysis |

---

## Methodology

Every audit works through:

1. **Fetch & crawl** — raw HTML via `curl`, rendered text via `web_extract`/
   `web_search`; robots.txt + sitemap enumeration.
2. **Industry detection** — SaaS / e-commerce / local / publisher / agency, which
   drives which checks apply.
3. **Analysis checklists** — 9 technical categories, Who/How/Why E-E-A-T test,
   Schema.org status rules (incl. FAQPage retired 2026-05-07 → use QAPage),
   image tiers, GEO/AI-crawler management, local SEO, sitemap.
4. **Health score (0–100)** — weighted across Technical / Content / Schema / Images /
   GEO / Local.
5. **Report** — JSON-driven `scripts/generate_report.py` renders a `.docx` with a
   color-coded scores table, per-category findings, and a priority recommendations matrix.

### Limitations (read before trusting output)

- **No external APIs.** Core Web Vitals field data, backlink metrics, GSC indexation,
  and GA4 traffic require PageSpeed/CrUX, Moz/Ahrefs/DataForSEO, or GSC — the skill
  names these tools rather than guessing.
- **HTML-only view.** JavaScript-rendered content is captured via `web_extract`
  fallback, but critical content should be indexable without JS.
- **Honesty guardrails.** It will not claim an AI citation boost or a specific CWV
  score it cannot measure.

---

## Repository layout

```
Hermes-SEO-GEO-AEO-SKILL/
├── SKILL.md                      # skill definition (frontmatter + methodology)
├── scripts/
│   └── generate_report.py        # JSON-driven DOCX report generator (python-docx)
├── README.md
├── LICENSE
└── .gitignore
```

---

## Author

**The Saint** — built with Hermes Agent.

Methodology ported from [`AgriciDaniel/claude-seo`](https://github.com/AgriciDaniel/claude-seo)
(used under MIT; we adapted the analysis framework and replaced Claude-only tooling
with Hermes equivalents).

---

## License

MIT — see [LICENSE](LICENSE).
