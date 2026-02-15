# COGNITIVE BELIEF REPOSITORY v1.0

## NORTH STAR [Identity Zone — user-approved changes only]

**Objective**: Hit $250K annual net profit for Techy Miramar through repair services + AI-powered SaaS products.

**Timeline**: Rolling annual target. Current tracking period: 2026.

**Key Results**:
- Launch ReviewGuard SaaS to 10 paying customers
- Automate 80% of repair shop operations via n8n workflows
- Establish Hamoriko & Friends as consistent bilingual content pipeline
- Reduce infrastructure costs by migrating eligible workloads to local MacBook M1s

## DYNAMIC USER MODEL

- **Role**: Owner/Operator at Techy Miramar + Technical Architect
- **High Energy**: 09:00-12:00 (deep technical), 20:00-23:00 (side projects)
- **Risk Profile**: High technical risk tolerance, moderate financial, low tolerance for UI bugs
- **Decision Style**: Prefers comprehensive automation; systematic problem-solver; wants documentation over tribal knowledge

## ACTIVE BELIEFS (high confidence) [Skill Zone]

Beliefs are stored in `memory/beliefs.jsonl`. This section summarizes the highest-confidence items:

- Parts sourcing: Dell->parts-people.com, phones->Injured Gadgets/Mobile Sentrix, MacBook->ProTech Restore, screens->LaptopScreen.com [frozen]
- Avoid: GBOLE, GinTai on Amazon [frozen]
- Screen lookup uses panel part number from LCD back, NOT laptop model [frozen]
- Dell uses Service Tag for parts lookup [frozen]
- FB Marketplace for whole devices to harvest parts or refurbish, not for individual parts [frozen]
- Clawdbot gateway lock: `/tmp/clawdbot-1001/gateway.820017cb.lock` — kill process + remove lock
- Brand colors: deep navy #0A2A4A, cyan-blue #06B0FD

## PROCEDURAL SKILLS (learned) [Skill Zone]

- **Parts Sourcing Analysis**: Compare vendor price vs eBay whole-device harvesting. If whole device < 1.5x part cost with usable remaining parts, recommend whole device.
- **n8n Workflow Building**: Identify trigger -> map data flow -> select nodes -> build JSON -> test -> deploy via API.
- **Iron Secretary Call Analysis**: Score calls against "Perfect Phone Call" criteria, integrate with RepairDesk (include X-Idempotency-Key header).

## EPISODIC LESSONS [Skill Zone]

- `ep-001`: User rejects proactive coding interventions during focus sessions. **Remedy**: Increase proactive threshold by 20% during high-energy windows.
- `ep-002`: RepairDesk webhook integration requires idempotency key. **Remedy**: Always include X-Idempotency-Key header.
- `ep-003`: Buying "for parts" MacBooks from FB Marketplace often more economical than individual part sourcing when 3+ parts are usable.

## LEAD SOURCES [Skill Zone]

Primary lead intake channels (for trigger signal routing):
- **RepairDesk**: New ticket webhook -> n8n lead-sla-guardian
- **HubSpot**: Contact created workflow -> n8n lead-sla-guardian
- **Website form**: Direct POST to n8n lead-sla-guardian webhook
- **Google Ads**: Routed through website form or HubSpot

All lead sources normalize to the same webhook payload format. See `workflows/lead-sla-guardian.json` for the normalizer.
