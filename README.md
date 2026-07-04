# cua-desktop-automation-skills

<div align="center">

[![GitHub stars](https://img.shields.io/github/stars/ChrisLamDev/cua-desktop-automation-skills?style=for-the-badge&logo=github)](https://github.com/ChrisLamDev/cua-desktop-automation-skills/stargazers)
[![GitHub forks](https://img.shields.io/github/forks/ChrisLamDev/cua-desktop-automation-skills?style=for-the-badge&logo=github)](https://github.com/ChrisLamDev/cua-desktop-automation-skills/forks)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](https://opensource.org/licenses/MIT)
[![Skills](https://img.shields.io/badge/Skills-9-8A2BE2?style=for-the-badge)](skills/)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-ready-8A2BE2?style=for-the-badge&logo=anthropic)](https://claude.ai)
[![OpenAI Codex](https://img.shields.io/badge/OpenAI%20Codex-ready-8A2BE2?style=for-the-badge&logo=openai)](https://openai.com)
[![Cursor](https://img.shields.io/badge/Cursor-ready-8A2BE2?style=for-the-badge&logo=cursor)](https://cursor.sh)
[![Hermes Agent](https://img.shields.io/badge/Hermes%20Agent-ready-8A2BE2?style=for-the-badge&logo=python)](https://github.com/NousResearch/hermes-agent)
[![Star History Chart](https://api.star-history.com/svg?repos=ChrisLamDev/cua-desktop-automation-skills&type=Date)](https://star-history.com/#ChrisLamDev/cua-desktop-automation-skills&Date)

</div>

**Cua Desktop Automation — 9 executable skills with a self-learning task router for reliable macOS GUI automation.**

## 📸 Demo

<div align="center">
  <img src="screenshots/demo.svg" alt="cua-desktop-automation-skills overview" width="100%">
  <br><br>
  <img src="screenshots/demo-real.png" alt="cua-desktop-automation-skills live demo" width="90%" style="border-radius: 8px; border: 1px solid #30363d;">
  <br>
  <em>Cua Desktop Automation live dashboard — Router Learning System scanning macOS desktop with 9 executable skills</em>
</div>

<br>



A collection of battle-tested skills and examples for building computer-use agents that control macOS desktops via Cua Driver — with an intelligent task router that learns from past executions.

## 💡 Why This Project?

**Desktop automation is fragile.** A single pixel shift, a renamed button, or a browser update can break an entire automation script. Most tools rely on hard-coded selectors, fixed coordinates, or brittle accessibility tree crawls — and when they break, they break silently.

**AI agents need a router that learns from failure.** Instead of guessing which tool to use, the router records what worked (and what didn't) across four execution tiers — CLI commands, browser CDP, pixel-level clicks, and vision-based fallback. Over time, it gets faster, smarter, and more resilient, automatically skipping strategies that have failed before and prioritizing the ones that actually work.

The result: automation that **survives UI changes** and **improves with every run** — no manual retraining required.

## 🌟 Star Feature: Router Learning System

The `examples/router.py` is a self-improving task routing engine that decides **what tool to use** based on past success:

```
Tier 1 — CLI/Git (fastest, no GUI needed)
Tier 2 — CDP WebSocket (direct browser DOM manipulation)
Tier 3 — Cua Driver (pixel clicks, keystrokes, AX tree)
Tier 4 — Vision AI (screenshot analysis for fallback)
```

### How it works

```python
from examples.router import do

# Route + auto-learn — the more you use it, the smarter it gets
result = do("git_commit", args={"message": "feat: ..."})
# → Router checks history, picks best path, records result
```

**Key capabilities:**
- 🧠 **Dynamic path sorting** — reorders tiers by historical success rate + execution time
- 📊 **SQLite history** (`~/.hermes/data/router_history.db`) — persistent learning across sessions
- 🔄 **Context-aware fallback** — auto-escalates through Tier 1 → 2 → 3 → 4 on failure
- ⏭️ **Consecutive failure skip** — skips paths that failed 3+ times in a row
- 🏆 **Cold start → warm** — static tier order initially, self-optimizes after 5+ samples

### Built-in task routes

| Task | Available paths |
|------|----------------|
| `create_github_token` | Device code flow → CDP browser → Safari AX |
| `browser_navigate` | Cua hotkey (Cmd+L) |
| `browser_fill_input` | CDP JS → Cua type_text |
| `browser_click_button` | CDP JS → Cua pixel click |
| `app_hotkey` | Cua hotkey |
| `app_type` | Cua type_text |
| `app_click` | Cua pixel click |
| `app_screenshot` | screencapture |
| `git_commit` | git CLI |
| `git_push` | git CLI |

## 📋 Real-World Scenarios

These scenarios show how the Router Learning System picks the optimal path — and escalates intelligently when things go wrong.

### Scenario 1: Automated GitHub Release

**Pain point:** Releasing software involves multiple steps — version bump, changelog update, git tag, push, and GitHub Release creation. One failed credential or network hiccup and the whole pipeline stalls.

```python
# Router decides: Tier 1 (CLI git) first — fastest path
do("git_commit", args={"message": "chore: bump v1.2.3"})  # ✅ CLI succeeds

# But if git push fails due to auth...
do("git_push")  # ❌ CLI auth error
# Router auto-escalates: Tier 3 (Cua click → Safari browser)
# → Opens GitHub.com in Safari, fills credentials, clicks "New Release"
do("create_github_token")  # ✅ Cua Driver fallback
```

| Step | Chosen path | Why |
|------|-------------|-----|
| `git_commit` | Tier 1 — `git` CLI | Fastest, no GUI needed |
| `git_push` | Tier 1 — `git` CLI | Preferred, but fails (auth) |
| `create_github_token` | Tier 3 — Cua Driver | Escalates on failure |

---

### Scenario 2: Browser Fill + Submit

**Pain point:** Filling web forms and clicking submit buttons should be simple — but many sites use dynamic JS-rendered fields that static selectors can't reach.

```python
# Router decides: Tier 2 (CDP JS) for DOM access
do("browser_fill_input", args={
    "field": "#email",
    "value": "user@example.com"
})  # ✅ CDP directly sets input.value

# Submit button is outside React's virtual DOM — CDP can't trigger it
do("browser_click_button", args={"button": "submit"})
# ❌ CDP JS fails — button not in accessible DOM tree
# Router escalates: Tier 3 (Cua pixel click)
# → Finds button coordinates via AX tree, clicks at pixel position
# ✅ Success!
```

| Step | Chosen path | Why |
|------|-------------|-----|
| `browser_fill_input` | Tier 2 — CDP JS | Direct DOM access, fast |
| `browser_click_button` | Tier 2 → Tier 3 | CDP fails → pixel/AX fallback |

---

### Scenario 3: Mac App Automation

**Pain point:** Native macOS apps (Finder, Calendar, Mail) have no web inspector, no CDP, no CLI interface for complex operations. You need full GUI control.

```python
# Router decides: Tier 3 (Cua Driver) for native app interaction
do("app_click", args={
    "app": "Finder",
    "target": "Documents folder"
})  # ✅ Cua AX tree locates and clicks

# Button not found in AX tree (custom renderer)...
do("app_click", args={
    "app": "Preview",
    "target": "Export button"
})  # ❌ Tier 3 fails — no AX match
# Router escalates: Tier 4 (Vision AI)
# → Takes screenshot, runs vision model, finds button coordinates
# → Clicks at detected pixel position
# ✅ Success!
```

| Step | Chosen path | Why |
|------|-------------|-----|
| `app_click` (Finder) | Tier 3 — Cua Driver | Native app, AX tree works |
| `app_click` (Preview) | Tier 3 → Tier 4 | AX fails → vision screenshot fallback |

---

## Skills

| Skill | What it does |
|-------|-------------|
| `cua-driver-install-macos` | Install, configure, and integrate Cua Driver on macOS — permission granting, MCP setup, Hermes integration |

## Quick Start

```bash
# Clone
git clone https://github.com/ChrisLamDev/cua-desktop-automation-skills.git
cd cua-desktop-automation-skills

# Try the Router
python3 examples/router.py stats
python3 examples/router.py git_commit
```

## For Cua Driver Contributors

This repo demonstrates real-world patterns for building computer-use agents with Cua Driver:

1. **Task routing with fallback** — the key architectural pattern for reliable desktop automation
2. **Self-learning from history** — essential for production agents that improve over time
3. **Multi-tier escalation** — CLI → CDP → Cua Driver → Vision — handles all failure modes

## Platform Compatibility

- **macOS** (primary target — tested on Apple Silicon)
- Requires Cua Driver: `brew install trycua/tap/cua`

## 🧬 Skill Anatomy

Each skill in this repo follows a standard structure that the Router Learning System understands:

```
skills/cua-driver-install-macos/
├── SKILL.md         # Human-readable: what this skill does, prerequisites, usage
└── router.py        # Machine-readable: execution logic for the router
```

**`SKILL.md`** — The documentation layer. Contains a clear description, environment prerequisites (e.g., "requires macOS 14+"), step-by-step usage instructions, and any notes on common failure modes. This is what contributors read and what AI agents can parse for context.

**`router.py`** — The execution layer. Exports one or more functions that align with the built-in task routes (e.g., `install()`, `verify()`). The router imports these functions dynamically and calls them with the appropriate tier strategy. Each function should return a `(success: bool, result: str)` tuple so the router can record the outcome and learn from it.

**How the router uses a skill:**

```python
# The router discovers and invokes a skill dynamically
from skills.cua_driver_install_macos.router import install

success, result = install()
# Router records: {skill_name, tier, success, duration_ms}
# Next time: router prioritizes this path if it worked
```

To add a new skill: create `skills/<your-skill>/SKILL.md` and `skills/<your-skill>/router.py`, then register the task route in the router's configuration table.

## License

MIT — use freely, improve openly.
