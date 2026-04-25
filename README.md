# resy-skill

A browser-based Resy booking skill for AI agents. Works with any agent that has access to a browser tool — Claude Code, Codex, OpenClaw, or anything else.

No API key. No third-party service. No per-booking fee. Just your own Resy account, your own browser session, your agent doing the clicking.

## How it works

Your agent navigates Resy in a browser the same way you would — searches for the venue, checks availability, picks a time, and confirms the booking. The session stays signed in between uses, so setup is one-time.

## Setup

### 1. Sign into Resy in your agent's browser profile

Your agent's browser tool runs Chromium with its own profile (separate from your everyday Chrome). That profile starts signed-out. To use this skill, you sign in **once** inside that profile, and the cookies persist for future runs.

The trick: the agent has to launch the browser, then you log in in the window it opens. You can't just open your normal Chrome — that's a different profile, and the agent won't see the session.

**Claude Code (chrome-devtools MCP):** This is the easiest case. The chrome-devtools MCP server uses a persistent user-data-dir by default, so cookies stick across runs. In Claude Code:

> "Use the chrome-devtools tool to open https://resy.com in a new page. Leave the page open."

A visible Chromium window appears. Sign in normally. Tell the agent to close the page (or just close the window). Done — next time your agent opens Resy, you're already logged in.

If your config has `isolated: true`, remove it — that flag wipes the profile after each session. See the [chrome-devtools-mcp docs](https://github.com/ChromeDevTools/chrome-devtools-mcp).

**Claude Code (Playwright MCP):** Same idea — ask the agent to navigate to resy.com, log in in the window, close. Make sure your Playwright MCP config uses a persistent context (a fixed `userDataDir`), not the default fresh-context-per-session.

**OpenClaw:** OpenClaw manages a dedicated browser profile through its Gateway:
```bash
openclaw browser open https://resy.com
```
Sign in manually, close. The OpenClaw profile stays signed in across runs. See [OpenClaw browser docs](https://docs.openclaw.ai/tools/browser).

**Codex CLI / other agents:** Whatever browser tool your agent uses, the rule is the same:
1. Configure it to use a persistent profile (look for `--user-data-dir`, `userDataDir`, `storageState`, or "persistent context" in your tool's docs).
2. Have your agent navigate to `https://resy.com`.
3. Log in in the window that opens.
4. Close.

If you can't get a persistent profile working, this skill won't work for you — Resy will see a fresh, signed-out browser every time.

### 2. Install the skill

Skills go in a specific folder your agent watches. Pick the right one for your tool:

**Claude Code** — clone into your personal skills folder:
```bash
git clone https://github.com/Avanderheyde/resy-skills ~/.claude/skills/resy
```
Restart Claude Code (or run `/skills` to refresh). The `resy` skill should now be available across all projects. Project-scoped install: clone into `.claude/skills/resy` inside your repo instead. See [Claude Code skills docs](https://code.claude.com/docs/en/skills).

**OpenClaw** — clone into your user skills folder:
```bash
git clone https://github.com/Avanderheyde/resy-skills ~/.openclaw/skills/resy
```
OpenClaw watches the folder and picks up SKILL.md changes automatically. See [OpenClaw skills docs](https://docs.openclaw.ai/tools/skills).

**Codex CLI / other agents** — if your tool doesn't have a skills directory convention, paste the contents of `SKILL.md` into your agent's system prompt, AGENTS.md, or whatever context file it loads.

To verify it's installed, ask your agent: "Do you have the resy skill?" — it should describe what the skill does.

### 3. Book

Ask your agent:
> "Book me a table at Lilia next Friday for 2, around 7pm"

The agent will search, surface available slots, and confirm with you before completing the booking.

## Features

- Search by restaurant name and city
- Check availability for any date and party size
- Handles "no availability" gracefully — surfaces next available dates
- Flags deposit requirements before submitting
- Detects session expiry and prompts re-login
- Loop prevention — won't spiral on venues not listed on Resy

## Confirmation behavior

By default, the agent **confirms with you before clicking the final booking button**. This is intentional — Resy reservations can carry no-show fees and deposits.

If you trust your agent to book autonomously (e.g. for a cron job that fires when reservations drop), you can tell it: "You have permission to complete the booking without asking me first."

## Automation

Two ways to use this skill:

### Book now

Just ask your agent in chat:

> "Use the resy skill to book Lilia next Friday for 2 around 7pm."

The agent searches, surfaces options, and confirms with you before submitting.

### Schedule for later

Useful for catching hot venue drops — Carbone, Torrisi, 4 Charles, etc. release reservations exactly 30 days out, often at 9–10am local. Schedule a run for that exact moment.

**Claude Code** — use the built-in `/schedule` skill or the Agent SDK to fire on a cron:
- Slash command: https://docs.claude.com/en/docs/claude-code
- Agent SDK: https://docs.claude.com/en/api/agent-sdk

Tell the agent:
> "On 2026-05-15 at 8:59am ET, use the resy skill to book Carbone for 2 at 9:00pm on 2026-06-14. You have permission to complete the booking without asking me first."

**OpenClaw** — use the built-in `cron` tool to schedule a job that runs your agent at a specific time:
- Cron jobs: https://docs.openclaw.ai/automation/cron-jobs
- CLI reference: https://docs.openclaw.ai/cli/cron

> "Schedule a job for [time]: use the resy skill to book [venue] for [N] at [time] on [date]. Pre-approved."

**Any other agent** — if your framework has a cron/scheduler and a browser tool, the same pattern works. Point it at this skill, give it a pre-approved booking instruction, fire it at the release moment.

### Pre-approving autonomous bookings

By default the skill asks before clicking the final reserve button. For scheduled runs, that won't work — there's no human in the loop. Add this line to your agent instruction to pre-approve a *specific* booking:

> "You have permission to complete this specific booking without asking me first. Still stop and notify me if a deposit is required or the requested time is unavailable."

Don't grant blanket booking permission — scope it to the single reservation you're trying to catch.

## Supported cities

`new-york-ny` · `los-angeles-ca` · `miami-fl` · `chicago-il` · `san-francisco-bay-area` · `las-vegas` · `washington-dc-area` · `london`

## Tips

- **Hot venue release windows:** Carbone, Torrisi, and similar spots drop reservations ~30 days out at a fixed time (often 9–10am local). Schedule your agent to run right at that moment.
- **Session expiry:** If your agent reports a login wall, just sign back into Resy in the browser profile. Sessions last weeks to months typically.
- **Party sizes > 6:** Resy routes large parties differently. This skill works best for 2–6 guests.

## Why not just use the Resy API?

Unofficial Resy API access risks getting your account banned. Browser automation mimics a real user session — much harder to detect, and your actual account stays safe.

## License

MIT — use it, fork it, improve it.
