# ZINTERACTIVE_REPORT

A classical **interactive ABAP report** for Purchase Orders. It uses drill-down
lists (`AT LINE-SELECTION`) instead of a single flat list — click a row to go
deeper.

## What it shows

| Level | Data | Source Table | Triggered by |
|-------|------|--------------|--------------|
| 1 | Purchase Order header (PO number, vendor, date, company, currency) | `EKKO` | Report execution |
| 2 | PO line items (material, plant, quantity, net price, deletion flag) | `EKPO` | Click a PO number on Level 1 |
| 3 | Goods Receipt / Invoice history (doc number, movement type, posting date, qty, amount) | `EKBE` | Click an item on Level 2 |

## Selection screen fields

- `S_EBELN` – Purchase Order number
- `S_BUKRS` – Company code
- `S_WERKS` – Plant
- `S_AEDAT` – Document date (defaults to last 365 days)
- `S_LIFNR` – Vendor

## Installation

1. Go to transaction `SE38` (or `SE80`).
2. Create a new program named `ZPUR_INTERACTIVE_REPORT`, type **Executable Program**.
3. Paste in the code from `ZPUR_INTERACTIVE_REPORT.abap`.
4. Go to the program's **Text Elements** (`Goto > Text Elements`) and add:
   - Text symbol `001` = `Selection Criteria`
5. Activate (`Ctrl+F3`).
6. Run (`F8`).

## How to use it

1. Enter selection criteria (or leave blank for defaults) and execute.
2. The first list shows PO headers — PO numbers are shown as clickable **hotspots**.
3. Click a PO number → drills into that PO's line items.
4. Click a line item → drills into GR/Invoice history for that item.
5. Use the standard **Back** button (green arrow) to return to the previous list level.

## Known limitations / things to adapt for production use

- No authority checks — add `AUTHORITY-CHECK` against object `M_BEST_BSA` / `M_BEST_WRK` before displaying data.
- No deletion-flag or status filtering beyond what's shown — add `WHERE loekz = ''` if you only want active items.
- Uses direct `SELECT` on `EKKO`/`EKPO`/`EKBE` for simplicity — consider CDS views or a logical database for large datasets or better performance.
- No sorting, subtotals, or export options — this is the minimal drill-down structure. Ask if you want these added, or want a rebuild using ALV Grid (`CL_GUI_ALV_GRID`) with double-click drill-down instead of classical lists.

