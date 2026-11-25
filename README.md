# LeadLoop Checkpoint (demo MVP)

A working demo of LeadLoop's QR checkpoint for dead e-rickshaw batteries: one
sticker per battery, three scans (scrap vendor pickup, LeadLoop yard weigh-in,
recycler gate), the money at each step, and LeadLoop's control room. Dummy data
only; not production software.

- **The 45-second story:** the landing button plays eight beats with a voice in
  English or Hindi, or open `…/#tour` directly. It saves nothing.
- **The tour:** *See the real screens* (in the Demo menu, or `…/#screens`) plays
  through the app's own screens on its own, eight stops, about 45 seconds.
- **Try it step by step:** a real pickup → weigh-in → dispatch → gate check →
  settlement on a fresh sticker, switching role with one tap. This one saves
  records.
- **What stops cheating:** Demo menu → *What stops cheating*, or control room →
  Demo → *What stops cheating*. Twelve attacks, the counter for each, and what
  the design does not stop.
- **Deep links:** `…/#LL-0021` opens battery LL-0021. The QR stickers hold these
  links.
- **Offline:** the page needs no network at all. The QR libraries and the fonts
  are inside `index.html`, so it runs from a laptop with no internet and no
  Wi-Fi. See *The QR sticker sheet* for printing stickers that point at a LAN
  address.

## What's in here

| Path | What it is |
|---|---|
| `index.html` | The whole app: one self-contained page (HTML, CSS, JS, embedded fonts, QR libraries and inline demo data). Edit this. |
| `vercel.json` | Static hosting settings: no build step, `no-cache`, and the security headers. |
| `AUDIT.md` | The independent audit, red-team findings and the phases that fixed them. |

There is no build step, no package manager and no test runner in this repo. The
demo data lives in the `<script type="application/json" id="seedData">` block
inside `index.html`, so the page needs no network request to start.

The two QR libraries (jsQR 1.4.0 and qrcode-generator 1.4.4, both MIT) and the
Bricolage Grotesque / Mukta font subsets are embedded in the page. Nothing is
fetched from a CDN, so the demo works on a laptop in a room with no internet.

## Where the data lives

The same `index.html` runs in two places:

- **As the Claude artifact** it uses the artifact's shared database, so every
  viewer sees the same records.
- **On Vercel (or any static host, or a file on disk)** it keeps a copy of the
  built-in seed in each browser (`localStorage`, key `ll_store_v1`). Each device
  starts from the same demo data, and what one juror records stays on that
  device. *Control room → Settings → Reset demo data* puts a device back to the
  start.

Photos are compressed before they are stored. A device that runs out of browser
storage now says so instead of dropping records silently.

## Put it on GitHub and Vercel

1. Create an empty repository on github.com, without a README.
2. In this folder:
   ```bash
   git init
   git add .
   git commit -m "LeadLoop Checkpoint demo"
   git branch -M main
   git remote add origin https://github.com/<your-username>/leadloop-checkpoint.git
   git push -u origin main
   ```
3. On vercel.com: **Add New → Project → Import** the repository. Framework
   preset: **Other**. No build command, no output directory. Deploy.
4. Every later `git push` to `main` redeploys automatically; the address stays
   the same.

## Run it locally

Double-click `index.html` for a pitch with no network: it renders and the whole
demo works from `file://`. Camera, geolocation and clipboard access need a real
origin, so use a server when you want to scan with a phone:

```bash
python -m http.server 8000
# then open http://localhost:8000/
```

On the same Wi-Fi, a phone can open `http://<this computer's LAN address>:8000/`
(the address `ipconfig` shows, e.g. `192.168.1.24`). Put that address on the
sticker sheet so printed codes open there.

## The QR sticker sheet

The control room's **QR stickers** tab renders all 32 codes in the browser, each
holding a web address plus the battery ID. Fresh stickers start a pickup; used
ones open their custody trail.

**Where the codes point.** The sheet has one field for the address phones should
open. It defaults to the address the page was opened from; for an offline pitch,
type the laptop's LAN address (`http://192.168.1.24:8000/`) so stickers work on
phones with no internet. The choice is remembered in `localStorage`
(`ll_qr_base`). A warning appears when the address is blank, a `file://` path or
`localhost`, because phones cannot open those.

**Printing.** *Print sticker sheet* prints A4 pages, three stickers across with
cut lines, a header naming the address the codes open, and black-on-white QR
codes. Stick one on each battery before the pickup.

## Rules the demo follows (from the Grand Finale deck)

- Driver paid the local rate (₹60–62/kg) by UPI, target 15 minutes or less.
- Partner scrap vendor paid ₹67/kg + ₹2/kg match = ₹69/kg of yard weight, the
  same day, on the recycler's behalf (₹2,070 for 30 kg).
- Recycler repays those payments plus ₹4/kg (₹120 for 30 kg), about 10 days
  after its gate check.
- Weight must match within ±5% at the yard (vs pickup) and at the gate (vs
  yard); a gap over 5% comes off the partner's next payment.
- ₹10 per tagged battery (delivery-and-record allowance, or the dealer's tagging
  fee in phase 2). Dealers keep take-backs in a crate until a partner scrap
  vendor collects them.
- Storage limit 90 days (CPCB, Jul 2026); LeadLoop target ~10 days. Partners
  deliver at least weekly.
- Only registered recyclers generate EPR certificates; LeadLoop supplies the
  evidence trail.

## Security notes

`vercel.json` sends a Content-Security-Policy with no external origins at all:
`default-src 'self'`, `img-src 'self' data: blob:` for photos and
`font-src 'self' data:` for the embedded fonts. The app script and the two QR
libraries are inline, so `script-src` still needs `'unsafe-inline'`; moving them
to their own files would let the policy drop it. See `AUDIT.md` for the rest of
the findings and what was left alone.
