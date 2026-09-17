# Reed — listing checklist (skills.sh / ClawHub)

Owner: Head of Product. Atlas drafted the skills; you own public listing copy and “ship” call.

## Already done

- Public GitHub repo: `maticijus/hexenkraft-agent-skills`
- Two skills with trigger-oriented descriptions and live curl/MCP recipes
- Checkout path remains human Stripe URL (founder risk posture)

## Your steps

1. Skim both `skills/*/SKILL.md` descriptions — trigger phrases must match how agencies ask.
2. Confirm prices still match live `/api/offers.json` (skills say “re-read offers”).
3. Publish / verify on [skills.sh](https://skills.sh) via the GitHub repo (or `npx skills` docs current path).
4. Optional: mirror listing on ClawHub if you want OpenClaw/Hermes traffic.
5. Add one line each to inkluso + cybiq `llms.txt` / for-agents pointing at install command once listed.
6. Ping Atlas for Hetzner deploy of any llms/for-agents copy changes (SHIP).

## Install commands (after GitHub is public)

```bash
npx skills add maticijus/hexenkraft-agent-skills --skill inkluso-eaa-scan
npx skills add maticijus/hexenkraft-agent-skills --skill cybiq-cra-scope
```

## Do not

- Put full CRA/EAA statute text in the skills
- Enable x402 or API-key metering in v1 listing
- Claim full legal compliance from automated scan/classify
