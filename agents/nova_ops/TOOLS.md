# NovaOps Tools — LEDStock API

## SKU Handling Rule
**When given a SKU code (e.g., NLW1-NKL, NLW2-BRS), use it exactly as provided.** Do not attempt to correct, modify, or substitute it with another variant. Look up the exact SKU via API and report what you find — nothing more, nothing less.

## Auth

```bash
BASE="http://localhost:3003"
KEY="<LEDSTOCK_SERVICE_API_KEY>"   # stored in TOOLS.md on server; do not commit to repo
# All requests:
curl -s -H "X-Api-Key: $KEY" ...
```

---

## Products

### Look up product and finish names (source of truth)
```bash
curl -s -H "X-Api-Key: $KEY" "$BASE/products" | python3 -m json.tool
```
Returns each product's `sku`, `finish_code`, `finish_name`, `family`, `status`.
**Use this whenever you need a finish name and the stock response is not already in context.**
Never guess or infer names from codes — always read from the API response.

---

## Stock

### Get available stock (all SKUs)
```bash
curl -s -H "X-Api-Key: $KEY" "$BASE/stock/available" | python3 -m json.tool
```
Returns: `sku, family, finish_name, internal_qty, reserved_qty, available_qty, updated_at`

### Get available stock (single SKU)
```bash
curl -s -H "X-Api-Key: $KEY" "$BASE/stock/available/NLW1-BRS" | python3 -m json.tool
```

### Get stock under treatment
```bash
curl -s -H "X-Api-Key: $KEY" "$BASE/stock/under-treatment" | python3 -m json.tool
```

### Get movement history
```bash
# All movements (paginated)
curl -s -H "X-Api-Key: $KEY" "$BASE/stock/movements?page=1&page_size=50" | python3 -m json.tool

# Filter by SKU
curl -s -H "X-Api-Key: $KEY" "$BASE/stock/movements?sku=NLW1-BRS" | python3 -m json.tool

# Filter by type
curl -s -H "X-Api-Key: $KEY" "$BASE/stock/movements?movement_type=COUNT_ADJUSTMENT" | python3 -m json.tool
```

---

## Stock Counts

### List all stock counts
```bash
curl -s -H "X-Api-Key: $KEY" "$BASE/stock-counts" | python3 -m json.tool
```

### Get a specific count
```bash
curl -s -H "X-Api-Key: $KEY" "$BASE/stock-counts/1" | python3 -m json.tool
```

### Initiate a new stock count (snapshots all active stock)
```bash
curl -s -X POST -H "X-Api-Key: $KEY" -H "Content-Type: application/json" \
  -d '{"count_date": "2026-05-24", "notes": "Monthly count"}' \
  "$BASE/stock-counts" | python3 -m json.tool
```

### Update a count line (enter physical quantity)
```bash
curl -s -X PATCH -H "X-Api-Key: $KEY" -H "Content-Type: application/json" \
  -d '{"physical_quantity": 42, "reason": "recount confirmed"}' \
  "$BASE/stock-counts/{count_id}/lines/{line_id}" | python3 -m json.tool
```

### Submit count for review
```bash
curl -s -X POST -H "X-Api-Key: $KEY" "$BASE/stock-counts/{count_id}/submit" | python3 -m json.tool
```
⚠️ Approval requires Lars or Daniel — NovaOps cannot approve.

### Get adjustments applied to a count
```bash
curl -s -H "X-Api-Key: $KEY" "$BASE/stock-counts/{count_id}/movements" | python3 -m json.tool
```

---

## Production Batches (Incoming from China)

### List all batches
```bash
curl -s -H "X-Api-Key: $KEY" "$BASE/production-batches" | python3 -m json.tool
```

### Get a specific batch
```bash
curl -s -H "X-Api-Key: $KEY" "$BASE/production-batches/1" | python3 -m json.tool
```

### Receive goods from a batch
```bash
curl -s -X POST -H "X-Api-Key: $KEY" -H "Content-Type: application/json" \
  -d '[{"line_id": 1, "quantity_received": 100}]' \
  "$BASE/production-batches/1/receive" | python3 -m json.tool
```

---

## Treatment Orders (TempoKrom)

### List all treatment orders
```bash
curl -s -H "X-Api-Key: $KEY" "$BASE/treatment-orders" | python3 -m json.tool
```

### Get a specific treatment order
```bash
curl -s -H "X-Api-Key: $KEY" "$BASE/treatment-orders/1" | python3 -m json.tool
```

---

## Sales Orders

### List all sales orders
```bash
curl -s -H "X-Api-Key: $KEY" "$BASE/sales-orders" | python3 -m json.tool
```

### Check reserved stock by order
```bash
curl -s -H "X-Api-Key: $KEY" "$BASE/sales-orders" | \
  python3 -c "import json,sys; orders=json.load(sys.stdin); \
  [print(o['id'], o['customer_name'], o['status']) for o in orders if o['status'] in ['open','reserved']]"
```

---

## Sample Loans

### List all loans
```bash
curl -s -H "X-Api-Key: $KEY" "$BASE/sample-loans" | python3 -m json.tool
```

### Get a specific loan
```bash
curl -s -H "X-Api-Key: $KEY" "$BASE/sample-loans/1" | python3 -m json.tool
```

### Create a new loan (automatically generates SEND_SAMPLE movements)
```bash
curl -s -X POST -H "X-Api-Key: $KEY" -H "Content-Type: application/json" \
  -d '{"loan_date": "2026-05-27", "partner": "Company Name", "contact_location": "City", "purpose": "Showroom display", "lines": [{"product_sku": "NLW1-BRS", "quantity": 1}]}' \
  "$BASE/sample-loans" | python3 -m json.tool
```
⚠️ Loans are created directly in `out` status. SEND_SAMPLE movements are generated immediately at creation — there is no separate send step.

### Return a loan (generates RETURN_SAMPLE movements)
```bash
curl -s -X POST -H "X-Api-Key: $KEY" "$BASE/sample-loans/1/return" | python3 -m json.tool
```

---

## Shopify Inventory Sync

### Push available stock to Shopify
⚠️ **ALWAYS confirm quantities are correct before running. Ask if uncertain.**
```bash
curl -s -X POST -H "X-Api-Key: $KEY" "$BASE/shopify/push-inventory" | python3 -m json.tool
```
Returns: `pushed_count, skipped_count, errors[]`

---

## Ledger

### Full movement ledger (all SKUs, newest first)
```bash
curl -s -H "X-Api-Key: $KEY" "$BASE/stock/ledger?page=1&page_size=50" | python3 -m json.tool
```

### Filter ledger by SKU
```bash
curl -s -H "X-Api-Key: $KEY" "$BASE/stock/ledger?sku=NLW2-BRS" | python3 -m json.tool
```

### Filter ledger by movement type
```bash
curl -s -H "X-Api-Key: $KEY" "$BASE/stock/ledger?movement_type=COUNT_ADJUSTMENT" | python3 -m json.tool
```
Valid movement types: `RECEIVE_CHINA`, `SEND_TREATMENT`, `RECEIVE_TREATMENT`, `SELL_INVOICE`, `SELL_SHOPIFY`, `RESERVE`, `RESERVE_RELEASE`, `SEND_SAMPLE`, `RETURN_SAMPLE`, `COUNT_ADJUSTMENT`, `SCRAP`, `DEVELOPMENT`

---

## Consistency Check

### Verify ledger vs stored balances (read-only)
```bash
curl -s -H "X-Api-Key: $KEY" "$BASE/stock/consistency-check" | python3 -m json.tool
```
Replays all movements and compares to `stock_balances`. Returns `{"ok": true}` or a list of mismatches with `stored_qty`, `expected_qty`, and `diff`.
⚠️ Read-only — nothing is changed. If mismatches are found, escalate to a developer.

---

## Quick Diagnostics

### What to report when asked about "total stock"

The system tracks multiple physical locations (INTERNAL, SHOWROOM, TEMPO_KROM, CUSTOMER).
**Always use `available_qty` from `/stock/available` when reporting how much stock we have.**
- `available_qty` = INTERNAL balance minus reserved — this is what the LEDStock UI shows
- SHOWROOM (7 units), TEMPO_KROM (under treatment), CUSTOMER (on loan) are NOT available — never add them to the total
- Correct answer for "total stock": sum of `available_qty` only

### Full stock snapshot (table format)
```bash
curl -s -H "X-Api-Key: $KEY" "$BASE/stock/available" | \
  python3 -c "
import json, sys
rows = json.load(sys.stdin)
print(f'{'SKU':<14} {'Available':>10} {'Reserved':>10} {'Internal':>10}')
print('-' * 46)
total = 0
for r in sorted(rows, key=lambda x: x['sku']):
    flag = ' MANCO' if r['available_qty'] < 0 else ''
    print(f'{r[\"sku\"]:<14} {r[\"available_qty\"]:>10} {r[\"reserved_qty\"]:>10} {r[\"internal_qty\"]:>10}{flag}')
    total += r['available_qty']
print('-' * 46)
print(f'TOTAL AVAILABLE: {total}')
"
```

### Check for mancos
```bash
curl -s -H "X-Api-Key: $KEY" "$BASE/stock/available" | \
  python3 -c "import json,sys; rows=json.load(sys.stdin); mancos=[r for r in rows if r['available_qty']<0]; print('MANCOS:', len(mancos)); [print(' -', r['sku'], r['available_qty']) for r in mancos]"
```
