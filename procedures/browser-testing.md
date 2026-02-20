# Browser Testing Doctrine

> How to set up and operate browser testing agents for automated visual testing.

---

## Agent Architecture

Each testing agent is an isolated environment consisting of:
- **Git worktree** - Isolated copy of the codebase
- **Dev server** - Running on a unique port
- **Browser instance** - Chrome with remote debugging enabled

This isolation allows parallel testing without interference.

---

## Setting Up an Agent

### Prerequisites

- Chrome/Chromium installed
- Project supports running on custom ports
- `agent-browser` CLI available (`npx agent-browser` or installed globally)

### Single Agent Setup

```bash
# Variables - customize for your project
PROJECT_DIR="/path/to/your/project"
AGENT_NUM=1
CHROME_PORT=$((9222 + AGENT_NUM))
DEV_PORT=$((3010 + AGENT_NUM))

# 1. Create worktree
cd $PROJECT_DIR
git worktree add .worktrees/agent-$AGENT_NUM -b agent-$AGENT_NUM

# 2. Install dependencies (if needed)
cd .worktrees/agent-$AGENT_NUM
npm install  # or yarn, pnpm, etc.

# 3. Start dev server (disable HMR for stable screenshots)
NO_HMR=true PORT=$DEV_PORT npm run dev &

# 4. Start Chrome with remote debugging
google-chrome-stable \
  --remote-debugging-port=$CHROME_PORT \
  --user-data-dir=/tmp/chrome-agent-$AGENT_NUM \
  --no-first-run \
  --no-default-browser-check \
  --disable-sync \
  --disable-extensions &
```

### Multiple Agents Setup

```bash
PROJECT_DIR="/path/to/your/project"
NUM_AGENTS=5

cd $PROJECT_DIR

# Create all worktrees
for i in $(seq 1 $NUM_AGENTS); do
  git worktree add .worktrees/agent-$i -b agent-$i
done

# Start all dev servers
for i in $(seq 1 $NUM_AGENTS); do
  cd $PROJECT_DIR/.worktrees/agent-$i
  NO_HMR=true PORT=$((3010 + i)) npm run dev &
done

# Start all Chrome instances
for i in $(seq 1 $NUM_AGENTS); do
  google-chrome-stable \
    --remote-debugging-port=$((9222 + i)) \
    --user-data-dir=/tmp/chrome-agent-$i \
    --no-first-run \
    --no-default-browser-check \
    --disable-sync \
    --disable-extensions &
done
```

---

## Chrome Flags Explained

| Flag | Purpose |
|------|---------|
| `--remote-debugging-port=XXXX` | Enables CDP (Chrome DevTools Protocol) for automation |
| `--user-data-dir=/tmp/chrome-agent-N` | Isolated profile - no cross-contamination between agents |
| `--no-first-run` | Skip "Welcome to Chrome" dialogs |
| `--no-default-browser-check` | Skip "Set as default browser?" prompts |
| `--disable-sync` | No Google account sync interference |
| `--disable-extensions` | Clean environment, no extension side effects |

---

## Verification

```bash
# Check Chrome instances are running
for p in 9223 9224 9225; do
  curl -s http://127.0.0.1:$p/json/version > /dev/null && echo "Chrome $p: UP" || echo "Chrome $p: DOWN"
done

# Check dev servers are running
for p in 3011 3012 3013; do
  curl -s http://127.0.0.1:$p > /dev/null && echo "Server $p: UP" || echo "Server $p: DOWN"
done
```

---

## DO's and DON'Ts

### DO

- **Disable HMR/Hot Reload** - Hot module replacement causes inconsistent screenshots
- **Use isolated user data dirs** - Each agent needs its own Chrome profile
- **Use unique ports** - No collisions between agents or main dev server
- **Wait for page stability** - After navigation, wait for network idle before screenshots
- **Use consistent viewport sizes** - Set viewport before taking screenshots
- **Clear state between tests** - localStorage, cookies, etc.

### DON'T

- **Don't share Chrome profiles** - Cross-contamination breaks isolation
- **Don't use your daily browser** - Extensions and settings will interfere
- **Don't skip the remote debugging flag** - MCP tools need CDP access
- **Don't take screenshots during animations** - Wait for completion
- **Don't ignore network requests** - Wait for all XHR/fetch to complete
- **Don't hardcode absolute paths** - Use relative paths or variables

---

## Browser Interaction - agent-browser CLI

Use `agent-browser` CLI (via Bash) for all browser automation. No MCP servers needed.

```bash
# Core workflow
agent-browser --session agent-N open <url>       # Navigate
agent-browser --session agent-N snapshot -i       # Get element refs (@e1, @e2...)
agent-browser --session agent-N click @e1         # Click
agent-browser --session agent-N fill @e2 "text"   # Fill input
agent-browser --session agent-N screenshot        # Capture
agent-browser --session agent-N wait --load networkidle  # Wait for load
```

| Command | Purpose |
|---------|---------|
| `open <url>` | Navigate to URL |
| `snapshot -i` | Get interactive element refs |
| `click @eN` | Click element |
| `fill @eN "text"` | Enter text in input |
| `select @eN "opt"` | Select dropdown option |
| `get text @eN` | Extract text content |
| `get url` | Get current URL |
| `screenshot` | Capture screenshot |
| `wait --load networkidle` | Wait for network idle |
| `close` | Close browser session |

**Refs invalidate on navigation/DOM changes — always re-snapshot after.**

---

## Cleanup

```bash
PROJECT_DIR="/path/to/your/project"
NUM_AGENTS=5

cd $PROJECT_DIR

# Kill Chrome instances
pkill -f "chrome.*remote-debugging-port=922"

# Kill dev servers (adjust pattern for your setup)
pkill -f "npm run dev"

# Remove worktrees
for i in $(seq 1 $NUM_AGENTS); do
  git worktree remove .worktrees/agent-$i --force
  git branch -D agent-$i
done
```

---

## Troubleshooting

### Chrome won't start
- Check if port is already in use: `lsof -i :9223`
- Remove stale user data: `rm -rf /tmp/chrome-agent-*`

### agent-browser can't connect
- Verify Chrome is running: `curl http://127.0.0.1:9223/json/version`
- Check that SSH tunnels are up (Chrome debug ports tunneled from Lappy)

### Screenshots are inconsistent
- Disable HMR: `NO_HMR=true`
- Wait for network idle before screenshot
- Set explicit viewport size
- Disable animations in test mode

### Dev server won't start
- Check if port is in use: `lsof -i :3011`
- Verify worktree has node_modules (may need `npm install`)
