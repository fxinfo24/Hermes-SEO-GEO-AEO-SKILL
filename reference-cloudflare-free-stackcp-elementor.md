# Hosting-stack reference: Cloudflare Free + StackCP + WordPress/Elementor

Real-world fixes applied to a WordPress site on StackCP hosting, fronted by Cloudflare
**Free**. Documented so the next audit on this stack doesn't repeat the same dead ends.
Swap in your own domain where examples show one.

## 1. Leftover page-builder `.htaccess` residue
A removed caching plugin can leave its `# BEGIN … # END` block in
`public_html/<docroot>/.htaccess`, still setting headers (e.g. a stale
`X-Powered-By`). The plugin directory may be gone (site loads) but the fingerprint
lives on.
**Fix:** delete the entire block between the plugin's `# BEGIN` / `# END` markers.
Verify the stale header disappears from a `curl -I` response.

## 2. Security headers (origin `.htaccess`, NOT Cloudflare)
On Cloudflare **Free**, Transform Rules (Modify Response Header) are unavailable.
Correction to a common assumption: Cloudflare **passes origin response headers
through** to visitors, so adding them in origin `.htaccess` DOES reach the browser.
Applied via:

```
<IfModule mod_headers.c>
    Header always set Strict-Transport-Security "max-age=63072000; includeSubDomains; preload"
    Header always set Content-Security-Policy "default-src 'self'; img-src 'self' data: https:; style-src 'self' 'unsafe-inline'; script-src 'self' 'unsafe-inline'"
    Header always set X-Content-Type-Options "nosniff"
    Header always set Referrer-Policy "strict-origin-when-cross-origin"
    Header always set Permissions-Policy "geolocation=(), microphone=(), camera=()"
    Header unset X-Powered-By
</IfModule>
```
Add this outside any plugin-managed marker block (WordPress owns the
`# BEGIN WordPress … # END WordPress` section; don't edit inside it).

## 3. `X-Powered-By` PHP leak — PARTIALLY CLOSED (key gotcha)
- `Header unset X-Powered-By` in `.htaccess` does NOT reliably strip PHP-FPM's own emit.
- `expose_php = Off` via `.user.ini` was tested and **crashed the site with HTTP 500**:
  the same `.user.ini` also set `memory_limit = 4096M`, exceeding the hosting
  account cap (often 2 GB). PHP rejected the value, fell back to its compiled-in
  default (128 MB), and the page builder (Elementor) OOM'd.
- **Resolution:** strip `.user.ini` to a SINGLE line — `expose_php = Off` —
  removing the entire host-generated directive block. The site returned to HTTP 200.
- **Residual:** some StackCP PHP-FPM setups still ignore `expose_php` in `.user.ini`,
  so the `X-Powered-By` header can persist. Accepted as low-severity info disclosure;
  only host support can disable it at the FPM pool level. **Do NOT re-add
  `memory_limit`** — that is what caused the outage.

## 4. robots.txt changes need a CDN purge
After uploading new `robots.txt` (e.g. explicit AI-crawler allow rules for `GPTBot`,
`ChatGPT-User`, `ClaudeBot`, `PerplexityBot`, `Google-Extended`), the file may still
serve stale because BOTH Cloudflare's default static cache AND StackCP's StackCDN
cache it. **A Cloudflare cache purge (Caching → Purge Everything, or Custom Purge →
the robots.txt URL) was required** to make the new version live — even with zero
cache *rules* configured. StackCP's own cache clear may also be needed.

## Lessons
- On Free plans, do edge-style fixes at origin, not Cloudflare.
- Never set `memory_limit` above the hosting account cap in `.user.ini`.
- "X-Powered-By gone" during a 500 is a measurement artifact (error pages emit no header).
- robots.txt / static-file changes need a CDN purge to show up.
- Keep `.user.ini` minimal; host-generated directive dumps often contain values that
  exceed account limits.
