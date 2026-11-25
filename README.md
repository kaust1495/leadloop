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
- **Deep links:** `…/#LL-0021` opens battery LL-0021. The QR stickers hold these
  links.

## What's in here

| Path | What it is |
|---|---|
| `index.html` | The whole app: one self-contained page (HTML, CSS, JS, inline demo data). Edit this. |
| `vercel.json` | Static hosting settings: no build step, `no-cache`, and the security headers. |
| `AUDIT.md` | The independent audit, red-team findings and the phases that fixed them. |

There is no build step, no package manager and no test runner in this repo. The
demo data lives in the `<script type="application/json" id="seedData">` block
inside `index.html`, so the page needs no network request to start.

## Where the data lives

The same `index.html` runs in two places:

- **As the Claude artifact** it uses the artifact's shared database, so every
  viewer sees the same records.
- **On Vercel (or any static host)** it keeps a copy of the built-in seed in
  each visitor's browser (`localStorage`, key `ll_store_v1`). Each device starts
  from the same demo data, and what one juror records stays on their device.
  *Control room → Settings → Reset demo data* puts a device back to the start.

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

Any static server works:
```bash
python -m http.server 8000
# then open http://localhost:8000/
```
The built-in seed means the page also renders from `file://`, but camera,
geolocation and clipboard access need a real origin, so use the server for a
demo.

## The QR sticker sheet

The control room's **QR stickers** tab renders all 32 codes in the browser, each
holding the site's address plus the battery ID. Print that page (or show it on a
second screen) and point a phone camera at a sticker: the phone opens the app at
that battery, the way a scrap vendor's scan would in the field. Fresh stickers
start a pickup; used ones open their custody trail. The sheet always matches the
address you are viewing it from.

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

`vercel.json` sends a Content-Security-Policy that allows scripts from this
origin and `cdn.jsdelivr.net` only (the two pinned QR libraries, both with SRI
hashes), plus `img-src data: blob:` for photos and `font-src` for
`fonts.gstatic.com`. The app script itself is inline, so `script-src` still
needs `'unsafe-inline'`; moving the script to its own file would let the policy
drop it. See `AUDIT.md` for the rest of the findings and what was left alone.
