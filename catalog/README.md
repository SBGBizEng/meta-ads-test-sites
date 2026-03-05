# Product Catalog Feed

**Catalog ID:** `1597133564781093`  
**Connected Pixel:** Ecomm Pixel `1981397602752829`  
**Feed file:** `product_feed.csv`

---

## Product Inventory & Intentional Issues

This catalog contains 12 products spread across three test sites. Each product is designed to demonstrate a specific issue (or correct behaviour) when matched against pixel events.

### Site 06 — Catalog Optimization Issues (`opt_product_catalog`)

These products demonstrate feed-level quality problems.

| Product ID | Name | Intentional Issue |
|---|---|---|
| `SHOE_001` | Classic Runner | **OOS showing as in_stock** + stale price ($50 feed vs $45 website) + missing `product_type` |
| `BAG_002` | Canvas Weekender | **Missing GTIN and MPN** — hurts ad delivery and catalog match quality |
| `HAT_003` | Bucket Hat | **Bad image URL** — `image_link` points to a 404 path |
| `SCARF_004` | Wool Scarf | **Image too small** (URL implies <500px) + missing `product_type` |
| `PANT_005` | Slim Chino | **Stale price** — feed shows $79.99 but website shows $59.99 sale price |

---

### Site 01 — Catalog Sales Issues (`sales_catalog_sales`)

These products demonstrate pixel-vs-catalog mismatches.

| Product ID | Name | Intentional Issue |
|---|---|---|
| `prod_123` | Classic Leather Sneaker | **ID mismatch** — pixel fires `SKU123` but catalog has `prod_123` → breaks Dynamic Ads |
| `BAG_001` | Canvas Tote Bag | **Currency mismatch** — catalog priced in `EUR`, pixel fires `USD` |
| `BEAN_007` | Wool Beanie | **Price mismatch** — catalog shows `89.99 USD` but pixel sends `value: 99.99` |
| `SHORT_02` | Running Shorts | **Missing `content_type`** — pixel Purchase event omits `content_type: 'product'` |

---

### Site 11 — Live Video Shopping Issues (`product_live_video`)

These products demonstrate live video tagging mismatches.

| Product ID | Name | Intentional Issue |
|---|---|---|
| `SHOE_CATALOG_99` | Air Runner (Catalog) | **ID mismatch** — stream shows `SHOE_LIVE_01` but catalog has `SHOE_CATALOG_99` |
| `BAG_LIVE_02` | Tote Bag (Live) | **Correctly tagged** — `BAG_LIVE_02` matches in both stream and catalog (reference/correct case) |
| `JACKET_CATALOG_04` | Sport Jacket (Catalog) | **ID mismatch** — stream shows `JACKET_LIVE_04` but catalog has `JACKET_CATALOG_04` |

> **Note:** `HAT_LIVE_03` (Wool Hat) is intentionally **not in the catalog** — it has no product tag at all, demonstrating the missing tag issue.

---

## How to Upload

### Option A — Manual Upload (Recommended for first time)

1. Go to [Commerce Manager → Catalog 1597133564781093 → Data Sources](https://business.facebook.com/commerce/catalogs/1597133564781093/datasources)
2. Click **Add Items** → **Use Data Feed**
3. Select **Upload File** → choose `product_feed.csv` from this folder
4. Set currency to **USD** (BAG_001 overrides to EUR inline)
5. Click **Upload** — products will appear within a few minutes

### Option B — Scheduled Feed URL (for ongoing sync)

Once GitHub Pages is live, you can point Commerce Manager to the feed URL directly:

```
https://rohantirumeta.github.io/meta-ads-test-sites/catalog/product_feed.csv
```

1. Go to **Data Sources → Add Items → Use Data Feed → Use a URL**
2. Paste the URL above
3. Set schedule to **Daily** (or Manual)

---

## Notes on Intentional Issues

- `HAT_003` image URL is intentionally broken (404) — Meta will flag this in feed diagnostics
- `SCARF_004` image URL implies a small image — Meta may reject or warn
- `SHOE_001` is marked `in_stock` but should be OOS — ads will serve for an unavailable product
- `BAG_001` is priced in `EUR` while the pixel fires `USD` — currency mismatch will show in reporting
- `BEAN_007` catalog price (`89.99`) intentionally differs from pixel value (`99.99`) — price discrepancy warning
