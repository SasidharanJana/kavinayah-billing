# JK Hardwares — Billing

Rebranded from the Kavinayah Industries billing app you had at
github.com/SasidharanJana/kavinayah-billing. Same single-file
architecture: no server, no build step, no login gate — it boots
straight to the dashboard, with data stored in this browser's
`localStorage`.

## The logo

Your uploaded logo had "UPVC HARDWARE'S" in blue serif on a gray
gradient. I removed "UPVC" and redrew "HARDWARE'S" freshly centered,
matching the exact background gradient and brand blue color sampled
directly from your original file — not a rough crop-and-paste. Used as
both the app icon (`icon-192.png`, `icon-512.png`) and inline wherever the
app previously showed a small logo mark (sidebar, invoice letterhead).

## Real bugs found and fixed while rebranding this

This is the part worth reading carefully. The original file had your old
company's real details **hardcoded directly in four separate places** —
not read from Settings at all. This meant that even after changing the
Settings page, the actual printed invoices would have kept showing the
old company's information:

1. Invoice letterhead — company name and tagline were literally typed as
   `KAVINAYAH INDUSTRIES` / `UPVC WINDOWS · DOORS · PARTITIONS` in the
   template, ignoring the `companyName` setting entirely
2. Invoice footer — the old Salem address was hardcoded on every printed
   invoice, regardless of what Settings said
3. Signature block — `For KAVINAYAH INDUSTRIES` was fixed text
4. **Sidebar footer** — a second, completely separate hardcoded copy of
   the same old address, in the main app shell (this one wouldn't even
   have shown up if I'd only grepped for the company name — I only found
   it because I tested by actually setting new values and checking what
   printed, not by reading the code and assuming it was wired up correctly)

All four now read from Settings live. I also noticed `tagline` was a
setting that existed in the data but had **no way to edit it and was
never displayed anywhere** — dead code. Added a real "Tagline" field to
Settings that now actually appears on printed invoices.

## Logo not appearing on printed invoices — found and fixed after your report

You caught a real one: `printInvoice()` copied the invoice preview
(including the logo `<img>`) into the print area, then called
`window.print()` on the very next line. Setting an image's `src` starts
an asynchronous load — it isn't finished by the next line of code, even
if the browser already has it cached. Calling print immediately after
could open the print dialog before the logo had actually painted, so it
came out as blank space on the printed page while showing fine
everywhere else on screen. It now waits for the logo to actually finish
loading (or fail, or a short safety-net timeout if neither happens)
before printing — I tested all three of those paths directly, not just
the one that was already working.

## What's blank, on purpose

Proprietor name, phone, address, state, and state code — I don't have
JK Hardwares' real details, and carrying over Kavinayah's real business
information onto a different company's invoices would have been actively
wrong, not just incomplete. Fill these in under **Settings** first thing.

Also cleared the old product catalog (UPVC sliding windows, casement
windows, main doors, partition panels) — those are finished window/door
products, not hardware components, so they don't fit a hardware
supplier's inventory. Starts empty; add your real stock from Settings →
Inventory.

## Testing performed

Real simulated clicks, not direct function calls — the only way the
sidebar-footer bug above actually got caught:
- Confirmed no login gate, straight to dashboard (this app's real
  architecture — no PIN system like some of the other billing apps built
  in this series)
- Confirmed every field that should be blank is blank, and the new
  Tagline field exists and saves correctly
- Added a real customer and a real hardware product
- Created a full invoice and specifically checked the rendered letterhead,
  footer, and signature block for old hardcoded text — all four
  previously-hardcoded spots now correctly show live settings data
- Zero JavaScript errors across the full run

## Deploying

Same repo, same process — this replaces what's currently in
`kavinayah-billing` (or push to a renamed repo if you'd rather):

```
cd jk-hardwares-billing
git remote add origin https://github.com/SasidharanJana/<your-repo-name>.git
git push -u origin main
```
