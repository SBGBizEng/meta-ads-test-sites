# Meta Ads Test Sites — Next Steps Guide

## What's Live Now

All 11 test sites are deployed and accessible. Each page is a self-contained simulation with an interactive event log that shows what pixel/CAPI payloads *would* fire — including the intentional issues. No real tracking is active yet.

---

## What to Add Next

### Total Meta Assets Required

To fully implement all 11 test scenarios, you only need a minimal number of core Meta assets. The goal is to use a single set of assets and introduce the specific errors at the implementation level for each site.

| Asset Type | Quantity Needed | Notes |
|---|:---:|---|
| **Meta Pixel / Dataset** | 1 | A single Pixel ID can be used across all 11 sites. The different issues (e.g., duplicate install, wrong ID) are simulated in the code. |
| **Product Catalog** | 1 | A single catalog can hold all the products needed for Sites 01, 06, and 11. Each product within the catalog can be crafted to have the specific intentional issue (e.g., stale price, bad image URL). |
| **CAPI Server-Side App** | 1 | A single server-side application (e.g., a small Python/Node.js app) can handle all CAPI events. Logic within the app will introduce the specific errors for each test case (e.g., omitting `event_id` for Site 05). |
| **CRM Webhook Endpoint** | 1 | A single webhook endpoint (e.g., on Zapier or Make.com) can receive leads from all forms. Logic in the webhook flow will determine how to process them to simulate the issues on Sites 02, 09, and 10. |

---


### Step 1 — Install the Meta Pixel (Base Code)

The first thing to add to each site is the Meta Pixel base code. This goes in the `<head>` of every page and enables browser-side event tracking.

**Where to get it:** Meta Events Manager → Data Sources → Add New → Web → Meta Pixel

**What to add to each `index.html`:**

```html
<!-- Meta Pixel Code -->
<script>
!function(f,b,e,v,n,t,s)
{if(f.fbq)return;n=f.fbq=function(){n.callMethod?
n.callMethod.apply(n,arguments):n.queue.push(arguments)};
if(!f._fbq)f._fbq=n;n.push=n;n.loaded=!0;n.version='2.0';
n.queue=[];t=b.createElement(e);t.async=!0;
t.src=v;s=b.getElementsByTagName(e)[0];
s.parentNode.insertBefore(t,s)}(window, document,'script',
'https://connect.facebook.net/en_US/fbevents.js');
fbq('init', 'YOUR_PIXEL_ID');
fbq('track', 'PageView');
</script>
<noscript><img height="1" width="1" style="display:none"
src="https://www.facebook.com/tr?id=YOUR_PIXEL_ID&ev=PageView&noscript=1"
/></noscript>
<!-- End Meta Pixel Code -->
```

**Key decisions:**
- Use a **single Pixel ID** across all 11 sites (they share one test account)
- Site 03 (`edu_pixel_setup`) intentionally needs a **duplicate** install to demo that issue
- Site 03 also needs a **consent manager stub** to demo the blocking issue

---

### Step 2 — Wire Up the Simulated Events to Real `fbq()` Calls

Each site already has a JavaScript `log()` function that outputs what events *would* fire. The next step is to replace those log calls with real `fbq('track', ...)` calls.

Each site's event log already shows the exact payload — just copy those into real `fbq()` calls.

**Priority order by site:**

| Site | Events to Wire Up |
|------|-------------------|
| 01 · Catalog Sales | `ViewContent`, `AddToCart`, `Purchase` |
| 03 · Pixel Setup | `PageView` (duplicate), `ViewContent` (wrong page) |
| 04 · Value Optimization | `Purchase` (with each broken value mode) |
| 05 · CAPI | `ViewContent`, `AddToCart`, `InitiateCheckout`, `Purchase` |
| 07 · Pixel Opt | `PageView`, `AddToCart` (over-fire), `Purchase` (no params) |
| 11 · Live Video | `ViewContent` (missing), `Purchase` |

---

### Step 3 — Set Up a Product Catalog

Sites 01 and 06 are specifically designed to test catalog issues. To make Dynamic Ads work (and to demo the mismatches), you need a Meta Product Catalog.

**Steps:**
1. Go to **Commerce Manager** → Create a Catalog → Manual feed or Google Sheet
2. Add products with the IDs used in the test sites (e.g. `prod_123`, `BAG_001`, etc.)
3. Intentionally introduce the issues from the spec:
   - Use `prod_123` in catalog but fire `SKU123` from pixel (Site 01)
   - Set prices that differ from what the pixel sends
   - Set `availability: in_stock` for OOS items (Site 06)
   - Leave GTIN/MPN blank on some rows (Site 06)
   - Use a broken image URL for one product (Site 06)

**Catalog CSV template for Site 01:**

```csv
id,title,description,availability,price,link,image_link,brand,gtin
prod_123,Classic Leather Sneaker,Test product,in stock,89.99 USD,https://rohantirumeta.github.io/meta-ads-test-sites/sales_catalog_sales/,https://example.com/shoe.jpg,TestBrand,4901234567890
BAG_001,Canvas Tote Bag,Test product,in stock,45.00 EUR,https://rohantirumeta.github.io/meta-ads-test-sites/sales_catalog_sales/,https://example.com/bag.jpg,TestBrand,4901234567891
```

---

### Step 4 — Implement Conversions API (CAPI)

Sites 02, 05, 08, and 10 are built to test CAPI issues. To make them real, you need a server-side endpoint that posts to the Meta Conversions API.

**Minimum viable CAPI setup (Node.js / Python):**

```python
import requests, hashlib, time

def send_capi_event(event_name, email, event_id=None):
    hashed_email = hashlib.sha256(email.lower().encode()).hexdigest()
    payload = {
        "data": [{
            "event_name": event_name,
            "event_time": int(time.time()),
            "event_id": event_id,  # Required for deduplication
            "action_source": "website",
            "user_data": {
                "em": [hashed_email]
            }
        }]
    }
    r = requests.post(
        f"https://graph.facebook.com/v18.0/{PIXEL_ID}/events",
        params={"access_token": ACCESS_TOKEN},
        json=payload
    )
    return r.json()
```

**To demo the issues on each site:**
- **Site 05**: Omit `event_id` to show deduplication failure
- **Site 08**: Use a wrong `dataset_id` and leave `test_event_code` in
- **Site 02**: Send `em` unhashed (raw email string) to show PII hashing issue
- **Site 10**: Send `Purchase` instead of `Contact` for the Contacted stage

---

### Step 5 — Connect Lead Forms to a CRM Webhook

Sites 02, 09, and 10 test lead flow issues. To make these realistic:

1. Set up a simple webhook endpoint (e.g. using **Zapier**, **Make**, or a custom server)
2. Connect it to Meta Lead Ads via **Meta Business Suite → Instant Forms → CRM Integration**
3. For Site 09, intentionally **do not** configure the webhook to demo the "no CRM webhook" issue
4. For Site 10, configure the webhook but **do not** send downstream events (Contact, QualifiedLead, etc.)

---

### Step 6 — Add the Meta Pixel Helper / Test Events Tool

Once the pixel is installed, use these tools to verify:

- **Meta Pixel Helper** (Chrome extension) — shows which events fire on each page in real time
- **Test Events tool** in Events Manager — confirms events are received server-side
- **Event Deduplication** — check the "Deduplicated" column in Events Manager to verify Site 05's double-counting issue

---

## Site-by-Site Pixel/CAPI Readiness Checklist

| # | Site | Pixel Needed | CAPI Needed | Catalog Needed |
|---|------|:---:|:---:|:---:|
| 01 | Catalog Sales Issues | ✓ | — | ✓ |
| 02 | Lead Ads + CAPI for CRM | ✓ | ✓ | — |
| 03 | Basic Pixel Setup Mistakes | ✓ (×2) | — | — |
| 04 | Value Optimization Issues | ✓ | — | — |
| 05 | CAPI Implementation Issues | ✓ | ✓ | — |
| 06 | Catalog Optimization Issues | — | — | ✓ |
| 07 | Pixel Optimization Education | ✓ | — | — |
| 08 | CAPI Gateway Issues | — | ✓ | — |
| 09 | Instant Form Issues | ✓ | — | — |
| 10 | Conversion Leads Setup | — | ✓ | — |
| 11 | Live Video Shopping | ✓ | — | ✓ |
