# Receiving 0.2 — Project Instructions

This project contains one warehouse receiving tool for **Arrow Electronics, Microsoft Department, Sevenum NL**.

## Apps Overview

### 1. **PackCalc.html** — Smart Single-Pallet Counter
- **Purpose**: Quick box-by-box counting for a single pallet; generates a receiving ticket
- **Workflow**: Configure scan profile → Scan boxes (PN + COO + Qty optional) → Generate ticket
- **Key features**:
  - Configurable scan fields (PN, COO, QTY toggles; QTY always required)
  - Step-by-step guidance; "skip" link for optional fields
  - Real-time box list with edit (qty) and delete
  - Mixed pallet detection (warns if multiple PN/COO on same pallet)
  - Auto-complete PN/COO in ticket metadata if all boxes match
  - Thermal label output (12.8 cm Zebra ZT)

## Design System & Palette

All components use a dark industrial warehouse theme:
- **Background**: `#0d0f14`
- **Surface**: `#141720`
- **Accent (Orange)**: `#f47c1f` — primary action, focus
- **Green**: `#23c55e` — complete, success, positive
- **Yellow**: `#f59e0b` — warning, discrepancy
- **Red**: `#ef4444` — error, critical
- **Typography**: Inter (UI), JetBrains Mono (data/codes)

## State Management

Both apps use **React hooks** + **localStorage** for persistence:
- **PackMatch**: localStorage key `pm6` (targets, results, screen state)
- **PackCalc**: localStorage key `packcalc` (config, metadata, boxes)

## Scanning & Barcode Handling

### Scan Profile (PackMatch)
- Defined at setup; tells the app which fields come from the barcode vs. are master data
- Example: if PN + COO + QTY enabled, the barcode carries those three; PO and Delivery are master
- Optional fields (PN, COO) can be "off the barcode" → operador selects target manually

### PackCalc Configuration
- PN and COO are toggle-able optional fields; QTY is always required
- Allows flexible workflows (full PN+COO+QTY, or just QTY for fast-tracking)

## Pallet Numbering & IDs

- Pallet ID format: `P01`, `P02`, etc. (padded 2-digit)
- Sequential within a session; resets on "New Session"
- Labels show "Pallet X of Y" to indicate multi-pallet targets

## Printing & Labels

### Zebra ZT Thermal (12.8 cm width)

**PackCalc Ticket**:
- Similar to pallet label, but for a single uncompleted pallet
- Can show mixed PN/COO with "— mixed / no PN —" fallback

## Editing & Data Corrections

### PackCalc Box Edit
- Inline edit (pencil) for piece quantity
- Delete box (trash) to remove
- Add new box via "Add box" form at the bottom of the list

## Error Handling & Alerts

### PackCalc
- **Mixed Pallet** (yellow): Multiple distinct PN or COO detected → informational warning, no block

## Persistence & Recovery

Both apps auto-save to localStorage on every state change:
- **PackCalc**: config, metadata, boxes

Refreshing the page restores the last session. "New Session" clears localStorage.

## Future Integration

PackCalc is currently independent. Potential connection points:
- Use PackCalc as a "quick verify"

## Contact & Support

For changes, bug reports, or feature requests:
- Edit the respective HTML files directly (Babel JSX inline)
- Test in the preview; verify with fork_verifier_agent if making significant changes
- Update localStorage keys if the data model changes substantially
