# LeadLoop Checkpoint — independent audit and fix plan

Audit of `index.html`, `vercel.json`, `README.md` and the deployed site
(`https://robinhood-leadloop.vercel.app/`), at commit `27d3009`.
Severity: **P0** breaks the page or loses data, **P1** security or access
barrier, **P2** polish, performance or repo hygiene.

## Scope of the audit

Every rendered screen was read and exercised:

| Screen | Route / entry | Notes |
| --- | --- | --- |
| Landing | `#` (no session) | hero, role grid, header |
| How it works | `#how` | ten steps, three scans |
| Role home ×4 | vendor, dealer, yard, recycler | tiles, lists, scan buttons |
| Pickup form | scan 1 | chips, weight dial, photos, payment |
| Done | after save | ring, result rows, payout slip |
| Yard weigh-in | scan 2 | diff banner, deduction netting |
| Gate check | scan 3 | timeline, weight input |
| Timeline | any battery | custody trail, flags |
| My payouts | vendor/dealer | ledger |
| Dispatch | yard | selection, manifest |
| Truckload | `#` ref | progress, rows |
| Settlement | `#` ref | statement, repayment |
| Control room | admin | alerts, batteries, cash, partners, report, audit, settings |
| QR stickers | admin | 32 codes |
| Scan walkthrough | `#scan` | five steps, live QR |
| Story | `#story`, `#tour` | eight beats, EN/HI |
| Tour | `#screens` | eight stops |

## Findings

### P0 — crashes, silent data loss

1. **A malformed hash blanks the whole app.** `readHash()` calls
   `decodeURIComponent(location.hash)` with no guard, and it runs at top
   level before `render()`. A link ending in `#%` or `#%E0%A4` throws
   `URIError`, aborts the script, and leaves the page on the loading
   spinner forever. Any shareable link can be poisoned this way.
2. **A QR containing `%` breaks the scan.** `wireScan()` calls
   `decodeURIComponent(txt)` unguarded; the throw leaves the button stuck
   on "Reading the QR…" with no way back.
3. **Missing-record guards.** `saveYard()` and `saveGate()` read
   `cur.status` without checking `cur` exists — a reset or a second tab
   while a form is open throws instead of recovering.
4. **Corrupt `ll_demo` can blank the page.** The stored demo state is
   merged with `Object.assign` after only a `typeof step === "number"`
   check; a hand-edited or stale value such as `step: 99` makes
   `DEMO[i]` undefined and `renderDemo()` throws on every render.
5. **Photo storage fails silently.** `persist()` swallows the quota
   error. Two compressed photos per battery reach the ~5 MB
   `localStorage` ceiling quickly, after which saves stop being kept with
   no signal to the user.
6. **`pickFile()` never settles on cancel.** Only `change` is listened
   for, so cancelling the file picker leaves the promise pending.
7. **Unknown control-room tab renders nothing.** `viewAdmin()` builds an
   empty body for an unrecognised `S.crTab`.
8. **`viewDone` without state.** Direct render with no `S.done` prints
   `undefined` as a sticker id.

### P1 — security

9. **CSP has no fetch directives.** `frame-ancestors`, `object-src`,
   `base-uri` and `form-action` are set, but with no `script-src`,
   `style-src`, `img-src`, `font-src` or `connect-src` an injected
   `<script src>` from any origin still runs. The inline app script needs
   `'unsafe-inline'`, but the origin allow-list is free to add.
10. **Nine inline `onclick="closeModal()"` handlers.** A DOM-XSS sink and
    the blocker for ever tightening `script-src` to a hash.
11. **CSV formula injection.** `csvCell()` quotes commas and quotes but
    passes a leading `=`, `+`, `-` or `@` straight through, so an audit
    note or partner field can execute in Excel/Sheets on open.
12. **No `Cross-Origin-Resource-Policy`.** Static assets are readable
    cross-origin; `Access-Control-Allow-Origin: *` is Vercel's default for
    static files (no credentials, no API behind it, so not exploitable —
    recorded rather than "fixed").
13. **`X-XSS-Protection` unset.** The legacy auditor is best explicitly
    disabled.

### P1 — accessibility

14. **89 table headers without `scope`.** Screen readers cannot tie
    cells to columns.
15. **Five clickable table rows are mouse-only.** `tr.click` has a click
    handler but no `tabindex`, role or key handler.
16. **Modal has no focus management.** Focus stays behind the overlay,
    Escape does nothing, and focus is not restored on close.
17. **Language never switches.** `<html lang="en">` is fixed, so the Hindi
    form and story are announced with an English voice.
18. **Unlabelled control groups.** Chip groups (source, brand, died,
    leaking, sale, condition) sit under a `<label>` with no `for`, and the
    simulate-scan input has no label at all.
19. **Wide tables are not keyboard-scrollable.** `.tablewrap` scrolls
    sideways but cannot receive focus.
20. **No skip link** past the header.

### P2 — paint, performance, repo

21. **The landing is JS-only.** `<main id="app">` ships a spinner; the
    `h1` and copy exist only after the script runs, so crawlers and the
    first paint depend on JS. The largest element is therefore late.
22. **Seven full re-renders on load.** Each collection snapshot calls
    `softRender()` → `render()`, uncoalesced.
23. **Fonts are not preloaded.** `preconnect` is set but the two Latin
    `woff2` files are only discovered after CSS parse.
24. **README describes files that do not exist** (`data/seed.json`,
    `tools/seed.py`).
25. **`.cloud-attachments/` is not ignored**, so local screenshots can be
    committed by accident.

## Fix phases

Each phase is committed and verified on its own, then the whole set is
re-verified across every screen.

- **Phase 1 — P0:** hash and QR decode guards, record guards, demo-state
  validation, quota signalling, cancel handling, tab fallback, done guard.
- **Phase 2 — security:** CSP fetch directives, inline handlers removed,
  CSV neutralisation, CORP and `X-XSS-Protection`.
- **Phase 3 — accessibility:** `th scope`, keyboard rows, modal focus,
  `lang` sync, labelled groups, focusable table regions, skip link.
- **Phase 4 — paint and performance:** static hero, coalesced renders,
  font preload.
- **Phase 5 — repo hygiene:** README accuracy, ignore rules, copy checks.

## Status

All five phases landed and are live: `e1c46d6` (P0), `6426df7` (security),
`ae0732d` (accessibility), `7d10a71` (paint and performance), `57ddaea` (repo).
Each phase was checked on its own, then all 22 view states were swept at 375 px
and 1440 px — no console messages, no empty screens, no horizontal overflow —
and the same sweep was repeated on the deployed origin under the live CSP.

## Deliberately not changed

- **`Access-Control-Allow-Origin: *`** — Vercel's static default; the site
  serves no credentials and has no API, so wildcard CORS exposes nothing
  beyond the public page. Recorded, not overridden.
- **`'unsafe-inline'` in `script-src`** — the app is one inline script by
  design (single-file requirement). A hash-based policy is possible once
  inline handlers are gone, but it breaks on every hand edit, so the
  origin allow-list is the honest middle ground.
- **Rate limiting / DMARC** — no server and no mail domain in this repo.

## Round 2 — offline, print, and the cheat screen

Three follow-ups from the ranked list landed after the audit, all inside
`index.html`:

- **Offline.** jsQR 1.4.0 and qrcode-generator 1.4.4 are embedded in the page,
  and the Bricolage Grotesque / Mukta subsets are `data:` URIs in the font
  block. The page now makes no network request at all: verified as zero
  resource entries, zero console messages, and `document.fonts` showing the
  embedded Latin and Devanagari faces loaded. Supersedes finding 23 (nothing to
  preload) and drops the CDN origin from the CSP.
- **Print.** The sticker sheet has an address field (defaults to the page's own
  origin, remembered in `ll_qr_base`, warns on `file://` and `localhost`) and
  prints A4: `@page` with 12 mm / 10 mm margins, three 63 mm columns, 34 mm
  codes, 12 stickers per page, three pages for 32.
- **What stops cheating.** A new screen (control room → Demo, the Demo menu, or
  How it works) maps twelve attacks to the counter for each, then lists what the
  design does not stop.

The CSP is now origin-free apart from `'self'` and `data:`/`blob:` for images
and fonts; `'unsafe-inline'` stays for the inline scripts, as above.
