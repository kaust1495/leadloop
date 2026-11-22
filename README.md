# LeadLoop Checkpoint (demo MVP)

A working demo of LeadLoop's QR checkpoint for dead e-rickshaw batteries: one sticker per battery, three scans (scrap vendor pickup, LeadLoop yard weigh-in, recycler gate), the money at each step, and LeadLoop's control room. Dummy data only; not production software.

- **Play the tour:** open the site and press *Play the 2-minute tour*, or go straight to `…/#tour`. It walks through every screen on its own and saves nothing.
- **Try it step by step:** *Try it step by step* takes you through a real pickup → weigh-in → dispatch → gate check → settlement on a fresh sticker, switching role with one tap.
- **Deep links:** `…/#LL-0021` opens battery LL-0021. The QR stickers hold these links.

## What's in here

| Path | What it is |
|---|---|
| `index.html` | The whole app: one self-contained page (HTML, CSS, JS). Edit this. |
| `data/seed.json` | Starting demo data: 32 stickers, payouts, truckloads, settlements, partners, audit log. |
| `tools/seed.py` | Regenerates `data/seed.json`. |
| `tools/make_qr_sheet.py` | Printable QR sheet pointing at your site's address. |
| `tests/` | Browser tests (Playwright) that run every flow against a mock database. |
| `vercel.json` | Static hosting settings (no build step). |

## Where the data lives

The same `index.html` runs in two places:

- **As the Claude artifact** it uses the artifact's shared database, so every viewer sees the same records.
- **On Vercel (or any static host)** it keeps a copy of `data/seed.json` in each visitor's browser. Each device starts from the same demo data, and what one juror records stays on their device. *Control room → Settings → Reset demo data* puts a device back to the start.

## Put it on GitHub and Vercel

1. Create an empty repository on github.com (e.g. `leadloop-checkpoint`), without a README.
2. In this folder:
   ```bash
   git init
   git add .
   git commit -m "LeadLoop Checkpoint demo"
   git branch -M main
   git remote add origin https://github.com/<your-username>/leadloop-checkpoint.git
   git push -u origin main
   ```
3. On vercel.com: **Add New → Project → Import** the repository. Framework preset: **Other**. No build command, no output directory. Deploy.
4. Every later `git push` to `main` redeploys automatically; the address stays the same.
5. Print stickers that open your Vercel address:
   ```bash
   pip install qrcode reportlab pillow
   python tools/make_qr_sheet.py https://<your-project>.vercel.app/
   ```

## The QR sticker sheet

`tools/make_qr_sheet.py` makes a printable A4 PDF of the 32 demo stickers. It is for the live demo, not part of the website: print it (or show it on a second screen), and a juror points a phone camera at a sticker. The phone opens the app at that battery, the way a scrap vendor's scan would in the field. Fresh stickers (dashed border) start a pickup. Used ones open their custody trail. The PDF is not committed (`.gitignore`); make it again whenever the site address changes.

## Run it locally

Open it through a web server, not by double-clicking the file (the browser blocks loading `data/seed.json` from disk):
```bash
python -m http.server 8000
# then open http://localhost:8000/
```

## Tests

```bash
cd tests
npm install                 # local copies of jsQR and qrcode-generator
pip install playwright && playwright install chromium
python run_flows.py         # pickup → yard → payouts → dispatch → gate → settlement → control room, mobile and desktop
python run_tour.py          # the hands-free tour: every stop opens, nothing is saved
python run_hosted.py        # hosted mode (start `python -m http.server 8765` in the repo root first)
```

## Rules the demo follows (from the Grand Finale deck)

- Driver paid the local rate (₹60–62/kg) by UPI, target 15 minutes or less.
- Partner scrap vendor paid ₹67/kg + ₹2/kg match = ₹69/kg of yard weight, the same day, on the recycler's behalf (₹2,070 for 30 kg).
- Recycler repays those payments plus ₹4/kg (₹120 for 30 kg), about 10 days after its gate check.
- Weight must match within ±5% at the yard (vs pickup) and at the gate (vs yard); a gap over 5% comes off the partner's next payment.
- ₹10 per tagged battery (delivery-and-record allowance, or the dealer's tagging fee in phase 2). Dealers keep take-backs in a crate until a partner scrap vendor collects them.
- Storage limit 90 days (CPCB, Jul 2026); LeadLoop target ~10 days. Partners deliver at least weekly.
- Only registered recyclers generate EPR certificates; LeadLoop supplies the evidence trail.
