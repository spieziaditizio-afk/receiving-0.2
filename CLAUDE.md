# receiving 0.2 — Agent Instructions

## Project
Arrow Electronics · Microsoft Dept · Sevenum NL — warehouse receiving tools.
Standalone single-file HTML apps (React 18 CDN + Babel standalone). No npm, no build step. Edit HTML files directly.

## Files
- `PackCalc.html` — Single-pallet piece counter + thermal ticket generator (dual-device)

## Target Hardware
| Station | Device | Printer | Label |
|---------|--------|---------|-------|
| Receiving desk | Desktop + Zebra DS3678 scanner | Zebra ZT (industrial) | 12.8 × 6.3 cm |
| Floor / mobile | Zebra TC520L handheld (5" touch, ~360px CSS width) | Zebra QLn420 (portable) | 11.3 × 6.3 cm |

- Printer selector in UI toggles `cfg.printer: 'zt' | 'qln'` — controls label width in print popup
- TC520L layout: `@media(max-width:639px)` — tab bar (SCAN / BOXES), `100dvh` fixed, no page scroll
- Desktop layout: `@media(min-width:640px)` — side-by-side panels, `overflow:hidden`

## Design System (established session 2026-06-04)
- **Fonts**: Bebas Neue (display numbers/titles) · Barlow Condensed (UI chrome) · JetBrains Mono (codes/data)
- **Palette**: bg `#07080b` · surface `#0c0e14` · amber `#ffb700` (primary) · green `#00e569` (success) · red `#ff3838` · yellow `#f7c048`
- **Style**: Industrial dark · corner-bracket scan zones · Bebas Neue 50px counters · uppercase labels
- Note: `design_extract/receiving-0-2/CLAUDE.md` has the original design spec (orange #f47c1f / Inter); the implemented version above supersedes it.

## State
- `PackCalc` session → localStorage key `packcalc` — cfg, meta, boxes, palletId
- `PackCalc` history → localStorage key `packcalc_pallets` — array of up to 10 pallets (most recent first)

## Modes
- **RECEIVE** (desktop): scan boxes → generate ticket → saves pallet to history with QR code
- **FLOOR** (TC520L): scan label QR → loads pallet data → tap PICK to remove boxes → reprint updated label
  - QR encodes full pallet JSON: `{v:1, id, m:{pn,coo,po,ref}, b:[{i,pn,coo,q}]}`
  - History list shows 10 most recent pallets as fallback if QR unavailable

## Print layouts
- **LISTA**: vertical box breakdown with PN/COO/qty per row
- **BURBUJAS**: horizontal bubbles — qty large (Mono) + box number (B01) below; left-to-right wrapping
- Both layouts include QR code in label header (46×46px) — scannable by TC520L integrated imager

## Environment
- Shell: PowerShell (Windows 11) — never use bash syntax
- Python: `python` (not `python3`) → `C:\Users\aspiezia\AppData\Local\Programs\Python\Python312\python.exe`
- ui-ux-pro-max script: `python "C:\Users\aspiezia\.claude\plugins\cache\ui-ux-pro-max-skill\ui-ux-pro-max\2.5.0\cli\assets\scripts\search.py"`

## Receiving Design Bundles (api.anthropic.com/v1/design/h/...)
WebFetch saves the binary to a `.bin` temp file. Extract with:
```powershell
tar -xzf <path-to-.bin> -C <destination-folder>
```
Read `design_extract/receiving-0-2/README.md` and the chat transcripts before implementing.
