# HEXENKRAFT agent skills

Thin `SKILL.md` packages that teach coding agents when and how to call the **live** Inkluso and Cybiq HTTP/MCP surfaces.

These skills wrap what already ships in production. They do **not** invent new APIs.

| Skill | Product | Install |
|---|---|---|
| `inkluso-eaa-scan` | [inkluso.eu](https://inkluso.eu) EAA/WCAG | `npx skills add maticijus/hexenkraft-agent-skills --skill inkluso-eaa-scan` |
| `cybiq-cra-scope` | [cybiq.eu](https://cybiq.eu) CRA scope | `npx skills add maticijus/hexenkraft-agent-skills --skill cybiq-cra-scope` |

## What agents get

- Free classify/scan over JSON
- MCP tool names and REST equivalents
- Honest limits (automation ≠ full legal compliance; not legal advice)
- Paid packs/audits via **human** Stripe Checkout URLs (no agent wallet / x402 yet)

## Source of truth

Always prefer live discovery over this repo if they diverge:

- https://inkluso.eu/llms.txt · https://inkluso.eu/openapi.json · https://inkluso.eu/.well-known/mcp.json
- https://cybiq.eu/llms.txt · https://cybiq.eu/openapi.json · https://cybiq.eu/.well-known/mcp.json

## Operator notes

See [REED-LISTING.md](./REED-LISTING.md) for skills.sh / ClawHub listing checklist.
