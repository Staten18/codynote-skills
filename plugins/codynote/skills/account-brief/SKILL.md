---
name: account-brief
description: Brief the owner (a Panzura CSM) on one customer account from CodyNote, HubSpot and Fathom. Before a call - the read, signals, who's who, environment, what changed, storylines, open items, deals and what to raise. In QBR mode - prep for a QBR, business review or roadmap call - the quarter's story, who will be in the room, a draft agenda, questions and a to-do list (with "with email", the quarter's Outlook threads too). Handles MSPs. Use when the owner asks "what's going on with Acme?", "prep me for my call with Acme", "brief me on Acme", "refresh me on Acme", "what's the read on Acme?", "brief me on Acme for the QBR", "prep the Acme QBR", "QBR prep for Acme", "get me ready for the Acme business review / EBR / roadmap / QBR + Roadmap" (any customer), or similar. Read-only.
---

# Account brief

Before a call, or any time the owner wants to know where an account stands: one screen that starts from the account's stored read, says what has moved since, and thinks about it.

This skill only reads. Never save, suggest or change anything in CodyNote or HubSpot, and never send anything. If the owner wants changes afterwards, that's a separate ask. It never reads Outlook (QBR mode does, only when the owner adds "with email"): email comes from what CodyNote has indexed.

Read `account-picture.md` (next to this file) for what the read, sentiment and storylines are, and "The book and the lint" for the signals, the lint kinds and the Ring environment fields; `cards.md` for the boxes on storylines and action items.

**First, once:** `codynote_ping(skills: "0.55.0")` (the `skills` value tells CodyNote which skills Claude has). Its `version` must be **0.55.0** or later (an older CodyNote answers in one line, without `version` and `tools`): if it is older, say "Update CodyNote to 0.55.0 or later, then ask again" and stop. Call only the tools in its `tools`.

**QBR mode.** For a QBR, business review or roadmap call ("brief me on Acme for the QBR", "prep the Acme QBR", "QBR prep for Acme", "get me ready for the Acme business review / EBR / roadmap / QBR + Roadmap", any of these "with email"), read `mode-qbr.md` (next to this file) and follow it instead of the steps below. Everything else on this page is the brief before an ordinary call.

## 1. Gather (read everything before writing anything)

**CodyNote** (the owner's notebook; the main source):
1. `codynote_get_customer` with the name. If it matches more than one account, or none, ask which one and stop. Start from **`summary`** (0.50.0, first in the answer): CodyNote's executive summary, worked out from the records: first (0.52.0) the read's **Do first** (the one thing to do and why, then the next two) and **Situation** (the account in two or three short paragraphs; "not written yet" when the read predates 0.52.0: then write the brief's Situation yourself from the rest, marked "my read"), then the account's **Goal** (what they want from Panzura this year) and **Next meeting**, health and why (with the storylines and items the read cites as `[3f2a9c1d]`), signals, money, environment, work (past due, next steps, waiting on them, no next step, decisions for the owner), the story (each open storyline in one line, "no goal yet" when its boxes are empty) and what changed since the read. It is the brief's backbone: use its numbers as they are, never redo them. Then note `overview.company_id`, **the read** (its Do first and Situation, its five short lines, health and date), the owner's **sentiment**, the **storylines** (kind, status, watch date, Now, last History line, Open questions not yet ticked, Hurts and Signals to watch lines), the facts, the newest meeting (`meetings`: `meeting_id`, `held_on`, `title`), renewal date and ARR, both $/TB, owner and RSM, MC used vs licensed (`environment_history` for the trend), each Ring's environment (`rings`: CloudFS version, nodes, object store, hypervisor, HA), `needs_a_look`, `signals` (this account's signals as the book works them out: each with its tone and reason, `needs_attention` and `why`), `still_true` (storylines and bullets nothing has mentioned for 6 months, not resolved and not on watch until a later date), `days_since_touch`, products, deals, open and waiting items (each with its Situation, read first, and Next step; Goal only on an old item with no Situation), `contacts` (role, `last_met_on`, `last_emailed_on`), `people` (each contact's and teammate's persona `role`, `sentiment`, `background`, `personality`, `interests`, `sore_spots`, newest `observations`, `last_met_on`, `last_emailed_on`), the newest 3 `emails` (date, subject, people, gist) and `emails_through`. Managed accounts: `managed_by` (the MSP that manages this account, if any) with `managed_by_people` (the MSP's contacts with their profiles, read from the MSP, each marked `from: <MSP>`, those still there first), and on an MSP `manages_count` and `manages` (one line per account it manages: health, sentiment, next renewal and its ARR, open and waiting items, last touch, `needs_attention` and `why`).
2. **The map:** `codynote_map` with the company: one line per record (meetings with their notes and transcript words, documents, items, deals, storylines, emails, people), ids and dates, no text; pick what to open from it.
3. `codynote_changes` with the company and `since` = the read's date (no read: the last meeting's date; neither: leave it out).
4. The last meeting's notes and next steps: `codynote_read` on the newest meeting's id with `find` for next steps, problems or concerns (the passages, not the whole transcript); the whole meeting only when it has no transcript or the passages don't answer.
5. "Still true?": the briefing's `still_true` (each with its `last_mentioned` date).
6. **An MSP** (the briefing has `manages`): the MSP is an ordinary account of the owner's that manages others (a Panzura partner); the managed accounts keep their own deals, items, meetings and contacts. The `manages` lines are enough for most; call `codynote_get_customer` on a managed account only when it needs attention or renews within 90 days (at most 3), for its read, Now lines and open items.

**HubSpot** (read only; skip quietly if not connected): find the company, then open support tickets (subject, opened, status, priority); emails, calls, meetings and notes logged since the read's date; open deals. A close date, amount or stage that differs from CodyNote's goes under Watch out as a mismatch; don't decide which is right.

**Fathom** (skip quietly if not connected): a call newer than CodyNote's last meeting is the latest call; read its summary.

## 2. Think before writing

- **Does the read still hold?** Weigh what changed since its date (meetings, items done or overdue, deals, tickets, MC readings, storylines opened or resolved) and the signals that are bad or on watch against its health and lines. Say whether you'd keep or change the health, and why ("my read"). If the owner's sentiment disagrees with the read, say so in a few words.
- **Connections:** facts that share a cause or affect each other (a ticket and a stalled ask at the same site; capacity growth vs the renewal date).
- **Who's who:** from `people` (contacts still there; those no longer there, `active: false`, are only named, from `inactive_people`), the people who matter for this call (a persona set, or met or emailed in the last year; at most 6, the decision makers first): the one thing to know about each (from their sentiment, background and newest Observations), an opener from their Interests and anything in Sore spots to steer around. Someone negative, or a champion gone quiet, is worth a line in My read now. With no profiles yet, list the contacts with a role, met and emailed dates. Someone on more than one account (`accounts` on their entry; `linked: true` when this account isn't their home): end their line with "also on <the other accounts>" (a partner's account executive who covers several of the owner's accounts is worth knowing before the call).
- **An MSP:** reason about its managed accounts together, not one by one: renewals that bunch up, the same MSP people across them, which need attention and whether one cause runs through them (the MSP's engagement, a shared project, the same ticket). Its own deals and items are its own: say "with managed accounts" whenever you add their ARR to the MSP's.
- **A managed account:** say who manages it. The MSP's people (`managed_by_people`) are people on this account too: include those who matter in Who's who after the account's own, marked with the MSP, and count them when you say who's been met or emailed.
- **The one thing** this call must achieve, and why it outranks the rest.
- **The goal** (0.50.0): weigh the storylines and deals against the account's Goal. No goal yet, or storylines that say "no goal yet": asking what they want from Panzura this year (or from that storyline) is worth raising on this call. A next meeting that has passed: say so in one line.
- **Where Panzura fits:** match what's worth raising to `panzura.md` (the product, what it needs first, a customer story).

Ground every judgment in a fact you found; mark inferences "my read"; say what you couldn't see. With no stored read, work out the health yourself (engagement, renewal timing, tickets, capacity vs licensed, competitors, products bought but not set up, past-due items) and say it's yours.

## 3. Write

One screen, about 25-40 lines, in this order. Short lines; every fact names its source in a few words. Dates "Oct 9"; money "$84k"; waits "12 days".

```
**<Company>** · renews <date> · <renewal ARR> ARR · <CloudFS $/TB>/TB · RSM: <name>

**Summary** (CodyNote's, from `summary`)
Do first: <the summary's Do first line, or "not written yet">
Situation: <the summary's Situation, as it is; or your own two or three sentences from the rest, marked "my read">
Wants this year: <the goal, or "not set: ask what they want from Panzura this year"> · Next meeting: <date (in N days) | past: update | not set>
Work: <n> open · <n> waiting · <n> past due<; decisions for you: <the summary's, short>>

**The read** (<date>): **<Healthy | Watch | At risk>** · your sentiment: <green | yellow | red, date, or "not set">
> <Health and why>
> <Renewal> · <What's changing> · <Risks and openings>
> Next: <the one thing to do next>
**Signals:** <the bad and watch ones, from `signals`: "No touch 34 days · renewal in 80 days, no opp · MC 91 %"; missing ones as "no sentiment", "no MC reading"; or "all fine">

**Who's who**
- <name> (<persona>) · <positive | neutral | negative, or "no sentiment yet"> · met <date> · emailed <date>: <the one thing to know about them now>. Open with: <an interest>. Avoid: <a sore spot>. <also on <other accounts>, when on more than one>
- With you: <teammates on the account: name (what they do here)>
- <MSP person> (<MSP>, <persona>) · ...: as above (a managed account: the MSP's people who matter, from `managed_by_people`)
- No longer there: <names from `inactive_people`, in one line>
**Environment:** <per Ring: <Ring>: CloudFS <version> · <n> nodes · <object store> · <hypervisor> · HA <notes>; "not recorded" for what's empty>
**Since then:** <one or two lines: meetings, items, deals, tickets, MC; or "nothing new">
**My read now:** <same health, or "I'd move it to <X>": why.>
**Get out of this call:** <the one thing, and why.>

**Storylines** (open and watch first)
- [<Kind>] <line>: <its line from the summary's story: Goal (or "no goal yet") · status · Now, short · next step · money · moved> · <n> open items
- Resolved since the read: <line> (<outcome>)

**Since your last call (<date>, <meeting title>)**
- <what changed: deals and stages, tickets, logged emails, documents, MC readings, items done>
- Email: <the newest 3 from CodyNote: <date> · <subject> · <gist>>

**Needs a look in CodyNote**
- <title>: <detail, short> (one line per `needs_a_look` row)

**Open between you**
- You owe: <item> (due <date>, **past due** if so) · next: <its Next step>
- Waiting on <person>: <item>, **<n> days**

**Deals**
- <deal name>: <stage> · <forecast> · closes <date> · <ARR>

**Watch out**
- <open tickets, competitor mentions, still-true items ("hasn't come up in 8 months: still true?") not already under Needs a look, HubSpot mismatches>

**Worth raising on this call**
1. <topic, and why now (an Open question the call can answer counts)>
2. ...
```

On a managed account the header reads `**<Company>** · managed by <MSP> · renews ...`.

On an MSP, open with the header and **Who's who** of the MSP itself (its own people; teammates on it), then its managed accounts, then the MSP's own read, signals and the rest as above:

```
**Managed accounts** (<n>)
| Account | Health | Sentiment | Renews | Renewal ARR | Open / waiting | Last touch | Needs attention |
| --- | --- | --- | --- | --- | --- | --- | --- |
| <name> | <Healthy / Watch / At risk / no read> | <green / yellow / red / not set> | <date> | <$k> | <n> / <n> | <date> | <why, short, or "-"> |
**Across them:** <one or two lines: renewals that bunch up, the people they share, the one that needs the owner first and why; "with managed accounts" renewal ARR if it helps>
```

With no stored read, the top block is **The read (mine, not saved)** with the health and 2-4 sentences. Leave out other sections with nothing in them; leave out an Open with or Avoid that the profile doesn't have, and the No longer there line when nobody has left. Don't include the full Overview, license lines, deal history or uninvolved contacts.

If `emails_through` is missing or older than 14 days, end with: "Email read through <date> (or: Email not read yet): say 'catch up on email for <Company>' to refresh." If most people on the account have no profile yet, end with: "Say 'profile the people at <Company>' to build them." If the read is older than the newest meeting (or there is none), end with one line: "The read is from <date> (or: There's no saved read yet): say 'refresh <Company>' or 'wrap up my <Company> call' to refresh it."
