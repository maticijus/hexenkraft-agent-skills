---
name: cybiq-cra-scope
description: >
  Decide whether the EU Cyber Resilience Act (Regulation (EU) 2024/2847) applies
  to a product with digital elements, which conformity route applies, and which
  obligations/dates matter. Use for CRA scope, Annex III/IV class, manufacturer
  vs importer duties, or buying the €49/€199 document pack. Calls Cybiq API/MCP.
  Not legal advice.
---

# Cybiq CRA scope check

Operator: HEXENKRAFT s.r.o. (IČO 29665931). Product site: https://cybiq.eu

## When to use

- “Does the Cyber Resilience Act apply to this product?”
- Conformity route / Annex III Class I–II / Annex IV critical
- Manufacturer, importer, or distributor obligations
- Key dates (reporting duties, full application)
- Buy verification checklist (€49) or compliance pack (€199)

## When not to use

- Asking the model to invent CRA law text (use Cybiq; every claim is OJ-anchored server-side)
- Treating the result as legal advice or a notified-body certificate
- Paying / waiving withdrawal for the human

## Discovery (read first)

| Resource | URL |
|---|---|
| llms.txt | https://cybiq.eu/llms.txt |
| OpenAPI | https://cybiq.eu/openapi.json |
| Offers | https://cybiq.eu/api/offers.json |
| Decisions dictionary (EN) | https://cybiq.eu/api/decisions.json |
| Decisions dictionary (DE) | https://cybiq.eu/api/decisions.de.json |
| MCP card | https://cybiq.eu/.well-known/mcp.json |
| MCP endpoint | https://cybiq.eu/mcp (Streamable HTTP, POST only, no auth) |
| Human guide | https://cybiq.eu/for-agents.html |

## Auth

None for classify. Pack purchase = Stripe Checkout URL via MCP; **human** pays.

## Tools

MCP tools: `classify_cra`, `start_pack_checkout`, `check_order_status`

### 1) Classify — `classify_cra` / `POST /api/classify`

Deterministic. Partial answers return `verdict: "needs_review"` and list what is missing (no guessing). Unknown keys/values → 422.

```bash
curl -sS -X POST https://cybiq.eu/api/classify \
  -H 'Content-Type: application/json' \
  -d '{
    "locale":"en",
    "answers":{
      "market":"commercial",
      "ossMonetized":false,
      "productKind":"hardware_with_software",
      "otherRegime":"none",
      "role":"manufacturer",
      "category":"network_device"
    }
  }'
```

`answers` fields (any subset allowed):

| Field | Values |
|---|---|
| `market` | `commercial` \| `internal_only` \| `free_oss` |
| `ossMonetized` | boolean (only meaningful when `market` is `free_oss`) |
| `productKind` | `installable_software` \| `hardware_with_software` \| `standalone_hardware` \| `pure_saas` |
| `otherRegime` | `none` \| `medical_devices` \| `in_vitro_diagnostics` \| `aviation` \| `marine_equipment` \| `road_vehicles` \| `high_risk_ai` |
| `role` | `manufacturer` \| `importer` \| `distributor` |
| `category` | see category enum below |

`locale`: `en` | `de` (prose language only; legal anchors unchanged).

**Category enum:** `general_app`, `browser`, `password_manager`, `identity_access_management`, `anti_malware`, `vpn`, `network_management`, `siem`, `boot_manager`, `pki_certificate_issuance`, `operating_system`, `security_microprocessor_microcontroller`, `smart_home_assistant_or_security`, `connected_toy`, `health_wearable`, `network_device`, `network_critical_infrastructure`, `firewall_ids_ips`, `tamper_resistant_chip`, `hypervisor_container_runtime`, `hsm_secure_element`, `industrial_control`, `other`

Offline option: map answers via `GET /api/decisions.json` (large). Prefer `POST /api/classify` for a single product.

### 2) Buy pack — `start_pack_checkout` (MCP)

Tiers from offers (re-read before quoting):

| plan | price | use |
|---|---|---|
| `verification_checklist_49` | €49 | importer/distributor verification checklist |
| `compliance_pack_199` | €199 | manufacturer compliance pack (templates/indexes; Cybiq does not file for you) |

Call MCP tool `start_pack_checkout` with the tier. Hand the returned Checkout URL to the human. Do not open/pay it as the agent. Poll `check_order_status` with `session_id`; do not fetch one-time unlock URLs yourself.

Idempotency: if the tool accepts `idempotency_key`, reuse it for retries of the same tier/locale.

### 3) Optional MCP JSON-RPC shape

```bash
curl -sS -X POST https://cybiq.eu/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{
    "jsonrpc":"2.0",
    "id":1,
    "method":"tools/call",
    "params":{
      "name":"classify_cra",
      "arguments":{
        "locale":"en",
        "answers":{"market":"commercial","productKind":"installable_software","role":"manufacturer","category":"general_app","otherRegime":"none"}
      }
    }
  }'
```

(If the server expects an initialize handshake first, follow the MCP Streamable HTTP sequence from the OpenAPI `/mcp` description.)

## Errors

Failures are RFC 9457 problem+json when applicable. Read `code` when present.

| HTTP | Meaning |
|---|---|
| 400 | Not JSON / not an object |
| 422 | Unknown answer key/value or field |
| 429 | Rate limit |
| 413 | Body > 64KB |

## Output to the human

1. Verdict + conformity route + key obligations with OJ citations from the response.
2. Explicit: **not legal advice**; Cybiq does not file with authorities.
3. If they need templates: €49 or €199 checkout link for human payment.
4. Mention reporting-duty / application dates only as returned by the API (don’t invent dates).

## Do not

- Paste long CRA excerpts into the skill or chat; call the API
- Silently drop unknown answers
- Claim notified-body certification from a classify result
