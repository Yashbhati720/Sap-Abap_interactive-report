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

## Program Structure: Include Programs

This project is modularized using **Include programs** rather than one monolithic report. The main program (`ZPUR_INTERACTIVE_REPORT`) only contains the `INCLUDE` statements and the overall flow; each functional area lives in its own Include.

| Include Name | Purpose | Contains |
|--------------|---------|----------|
| `ZPUR_INT_TOP` | Global declarations | `TABLES`, `TYPES`, `DATA`, `FIELD-SYMBOLS`, constants used across the whole program |
| `ZPUR_INT_SEL` | Selection screen | `SELECT-OPTIONS`, `PARAMETERS`, `SELECTION-SCREEN` blocks (`S_EBELN`, `S_BUKRS`, `S_WERKS`, `S_AEDAT`, `S_LIFNR`) |
| `ZPUR_INT_F01` | Data retrieval (forms) | `FORM` routines that `SELECT` from `EKKO`, `EKPO`, `EKBE` |
| `ZPUR_INT_O01` | List display (forms) | `FORM` routines that build/format each drill-down level's output list |
| `ZPUR_INT_E01` | Event handling | `AT LINE-SELECTION`, `AT SELECTION-SCREEN`, `TOP-OF-PAGE`, `INITIALIZATION` logic, dispatched by current list level (`sy-lsind`) |

### Naming Convention

- All Includes are prefixed with the main program's short name (`ZPUR_INT_`) so they group together and are easy to find via `SE38`/`SE80`.
- Suffixes indicate purpose: `_TOP` (top include / global data), `_SEL` (selection screen), `_F0x` (form routines), `_O0x` (output/list routines), `_E0x` (event blocks).

### Why Includes Are Used Here

- Keeps the main program short and readable — it's essentially a table of contents.
- Separates **data retrieval** (`_F01`) from **presentation** (`_O01`), so changing how a list looks doesn't risk touching the `SELECT` logic.
- Event logic (`_E01`) is isolated so the drill-down flow (`sy-lsind` level checks in `AT LINE-SELECTION`) is easy to trace in one place.
- Makes future extensions (e.g., adding a 4th drill-down level, or swapping to ALV) easier — you'd mainly touch `_O01` and `_E01` without disturbing selection or data-fetch logic.

### Adding a New Include

1. In `SE38`, open the main program `ZPUR_INTERACTIVE_REPORT`.
2. Place your cursor where the new Include should be inserted and add:
   ```abap
   INCLUDE zpur_int_f02.
   ```
3. Double-click the Include name → system prompts to create it → confirm.
4. Choose **Include Program** as the object type, assign it to the same package/transport as the main program.
5. Write your logic inside the new Include, then activate both the Include and the main program.

## Installation

1. Go to transaction `SE38` (or `SE80`).
2. Create a new program named `ZPUR_INTERACTIVE_REPORT`, type **Executable Program**.
3. Paste in the code from `ZPUR_INTERACTIVE_REPORT.abap` — note this will primarily consist of `INCLUDE` statements as described above (`ZPUR_INT_TOP`, `ZPUR_INT_SEL`, `ZPUR_INT_F01`, `ZPUR_INT_O01`, `ZPUR_INT_E01`).
4. Create each Include program listed above (via double-click-to-create, as described in **Adding a New Include**) and paste in its corresponding section of code.
5. Go to the program's **Text Elements** (`Goto > Text Elements`) and add:
   - Text symbol `001` = `Selection Criteria`
6. Activate the main program and **all** Includes (`Ctrl+F3`) — an Include will not run correctly if it isn't activated along with the main program.
7. Run (`F8`).

## How to use it

1. Enter selection criteria (or leave blank for defaults) and execute.
2. The first list shows PO headers — PO numbers are shown as clickable **hotspots**.
3. Click a PO number → drills into that PO's line items.
4. Click a line item → drills into GR/Invoice history for that item.
5. Use the standard **Back** button (green arrow) to return to the previous list level.

## Known limitations / things to adapt for production use

- No authority checks — add `AUTHORITY-CHECK` against object `M_BEST_BSA` / `M_BEST_WRK` before displaying data (would live in `ZPUR_INT_F01` alongside the existing `SELECT`s).
- No deletion-flag or status filtering beyond what's shown — add `WHERE loekz = ''` if you only want active items.
- Uses direct `SELECT` on `EKKO`/`EKPO`/`EKBE` for simplicity — consider CDS views or a logical database for large datasets or better performance.
- No sorting, subtotals, or export options — this is the minimal drill-down structure. Ask if you want these added, or want a rebuild using ALV Grid (`CL_GUI_ALV_GRID`) with double-click drill-down instead of classical lists.
- Include programs are not independently executable or transportable on their own — always test and transport them together with the main program `ZPUR_INTERACTIVE_REPORT` as one unit.
