# PathosBay Fix Log (Cloudflare Free plan)

Real-world fixes applied to `pathosbay.com` (WordPress, StackCP hosting, Cloudflare Free).
Documented so the next audit doesn't repeat the same dead ends.

## 1. WP Rocket residue
The plugin was removed but its `.htaccess` block (`# BEGIN WP Rocket ... # END WP Rocket`)
was STILL present and actively setting `X-Powered-By: WP Rocket/3.23.1.1`.
**Fix:** delete the entire WP Rocket block from `public_html/pathosbay/.htaccess`.
Verified: that header disappeared.

## 2. Security headers (origin `.htaccess`, NOT Cloudflare)
On Cloudflare **Free**, Transform Rules (Modify Response Header) are unavailable.
Correction to earlier advice: Cloudflare **passes origin response headers through**, so
adding them in origin `.htaccess` DOES reach the browser. Applied via:

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

## 3. `X-Powered-By: PHP/8.5.9` leak — PARTIALLY CLOSED
- `Header unset X-Powered-By` in `.htaccess` does NOT reliably strip PHP-FPM's own emit.
- `expose_php = Off` in `.user.ini` was tested: it **crashed the site with HTTP 500**
  because the same `.user.ini` also set `memory_limit = 4096M`, which exceeds the
  hosting account cap of 2 GB. PHP rejected it, fell back to 128 MB, and Elementor OOM'd.
- **Resolution:** strip `.user.ini` to a SINGLE line: `expose_php = Off` (remove the
  StackCP directive block entirely). Site stable at 200.
- **Residual:** host PHP-FPM still ignores `expose_php` in `.user.ini`, so the
  `X-Powered-By: PHP/8.5.9` header persists. Accepted as low-severity info disclosure;
  only host support can disable it at the FPM pool level. Do NOT re-add `memory_limit`.

## 4. robots.txt AI-crawler rules
Added explicit allow blocks for `GPTBot`, `ChatGPT-User`, `ClaudeBot`, `PerplexityBot`,
`Google-Extended`. After upload, the file is cached by BOTH Cloudflare default static
cache AND StackCP's StackCDN. **A Cloudflare cache purge (Caching → Purge Everything)
was required** to make the new version live — even with zero cache *rules* configured.

## Lessons
- On Free plans, do edge-style fixes at origin, not Cloudflare.
- Never set `memory_limit` above the hosting account cap in `.user.ini`.
- "X-Powered-By gone" during a 500 is a measurement artifact (error pages have no header).
- robots.txt changes need a CDN purge to show up.
