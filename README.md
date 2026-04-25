# resy-skill

A browser-based Resy booking skill for AI agents. Works with any agent that has access to a browser tool — Claude Code, Codex, OpenClaw, or anything else.

No API key. No third-party service. No per-booking fee. Just your own Resy account, your own browser session, your agent doing the clicking.

## How it works

Your agent navigates Resy in a browser the same way you would — searches for the venue, checks availability, picks a time, and confirms the booking. The session stays signed in between uses, so setup is one-time.

## Setup

### 1. Sign into Resy in your agent's browser profile

Your agent needs access to a browser profile that's already signed into Resy. How you do this depends on your agent framework:

**OpenClaw:**
```bash
openclaw browser open https://resy.com
```
Sign in manually, then close. The `openclaw` profile stays signed in.

**Claude Code / other agents with Chrome access:**
Open Chrome (or Chromium) using the persistent profile your agent uses, navigate to `https://resy.com`, and sign in. Your agent will reuse that session.

**Generic:**
Whatever profile your agent's browser tool points to — open it, go to resy.com, log in. Done.

### 2. Add the skill

Drop `SKILL.md` wherever your agent reads skills from, or paste its contents into your agent's system prompt / context.

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
