# Jay Agent — Setup Guide
## Ready to paste when Shuki arrives at the store

---

## Step 1: Create Jay's Telegram Bot

1. Open Telegram → search **@BotFather**
2. Send `/newbot`
3. Name: `Jay - Squad Lead` (or whatever you prefer)
4. Username: `jay_squadlead_bot` (must be unique, add numbers if taken)
5. **Save the bot token** BotFather gives you (looks like `1234567890:ABCdefGHIjklMNOpqrSTUvwxYZ`)

---

## Step 2: Install OpenClaw for Jay

You can run Jay on the same MacBook or on a separate machine. Same machine is easiest to start:

```bash
# Create Jay's config directory
mkdir -p ~/.openclaw-jay

# Initialize Jay as a separate OpenClaw instance
openclaw configure --config ~/.openclaw-jay/openclaw.json
```

**During setup, choose:**
- Model: `anthropic/claude-sonnet-4-20250514` (cheaper than Opus for execution work)
- Channel: Telegram
- Paste Jay's bot token when asked

**OR** — simpler option: Add Jay as a second agent in your existing OpenClaw config. Run:
```bash
openclaw configure
```
And add a second agent with Jay's Telegram bot token.

---

## Step 3: Paste Jay's System Files

Jay's SOUL.md, IDENTITY.md, USER.md, AGENTS.md, and TOOLS.md are already prepared in:
```
~/.openclaw/workspace/agents/jay/
```

Copy these into Jay's workspace when his instance is running.

---

## Step 4: Create the Group Chat

1. Open Telegram → New Group
2. Name: **"Squad HQ"** (or your preference)
3. Add: Kai bot + Jay bot + yourself
4. Both bots need **admin permissions** to read group messages

---

## Step 5: Tell Kai

Once Jay is online in the group, Kai will brief him on:
- Current priorities and open items
- Parts Scout agent spec
- Lister agent spec
- Security fixes for Iron Secretary
- Chain of command

---

## Jay's Core Config Summary

**Name:** Jay
**Role:** Squad Lead — operational execution
**Reports to:** Kai (Chief of Staff)
**Manages:** Worker agents (future)
**Tone:** Direct, action-first, no fluff
**Model recommendation:** Claude Sonnet 4 (cost-efficient for execution tasks)

**Key behaviors:**
- Executes without waiting for instructions
- Escalates to Kai with recommendation (never just a question)
- Filters everything through Repair Shop revenue impact
- Logs everything

---

## Jay's First Briefing Checklist (Kai will deliver)

1. ✅ Chain of command: Shuki → Kai → Jay → Workers
2. ✅ Four businesses overview
3. ✅ Current priorities:
   - Iron Secretary security fixes (2 blockers)
   - Parts Scout agent spec (buy-side)
   - Lister agent spec (sell-side)
   - Reddit engagement for Iron Secretary
   - Missed Call Calculator landing page
4. ✅ Open items from Feb 13
5. ✅ Access to n8n instance
6. ✅ Access to GitHub (shukik73/techy-docs)
