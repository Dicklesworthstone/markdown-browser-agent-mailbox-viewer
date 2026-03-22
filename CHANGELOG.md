# Changelog

All notable changes to the **markdown-browser-agent-mailbox-viewer** repository are documented here.

This project is a static export of an [MCP Agent Mail](https://github.com/Dicklesworthstone/mcp_agent_mail) mailbox. It bundles a scrubbed SQLite snapshot with a self-contained browser-based viewer so that teammates can browse threads, attachments, and metadata without direct database access.

> **Note:** This repository has no tags or formal GitHub releases. History is tracked entirely through commits on `main`.

---

## [ae5cea1] — 2026-02-21

**License: MIT with OpenAI/Anthropic Rider**

Added a `LICENSE` file containing the MIT license with an additional rider that restricts use by OpenAI, Anthropic, and their affiliates without express written permission from Jeffrey Emanuel.

### Licensing

- Added 73-line `LICENSE` with the custom "MIT + OpenAI/Anthropic Rider" terms.
- Rider defines "Restricted Parties" (OpenAI, Anthropic, affiliates) and prohibits any use, distribution, or derivative work by those parties without prior written consent.
- Breach auto-terminates all permissions; prevailing-party attorney-fee provision included.

[ae5cea1]: https://github.com/Dicklesworthstone/markdown-browser-agent-mailbox-viewer/commit/ae5cea13c9217aaf5f87461b3647d2b60cc9dd5a

---

## [26af3a5] — 2026-02-21

**Social preview image**

Added a GitHub Open Graph share image for consistent social media link previews when sharing the repository URL.

### Repository metadata

- Added `gh_og_share_image.png` (1280x640, ~35 KB) for repository social card.

[26af3a5]: https://github.com/Dicklesworthstone/markdown-browser-agent-mailbox-viewer/commit/26af3a520ff7cbaba2ffc2930cbb1c4a0f9ae1cd

---

## [22faf83] — 2025-11-08

**Initial mailbox export — full viewer bundle**

The foundational commit that created the entire repository: a scrubbed SQLite mailbox snapshot (1,060 messages from the `markdown_web_browser` project, 38 agents) plus a complete static web viewer.

### Viewer application (`viewer/`)

- **Single-page app** built on Alpine.js and Tailwind CSS with a split-pane, list, or thread view mode.
- **In-browser SQLite engine** via sql.js (v1.10.1) — loads `mailbox.sqlite3` directly in the browser and runs live SQL queries for filtering, search, and thread assembly.
- **Advanced search** with boolean operators (AND, OR, NOT), quoted phrases, parentheses, and proper operator precedence via a shunting-yard tokenizer/parser. Falls back from FTS5 to LIKE automatically when FTS is unavailable.
- **Thread explorer** — groups messages by `thread_id`, shows per-thread message counts, latest subject/snippet, and chronological message ordering within threads.
- **Multi-axis filtering** — filter by project, sender, recipient, importance level (urgent/high/normal/low), thread membership, and message kind (user vs. administrative).
- **Administrative message detection** — auto-classifies messages matching patterns like "contact request from" or "auto-handshake" and allows toggling visibility (user-only, admin-only, or all).
- **Markdown rendering** via Marked.js (v11.0.0) with GFM support, sanitized through DOMPurify (v3.0.8).
- **Trusted Types integration** — creates `mailViewerDOMPurify` and `default` policies for CSP-compliant innerHTML usage; vendor script URLs restricted to `./vendor/` paths.
- **Virtual scrolling** — custom windowed renderer (spacer-based approach) with adaptive row-height estimation, overscan, and `requestAnimationFrame` scheduling for smooth performance on large mailboxes.
- **OPFS caching** — opportunistically caches the SQLite database to the Origin Private File System after first load, with SHA256-keyed cache invalidation and version metadata; falls back gracefully when OPFS is unavailable.
- **Dark mode** — system-preference-aware with localStorage persistence and instant toggle; flash-of-wrong-theme prevented by an inline `<script>` in `<head>`.
- **Responsive/mobile layout** — breakpoint detection via `matchMedia`, auto-collapsing filters on scroll, mobile message modal with body-scroll lock.
- **Sorting** — by newest, oldest, subject, sender, or body length.
- **Bulk actions** — select all / individual message selection with local read-state tracking.
- **Auto-refresh UI** — simulated 30-second refresh cycle for static viewer; designed for parity with the live MCP Agent Mail web UI.
- **Diagnostics panel** — shows database source (network/OPFS), cache state, FTS status, and total message count.
- **EXPLAIN mode** — `state.explainMode` flag logs `EXPLAIN QUERY PLAN` output for all SQL queries to the browser console for performance debugging.
- **Content Security Policy** — strict CSP meta tag allowing only `self`, specific CDNs (Tailwind, Google Fonts, Lucide icons), and `unsafe-inline`/`unsafe-eval` for Alpine.js/Tailwind runtime.
- **Asset preloading** — `<link rel="preload">` for sql-wasm.js, sql-wasm.wasm, and mailbox.sqlite3 to minimize time-to-interactive.
- **Inline SVG favicon** — envelope icon embedded as a data URI to prevent 404s in preview.

### Vendor libraries (`viewer/vendor/`)

| Library | Version | Purpose |
|---|---|---|
| sql.js | 1.10.1 | In-browser SQLite via WebAssembly |
| Marked.js | 11.0.0 | GitHub-Flavored Markdown parser |
| DOMPurify | 3.0.8 | HTML sanitization |
| Clusterize.js | 0.18.0 | Virtual list scaffolding (legacy, replaced by custom renderer) |

All vendor files tracked in `viewer/vendor_manifest.json` with SHA256 checksums for integrity verification.

### Data layer

- `mailbox.sqlite3` — 3.7 MB scrubbed database. Standard scrub preset: 1,060 ack flags cleared, 263 agent links removed, 336 file reservations removed, 2,933 recipients cleared. Agent names retained (38 agents, 0 pseudonymized).
- `viewer/data/messages.json` — pre-rendered JSON cache of 500 messages for offline/fallback use.
- `viewer/data/meta.json` — metadata (message count, FTS status, generation timestamp).

### Manifest and integrity

- `manifest.json` (schema v0.1.0) — machine-readable metadata covering database path/SHA256/size, export configuration, scrub statistics, project scope, hosting detection, and SRI hashes for all viewer assets.
- `manifest.sig.json` — Ed25519 signature over the manifest SHA256 digest with embedded public key for bundle verification via `mcp_agent_mail.cli share verify`.

### Deployment infrastructure

- `index.html` — root redirect page that sends visitors to `viewer/index.html` via `<meta http-equiv="refresh">` and JavaScript `location.replace`.
- `_headers` — Netlify/Cloudflare Pages header rules for Cross-Origin-Isolation (COOP: same-origin, COEP: require-corp), CORP for viewer assets, and correct MIME types for `.sqlite3` and chunk files.
- `coi-serviceworker.js` (v0.1.7) — service worker that retrofits COOP/COEP headers on hosts like GitHub Pages that lack custom header support; enables SharedArrayBuffer and OPFS.
- `preview-reload.js` — development-time hot-reload poller (2-second interval) that watches a `/__preview__/status` endpoint for signature changes and triggers page reload.
- `.nojekyll` — disables GitHub Pages Jekyll processing so `_headers` and `.wasm` files are served untouched.
- `HOW_TO_DEPLOY.md` — deployment guides for GitHub Pages, Cloudflare Pages, Netlify, Amazon S3, and generic static hosts, including cross-origin isolation verification steps.

### Python packaging

- `viewer/__init__.py` — minimal package marker exposing `__path__` so the `mcp_agent_mail` CLI can locate viewer static assets at runtime.

[22faf83]: https://github.com/Dicklesworthstone/markdown-browser-agent-mailbox-viewer/commit/22faf831fe065e03fdc1ef99c7e92ca8d75a3885
