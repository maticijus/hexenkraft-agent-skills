---
name: inkluso-eaa-scan
description: >
  Audit a public website for EU EAA / WCAG 2.1 AA / national law (CZ 424/2023,
  DE BFSG, PL ustawa 26.04.2024). Use when the user needs accessibility findings,
  a statement draft, a Shoptet e-shop check, or a paid audit/monitoring checkout
  link. Calls the live Inkluso API/MCP. Not a substitute for a full legal audit.
---

# Inkluso EAA / WCAG scan

Operator: HEXENKRAFT s.r.o. (IČO 29665931). Product site: https://inkluso.eu

## When to use

- “Does this site meet EAA / WCAG 2.1 AA?”
- Czech Act 424/2023, German BFSG, Polish accessibility act mapping
- Draft an accessibility statement
- Shoptet e-shop accessibility audit
- Start paid audit/monitoring (returns a Stripe URL for a **human** to pay)

## When not to use

- Private/intranet URLs (API rejects non-public hosts with 422)
- Claiming full legal compliance from an automated scan alone
- Sending `consent: true` on checkout without the human’s real agreement

## Discovery (read first)

| Resource | URL |
|---|---|
| llms.txt | https://inkluso.eu/llms.txt |
| OpenAPI | https://inkluso.eu/openapi.json |
| Offers | https://inkluso.eu/api/offers.json |
| MCP card | https://inkluso.eu/.well-known/mcp.json |
| MCP endpoint | https://inkluso.eu/mcp (Streamable HTTP, POST only, no auth) |
| Human guide | https://inkluso.eu/for-agents |

## Auth

None for free tools. Paid purchase = Stripe Checkout URL; the **human** pays. Do not tick withdrawal waiver / consent for them.

## Tools (prefer MCP; REST equivalents below)

MCP tools: `scan_accessibility`, `draft_accessibility_statement`, `start_audit_checkout`, `check_order_status`

### 1) Free page scan — `scan_accessibility` / `POST /api/scan/free`

Homepage teaser scan (~30–60s). Rate limit ~5/hour/IP.

```bash
curl -sS -X POST https://inkluso.eu/api/scan/free \
  -H 'Content-Type: application/json' \
  --max-time 90 \
  -d '{"url":"https://example.cz","lang":"cs","source":"skill-inkluso-eaa-scan"}'
```

Body:
- `url` (required) — public https URL; scheme optional (https assumed)
- `lang` — `cs` | `de` | `pl` | `en` (default `cs`; selects national law cited)
- `source` — optional attribution tag (≤64 chars)

On 200, read `scan_status`: only `"ok"` means the page was assessed. Other values (e.g. bot challenge) mean findings are incomplete.

**Honesty rule:** Automated scans catch a minority of WCAG issues. Say that. Do not invent “fully compliant.”

### 2) Statement draft — `draft_accessibility_statement` / `POST /api/statement/draft`

```bash
curl -sS -X POST https://inkluso.eu/api/statement/draft \
  -H 'Content-Type: application/json' \
  -d '{
    "organisation":"Example s.r.o.",
    "url":"https://example.cz",
    "lang":"cs",
    "conformance_status":"partial",
    "contact_email":"a11y@example.cz",
    "known_issues":["Two pre-2024 PDFs are not tagged."]
  }'
```

`conformance_status`: `partial` | `none` | `not_assessed` only (`full` is rejected).

### 3) Paid checkout — `start_audit_checkout` / `POST /api/billing/checkout`

Returns a Stripe Checkout URL. Human opens it and pays.

```bash
curl -sS -X POST https://inkluso.eu/api/billing/checkout \
  -H 'Content-Type: application/json' \
  -H 'Idempotency-Key: YOUR-UNIQUE-KEY' \
  -d '{
    "email":"buyer@example.cz",
    "tier":"scan_report",
    "url":"https://example.cz",
    "lang":"cs",
    "consent":true,
    "source":"skill-inkluso-eaa-scan"
  }'
```

`tier` values: `scan_report` | `shoptet_audit` | `monitor` | `monitor_annual` | `rescan` | `agency`

Indicative prices (always re-read `/api/offers.json` before quoting):

| tier | cs | de/en | pl |
|---|---|---|---|
| scan_report | 3 590 Kč | €149 | 649 zł |
| shoptet_audit | 2 490 Kč | €99 | 449 zł |
| monitor | 1 190 Kč/mo | €49/mo | 219 zł/mo |
| monitor_annual | 11 900 Kč/yr | €490/yr | 2 190 zł/yr |
| rescan | 1 190 Kč | €49 | 219 zł |
| agency | floor via API — prefer sales-led | | |

`consent` must be `true` only after the human agrees to immediate digital delivery / withdrawal waiver. Agents must not fabricate consent.

### 4) Order status — `check_order_status` / `GET /api/orders/{ref}`

Poll until fulfilled. 202 = pending; 200 = `report_url`; 404 = stop.

## Errors

| HTTP | Meaning |
|---|---|
| 422 | Bad URL / validation / consent false |
| 429 | Rate limit |
| 503 | Scanner busy |
| 504 | Scan wall-clock timeout |

Prefer machine fields (`scan_status`, problem `code` if present) over parsing HTML.

## Output to the human

1. JSON findings summary (score/grade, top violations, national law refs).
2. Clear limit: automated ≠ full EAA conformity assessment.
3. If they want a filed PDF / multi-page audit: checkout link + ask them to pay.
4. Optional statement draft text.

## Do not

- Double-create checkout without a new Idempotency-Key
- Scan localhost / private IPs
- Over-claim compliance
