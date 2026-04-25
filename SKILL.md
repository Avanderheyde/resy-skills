---
name: resy
description: Book restaurant reservations on Resy. Use when the user asks to book a table, make a reservation, find availability, or check a specific restaurant on Resy.
triggers:
  - resy
  - book a table
  - book a reservation
  - find me a table
  - restaurant reservation
---

# Resy Booking Skill

Your agent books Resy reservations using a persistent browser session signed into the user's Resy account. No API key, no third-party service — just a browser acting like a human.

## Prerequisites

The user must be signed into Resy in your agent's browser profile before this skill runs. One-time setup. See README for instructions.

## Before you search: loop prevention

**Hard rule: one search per restaurant name. If it returns no match, STOP.**

The common failure mode is looping on a name that isn't on Resy — the agent keeps retrying variations. Don't do that. If the user's exact spelling returns zero matches, tell them: "Resy returned no match for X. Did you mean Y? Want me to try OpenTable?"

Only retry once, and only if the user confirms a new spelling.

## Flow

1. **Navigate to search:**
   ```
   browser navigate "https://resy.com/cities/<city>/search?date=YYYY-MM-DD&seats=<N>&query=<term>"
   ```
   where `<city>` is `new-york-ny`, `los-angeles-ca`, etc. Default to `new-york-ny` unless user specifies.

2. **Detect no-results:** Evaluate `document.body.innerText` and check for the string `No results for`. If present, STOP. Do not retry, do not guess variations.

3. **Pick the venue:** The first link element matching the user's requested name (case-insensitive) is the match. If the top result doesn't match the user's name, tell the user what you found and ask before proceeding.

4. **Open the venue page:**
   ```
   browser navigate "https://resy.com/cities/<city>/venues/<slug>?date=YYYY-MM-DD&seats=<N>"
   ```
   The `<slug>` comes from the link's href in the search results.

5. **Detect no-availability:** Read the page text. Signals:
   - Text contains `At the moment, there's no online availability for` — no slots for that date
   - Only a `Notify` button is present, no time buttons — waitlist only

   In both cases: surface the nearest alternative dates shown on the page and ask which the user wants to try. Do not auto-pick a different date.

6. **Pick a time:** Available time slots are rendered as buttons like `"7:00 PM"`. Click the one matching the user's request (or closest within ±30 min if they gave a soft window).

7. **Confirm before final submit:** After clicking the time, Resy shows a confirmation screen. **Stop here.** Tell the user: "Ready to book [Venue], [Day Date], [Time] for [N]. Confirm?" Only click the final "Complete Reservation" / "Reserve" button on explicit user approval.

8. **Handle deposit/credit card page:** High-demand venues (Carbone, Torrisi, 4 Charles Prime Rib, etc.) require a deposit. If a card form appears, stop and tell the user the deposit amount — do not auto-submit.

## URL patterns

- **Search:** `https://resy.com/cities/<city>/search?date=YYYY-MM-DD&seats=<N>&query=<term>`
- **Venue:** `https://resy.com/cities/<city>/venues/<slug>?date=YYYY-MM-DD&seats=<N>`

**Supported cities:** `new-york-ny`, `los-angeles-ca`, `miami-fl`, `chicago-il`, `san-francisco-bay-area`, `las-vegas`, `washington-dc-area`, `london`

## Known quirks

- **30-day release windows:** Hot venues (Carbone, Torrisi, etc.) drop reservations at a fixed time, often 9–10am local time, exactly 30 days out. If a date is unavailable, check if it's outside the release window before giving up.
- **Notify lists:** A "Notify" button means fully booked. Only join with explicit user confirmation.
- **Amex perks:** Some venues show "Global Dining Access" slots requiring an Amex card. Surface it, don't assume.
- **Party size mismatches:** If the user asks for 6 but only 2-top slots show, say so. Don't silently book for 2.

## Session expiry

If a login form appears, the session has expired. Tell the user to sign back into Resy in the browser profile manually. Do not ask for their password.

## Red lines

- Never click the final "Complete Reservation" button without explicit user approval for that specific booking.
- Never join a notify/waitlist without approval.
- Never submit credit card info or deposit without approval.
- Never retry the same search more than twice. If it didn't work, say so.
