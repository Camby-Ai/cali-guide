# Cali Setup Guide

Cali owns the lead list. Bad lists kill campaigns: guessed emails bounce, bounces hurt your sender reputation, and two people end up working the same business. Cali fixes that by being the one owner of one list. She finds local businesses that fit, notes why each one has a reason to buy now, records where every fact came from, and keeps the list clean after verification. She contacts no one. This guide sets her up on Hermes Agent, a free and open source agent from Nous Research that runs on your own machine.

The software is free. The model behind it is not always free, and the real work is deciding exactly who belongs on your list.

**Checked on 2026-11-21.** Commands and limits change. Sources are at the bottom.

**How this guide was made:** every command below was copied from the Hermes Agent documentation on that date. We have not test-run them on a fresh machine. If a step does not behave the way it says, see "If a step breaks" near the end: your AI can patch it for you in a minute.

## What Cali does

- Owns one lead list, start to finish. Nobody else adds to it or edits it.
- Finds local businesses that match your ideal customer, using web search.
- Writes down the reason each one might buy now, with the link she found it on.
- Finds the owner's name and a public email when there is one. She never guesses an email.
- Reads your email verification results, blocks the ones that bounce, and tags those businesses for more digging.
- Checks the list before adding anything, so no business is on it twice.
- Does not email, call or message anyone. She hands a clean list to whoever does outreach.

## What you need before you start

| You need | Why | Cost | Checked |
|---|---|---|---|
| A computer that stays on: macOS, Linux, Windows or WSL2 | The morning lead run only runs while Hermes' gateway is running | Yours already | 2026-11-21 |
| Git and curl | The installer uses them | Free | 2026-11-21 |
| Hermes Agent | Runs Cali | Free, MIT licence | 2026-11-21 |
| A model with at least 64,000 tokens of context | Hermes rejects smaller models at startup | Depends on the provider you pick. Not measured here | 2026-11-21 |
| An email verification service | Checks whether an address is real before anyone sends to it. We use Hunter | Check the service's own pricing page. Not listed here | 2026-11-21 |
| Telegram (optional) | To get the morning lead run on your phone | Free | 2026-11-21 |

On a brand new install the setup wizard may offer a "Free, start now" choice that needs no account. The Hermes docs say it is shown only when it is available, so do not plan around it.

In our own team Cali works from Neon Prospector, a lead tool we built for Las Vegas local businesses. It is not part of this guide. The version here uses Hermes' web search and your own sources, and it works for any city.

## The honest timeline

| When | What | Your time |
|---|---|---|
| Day 1 | Install Hermes and get one normal chat working | The Hermes docs say under 5 minutes. Allow 30 |
| Day 1 | Create Cali and give her a desk folder | 20 minutes |
| Day 1 to 2 | Write down exactly who belongs on the list | 1 hour. This is the part that decides everything |
| Day 2 | Turn on web search and the morning lead run | 30 minutes |
| Week 1 | Check her first rows against their sources, and run your first verification pass | 15 minutes a day |

The slow part is not the install. It is the definition. "Local businesses" gives you a phone book. "Owner-run HVAC companies in one city with a reason to buy this quarter" gives you a list somebody can work.

Three limits to know before you start:

- She verifies nothing on her own. Verification is done by the service you choose. You run the list through it and give her the results.
- She only records emails that are published. A guessed address is the thing this whole setup exists to stop.
- Cold outreach has rules, and some sites forbid automated collection in their terms. Check the law where you send and the rules of the sites you use. Cali does not send anything, but the list is still yours to answer for.

## Setup, in this order

### 1. Install Hermes and get one chat working

Do not add anything else until a plain conversation works. The Hermes quickstart is blunt about this, and it is right.

On macOS and Windows the Hermes docs recommend the Hermes Desktop installer from https://hermes-agent.nousresearch.com/. For a command line only install on macOS, Linux or WSL2:

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

Reload your shell, pick a model, and start a chat:

```bash
source ~/.zshrc
hermes setup
hermes
```

You should see: a welcome banner naming your model, and a reply to a simple question. If replies are empty or broken, run `hermes model` again and confirm the provider, the model and the login. If anything else looks off, run `hermes doctor`.

### 2. Create Cali

Every agent in Hermes gets its own profile: its own settings, memory and identity. Never point two agents at the same profile.

```bash
hermes profile create cali --description "Lead sourcing. Owns one lead list start to finish: finds local businesses that fit, records the reason to buy and the source, keeps the list clean after email verification. Never guesses an email. Contacts no one."
```

Creating the profile also creates a `cali` command. Check it:

```bash
cali chat
```

You should see: the same kind of chat as step 1, now running as the `cali` profile.

### 3. Give Cali her identity

In Hermes, who an agent is lives in a file called `SOUL.md` inside that agent's home. For a profile named `cali` that is:

```text
~/.hermes/profiles/cali/SOUL.md
```

Hermes creates a starter file there. Replace its contents with the text in "Cali's instructions" below. Keep it about who she is and how she talks. Your ideal customer and your rules go in different files, in the next step.

You should see: after starting a new `cali chat`, she introduces herself as the owner of your lead list when asked who she is.

### 4. Build her desk: the rules, the ideal customer, the list

This is the step everybody rushes, and it is the one that makes Cali useful.

Make a folder for her to work from:

```bash
mkdir -p ~/cali-desk/verification ~/cali-desk/handoff
```

Put three files in it. Hermes loads `AGENTS.md` from the folder you start a chat in, and Cali reads and updates the others as she works.

`~/cali-desk/AGENTS.md`, the rules:

```text
# How Cali works this desk

- leads.csv is the one list. You are its only owner. Nobody else adds or edits rows.
- icp.md says who belongs on the list. If a business does not match it, it does not go on.
- Before adding a business, search leads.csv for it. One business, one row, ever.
- Every row needs a source link and the date you found it. No source, no row.
- reason_to_buy is one line, and it has to come from something you read: a job post, a review, a news item, a change on their site.
- Record an email only if it is published where you found it. Never build or guess an address. If there is none, leave email empty and set status to needs_digging.
- verification/ holds the result files I bring back from the email verification service. When I add one, update email_status for every row in it.
- If an email bounced or failed, clear nothing: set email_status to blocked and status to needs_digging, then look for another way in (a different published address, a contact form, a phone number) and note it.
- When I ask for a handoff, write only rows with email_status verified into handoff/ as a dated file, and set their status to handed_off.
- Never email, call or message anyone.
```

`~/cali-desk/icp.md`, the part only you can write:

```text
# Who belongs on the list

- Kind of business: {for example owner-run HVAC and plumbing companies}
- Where: {city or service area}
- Size: {for example 5 to 50 staff}
- Who we want to talk to: {owner, general manager, office manager}
- Reasons to buy now: {what would make this month a good month for them, in your words}
- Who does not belong: {franchises, national chains, anyone already a customer}
- Where to look: {directories, licence boards, review sites and local sources you trust}
```

`~/cali-desk/leads.csv`: start it with one header line.

```text
business,owner,role,email,email_status,phone,reason_to_buy,source_url,found_on,status,notes
```

You should see: when you run `cali chat` from inside `~/cali-desk` and ask who belongs on the list, she answers from `icp.md`.

### 5. Turn on web search

Cali finds businesses and reads their pages with web search. Open the tools menu for her profile and confirm it is on:

```bash
hermes -p cali tools
```

Start a new `cali chat` after changing her tools. An old conversation keeps its old permissions.

You should see: when you ask her for three businesses that match `icp.md`, each one comes back with a link you can open.

### 6. Schedule the morning lead run

Hermes has a built-in scheduler. Give the job Cali's folder so she reads your rules and your ideal customer when she runs. The folder must be written as a full path.

```bash
hermes -p cali cron create "weekdays at 8am" \
  "Morning lead run. Read icp.md and leads.csv. Find up to ten new businesses that match icp.md and are not already in leads.csv. For each, add a row with the owner if published, a published email if there is one, one line on the reason to buy, the source link and today's date. Never guess an email. Then tell me how many rows are waiting for verification and how many are tagged needs_digging. Contact no one. Keep the summary under 150 words." \
  --workdir /Users/you/cali-desk \
  --name "Lead run"
```

Schedules run from a gateway, and each profile has its own. Start Cali's, and check that her scheduler is alive:

```bash
cali gateway start
hermes -p cali cron status
```

`cali gateway install` keeps it running after a restart.

By default the summary is saved on your machine. To get it on your phone, connect Telegram first with `hermes gateway setup`, then create the job with `--deliver telegram` added.

Two things to check on your machine:

- Whether a scheduled run has web search with the default settings. Run `hermes -p cali tools` to see which toolsets are on for each place Cali runs.
- Whether your times come out in your own timezone. Hermes reads the schedule in the profile's `timezone:` setting. Add `--timezone America/Los_Angeles` (or your zone) to pin it.

You should see: new rows in `leads.csv` the next weekday morning, each with a source link.

### 7. Run your first verification pass, by hand

Verification is the step that keeps a list clean, and it happens outside Hermes. Take the rows that have an email, run them through your email verification service, and save the result file in `verification/`. Then from inside `~/cali-desk` run `cali chat` and send her this:

```text
Read the newest file in verification/. For every email in it, update email_status in leads.csv: verified if it passed, blocked if it bounced or failed. For each blocked row, set status to needs_digging, look for another published way in, and write what you found in notes. Then write the verified rows to a dated file in handoff/ and set their status to handed_off. Tell me the counts. Contact no one.
```

You should see: every row in the result file updated, the blocked ones tagged `needs_digging` with a note, and a new file in `handoff/` holding only verified rows.

## Cali's instructions

Paste this into `~/.hermes/profiles/cali/SOUL.md`. Change the parts in braces.

```text
# Who you are

You are Cali, lead sourcing for {business name}. You work for {owner name}.

You own the lead list, start to finish. One list, one owner. Your job is to make sure everyone on it is real, fits, has a reason to buy now, and is on it exactly once.

# How you work

- You add a business only if it matches the written ideal customer.
- You check the list before you add anything. Nobody works the same lead twice.
- Every fact has a source link and a date. If you cannot source it, you leave it out.
- You never guess, build or pattern-match an email address. Published or empty.
- You do not verify emails yourself. You read the verification results you are given and keep the list honest about them.
- A bounce is not the end. You block the address, tag the business for more digging, and find another way in.
- You never contact anyone. You hand clean rows to whoever does outreach.
- You never invent a business, a name, a number or a reason to buy.

# How you talk

- Bright and quick. A detail nerd, and proud of it.
- Lead with the counts: new, verified, blocked, needs digging.
- One line per business: who they are, why now, where you found it.
- If the list has a problem, say it first.
```

This persona was written from Cali's job description for this guide. It is not a copy of the instructions she runs on in our own team.

## First test

Run `cali chat` from inside `~/cali-desk` and send her this:

```text
Find five businesses that match icp.md and are not in leads.csv. Add a row for each with the source link, today's date and one line on the reason to buy. Record an email only if it is published on the page you found. Then show me the five rows and tell me which ones have no email yet. Contact no one.
```

A good answer adds five rows with links you can open, leaves the email empty where none is published, tags those rows `needs_digging`, and contacts no one.

If a row has an email you cannot find at its source link, she guessed. Tighten the "never guess" line in `SOUL.md` and delete the address.

## When it goes wrong

| What you see | Why | Fix |
|---|---|---|
| Empty or broken replies | Model or login is wrong | `hermes model`, then a plain chat before anything else |
| The businesses do not fit | `icp.md` is vague, or the chat was not started in `~/cali-desk` | Start `cali chat` from that folder, and make `icp.md` more specific |
| She cannot find anything | Web search is off, or the chat is older than the change | `hermes -p cali tools`, then a new chat |
| An email is not at its source link | She guessed | Delete it, tighten the "never guess" line in `SOUL.md`, and check the rest of the batch |
| The same business appears twice | She did not check the list first | Tell her which row to keep, and tighten the "one row, ever" rule in `AGENTS.md` |
| No morning lead run | Cali's gateway is not running, or the job is missing its folder | `hermes -p cali cron status`, `cali gateway start`, and recreate the job with `--workdir` |
| Anything else | | `hermes doctor` |

## If a step breaks

Hermes changes fast, so a command here can drift from what your version expects. You do not need us to fix it. Paste this into Hermes itself, or into Claude or ChatGPT, with the error underneath:

```text
I am following a setup guide for a lead sourcing agent on Hermes Agent. This step failed. Here is the step, the exact command I ran, and the full error. Check the current Hermes Agent docs at https://hermes-agent.nousresearch.com/docs, tell me what changed, and give me the corrected command.
```

`hermes doctor` is worth running first. It catches most setup problems on its own.

## What it costs to run

- Hermes Agent: free, MIT licence.
- The model: billed by whichever provider you choose in `hermes model`. Every lead run and every verification pass is model usage. Not measured here, so no number is given.
- Web search: Hermes supports several search providers and the docs list free options. Pick one in `hermes tools`.
- Email verification: billed by the service you choose. Check its own pricing page.

## Want it set up for you?

CAMBY Ai sets Cali up and runs her for you: https://mikecamby.com

## Credit

Cali runs on [Hermes Agent](https://github.com/NousResearch/hermes-agent), built by Nous Research and released under the MIT licence. Hermes is not ours. This guide is a walkthrough for using it for lead sourcing. Not affiliated with or endorsed by Nous Research, Telegram, Hunter, ZoomInfo or Smartlead.

## Sources

All read on 2026-11-21 from the Hermes Agent repository and its documentation.

- https://github.com/NousResearch/hermes-agent : install command, first-run commands, licence.
- https://hermes-agent.nousresearch.com/docs/getting-started/quickstart : install paths, `hermes setup`, the 64,000 token minimum, the "Free, start now" option, `hermes doctor`, `hermes tools`.
- https://hermes-agent.nousresearch.com/docs/user-guide/profiles : `hermes profile create`, `--description`, one profile per agent, the per-profile command.
- https://hermes-agent.nousresearch.com/docs/user-guide/features/personality : `SOUL.md`, where it lives and what belongs in it.
- https://hermes-agent.nousresearch.com/docs/user-guide/which-file-does-what : `SOUL.md` versus `AGENTS.md`.
- https://hermes-agent.nousresearch.com/docs/user-guide/features/web-search : web search and page extraction, the provider list.
- https://hermes-agent.nousresearch.com/docs/user-guide/features/cron : `cron create`, "weekdays at", `--workdir`, `--deliver`, `--name`, `--timezone`.
- https://hermes-agent.nousresearch.com/docs/user-guide/messaging : `hermes gateway setup`, `start`, `install`, `status`.
- https://hermes-agent.nousresearch.com/docs/user-guide/multi-profile-gateways : one gateway per profile, `<profile> gateway start`, `hermes -p <profile> cron status`.
