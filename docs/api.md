# Wayfare API

**Base URL:** `https://wayfare-cdb9.onrender.com`

## How this doc is structured

Every endpoint follows the same 4-part template to keep it scannable:

```
## [METHOD] /path

**Description:** 1-2 sentences — what it does and why it exists.

### Request
* Headers / Path / Query / Body params with type, required/optional, defaults, and allowed values.

### Response
* Success (status): Minimal valid JSON example with real issuer addresses, not placeholders.
* cURL: Copy-pasteable command for prod and localhost.
* Errors: List of status codes with exact error body shape from writeError.

### Notes
* Explanation of tricky fields (loss_pct, verdict, live vs stale, integrity).
* Edge cases (empty history, fallback behavior).
```

## GET /assets

**Description:** Lists all known Stellar assets and whether they can be used as a corridor destination.

### Request

**Headers:**
* `Accept`: application/json

### Response

**Success (200 OK):**
```json
{
  "assets": [
    {
      "code": "NGNC",
      "issuer": "GASBV6W7GGED66MXEVC7YZHTWWYMSVYEY35USF2HJZBLABLYIFQGXZY6",
      "peg": "NGN",
      "can_be_destination": true
    },
    {
      "code": "USDC",
      "issuer": "GA5ZSEJYB37JRC5AVCIA5MOP4RHTM335X2KGX3IHOJAPP5RE34K4KZVN",
      "peg": "USD",
      "can_be_destination": true
    },
    {
      "code": "GHSC",
      "issuer": "GASBV6W7GGED66MXEVC7YZHTWWYMSVYEY35USF2HJZBLABLYIFQGXZY6",
      "peg": "GHS",
      "can_be_destination": true
    },
    {
      "code": "KESC",
      "issuer": "GASBV6W7GGED66MXEVC7YZHTWWYMSVYEY35USF2HJZBLABLYIFQGXZY6",
      "peg": "KES",
      "can_be_destination": true
    }
  ]
```

**cURL:**
```bash
curl -s https://wayfare-cdb9.onrender.com/api/assets
```

**Errors:**
* `405 Method Not Allowed`: Only GET supported.
* `500 Internal Server Error`: Failed to load assets.

---

## GET /api/corridor

**Description:** Measures how much of the destination token you get for a given amount of source token by checking the live Stellar DEX orderbooks. Compares that to the real-world FX mid rate.

### Request

**Headers:**
* `Accept`: application/json

**Query Parameters:**
* `from` (string, optional, default: "USDC") - Asset you send. Must be in KnownCodes: GHSC, KESC, NGNC, USDC.
* `to` (string, optional, default: "NGNC") - Asset you want to receive. Must have a fiat peg.
* `sizes` (string, optional, default: "0.1,1,5,10,25,50,100,250,500,1000,2500,5000") - Comma-separated USDC amounts to test. Example: `sizes=10,100,500`.
* `live` (string, optional) - If set to `1`, forces a live DEX measurement. If omitted, serves last stored measurement from history when available.

### Response

**Success (200 OK):**
```json
{
  "send_asset": {
    "code": "USDC",
    "issuer": "GA5ZSEJYB37JRC5AVCIA5MOP4RHTM335X2KGX3IHOJAPP5RE34K4KZVN"
  },
  "receive_asset": {
    "code": "NGNC",
    "issuer": "GASBV6W7GGED66MXEVC7YZHTWWYMSVYEY35USF2HJZBLABLYIFQGXZY6",
    "peg": "NGN"
  },
  "integrity": "DIRECT",
  "depends_on": [],
  "reference_mid": "1350.753432",
  "reference_source": "exchangerate-api",
  "reference_pair": "USD/NGN",
  "reference_agreement": "AGREE",
  "reference_secondary_mid": "1346.90659134",
  "reference_secondary_source": "currency-api",
  "reference_divergence_pct": "0.2856",
  "scored": true,
  "reference_fetched_at": "2026-08-26T14:47:15Z",
  "floor_loss_pct": "4.31",
  "floor_size": "0.1",
  "worst_loss_pct": "97.23",
  "worst_size": "5000",
  "recommended": {
    "description": "USDC -> XRP -> XLM -> NGNC",
    "source": "stellar-dex",
    "receive_amount": "129.2574648",
    "effective_rate": "1292.574648",
    "loss_pct": "4.31",
    "loss_amount": "5.82",
    "verdict": "FAIR",
    "warnings": [
      "delivers NGNC tokens, not NGN in a bank account; redeeming to fiat is a separate step with its own cost"
    ]
  },
  "recommended_size": "0.1",
  "live": true,
  "measured_at": "2026-08-26T14:47:17Z",
  "finding": "Best available: 4.31% below the exchangerate-api mid at 0.1 USDC, graded FAIR. Loss reaches 97.23% at 5000 USDC.",
  "findings": {
    "checks": [
      {
        "id": "sep10.endpoint-responds",
        "scope": "anchor",
        "subject": "NGNC (GASB\u2026)",
        "severity": "warning",
        "determined": true,
        "passed": false,
        "summary": "the declared SEP-10 endpoint returned HTTP 403 rather than a challenge",
        "evidence": [
          {
            "source": "https://anchor.ngnc.online/auth?account=GAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAWHF",
            "observed": "HTTP 403",
            "observed_at": "2026-08-26T14:47:17Z"
          }
        ],
        "observed_at": "2026-08-26T14:47:17Z"
      },
      {
        "id": "issuer.auth-flags",
        "scope": "asset",
        "subject": "NGNC (GASB\u2026)",
        "severity": "critical",
        "determined": true,
        "passed": true,
        "summary": "the issuer can neither freeze nor claw back this asset",
        "evidence": [
          {
            "source": "https://horizon.stellar.org/accounts/GASBV6W7GGED66MXEVC7YZHTWWYMSVYEY35USF2HJZBLABLYIFQGXZY6 \u2192 flags",
            "observed": "auth_required=false auth_revocable=false auth_clawback_enabled=false auth_immutable=false",
            "observed_at": "2026-08-26T14:47:17Z"
          }
        ],
        "observed_at": "2026-08-26T14:47:18Z"
      }
    ],
    "passed": 3,
    "failed": 2,
    "undetermined": 0,
    "worst_severity": "warning"
  },
  "rungs": [
    {
      "send_amount": "5000",
      "priced": true,
      "integrity": "DIRECT",
      "quote": {
        "description": "USDC -> XLM -> NGNC",
        "source": "stellar-dex",
        "receive_amount": "186947.8515264",
        "effective_rate": "37.38957030528",
        "loss_pct": "97.23",
        "loss_amount": "6566819.31",
        "verdict": "UNUSABLE",
        "warnings": [
          "delivers NGNC tokens, not NGN in a bank account; redeeming to fiat is a separate step with its own cost",
          "thin liquidity: this size gets 96.9% worse pricing than a 10 USDC trade"
        ]
      },
      "cost": {
        "parts": [
          {
            "component": "fx_loss",
            "amount": "6566819.3084736",
            "pct": "97.23194704381251",
            "determined": true
          }
        ],
        "total_loss_pct": "97.23194704381251"
      },
      "notes": [
        "No viable route. The best of 1 priced route(s) still loses 97.2% against the exchangerate-api mid-market rate. Sending through this corridor at this size is not recommended."
      ]
    }
  ]
}
```

**cURL:**
```bash
# History-first (fast, may be stale)
curl -s "https://wayfare-cdb9.onrender.com/api/corridor?from=USDC&to=NGNC"

# Live measurement
curl -s "https://wayfare-cdb9.onrender.com/api/corridor?from=USDC&to=NGNC&live=1"

# Custom sizes
curl -s "https://wayfare-cdb9.onrender.com/api/corridor?from=USDC&to=NGNC&sizes=10,100,500&live=1"
```

**Errors:**
* `400 Bad Request`: Unknown `from` or `to` asset. Body: `{"error": "unknown send asset \"XYZ\"; verified assets are GHSC, KESC, NGNC, USDC"}`
* `400 Bad Request`: No fiat peg for destination. Body: `{"error": "no verified fiat peg for BTC, so there is no independent rate to score it against"}`
* `400 Bad Request`: Invalid sizes format. Body: `{"error": "invalid sizes ..."}`
* `405 Method Not Allowed`: Only GET supported.
* `502 Bad Gateway`: Live measurement failed and no history exists. Body: `{"error": "measuring corridor: no size could be measured; every request failed..."}`
* `504 Gateway Timeout`: Upstream DEX/FX timed out. Returns stale cache if available, otherwise this error.

### Notes

* **What it does:** Tells you "If I send 10 USDC on Stellar, how many NGNC will I actually get?" and "How bad is that compared to the real bank rate?"
* **from / to:** Think of this as a currency pair, but for Stellar tokens. `from` is what you have, `to` is what you want.
* **reference_mid:** The real-world mid rate (e.g. USD/NGN = 1350 from exchangerate-api). This is your benchmark.
* **loss_pct / effective_rate:** How much worse the DEX is than the benchmark. `loss_pct: 4.31` means you lose 4.31% vs the bank rate. Higher is worse.
* **verdict:** FAIR / POOR / UNUSABLE - auto-graded based on loss. UNUSABLE > ~25% loss.
* **rungs:** Same route tested at different sizes. Liquidity gets thin, so 0.1 USDC might be FAIR but 5000 USDC is 97% loss.
* **live vs history:** Without `live=1`, API returns last saved measurement (fast). With `live=1`, it hits the DEX live (slow, 2-5s). If live fails, it falls back to stale history.
* **findings:** Automated security/compliance checks (SEP-10 auth, SEP-24 info, issuer flags). `passed: false` with `severity: warning` means that check failed but corridor still works.
* **integrity:** `DIRECT` means measurement is direct from orderbooks. If stale, it still has same shape.

---

## GET /api/corridor/trend

**Description:** Returns historical corridor measurements over time for charting loss and rate trends. Reads from runstore, oldest first.

### Request

**Headers:**
* `Accept`: application/json

**Query Parameters:**
* `from` (string, optional, default: "USDC") - Send asset.
* `to` (string, optional, default: "NGNC") - Receive asset.
* `limit` (integer, optional, default: 30) - Max records to return. Parsed by `parseTrendLimit`. Example: `limit=7` for last 7 runs.

### Response

**Success (200 OK):**
```json
{
  "corridor": "USDC-NGNC",
  "send_asset": {"code": "USDC", "issuer": "GA5ZSEJYB37JRC5AVCIA5MOP4RHTM335X2KGX3IHOJAPP5RE34K4KZVN"},
  "receive_asset": {"code": "NGNC", "issuer": "GASBV6W7GGED66MXEVC7YZHTWWYMSVYEY35USF2HJZBLABLYIFQGXZY6", "peg": "NGN"},
  "reference_pair": "USD/NGN",
  "count": 1,
  "runs": [
    {
      "seq": 1,
      "recorded_at": "2026-08-22T12:09:59Z",
      "integrity": "DIRECT",
      "reference": {
        "mid": "1349.669672",
        "source": "exchangerate-api",
        "as_of": "2026-08-22T00:02:31Z",
        "divergence_pct": "0.0340"
      },
      "floor_loss_pct": "27.15",
      "worst_loss_pct": "97.52",
      "finding": "No usable size. Loss is 27.15% at 0.1 USDC...",
      "rungs": [
        {"send_amount": "0.1", "priced": true, "loss_pct": "27.15", "verdict": "UNUSABLE"},
        {"send_amount": "5000", "priced": true, "loss_pct": "97.52", "verdict": "UNUSABLE"}
      ]
    }
  ]
}
```

**cURL:**
```bash
curl -s "https://wayfare-cdb9.onrender.com/api/corridor/trend?from=USDC&to=NGNC&limit=30"
curl -s "https://wayfare-cdb9.onrender.com/api/corridor/trend?to=NGNC&limit=7"
```

**Errors:**
* `400 Bad Request`: Unknown asset. `{"error": "unknown send asset \"XYZ\"; verified assets are GHSC, KESC, NGNC, USDC"}`
* `400 Bad Request`: No fiat peg for receive asset.
* `400 Bad Request`: Invalid limit. `{"error": "invalid limit ..."}`
* `405 Method Not Allowed`: Only GET supported.
* `500 Internal Server Error`: `{"error": "reading stored history: ..."}`

### Notes

* **What it does:** Shows history. Instead of one snapshot, you get last N measurements (e.g. last 30 runs). Good for graphing "is NGNC liquidity getting better or worse?"
* **runs[] order:** Oldest first (timeline order). Your chart can plot `recorded_at` on X-axis, `floor_loss_pct` on Y-axis.
* **seq / recorded_at:** `seq` is incrementing ID in runstore. `recorded_at` is when that measurement was taken.
* **floor_loss_pct vs worst_loss_pct:** `floor` = best case (smallest size, 0.1 USDC) — shows structural cost. `worst` = largest size tested. If floor is already 27%, corridor is unusable even for tiny amounts.
* **rungs inside each run:** Simplified to `send_amount`, `loss_pct`, `verdict` for trend view. No full quote/cost to keep payload small.
* **count:** Actual records returned (may be < limit if not enough history).
* **Empty history:** If no records yet, returns `{"corridor":"USDC-NGNC","count":0,"runs":[]}` with 200 OK, not an error.
* **reference:** Same FX benchmark as corridor endpoint, but snapshot at that time. `divergence_pct` tells you if two FX sources disagreed that day.

---

## GET /healthz

**Description:** Liveness probe. Returns ok if server is running. No dependencies checked.

### Request

**Headers:**
* None

**Query Parameters:**
* None

### Response

**Success (200 OK):**
```json
{
  "status": "ok"
}
```

**cURL:**
```bash
curl -s https://wayfare-cdb9.onrender.com/healthz | jq
curl -s http://localhost:8080/healthz | jq
```

**Errors:**
* None. Always 200 if process is up. If server is down, connection fails.

### Notes
* Use for load balancer / Render health checks. Should be cheap and unauthenticated.

---