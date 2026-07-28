# The Cape Cod Comedy Carnival — Box Office, Step One
**The 1140A Corporation · 1140A.com · Sat 29 Aug 2026 · Tilden Arts Center**

Three self-contained files. No build step, no server code, no database — open them in a browser and they work.

| File | Who it's for | What it does |
|---|---|---|
| `index.html` | Buyers | The ticket site: show picker (SAGALOW / CANNON / CARNIVAL), interactive seat map from the venue's real chart, GA zones with back-fill meter, 3-minute cart hold, Zeffy checkout handoff, wheelchair booking with scroll-to-agree terms, contact routing, mature-content notice, mailing list, merch placeholder, Event JSON-LD for Google. |
| `admin.html` | You | The console: sales dashboard (by pass and zone, revenue, pace vs 20/day), seat manager (block / release / sponsor / marketing / comp / undo per seat), one-click house moves (sponsor release, marketing in & out, wheelchair→regular conversions per the spec), settings, export/import. Password: `greenroom` (change `ADMIN_PW` in the file). |
| `door.html` | Door crew | Check-in rehearsal: camera or paste, VALID / ALREADY IN / VOID verdicts, gold banner for both-shows passes, per-device door log. |

## Try it right now
1. Open `index.html`. Site password: **`pitrow`**.
2. Pick CARNIVAL, tap two seats, add GA, Continue to payment, press the **DEMO** button — tickets with QR codes appear and the "info@ ping" toast fires.
3. Open `admin.html` (password **`greenroom`**) — the same seeded sales appear on the dashboard.
4. Open `door.html`, tap **Mint test tickets**, and scan/paste — valid, duplicate, tampered.

Everything editable lives in **CONFIG** at the top of the `<script>` in `index.html` and `admin.html` (they carry identical copies — when you hand-edit one, mirror the other; the console's "Copy CONFIG patch" button formats the lines for you).

## The switch (test → live)
- `CONFIG.gate.enabled: true` → password-gated test mode (the default).
- Set it to `false` in both files → live. That is the whole flip.
- While testing on a real host, ALSO turn on the host's own password (Netlify → Site protection). Remove **both** at go-live — a password blocks Google from reading the page and its Event schema.
- `CONFIG.demo: true` seeds fake sales so every screen has life. Set `false` for launch (clean zero state).

## Zeffy wiring (the $0-fee checkout)
Create **three ticketing forms** in Zeffy (zeffy.com → Campaigns → Ticketing):
1. **Sagalow show** · ticket types: Orchestra $55 · Tier 1 $44 · Tier 2 $33 · Balcony $24
2. **Cannon show** · same prices
3. **Carnival — both shows** · Orchestra $88 · Tier 1 $66 · Tier 2 $55 · Balcony $38

For the two **reserved** zones on each form, add per-row **dropdown questions** ("Row C — pick your seat") listing each sellable seat once. Zeffy retires a dropdown option once it's bought — that's the double-sell lock. Tier 2 / Balcony ticket types are quantity-only (set their capacity: 185 and 121, minus anything you're holding).

Then paste the three form URLs into `CONFIG.zeffy` in both HTML files.

Per form, switch on: **real-time payment notifications** (Invite collaborators → notify on new payments → info@…), the e-ticket/QR option, and set ticket quantity caps. At the door, Zeffy e-tickets scan with any phone camera and reject reuse.

Truth lives in Zeffy once you're live: export its payment report and mark sold seats in the console (Seat manager → MARK SOLD) until Phase 2 automates the sync via Zeffy's API/Zapier.

## Deploy (Netlify or any static host)
1. Drag the three files into Netlify (or `netlify deploy`). Done — it's static.
2. Custom domain **1140A.com**, force HTTPS, and make `https://1140a.com` the canonical host with **301** redirects from `www` and `*.netlify.app`.
3. Test mode: Site protection password ON. Go-live: password OFF (both Netlify's and CONFIG's).
4. `admin.html` / `door.html` ship with `noindex` and passwords, but for real privacy consider renaming them to unguessable paths (e.g. `console-8k2p.html`) when you deploy.

## Superstition notes (the no-4 discipline)
Timers, counts we author, step labels, and generated ticket codes never show the number between 3 and 5 (the hold clock speaks in words; GA bars snap politely past it; ticket-code alphabet omits it). Three things keep their 4s deliberately: **your locked prices** ($44/$24), **the venue's printed seat numbers** (tickets must match armrest plates), and the college's street address. The GA quantity stepper counts naturally (1,2,3,4…) so a party of four can actually buy four tickets — flip that to superstitious counting by searching `maxPerOrder` and asking, and I'll wire it, but it costs real money.

## Phase 2 pockets (already architected, not yet built)
Comedian bio dropdowns · merch store behind the COMING SOON card (pre-sale at ~15% off) · scan-for-discount at concessions · source-of-sale analytics (UTM links per channel) · squatter pattern-recognition · live seat-map sync from Zeffy (API/Zapier → a tiny Cloudflare Worker) · styled dot-QR on the real e-ticket email. None requires a rebuild; the store functions in `core.js` are the single interface a backend swaps into.

— Built 28 Jul 2026 from `Ticketing_Build_Brief_1140A.docx`, the venue's seating chart, and `Tilden Seating - Final Option.xlsx`. Full decision log in `Build_Report_1140A_Ticketing.docx`.
