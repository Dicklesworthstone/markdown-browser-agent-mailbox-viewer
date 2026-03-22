# Changelog

All notable changes to the **MCP Agent Mail -- Shared Mailbox Snapshot** viewer are documented here. This project exports scrubbed MCP Agent Mail databases as self-contained static bundles with a browser-based viewer.

Entries are organized by capability rather than diff order. Commit links point to the canonical GitHub repository.

> **Note:** This repository has no tags or formal GitHub releases. History is tracked entirely through commits on `main`.

---

## 2026-02-21 -- Licensing and Social Preview

Two housekeeping commits added a social preview image and replaced the license.

### Social Preview

Added a 1280x640 Open Graph share image (`gh_og_share_image.png`, ~35 KB) so repository links render with a branded card on social platforms and chat previews.

- [26af3a5 -- chore: add GitHub social preview image (1280x640)](https://github.com/Dicklesworthstone/markdown-browser-agent-mailbox-viewer/commit/26af3a520ff7cbaba2ffc2930cbb1c4a0f9ae1cd)

### License

Replaced the implicit license with **MIT + OpenAI/Anthropic Rider** (`LICENSE`, 73 lines). The rider defines "Restricted Parties" (OpenAI, Anthropic, and their affiliates) and prohibits any use, distribution, or derivative work by those parties without express prior written permission from Jeffrey Emanuel. Breach auto-terminates all permissions; prevailing-party attorney-fee provision included.

- [ae5cea1 -- chore: update license to MIT with OpenAI/Anthropic Rider](https://github.com/Dicklesworthstone/markdown-browser-agent-mailbox-viewer/commit/ae5cea13c9217aaf5f87461b3647d2b60cc9dd5a)

---

## 2025-11-08 -- Initial Export (Project Genesis)

The founding commit established the complete static mailbox bundle and self-contained browser viewer. Everything described below shipped in a single commit.

- [22faf83 -- Initial mailbox export](https://github.com/Dicklesworthstone/markdown-browser-agent-mailbox-viewer/commit/22faf831fe065e03fdc1ef99c7e92ca8d75a3885)

### Mailbox Database

- Scrubbed SQLite snapshot (`mailbox.sqlite3`, ~3.7 MB) containing 1,060 messages across 38 agents from the `markdown_web_browser` project.
- Standard scrub preset: 1,060 ack flags cleared, 263 agent links removed, 336 file reservations removed, 2,933 recipients cleared. Agent names retained (0 pseudonymized) for readability.
- Pre-rendered JSON cache: `viewer/data/messages.json` (500 messages) and `viewer/data/meta.json` for offline/fallback use.

### Browser-Based Viewer (Alpine.js + Tailwind CSS)

#### Core Architecture

- Single-page application in `viewer/` powered by **Alpine.js** for reactivity and **Tailwind CSS** (CDN) for styling.
- In-browser **sql.js / sql-wasm** (v1.10.1) engine loads the SQLite database directly -- no server required.
- Root `index.html` redirect page forwards visitors to the viewer via `<meta http-equiv="refresh">` and JavaScript `location.replace`.
- **Lucide** icon set for all UI elements.
- Inline SVG favicon (envelope icon as data URI) to prevent 404s in preview.

#### Message Browsing

- **Inbox view** with split-pane layout: scrollable message list on the left, selected message body on the right.
- **Thread explorer**: groups messages by `thread_id`, shows per-thread message counts, latest subject, importance, and snippet; chronological ordering within threads.
- **Thread search**: filters the thread sidebar by subject, identifier, or snippet text.
- **Administrative message classification**: auto-detects contact requests and auto-handshake messages via configurable subject/body regex patterns; toggles between user, admin, and all message categories.
- Click-to-select with visual highlight, automatic first-message selection on load.
- View modes: split (list + detail), list-only, and threads.

#### Search Engine

- **Full-text search** with boolean operators (`AND`, `OR`, `NOT`), parenthesized grouping, and quoted-phrase support.
- **Shunting-yard parser** tokenizes queries into an AST with correct operator precedence (NOT > AND > OR).
- **FTS5 path**: when the database includes an `fts_messages` virtual table, queries compile to `MATCH` expressions for fast ranked retrieval.
- **LIKE fallback**: when FTS is unavailable, the same AST generates nested `LIKE` clauses against subjects and bodies.
- **EXPLAIN QUERY PLAN** diagnostics: optional `explainMode` flag logs query plans to the browser console for performance debugging.
- Debounced search input (140 ms) to avoid hammering sql.js on every keystroke.

#### Filtering and Sorting

- Multi-axis filter panel: project, sender, recipient, importance level (urgent/high/normal/low with counts), threaded vs. standalone, and message kind (user/admin/all).
- Sort by newest, oldest, subject, sender, or body length.
- One-click "Clear Filters" resets all axes, search, and selections.
- Dynamic filter population: unique values extracted from the live dataset at load time.

#### Performance and Caching

- **OPFS (Origin Private File System) caching**: after the first network fetch, the database is persisted to browser-local OPFS storage via `requestIdleCallback`. Subsequent visits load from cache with SHA256-keyed invalidation and version metadata tracking. Stale caches are detected and purged automatically.
- **Virtual scrolling**: custom windowed renderer with spacer-based approach, adaptive row-height estimation (exponential moving average after font load), configurable overscan, and `requestAnimationFrame`-throttled rendering. Handles thousands of messages without DOM bloat.
- Asset preloading: `<link rel="preload">` hints for sql-wasm.js, sql-wasm.wasm, and the database file to reduce time-to-interactive.
- Single-query recipients map construction (replaces N+1 per-message lookups).

#### Security

- **Trusted Types** integration: `mailViewerDOMPurify` policy channels all dynamic HTML through DOMPurify; a `default` policy provides Clusterize.js innerHTML compatibility since all user content is pre-escaped.
- **DOMPurify** (v3.0.8) sanitizes all rendered Markdown.
- **Marked.js** (v11.0.0) with GFM mode renders message bodies; falls back to escaped plain text if unavailable.
- **Content Security Policy** meta tag restricting script sources to `self` and specific CDNs (Tailwind, Google Fonts, Lucide, jsDelivr, unpkg), disabling `object-src`.
- All user-controlled text passes through `escapeHtml()` before DOM insertion; `highlightText()` escapes before applying `<mark>` tags.
- Vendor scripts loaded with **Subresource Integrity** hashes recorded in both `viewer/index.html` and `manifest.json`.

#### Responsive Design and Dark Mode

- **Dark mode**: persisted to `localStorage`, respects `prefers-color-scheme` on first visit, togglable from the navigation bar. Flash-of-wrong-theme prevented by an inline `<script>` in `<head>`. Comprehensive dark-mode prose styling for Markdown content (headings, code blocks, blockquotes, tables, links, lists, strong/em, figcaptions).
- **Mobile-responsive layout**: `matchMedia` breakpoint at 768px triggers compact mode with collapsible filter panel, full-screen message modal with body-scroll lock (`mobile-modal-open` class), and scroll-direction-aware filter auto-hide.

#### Diagnostics Panel

- Collapsible diagnostics panel in the navigation bar showing database source (network vs. OPFS cache), total message count, FTS5 availability, cache state, schema version, exporter version, and scrub statistics.
- Real-time cache toggle button (green "Cached" badge when OPFS is active).

#### Bulk Actions and Refresh

- Checkbox selection per message row (appears on hover via group opacity), select-all toggle, and "mark as read" (local-only state in the static viewer).
- Simulated refresh with auto-refresh toggle (30-second interval) and timestamp label, designed for parity with the live MCP Agent Mail web UI.

#### Resource Cleanup

- `destroy()` lifecycle method tears down all event listeners (scroll, resize, matchMedia), clears auto-refresh intervals, removes body-scroll-lock classes, and nullifies virtual list state to prevent memory leaks.

### Vendor Libraries

| Library | Version | Purpose |
|---|---|---|
| sql.js | 1.10.1 | In-browser SQLite via WebAssembly |
| Marked.js | 11.0.0 | GitHub-Flavored Markdown parser |
| DOMPurify | 3.0.8 | HTML sanitization |
| Clusterize.js | 0.18.0 | Virtual list scaffolding (legacy; rendering replaced by custom windowed renderer) |

All vendor files tracked in `viewer/vendor_manifest.json` with SHA256 checksums for integrity verification.

### Deployment and Hosting

- `HOW_TO_DEPLOY.md` with platform-specific checklists for GitHub Pages, Cloudflare Pages, Netlify, Amazon S3/CloudFront, and generic static hosts, including cross-origin isolation verification steps.
- `_headers` file providing COOP/COEP headers for Cloudflare Pages and Netlify, plus correct MIME types for `.sqlite3` and chunk files.
- **coi-serviceworker.js** (v0.1.7): service worker that injects Cross-Origin-Isolation headers on hosts that lack custom header support (e.g., GitHub Pages), enabling SharedArrayBuffer and OPFS.
- `.nojekyll` marker disabling GitHub Pages' Jekyll processing so `_headers` and `.wasm` files are served correctly.
- **Preview server integration**: `preview-reload.js` polls a `/__preview__/status` endpoint every 2 seconds and auto-reloads the page when the export signature changes (used with `mcp_agent_mail.cli share preview`).

### Manifest and Integrity

- `manifest.json` (schema v0.1.0) recording export metadata: database path/SHA256/size, chunk configuration, scrub statistics, project scope, hosting detection signals, and per-asset SRI hashes under `viewer.sri`.
- `manifest.sig.json` with Ed25519 signature over the manifest SHA256 digest plus embedded public key for offline bundle verification via `mcp_agent_mail.cli share verify`.
- `viewer/vendor_manifest.json` pinning exact versions and SHA256 hashes for all vendored libraries.
- Chunked database support: the viewer can reassemble databases split across multiple chunk files (pattern-based naming with zero-padded indices), though this export ships as a single file.

### Python Packaging

- `viewer/__init__.py` -- minimal package marker exposing `__path__` so the `mcp_agent_mail` CLI can locate viewer static assets at runtime.
