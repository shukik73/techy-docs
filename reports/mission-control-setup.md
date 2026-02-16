# Mission Control — Setup Plan
## Status: TODO

### What Is It
A web dashboard to manage, monitor, and dispatch tasks to AI agents (Kai, Jay, future workers). Visual command center instead of just Telegram.

### Best Option: crshdn/mission-control
- GitHub: https://github.com/crshdn/mission-control
- Stack: Next.js 15 + SQLite + WebSocket to OpenClaw Gateway
- Features: Create tasks, AI-guided planning, auto-dispatch to agents, delivery tracking
- License: MIT

### Alternative: ClawDeck (clawdeck.io)
- Open source dashboard built specifically for OpenClaw
- Worth evaluating alongside crshdn

### What We Need
1. Node.js v18+ (already have v25.6.1 ✅)
2. OpenClaw Gateway running (already running ✅)
3. Gateway token: `7326cdac...` (already have ✅)
4. Gateway URL: `ws://127.0.0.1:18789` ✅

### Install Steps
```bash
# Clone
git clone https://github.com/crshdn/mission-control.git
cd mission-control
npm install

# Configure
cat > .env.local << EOF
OPENCLAW_GATEWAY_URL=ws://127.0.0.1:18789
OPENCLAW_GATEWAY_TOKEN=7326cdac165a7335396dd66da54aeb72a2d3ed60e8f7a893
PORT=3000
EOF

# Run
npm run dev
```

Then open http://localhost:3000

### Considerations
- Currently only works when MacBook is on (local gateway)
- For remote access: need Tailscale or expose via VPS
- Can run on one of the gaming laptops later for always-on access
- Should evaluate ClawDeck too before committing

### Dependencies
- [ ] Shuki approval to install
- [ ] Decide: crshdn/mission-control vs ClawDeck
- [ ] Evaluate if either supports multi-agent view (Kai + Jay)
