# Verify mode (Check what is done)

Refresh account's verify mode: "check Acme", "what's done on Acme", "is the Dallas quote still open?", "check Acme's items" (one account). The fact checker: every open thing on the account (open action items, unticked open questions, Signals-to-watch lines whose day has come, storylines not resolved) checked against every source in the owner's order, and one outcome for each. It is also the **last step of the "with email" modes** (call mode with email, email mode, "refresh Acme with email": the email is already read, so most checks cost nothing more), the **item checks of every mode that updates action items** (0.54.0, `scope: "items"`) and the **first step of a book run's refresh** (the Work through the book skill).

Everything is a suggestion the owner approves in CodyNote › Claude suggestions: that is the approval, so **never stop to ask first** (0.49.0). **Never close anything without proof** (owner, 2026-11-01): no source that shows it done, it stays open and the check says what was searched.

**For field X, follow its recipe** (`field-guide.md` next to this file; `codynote_recipe(id)` has the whole recipe): an item is `verify.item`, an open question `verify.question`, a Signals-to-watch line `verify.watch`, a storyline `verify.storyline`. Nothing is saved except **suggestions** through `codynote_suggest_changes`; HubSpot is read-only; Outlook and Teams are read here because checking is what the owner asked for (a "check" names the account, so its email and chats are in scope); never send anything.

## 1. Open the verify brief

`codynote_get_verify_brief(company)`; with a scope when the owner named one: `scope: "items"` ("check Acme's items"), `"questions"`, `"watch"`, `"storylines"`, or one thing's id ("is the Dallas quote still open?": find it with `codynote_map` first, then `scope: <its id>`). The brief gives:

- `how` and `proof`: the rules, once;
- `plan`: the order (`codynote → fathom → outlook → teams → hubspot`, 0.54.0), the Fathom domains and `since`, the Outlook addresses and domains, the Teams query and teammates, the HubSpot company id (or what to search for), its objects and how to pass tickets; each source's `since` is the oldest of the things' own (the day each source last checked it, else when it opened);
- `things`: each with its `id`, `kind`, `text`, `opened`, its own boxes (an item's `due`, `status`, `waiting_since`, `follow_ups`, `situation` (0.52.0; read it first), `goal` (old items only), `done_when`, `next_step`; a storyline's `status`, `now` with its date), `summary` (0.54.0), `last_checked`, and `search`: per source its own `since`, the meetings in CodyNote since then (ids), the addresses of the people it names (`outlook`), its HubSpot ticket ids;
- `waiting`: a check already waits for the owner: **skip that thing**; `found_by_codynote`: done CodyNote found by itself (a deal won for a PO / DocuSign / order form / quote item, a later meeting for a "book a call" item), waiting for the owner: skip those too.

Nothing open: say "<Company>: nothing open to check." and stop.

## 2. Read every source once, in order, and match against every thing

Read each source **once for the whole account** (since the plan's `since` for that source), then match what you found against every open thing, using each thing's recipe evidence. Item checks (0.54.0) read every source every time, not only until one settles it, because something new in a later source can change an item:

1. **CodyNote** (free): the meetings in `search.codynote` with `codynote_read(meeting_id, find: "<1-3 key words>")`, the email references (`codynote_search`), the item's Context lines.
2. **Fathom**: `codynote_fathom_missing(company, since: <plan.fathom.since>)` for the calls not in CodyNote yet: their titles, dates and summaries.
3. **Outlook**: `outlook_email_search` by the plan's addresses and domains (a thing's `search.outlook` first) since its `since`: a sent message that does the thing, or a reply that answers it. Read a body only for the thread that matters; quote that message. Read every time, "with email" or not.
4. **Teams**: search the owner's Teams chats for the account (`plan.teams.query`) and its teammates (`plan.teams.people`) since its `since`: a message that does the thing or answers it. Proof is the message: `evidence: {kind: "teams", id, url, date}`.
5. **HubSpot**: everything new on the company since its `since` (the plan's `objects`: tickets, deals, notes and activity), the tickets in `search.hubspot_tickets`, the deal's stage. **Tickets in one call:** pass every ticket you fetched, as HubSpot returns them, to `codynote_hubspot_deals(company, raw: {tickets})`: CodyNote suggests done for each closed ticket linked to an open item (with the ticket as proof); skip those items.

**A source that can't be read is skipped and said.** Teams and HubSpot may not be connected or allowed on this computer: try each; when there is no tool, no permission or an error, skip it, put `{source, looked_for: "skipped: <why>"}` in each check's `searched`, say it in the reply's one **Skipped** line, and carry on. Never stop the run for a missing source.

What counts is in the recipe (`lookFor`): the thing happened (the email sent with the quote, the reply that answers the question, the ticket closed, the meeting held), not a promise to do it.

## 3. Suggest one outcome per thing, in one pass

One `codynote_suggest_changes` call with every check (no `scan_id`: this isn't an update), each `{action: "add", record: "verification", company, id: <the thing's id>, fields: {result, searched, evidence?, note?, now?}, reason}` (`codynote_describe("write:verification")` has the shape). `searched` lists each source you looked at, in order, with what you looked for (`{source: "outlook", looked_for: "Dana since Oct 2"}`), skipped ones included. **Quiet checks (0.54.0):** a `still_open` or `unclear` check is saved straight away as the item's check (the answer's `checks_recorded`), not a suggestion; only `done` and `no_longer_true` wait for the owner. The four outcomes:

- **done with proof:** `result: "done"` with `evidence: {kind: meeting | document | email | teams | hubspot, id, title, url?, date}`: a meeting, document or email of the account by its CodyNote id, a Teams message (its id, link and date), or a HubSpot record as `<object type>:<id>` (`ticket:4410`). Approving closes the item (done, with that proof) or ticks the question. Never on a storyline (resolving one is a `bullet` change to `status: "resolved"` with its `outcome`).
- **no longer true:** `result: "no_longer_true"` with the `evidence` that shows it: a Signals-to-watch line is marked settled (it stays on the card); a storyline also gets `now`, its Now rewritten (`storyline.now` recipe: 1-3 sentences, at most 600 characters). Never on an item: one simply no longer wanted is a `remove` of the `action_item` with the reason (the owner approves it), or say so in the reply.
- **still open:** `result: "still_open"` with `searched`, and the newest thing found as `note` ("Dana replied Oct 28, no date yet") or `evidence`. Saved quietly as the check; nothing for the owner to approve.
- **unclear:** `result: "unclear"` with `note`: what would settle it ("ask Dana on Friday's call"). Saved quietly too.

**The Summary (0.54.0).** In the same call, an item whose `summary` is empty or no longer matches what the sources show gets one: a `box` change to `summary`, following recipe `item.summary`.

Give each change a `confidence` (`high`, `medium` or `low`: how sure you are), and copy every quote exactly as it is in the source. One check per thing; a dropped one: keep the rest and say what and why. Never close without proof: a "probably done" is `still_open` with what you searched.

**Last, before the reply (0.53.0): the cleanup.** Bring this account's open items that do not meet the rules up to them: a "Topic: what exactly, how much, for whom" title, a three-part Situation (The ask / Where it stands / Why it matters), a next step who / what / when, folds, follow-ups. Follow `mode-items.md` (its steps 2 and 3, rounds of at most 12 cards, one `codynote_suggest_changes` call per round without `scan_id` once the scan is closed). This is the account-by-account cleanup; there is no separate cleanup skill. Items that already pass get nothing; say what changed in the reply.

## 4. Reply

```
**Checked <Company>** · <n> things · <n> done · <n> still open · <n> no longer true · <n> unclear
Waiting in CodyNote › Claude suggestions: <n> changes · <n> checks recorded · <n> Summaries
- Done: "<item>" (<proof, date>)
- No longer true: "<line or storyline>" (<source>)
- Unclear: "<thing>": <what would settle it>
Still open: <one line naming them, the oldest first>
Skipped: <source>: <why> (only when a source was skipped)
```

When run as the last step of another mode (the item checks), add one line to that mode's reply instead: `**Checked:** <n> done · <n> still open · <n> unclear` (and `skipped: <source>` when one was), the done ones by name.
