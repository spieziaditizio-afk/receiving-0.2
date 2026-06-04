# Receiving 0.2 — Smart Warehouse Reconciliation Suite

One complementary app for real-time shipment receiving and verification at **Arrow Electronics, Microsoft Department, Sevenum**.

## 📦 Quick Start

### PackCalc — Single-Pallet Quick Counter
**For fast box counting on one pallet**

1. **Configure**: Toggle PN and COO fields (QTY always required)
2. **Scan/Count**: Step-by-step guidance; skip optional fields as needed
3. **Edit**: Inline qty correction or delete boxes
4. **Generate Ticket**: One-click thermal label with box breakdown

**Best for**: Receiving dock speed, pre-count verification, mixed or light pallets

---

## 🎯 Key Workflows

### Scenario B: Mixed Single Pallet (PackCalc)
```
One pallet arrives with assorted boxes from different PNs.
Operator:
1. Toggles: PN ✓, COO ✓, QTY ✓
2. Scans each box PN→COO→Qty
3. System warns "Mixed pallet — 3 part numbers"
4. Generates ticket; PO/Ref fields optional
5. Prints label for receiving dock sign-off
```

### Scenario C: Vendor with PN + QTY only (no COO barcode)
```
Operator:
1. In Setup, toggles off COO in Scan Profile
2. System shows "Master: PO, DEL, COO" (not scanned)
3. Selects delivery & target manually
4. Scans PN→QTY per box (COO comes from target)
5. Completes normally
```

---

## 🔧 Technical Details

### Storage
- **PackCalc**: `localStorage['packcalc']` — config + metadata + boxes

auto-persist on every change; refresh restores last session.

### Barcode Format

**PO Numbers**
- Format: `VT9A00000####` (e.g., `VT9A000002123`)
- Input auto-uppercases; flexible validation (no strict length check)

**Part Numbers & COO**
- Auto-uppercased on scan
- COO examples: `CN`, `US`, `MX`, or custom
- Optional on PackCalc

**Piece Quantities**
- Integers only; minimum 1
- Per-box granularity (PackCalc)

## 🖨️ Hardware & Printing

### Barcode Scanner
- **Model**: Zebra DS3678 (2D industrial scanner)
- **Connection**: USB (appears as keyboard input to web browser)
- **Compatibility**: Works with any barcode format (Code128, QR, etc.)
- **Barcodes scanned**: PN, COO, PO, Qty (configurable per vendor)

### Thermal Printer
- **Model**: Zebra ZT Series (ZT411, ZT421, or equivalent)
- **Connection**: Network or USB (system prints via web browser `window.print()`)
- **Label size**: **12.8 cm (width) × 6.3 cm (height)** — self-adhesive thermal labels
- **Resolution**: 203 dpi (8 dots/mm)
- **Media**: Zebra Thermal Transfer or Direct Thermal self-adhesive rolls
- **Part numbers**: Zebra 10015349 (4×6 inch) or equivalent 12.8×6.3 cm stock

### Label Formats

#### Pallet Label
**Dimensions**: 12.8 cm wide × 6.3 cm tall  
**Layout**:
```
┌─────────────────────────────────┐
│ ARROW ELECTRONICS               │ ← Header (10pt bold)
│ MICROSOFT DEPT · SEVENUM        │
├─────────────────────────────────┤
│ P01 PALLET 1 OF 2               │ ← Pallet ID + multi-pallet notation
├─────────────────────────────────┤
│ 0C19551                         │ ← Part Number (24pt bold, monospace)
├─────────────────────────────────┤
│ PO: VT9A000002123  DEL: 8100987 │ ← Metadata in 3-column grid
│ COO: CN            BOXES: 3      │
├─────────────────────────────────┤
│ BOX BREAKDOWN                   │ ← Each box: qty (13pt bold)
│ Box 01: 30 pcs                  │
│ Box 02: 35 pcs                  │
│ Box 03: 35 pcs                  │
├─────────────────────────────────┤
│ TOTAL BOXES: 3    TOTAL: 100 pcs│ ← Totals (16pt bold)
├─────────────────────────────────┤
│ SEVENUM · 2026-06-04 12:34      │ ← Timestamp + Status badge
│        ✓ RECONCILED             │
└─────────────────────────────────┘
```

**Font stack**: Inter (headers), JetBrains Mono (PN/data)  
**Barcode**: Optional QR code in footer (future enhancement)

### Print Workflow

1. **Complete reconciliation** (100% gate passed)
2. **Click "Generate Ticket"** (PackCalc) or **"Print All Pallet Labels"** (PackMatch)
3. Browser opens new window with formatted label
4. User clicks "🖨 Print on Zebra ZT"
5. Print dialog appears → select Zebra ZT printer
6. Label prints on self-adhesive thermal stock
7. Window auto-closes; return to app

**Note**: Labels are optimized for **12.8 cm Zebra thermal stock**. For office reference on A4, use "Print Summary" instead.

---

## 📋 Data Model

### PackCalc — Single Pallet

```javascript
{
  config: {
    pn: true,   // Scan PN?
    coo: true   // Scan COO?
  },
  meta: {
    pn: "0C19551",      // Auto-complete or manual
    coo: "CN",
    po: "VT9A000002123",
    ref: "8100987654"   // Delivery reference
  },
  boxes: [
    { id: "uid", pn: "0C19551", coo: "CN", qty: 30, time: "12:34:56" },
    { id: "uid", pn: "0C19551", coo: "CN", qty: 35, time: "12:35:10" }
  ]
}
```

---

## ⚠️ Edge Cases & Error Handling


### PackCalc
- **Mixed pallet warning**: Informational only; doesn't block
- **Edit pallet qty**: Click pencil, type, press Enter or click away
- **Generate ticket with no boxes**: Button disabled until ≥1 box added

---

## 🚀 Deployment Notes

Both apps are **self-contained HTML + React + Babel**:
- No build step required
- Works offline (localStorage)
- Can be hosted on any static web server or opened directly from disk
- Tested at 1440×900 viewport (warehouse desk/tablet)

---

## 📞 Support

For bugs, feature requests, or operational questions:
- Check `CLAUDE.md` for technical architecture
- Review app-specific workflows above
- Test changes in preview before deploying to production
- Verify with `fork_verifier_agent` for significant logic changes

---

**Last updated**: June 4, 2026  
**Version**: 0.2  
**Location**: Arrow Electronics, Microsoft Department, Sevenum NL
