# Add Order

Add a customer order to the Viking Heads inventory app.

## Instructions

The user will provide order information — this could be pasted text from an email, a screenshot of a PO, a description of what a customer wants, or any other format. Your job:

1. **Read the SKU table** from `viking-heads.html` (search for `const SKU_TABLE`) to get all valid SKUs and their product info (productType, size, color, hookSize, hookKey).

2. **Read `HEADS_PER_BAG`** from `viking-heads.html` to know pack sizes per product type.

3. **Parse the order** — match every line item to a SKU in the table. Handle:
   - Fuzzy matching (dashes, spaces, case differences)
   - Product descriptions → SKU mapping (e.g., "Swinghead 1/4oz Commando" → VKS14CMDO)
   - "Shad" = Commando for Finesse Heads
   - Combine duplicate SKUs by summing quantities

4. **Present the parsed order** to the user for confirmation before adding. Show a table with SKU, product description, and quantity. Flag any items that couldn't be matched.

5. **Once confirmed**, add the order via a one-time migration in the `applyMigrations()` function in `viking-heads.html`. Use this pattern:

```javascript
// Migration: seed [Shop Name] [PO number] ([date])
if (!d._seeded[UniqueKey]) {
  const items = [
    { sku:'SKU1', qty:N }, { sku:'SKU2', qty:N },
    // ...
  ].map(({ sku, qty }) => {
    const def = SKU_TABLE[sku];
    if (!def) return null;
    const heads = qty * (HEADS_PER_BAG[def.productType] || 1);
    return {
      sku, productType: def.productType, size: def.size, color: def.color,
      hookSize: def.hookSize, hookKey: def.hookKey, quantity: qty,
      components: calcDeductionsForSku(def, heads),
      itemStatus: { poured: false, loaded: false, painted: false },
    };
  }).filter(Boolean);
  d.orders.unshift({
    id: 'unique_order_id',
    shop: 'Shop Name',
    dueDate: 'YYYY-MM-DD',  // 2 weeks from today
    notes: 'PO number or relevant notes',
    createdAt: new Date().toISOString(),
    status: 'pending',
    archived: false,
    items,
  });
  d._seeded[UniqueKey] = true;
}
```

6. **If SKUs are missing** from the table, ask the user how to handle them (skip, map to existing SKU, or add new SKUs to the table).

7. **Due date** defaults to 2 weeks from today unless the user specifies otherwise.

8. **Commit and push** the changes after adding the order.
