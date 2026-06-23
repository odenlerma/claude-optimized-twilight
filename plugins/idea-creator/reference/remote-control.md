# Remote control: drive Claude Code from your phone

You're away from your Mac. A Claude Code session is mid-task — or you want to kick one off — and all you have is your phone. Which tool reaches your machine securely, without risking your Anthropic account?

**One-line answer:** Use Anthropic's **official Remote Control** feature (`claude remote-control` / `/rc`), introduced February 2026. It bridges a Claude Code session running on your own Mac to the Claude mobile app and `claude.ai/code` with **zero ban risk** (it's first-party), **no inbound ports**, and no extra infrastructure. If you need a raw shell or your plan lacks the feature, fall back to **SSH over Tailscale + tmux** — also no Anthropic-ToS dimension, since that's just networking to hardware you own.

---

## Comparison

| Option | How it works | Ban risk | Cost | Setup effort | Best for |
|---|---|---|---|---|---|
| **Remote Control** (`claude remote-control` / `/rc`) — *official* | Local Claude Code registers with Anthropic API, polls for work over outbound HTTPS; phone (Claude app) or browser (`claude.ai/code`) is a window into the local session. Session stays on your Mac. | **None** — first-party Anthropic feature, sanctioned use of your subscription. | Included with Pro/Max/Team/Enterprise. No API charge. | Low — one command, scan a QR code. | Steering or starting a session from your phone with minimal setup. |
| **Channels** (Telegram/Discord/iMessage) — *official* | Official plugin runs an MCP "channel" that pushes chat messages into your already-running local session; Claude replies back through the same chat. Runs on your Mac. | **None** — first-party plugins, your own bot credentials. | Free (plus your own bot). | Medium — install plugin, create bot, pair, allowlist. | Chatting with Claude from any messaging app; reacting to CI/webhooks. |
| **SSH from phone + tmux/mosh, over Tailscale** | Tailscale (WireGuard mesh) gives your Mac a private `100.x` address; phone SSH client connects, attaches a `tmux` session running Claude Code. No public port exposed. | **None** — pure networking to your own hardware; Claude Code runs locally as normal, no Anthropic-ToS dimension. | Free (Tailscale personal; Blink Shell one-time, Termius free tier, mosh/tmux free). | Medium — enable Remote Login, install Tailscale on both, learn tmux. | Full terminal control, raw shell, any CLI tool — not just Claude. |
| **VS Code Remote-SSH / Tunnels / `code-server`** | Browser or VS Code app reaches your Mac; Claude Code runs in the integrated terminal. Tunnels relay through Microsoft; `code-server`/Remote-SSH pair well with Tailscale. | **None** for Claude (still local first-party Claude Code). Tunnels route *editor* traffic through a third party — your code, not Claude credentials. | Free. | Medium–High — heavier than a shell; phone editing is cramped. | You already live in VS Code and want the editor too, not just a prompt. |
| **Claude Code on the web** (`claude.ai/code`, `--remote`) — *official* | Task runs in an **Anthropic-managed cloud VM** cloned from GitHub — *not* your Mac. Monitor/steer from the mobile app. | **None** — first-party. But it does **not** drive *your* machine; different tool for "no local setup." | Included with Pro/Max/Team. Shares your rate limits; no separate compute charge. | Low — connect GitHub, submit a task. | Async work on a cloned repo when you don't need your local environment. |
| **GitHub Codespaces / cloud dev box + Claude Code** | Claude Code installed on a cloud machine you own/rent; reach it from phone via browser or SSH. Solves "from phone" by not using your Mac at all. | **None** for Claude (your own first-party install). Codespaces billing/ToS is GitHub's. | Codespaces free tier then metered; or VPS cost. | Medium–High — provision box, install + auth Claude Code. | When your Mac is off and you want a persistent always-on environment. |
| **Unofficial chat-bridge bots** (DIY Telegram/Slack → terminal relay) | A third-party script relays messages to a terminal running Claude Code. | **Low–Medium** — see below. Safe *only* if it drives the official Claude Code CLI on your box (no credential routing). The official **Channels** feature now covers this need — prefer it. | Free–varies. | High — and you own the security of the relay. | Niche; superseded by official Channels. |
| **Routing subscription OAuth into a third-party harness** (OpenClaw-style) | Extracts your Pro/Max OAuth token and feeds inference to a non-Anthropic agent. | **High** — explicitly prohibited; tokens server-side blocked since Jan 2026. | — | — | **Avoid.** This is the actual ban vector. |

---

## Recommendation

**Primary: Remote Control (`claude remote-control` / `/rc`).** It's purpose-built for this exact need, it's first-party (so there is no Terms-of-Service question at all), the session never leaves your Mac, it opens **no inbound ports**, and setup is a single command plus a QR scan. Your full local environment — filesystem, MCP servers, project config — stays available, and you get push notifications when a long task finishes or Claude needs a decision. Requires a Pro/Max/Team/Enterprise subscription, claude.ai OAuth login (not an API key), and Claude Code **v2.1.51+**.

**Fallback: SSH over Tailscale + tmux.** Use this if (a) you're on an API-key / Bedrock / Vertex / Foundry login that Remote Control doesn't support, (b) the rollout hasn't reached your account, (c) your org admin disabled it, or (d) you want a raw shell, not just Claude. Tailscale means no public port and no port-forwarding; `tmux` keeps the session alive across disconnects; `mosh` survives flaky cellular. This is just networking to hardware you own — no Anthropic-ToS dimension.

A nice property: these compose. Run Claude Code inside `tmux` over SSH **and** type `/rc` in it — now you have both a raw terminal and the polished mobile UI into the same session.

---

## Setup sketch — Remote Control (recommended)

1. **Update Claude Code** on your Mac and confirm the version:
   ```bash
   claude --version   # need v2.1.51 or later (VS Code extension: v2.1.79+)
   ```
2. **Sign in with claude.ai** (OAuth, not an API key). If `ANTHROPIC_API_KEY` is set in your shell, unset it first:
   ```bash
   unset ANTHROPIC_API_KEY
   claude        # then run /login and choose the claude.ai option
   ```
3. **Install the Claude mobile app** — iOS / Android — and sign in with the **same account/organization**. (Inside Claude Code, `/mobile` shows a download QR.)
4. **Start a Remote Control session.** Three ways:
   - From an existing session, type `/remote-control` (or `/rc`) — carries over your current conversation.
   - Start fresh with remote enabled: `claude --remote-control "My Project"`.
   - Server mode (multiple concurrent sessions, stays running waiting for connections): `claude remote-control` — press **spacebar** to show the QR code.
5. **Connect the phone.** Scan the QR code to open the session directly in the Claude app, or open `claude.ai/code` (or tap **Code** in the mobile app) and pick the session by name — it shows a computer icon with a green dot when online. Now you can send messages from terminal, phone, and browser interchangeably; they stay in sync.
6. **(Optional) Push notifications.** Run `/config` and enable **Push when Claude decides** and/or **Push when actions required**. You can also prompt "notify me when the tests finish."
7. **(Optional) Always-on.** `/config` → set **Enable Remote Control for all sessions** to `true` so every interactive session is reachable.

**Keep it alive while you're away.** Remote Control runs as a local process — if you quit the `claude` process or close the terminal, the session ends, and an outage longer than ~10 minutes times it out. To keep it running when you shut your laptop lid or close the terminal, launch it inside a persistent multiplexer:
```bash
brew install tmux
tmux new -s claude
claude --remote-control "My Project"     # then detach with Ctrl-b d
```
A Mac with the lid closed sleeps unless on power with sleep disabled (`caffeinate -s`, or **System Settings → Battery/Lock Screen**), so for true away-from-desk use, leave it plugged in and awake.

**Plan gotchas.** On **Team/Enterprise**, Remote Control is **off by default** — an admin must enable the toggle at `claude.ai/admin-settings/claude-code`. It's unavailable on API-key/Console logins, on long-lived `setup-token`/`CLAUDE_CODE_OAUTH_TOKEN` tokens (inference-only scope), and where data-retention/compliance config blocks it. `claude doctor` tells you which check failed.

---

## Setup sketch — SSH over Tailscale + tmux (fallback)

1. **Enable Remote Login (SSH) on the Mac.** **Apple menu → System Settings → General → Sharing**, turn on **Remote Login** (off by default). Click the info button to restrict to specific users. (CLI equivalent: `sudo systemsetup -setremotelogin on`.) Use SSH **keys**, not passwords.
2. **Install Tailscale** on the Mac and on your phone (`tailscale.com/download`); sign both into the same tailnet. The Mac gets a stable private address (usually `100.x.x.x`) reachable from anywhere — **no port forwarding, no public exposure**. Tailscale's free Personal plan covers this.
3. **Install a phone SSH client.** **Blink Shell** (iOS, one-time purchase, native `mosh`) or **Termius** (iOS/Android, free tier). `mosh` is worth it on cellular — it survives IP changes and intermittent links.
4. **On the Mac, run Claude Code inside tmux** so it persists across disconnects:
   ```bash
   brew install tmux        # if needed
   tmux new -s claude
   claude                   # start your session; detach with Ctrl-b then d
   ```
5. **From the phone**, SSH in over the Tailscale address and re-attach:
   ```bash
   ssh you@100.x.x.x        # or: mosh you@100.x.x.x
   tmux attach -t claude
   ```
   Everything is exactly where you left it.

---

## Terms-of-service constraints (plain language)

The relevant policies are Anthropic's **Usage Policy** (effective 2025-09-15) and the **Consumer Terms of Service**. What they actually say, and how it maps to the options above:

- **Automated/programmatic access is restricted — with explicit exceptions.** The Consumer Terms prohibit accessing the services "through automated or non-human means, whether through a bot, script, or otherwise," *"except when you are accessing our Services via an Anthropic API Key or where we otherwise explicitly permit it."* **Claude Code is that explicit permission** — Anthropic's own product, built for scripted and interactive use. Running Claude Code on your Mac and driving it remotely is sanctioned use, however you reach the terminal.

- **Account and credential sharing is prohibited.** *"You may not share your Account login information, Anthropic API key, or Account credentials with anyone else. You also may not make your Account available to anyone else."* Remote-controlling your own session from your own phone is **not** sharing — it's one person, one account, multiple devices. Letting a friend use your login *would* violate this.

- **Bypassing limits/guardrails and ban-evasion are prohibited.** The Usage Policy's "Do Not Abuse our Platform" section bars intentionally bypassing "capabilities, restrictions, or guardrails," automating account creation/"spammy behavior," circumventing a ban with another account, and facilitating account/API access in violation of the Supported Regions Policy. None of the recommended options touch these.

- **The actual ban vector (avoid):** Routing **Pro/Max/Free subscription OAuth tokens into third-party harnesses** (OpenClaw, OpenCode, etc.) for cheaper inference. Anthropic deployed **server-side blocks in January 2026** (`"This credential is only authorized for use with Claude Code and cannot be used for other API requests"`) and clarified the Terms in February 2026: *"Using OAuth tokens obtained through Claude Free, Pro, or Max accounts in any other product, tool, or service — including the Agent SDK — is not permitted."* First-party Claude Code and claude.ai are explicitly exempt. Programmatic/third-party use belongs on **API keys** (Commercial Terms), which carry no automation restriction.

**The key distinction to internalize:**

> **Reaching your own Mac over the network** (Tailscale, SSH, VS Code tunnels, mosh) is a **networking** question, not an Anthropic question — there's no Anthropic-ToS dimension to logging into hardware you own. **Authentication routing** is the Anthropic question: as long as your subscription credential is used only by first-party Claude Code (or you use an API key for automation), you're inside the rules. Risk appears only when an *unauthorized third party* sits between your subscription and Anthropic's models.

By that test: Remote Control, Channels, SSH/Tailscale, VS Code Remote, Codespaces, and Claude Code on the web are all **ban-risk: None**. Only credential-routing harnesses are **High**.

---

## Caveats & staleness

- **Research-preview features move fast.** Remote Control and Channels are both labeled **research preview** as of mid-2026 — flags, behavior, version requirements, and plan availability can change. Re-check `claude --version` against the docs.
- **Policies change.** Policy language quoted here reflects the 2025-09-15 Usage Policy and the Feb 2026 Terms update. **Before relying on the ToS analysis above, re-read the live policies yourself:** [anthropic.com/aup](https://www.anthropic.com/aup) and the Consumer Terms.
- **Sign-in method matters.** Remote Control needs a full-scope claude.ai OAuth login. API-key, Bedrock, Vertex, Foundry, and long-lived inference-only tokens are not supported for it — those users should use the SSH/Tailscale fallback.
- **"From your phone" ≠ "on your Mac."** Claude Code on the web and Codespaces both solve "from my phone" but run **elsewhere**, not on your machine — choose them only if you don't need your local environment.
- **Verify your own setup's security.** Tailscale ACLs, SSH key hygiene, and disabling password auth are on you. Don't expose SSH to the public internet; let the mesh VPN handle reachability.

---

## Sources

- [code.claude.com/docs/en/remote-control](https://code.claude.com/docs/en/remote-control) — **primary**: official Remote Control docs — commands (`/rc`, `claude remote-control`, `--remote-control`), QR connect flow, "Claude keeps running locally… nothing moves to the cloud," outbound-HTTPS-only / no-inbound-ports security model, short-lived credentials, plan requirements (Pro/Max/Team/Enterprise; not API keys), v2.1.51+, ~10-min outage timeout, push notifications.
- [code.claude.com/docs/en/claude-code-on-the-web](https://code.claude.com/docs/en/claude-code-on-the-web) — Claude Code on the web running in **Anthropic-managed cloud VMs** (not your Mac), `--remote`/`--teleport`, isolation/credential-proxy model, shared rate limits, plan availability.
- [code.claude.com/docs/en/channels](https://code.claude.com/docs/en/channels) — the **official** Telegram/Discord/iMessage chat-bridge feature, events arriving in the local session running on your machine, sender allowlists/pairing security, plan/admin gating.
- [anthropic.com/aup](https://www.anthropic.com/aup) — Usage Policy (effective 2025-09-15): "Do Not Abuse our Platform" clauses on bypassing guardrails, automated account creation, ban circumvention, facilitating unauthorized access.
- Anthropic **Consumer Terms of Service** (anthropic.com/legal/consumer-terms) — exact clauses: automated-access prohibition with the API-key / "explicitly permit" exception, and "You may not share your Account login information… with anyone else."
- [theregister.com — Anthropic clarifies ban on third-party Claude access (2026-02-20)](https://www.theregister.com/2026/02/20/anthropic_clarifies_ban_third_party_claude_access/) — the ban targets *third-party tools routing subscription OAuth tokens*; "the official Claude Code tool and Claude.ai web interface are explicitly permitted"; the "OAuth tokens… in any other product, tool, or service — including the Agent SDK — is not permitted" wording.
- [thenewstack.io — Anthropic Agent SDK confusion](https://thenewstack.io/anthropic-agent-sdk-confusion/) — corroborates the subscription-token vs. API-key distinction and that third-party harnesses must use API keys.
- [venturebeat.com — Anthropic cracks down on unauthorized Claude usage by third-party harnesses](https://venturebeat.com/technology/anthropic-cracks-down-on-unauthorized-claude-usage-by-third-party-harnesses) — same crackdown (OpenClaw/OpenCode/Cline), Jan-2026 server-side token block, first-party exemption.
- [support.apple.com — Allow a remote computer to access your Mac](https://support.apple.com/guide/mac-help/allow-a-remote-computer-to-access-your-mac-mchlp1066/mac) — macOS Remote Login path (**System Settings → General → Sharing**), enables SSH/SFTP, off by default, per-user access control.
- [tailscale.com/docs/account/manage-plans/free-plans-discounts](https://tailscale.com/docs/account/manage-plans/free-plans-discounts) — Tailscale free Personal plan, WireGuard mesh, no port forwarding.
