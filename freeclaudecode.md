# 🚀 How to Use Claude Code for FREE Using NVIDIA's API + Kimi K2.6
### A Complete From-Scratch Guide for Ubuntu Linux

> **Written by:** Jatin (with help from Claude)
> **Goal:** Run Claude Code completely free by routing it through NVIDIA's free developer API using the powerful Kimi K2.6 AI model, with zero monthly cost.

---

## 🧠 Understanding What We're Building (Read This First!)

Before touching a single command, let's understand *why* this works — because when you understand the idea, nothing will confuse you.

**The Problem:** Claude Code is an amazing AI coding tool, but it normally requires a paid Anthropic API subscription. Most students and developers can't afford that.

**The Solution:** We build a clever "middleman" system on your own computer. Here's the chain:

```
Your Claude Code  →  LiteLLM Proxy (your PC)  →  NVIDIA Free API  →  Kimi K2.6 AI
```

Think of it like this: Claude Code is a customer who only speaks English. NVIDIA's servers only speak French. LiteLLM is the translator sitting in the middle, instantly converting every message between both sides — completely invisible to both of them. Claude Code thinks it's talking to Anthropic. NVIDIA thinks it's talking to a normal OpenAI-compatible app. Neither one knows about the other. LiteLLM makes them compatible.

**Kimi K2.6** is the free AI model running on NVIDIA's servers. It's a 1-trillion-parameter model from Moonshot AI, specifically designed for coding and tool use — making it one of the best free alternatives to Claude for real development work.

**NVIDIA's Free Developer Program** gives you access to run these powerful AI models for free after a simple registration. No credit card needed.

---

## ✅ Prerequisites Check

First, open your terminal and verify these four tools are installed by running each command:

```bash
python3 --version    # Should show Python 3.8 or higher
pip3 --version       # Should show pip 20 or higher
node --version       # Should show Node.js 18 or higher
npm --version        # Should show npm 8 or higher
```

If any of these say "command not found", install them first. For Node.js and npm on Ubuntu:

```bash
sudo apt update && sudo apt install nodejs npm python3 python3-pip -y
```

---

## 📋 Complete Step-by-Step Setup

### STEP 1 — Get Your Free NVIDIA API Key

Your NVIDIA API key is like a password that proves to NVIDIA's servers that you're a registered developer. Without it, every request gets rejected.

1. Go to **https://build.nvidia.com**
2. Click **Sign In** → create a free account (only needs email + phone verification)
3. Once logged in, find any model (like Kimi K2.6 at `build.nvidia.com/moonshotai/kimi-k2.6`)
4. Click **"View Code"** in the top right
5. Your API key will be visible in the code snippet — it starts with `nvapi-` followed by a long string of random characters, like `nvapi-aBcDeFgHiJkLmN...`
6. Copy it and keep it somewhere safe

---

### STEP 2 — Install LiteLLM

LiteLLM is the translator that runs a small local server on your computer. Modern Ubuntu (22.04+) protects its system Python from external packages, so we use **pipx** instead of pip — pipx creates a safe isolated environment for each tool automatically.

```bash
# Install pipx (Ubuntu's recommended way to install Python tools)
sudo apt install pipx

# Set up pipx so your terminal can find its tools
pipx ensurepath
```

**Close your terminal completely and open a fresh one** — this is important so the PATH update takes effect. Then install LiteLLM:

```bash
pipx install 'litellm[proxy]'
```

The `[proxy]` part in quotes is essential — it installs the server component that Claude Code will talk to. Verify it worked:

```bash
litellm --version    # Should show a version number like 1.51.0
```

---

### STEP 3 — Create the Proxy Config File

This is the "translation dictionary" that tells LiteLLM: whenever Claude Code asks for any Claude model by name, route that request to Kimi K2.6 on NVIDIA instead.

First, create a folder to keep everything organised:

```bash
mkdir -p ~/claude-code-proxy
```

Now paste this entire block into your terminal at once — it writes the full config file automatically:

```bash
cat > ~/claude-code-proxy/config.yaml << 'EOF'
model_list:

  # Every Claude model name that Claude Code might send gets mapped
  # to Kimi K2.6 on NVIDIA. The "openai/" prefix tells LiteLLM to use
  # OpenAI-compatible format (which NVIDIA's API speaks).

  - model_name: claude-opus-4-7          # Newest Claude Opus
    litellm_params:
      model: openai/moonshotai/kimi-k2.6
      api_base: https://integrate.api.nvidia.com/v1
      api_key: os.environ/NVIDIA_API_KEY

  - model_name: claude-opus-4-6
    litellm_params:
      model: openai/moonshotai/kimi-k2.6
      api_base: https://integrate.api.nvidia.com/v1
      api_key: os.environ/NVIDIA_API_KEY

  - model_name: claude-sonnet-4-6        # Claude Sonnet 4.6
    litellm_params:
      model: openai/moonshotai/kimi-k2.6
      api_base: https://integrate.api.nvidia.com/v1
      api_key: os.environ/NVIDIA_API_KEY

  - model_name: claude-haiku-4-5-20251001
    litellm_params:
      model: openai/moonshotai/kimi-k2.6
      api_base: https://integrate.api.nvidia.com/v1
      api_key: os.environ/NVIDIA_API_KEY

  - model_name: claude-sonnet-4-20250514
    litellm_params:
      model: openai/moonshotai/kimi-k2.6
      api_base: https://integrate.api.nvidia.com/v1
      api_key: os.environ/NVIDIA_API_KEY

  - model_name: claude-3-5-sonnet-20241022
    litellm_params:
      model: openai/moonshotai/kimi-k2.6
      api_base: https://integrate.api.nvidia.com/v1
      api_key: os.environ/NVIDIA_API_KEY

litellm_settings:
  # drop_params is CRITICAL — Claude Code sends Anthropic-specific
  # parameters that NVIDIA doesn't understand. This silently drops
  # them instead of crashing with an error.
  drop_params: true
  set_verbose: false
  default_litellm_params:
    temperature: 0.6     # Kimi K2.6's recommended temperature for coding

  # This disables the experimental Responses API adapter in newer
  # LiteLLM versions that breaks compatibility with NVIDIA's endpoint.
  enable_responses_api: false

general_settings:
  # This is the "password" Claude Code will send to YOUR local proxy.
  # It can be any string — just remember it matches ANTHROPIC_AUTH_TOKEN below.
  master_key: sk-my-local-proxy-key-1234
EOF
```

Verify it was written correctly:

```bash
cat ~/claude-code-proxy/config.yaml
```

You should see all the model entries printed back cleanly.

---

### STEP 4 — Install Claude Code

```bash
npm install -g @anthropic-ai/claude-code
```

Verify it installed:

```bash
claude --version    # Should show something like 2.1.141 (Claude Code)
```

---

### STEP 5 — Set Your Environment Variables Permanently

Environment variables are like sticky notes that your terminal reads every time it opens. We need to write three important ones into your shell profile so they're automatically available in every terminal, every folder, after every reboot — forever.

Replace `nvapi-YOUR-KEY-HERE` with your actual NVIDIA key from Step 1, then run all of this at once:

```bash
# Write everything permanently to your shell profile
echo 'export NVIDIA_API_KEY="nvapi-YOUR-KEY-HERE"' >> ~/.bashrc
echo 'export ANTHROPIC_BASE_URL="http://localhost:4000"' >> ~/.bashrc
echo 'export ANTHROPIC_AUTH_TOKEN="sk-my-local-proxy-key-1234"' >> ~/.bashrc

# Apply the changes to your current terminal right now
source ~/.bashrc

# Verify your key loaded correctly
echo $NVIDIA_API_KEY    # Should print your nvapi-... key
echo $ANTHROPIC_BASE_URL    # Should print http://localhost:4000
```

**What each variable does:**
- `NVIDIA_API_KEY` — authenticates your requests to NVIDIA's servers
- `ANTHROPIC_BASE_URL` — redirects Claude Code from Anthropic's servers to your local LiteLLM proxy at port 4000
- `ANTHROPIC_AUTH_TOKEN` — the password Claude Code sends to your local proxy (must match `master_key` in your config)

---

### STEP 6 — Set Up LiteLLM as a Permanent Background Service

This is the step that makes everything work automatically forever. Instead of starting LiteLLM manually every time, we register it as a **systemd service** — Ubuntu's built-in system service manager. Think of it like adding an app to your startup programs, except even more reliable: it starts before your desktop loads, and if it ever crashes, Ubuntu automatically restarts it within 5 seconds.

Run this entire block at once:

```bash
mkdir -p ~/.config/systemd/user

cat > ~/.config/systemd/user/litellm-proxy.service << EOF
[Unit]
Description=LiteLLM Proxy for Claude Code with Kimi K2.6 on NVIDIA
After=network.target

[Service]
Type=simple
# Replace the path below if "which litellm" gives a different path
ExecStart=/home/$USER/.local/bin/litellm --config /home/$USER/claude-code-proxy/config.yaml --port 4000
Restart=on-failure
RestartSec=5
# Inject your NVIDIA key directly so the service always has it
Environment="NVIDIA_API_KEY=$NVIDIA_API_KEY"

[Install]
WantedBy=default.target
EOF

# Tell systemd about the new service
systemctl --user daemon-reload

# Enable it so it starts automatically on every boot
systemctl --user enable litellm-proxy.service

# Start it right now
systemctl --user start litellm-proxy.service
```

Check if it's running:

```bash
systemctl --user status litellm-proxy.service
```

You should see **`Active: active (running)`** highlighted in green. If you see that, your proxy is alive and will stay that way permanently.

---

### STEP 7 — Test the Complete Chain

Wait 15 seconds for LiteLLM to fully load, then run this test. It sends a real Anthropic-format request through your proxy all the way to Kimi K2.6 on NVIDIA and back:

```bash
sleep 15 && curl -s -X POST http://localhost:4000/v1/messages \
  -H "Authorization: Bearer sk-my-local-proxy-key-1234" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-sonnet-4-6",
    "max_tokens": 50,
    "messages": [{"role": "user", "content": "Say hello in one sentence!"}]
  }'
```

If you get back a JSON response containing text like `"Hello! How can I help you?"`, your entire chain is working perfectly. If you get an error, see the Troubleshooting section below.

---

### STEP 8 — Launch Claude Code!

Open a **brand new terminal window** (so all your permanent environment variables from `.bashrc` load fresh), navigate to any project folder on your computer, and run:

```bash
claude
```

That's it. Claude Code will launch and you'll see its welcome screen.

**Important choices when Claude Code starts:**

When asked about model selection (`/model`), choose **Sonnet** (option 2) for everyday coding tasks. Here's why this matters with our setup:

- The **1M context** options tell Claude Code to pre-load your ENTIRE project before sending any message. For large projects this can take 5+ minutes even before the first API call. Avoid this for quick tasks.
- **Sonnet 4.6** is the sweet spot — fast, capable, and doesn't pre-load your whole codebase.
- **Opus 4.7** is best for complex architectural decisions but slower.
- **Haiku** is fastest for quick questions and simple code changes.

Remember: no matter which option you pick, ALL requests go through Kimi K2.6 on NVIDIA. The model name just affects how Claude Code manages its context and structures requests internally.

---

## ⚠️ Troubleshooting Common Issues

**"externally managed environment" error when installing with pip**
This is Ubuntu 22.04+ protecting its system Python. Use pipx instead, as described in Step 2.

**"command not found: litellm" after installation**
Run `pipx ensurepath && source ~/.bashrc`, then try again.

**`{"detail":"Not Found"}` error from curl test**
Your LiteLLM version is too old and doesn't have the `/v1/messages` endpoint. Run:
```bash
pipx install 'litellm[proxy]==1.51.0' --force
systemctl --user restart litellm-proxy.service
```

**`404 page not found` error from LiteLLM logs**
Your LiteLLM version is too new and uses an experimental Responses API adapter that NVIDIA doesn't support. The config already includes `enable_responses_api: false` to prevent this. If it still happens, pin to version 1.51.0 as above.

**Claude Code says "Marinating..." for 5+ minutes**
You selected the 1M context option. Press Escape, type `/model`, and switch to regular Sonnet 4.6 without the 1M context. 1M context pre-loads your entire project locally before any API call, which is very slow.

**The proxy service fails after a reboot**
Your NVIDIA API key might not be loading into the service correctly. Update the service file to hardcode the key:
```bash
# Edit the service file and replace $NVIDIA_API_KEY with your actual key
nano ~/.config/systemd/user/litellm-proxy.service
# Find the line: Environment="NVIDIA_API_KEY=..."
# Replace with your actual key, save, then:
systemctl --user daemon-reload
systemctl --user restart litellm-proxy.service
```

**Checking proxy logs for any issue**
```bash
journalctl --user -u litellm-proxy.service -n 50 --no-pager
```

---

## 🔄 Daily Usage

After setup is complete, your daily workflow is simply:

1. Open any terminal anywhere on your computer
2. `cd` into your project folder
3. Run `claude`
4. Start coding!

The LiteLLM proxy starts automatically with your computer, so you never need to manage it manually. Everything just works.

---

## 💡 Key Concepts Summary

| Concept | What It Actually Is |
|---|---|
| LiteLLM | A translation server running on your PC, port 4000 |
| `openai/` prefix in config | Tells LiteLLM to use OpenAI-compatible API format (NVIDIA speaks this) |
| `ANTHROPIC_BASE_URL` | Redirects Claude Code from Anthropic's servers to your local proxy |
| `master_key` | Password your proxy requires (you set this yourself) |
| `drop_params: true` | Silently ignores Claude-specific params NVIDIA doesn't understand |
| systemd service | Ubuntu's service manager — keeps your proxy alive automatically |
| Kimi K2.6 Instruct | The actual AI model doing all the thinking (1T parameters, free on NVIDIA) |

---

## 🙏 Credits

Built and tested by **Jatin** on Ubuntu Linux.
Special thanks to the open-source teams behind LiteLLM, Claude Code (Anthropic), and Moonshot AI (Kimi K2.6).
NVIDIA's free developer API made this possible at zero cost.

---

*If this guide helped you, share it with other developers who can't afford paid AI tools!*
*Feel free to adapt and improve this guide for your community.*
