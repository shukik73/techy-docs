# Lead SLA Guardian — Setup Guide

## What It Does

1. **New lead arrives** (via webhook from HubSpot, RepairDesk, website form, or any source)
2. **Kai drafts an SMS response** using Claude (under 300 chars, acknowledges their device)
3. **Pings Shuki on Telegram** with lead details + draft + action buttons
4. **If no response in 10 min** → escalation alert with estimated revenue loss

Trust level: **L1 (Suggest Only)** — Kai drafts but never sends. Shuki decides.

## Setup Steps

### 1. Import Workflows

Import both JSON files into n8n:
- `workflows/lead-sla-guardian.json` (main workflow)
- `workflows/lead-sla-escalation.json` (10-min escalation)

### 2. Configure Credentials

Replace these placeholders in both workflows:

| Placeholder | Where to Get |
|-------------|-------------|
| `ANTHROPIC_CREDENTIAL_ID` | n8n → Credentials → Add → HTTP Header Auth → `x-api-key: sk-ant-...` + base URL `https://api.anthropic.com/v1` |
| `YOUR_TELEGRAM_CHAT_ID` | Message @userinfobot on Telegram |
| `YOUR_TELEGRAM_CREDENTIAL_ID` | n8n → Credentials → Add → Telegram Bot → paste token from @BotFather |
| `YOUR_N8N_DOMAIN` | Your n8n instance URL (e.g., `n8n.srv1155599.hstgr.cloud`) |

### 3. Claude API Note

The "Draft Response" node uses the n8n OpenAI node pointed at Anthropic's API.
In n8n credential config:
- **Base URL**: `https://api.anthropic.com/v1`
- **API Key**: Your Anthropic key
- **Model**: `claude-sonnet-4-20250514`

Alternatively, use an HTTP Request node with raw Anthropic API call if the OpenAI node doesn't support custom base URLs in your n8n version.

### 4. Connect Lead Sources

Send a POST to your webhook URL with this payload format:

```bash
curl -X POST https://YOUR_N8N_DOMAIN/webhook/lead-sla-guardian \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John Smith",
    "phone": "+13055551234",
    "email": "john@example.com",
    "device": "iPhone 15 cracked screen",
    "message": "How much to fix a cracked screen?",
    "source": "google_ads"
  }'
```

#### HubSpot Integration
In HubSpot → Automation → Workflows:
- Trigger: "Contact is created"
- Action: Webhook → POST to `https://YOUR_N8N_DOMAIN/webhook/lead-sla-guardian`
- Map: `firstname` → `name`, `phone` → `phone`, etc.

#### RepairDesk Integration
In RepairDesk → Settings → Webhooks:
- Event: "New Ticket Created"
- URL: `https://YOUR_N8N_DOMAIN/webhook/lead-sla-guardian`

#### Website Form
Point your contact form's action to the webhook URL.

### 5. Test

```bash
curl -X POST https://YOUR_N8N_DOMAIN/webhook/lead-sla-guardian \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Test Lead",
    "phone": "+13055550000",
    "device": "MacBook Pro water damage",
    "message": "Spilled coffee on my MacBook, won't turn on",
    "source": "test"
  }'
```

You should receive a Telegram message within 5-10 seconds with the draft response.

### 6. Track Performance

Every intervention is logged. After 15 accepted suggestions, Kai can be promoted to L2 (Draft mode — creates artifacts but still doesn't send).

Track in `proactive/interventions.jsonl`:
- `user_response: "accepted"` → Shuki sent the draft
- `user_response: "modified"` → Shuki edited then sent
- `user_response: "rejected"` → Shuki skipped
- `user_response: "annoyed"` → Trust ramp demotion

## Cost Per Lead

- Claude Sonnet draft: ~200 tokens → ~$0.001
- Well under the $0.05 per-trigger budget in `triggers.yaml`
