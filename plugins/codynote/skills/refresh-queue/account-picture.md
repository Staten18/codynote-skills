# The account picture (CodyNote 0.27.0 and later)

What each company holds in CodyNote, who writes what, the book's signals and lint (0.28.0), and the exact shapes of Claude's suggestions. The skills read this once instead of repeating it. `codynote_get_update_brief` is the live rulebook for the installed CodyNote: where it and this file differ, follow the brief.

Claude only **suggests**; the owner approves each suggestion in CodyNote › Claude suggestions. Nothing here runs on its own: Claude reads and suggests only when the owner asks.

## The model

| Part | What it is | Who writes it |
|---|---|---|
| **The read** | Five lines: **Health and why** · **Renewal** · **What's changing** · **Risks and openings** · **The one thing to do next**. Health is `healthy`, `watch` or `at_risk`. Dated; old versions kept. | Claude suggests a whole new version; the owner approves. |
| **Sentiment** | The company's: the owner's own green / yellow / red, with a reason and a date. | **Owner only.** Claude quotes it, never suggests it. If it disagrees with the facts, say so in the reply. (A person's sentiment to Panzura is a box on their profile: "People" below.) |
| **Storylines** | The Projects and Issues bullets. Each has a **kind** (`project`, `issue`, `opportunity`, `risk`), a **status** (`open`; `watch` with a date it comes back; `resolved` with an **outcome** `worked`, `dropped` or `irrelevant`), one line, a **Now** paragraph, dated **History** lines with sources, **Helps** / **Hurts** / **Watch** lines, **Open questions** (a checklist) and the owner's **Notes** (the card's boxes: `cards.md`). | Both. One suggestion per change. |
| **Company facts** | `revenue`, `employees`, `offices`, `hq`, `industry`, `founded`, `website`, `acquisitions`, `other` (with a `label`). Each with a source link and a date. | Claude fills **blanks only**; the owner's values win. |
| **Knowledge** | Tech stack, Workloads, Use cases: bullets, as before. | Through the update brief and `codynote_suggest_changes` (the brief's bullet actions). |

`codynote_get_customer` returns the current read, the sentiment, the storylines (kind, status, Now, last History line, ids) and the facts. Storyline History in full: `codynote_read` on its `bullet_id`; old reads: `v_reads`.

## The book and the lint (0.28.0)

CodyNote works these out when asked; nothing runs on its own.

**Signals**, one per account, each on its own (never one score): **touch**, **renewal**, **capacity** (MC), **waiting**, **sentiment**, **risks**, **read**, **qbr**, **meeting** (0.50.0: the next meeting). `codynote_book` gives each a tone (`ok`, `watch`, `bad`) and a reason, and `why` (the bad ones joined) with the owner's saved rules; `codynote_get_customer` gives one account's the same way, as `signals`. **Only current accounts** (`account_group` `current`) can need attention; other accounts keep their signals but never a `why`. **Bad** (needs attention), with the default rules:
- touch: no meeting or follow-up in over **60 days** (or never); History lines and reads are notes, not touches;
- renewal: the next renewal **0-120 days** out with **no open opp**;
- capacity: MC used **≥ 85 %** of licensed (each Ring's latest reading, summed);
- sentiment: red, or older than **30 days** (**7** when the renewal is within 60 days); never set only with the renewal **0-120 days** out (watch otherwise);
- risks: a risk storyline open;
- read: at risk; none, or older than the last meeting, only with the renewal **0-120 days** out (watch otherwise).

Waiting is never bad (watch once the longest wait reaches 14 days). Nor is qbr, a nudge: watch when the last QBR is over **120 days** ago, or a current account has none on record. Nor is meeting (0.50.0): watch when the account's next meeting (`next_meeting_on`) is **0-7 days** away (prep), muted when it is not set or has passed ("past: update").

**The executive summary** (0.50.0): `codynote_get_customer` and the update brief open with `summary`, compact text CodyNote works out: first (0.52.0) the read's **Do first** (the one thing to do and why, then the next two) and **Situation** (the account in two or three short paragraphs; "not written yet" on an account whose read predates 0.52.0), then the account's **Goal** (what they want from Panzura this year) and **Next meeting**, health and why (the read's Health line, with the storylines and items it cites as `[3f2a9c1d]`), the signals, money, environment, work (next steps, waiting on them, past due, no next step, decisions for the owner), the story (each open storyline in one line: Goal · status · Now · next step · target · owner · money · moved · open questions; "no goal yet" when its boxes are empty) and what changed since the read. Read it first; never copy it into a box. Claude's parts are the narrative ones, each through its recipe: the read's Do first and Situation (`read.do_first`, `read.situation`, written with every read), the read's Health line (`summary.why`: cite storylines and items by id in brackets), a storyline's one line and Goal (`summary.storyline_line`), and the account's `goal` and `next_meeting_on` (`company.goal`, `company.next_meeting_on`).

**Needs a look** (`needs_a_look` in `codynote_get_customer` and the update brief; `v_lint` across the book: `company_id, company, kind, ref_type, ref_id, title, detail, since`). What each kind means and what fixes it:

| Kind | Means | Fix |
|---|---|---|
| `watch_due` | A Watch date has come | Storyline change: `open`, a new `watchUntil`, or resolve |
| `storyline_stale` | Open, nothing new in 90 days | Ask on the next call; a History line or resolve if the material says |
| `now_behind` | Now is older than its newest History line | Rewrite `nowText` |
| `fact_old` | A fact that changes (revenue, employees, offices, acquisitions, other) over a year old | Claude's: a `change` with a fresh source; the owner's: say it in the reply |
| `source_gone` | An unresolved storyline's source meeting or document is in Trash | Say it; the owner restores it or re-sources the storyline |
| `read_behind` | The read is older than the last meeting | A new read |
| `sentiment_stale` | The owner's sentiment is old | Remind the owner; never suggest it |
| `ring_license_mismatch` | The Rings' latest MC licensed TB (DR and test Rings left out) differs from the current renewals' CloudFS TB | Say both numbers; never change a reading to match |
| `item_unverified` | An open item not checked in 30 days (from the newer of when it opened and its last check; 0.49.0) | Verify mode: a `verification` (done with proof, still_open, unclear) |
| `watch_line_due` | A Signals-to-watch line whose day has passed, not checked since (0.49.0) | Verify mode: what happened, as a `verification` on the line (no_longer_true with a source, or still_open) |
| `promise_overdue` | An item the owner owes (not a teammate's, not waiting) past due, with no check that found the email sent (0.49.0) | Verify mode: look for the sent email; done with it as proof, else still_open and say it in the reply |

**Environment on Rings:** `codynote_get_customer` `rings` carry `ring_id`, `cloudfs_version`, `node_count`, `object_store`, `hypervisor`, `ha_notes` with the MC reading (empty ones left out). Fill only what a call or document said, as said ("8.5.1", not "8.5"); never guess the newest CloudFS version.

## Formats

- **The read's lines:** the five short ones one or two sentences each, **200 characters at most**; since 0.52.0 also `do_first` (one sentence, **300 at most**: the one thing to do and why, then "Then: …" with the next two, recipe `read.do_first`) and `situation` (two or three short paragraphs, **1,500 at most**: who they are and how they use Panzura, where the renewal stands, what is open and who owes what, the openings; recipe `read.situation`), both written with every read. Reason over the whole account (storylines, deals, MC, open items, the latest call), not only the last call. Health gets a reason. Anything inferred is marked "my read".
- **Now:** the Now box's rule in `cards.md` (the one CodyNote checks). Rewritten whole each time (never appended to); the app dates it.
- **History line:** one past-tense sentence of what happened, dated the day it happened (not today), sourced. History is **only appended**: never reword, merge or drop a line.
- **The card's other boxes** (a storyline's Helps, Hurts, Signals to watch and Open questions; an action item's Situation (0.52.0), Context, Done when and Next step, and its Goal only on an old item with no Situation yet): see `cards.md`, generated from the app's own list, for each box, its limits and the `box` / `line` / `tick` / `split` shapes.
- **Resolve** when it's over: `worked` (done, or it went our way), `dropped` (they stopped, or it went against us), `irrelevant` (no longer matters). **Watch** when waiting on something with a date (a decision, a budget cycle, a go-live), with `watchUntil` = that date.

## Suggestion shapes

**The read** goes only in `close_scan` (the last step of an update, `work-order.md`): `close_scan: {company, coverage, read}` on the last `codynote_suggest_changes` call, with the brief's `scan_id` and `coverage` for every step; sent as a change in `changes`, it is refused. The other records here (fact, history, ring, bullet changes and new storylines) go as items in `changes`. Each has `action` (`add` | `change`), `record`, `company`, `id` (the storyline's `bullet_id` on a change), `fields`, `reason` (one line), `sources` (at least one: `{kind: "meeting", id, title, date, quote}` for a call; `{kind: "other", title, url, date}` for a web page). Examples use made-up names.

**The read** (in `close_scan` as its `read`, where `action` and `record` may be left out; one per scan, only when something material changed):
```json
{"action": "add", "record": "read", "company": "Acme Engineering",
 "fields": {"health": "watch", "asOf": "2026-10-28", "lines": {
   "health": "Watch: engaged on calls, but the Revit slowness ticket is 6 weeks old (my read: it colours the renewal).",
   "renewal": "Renews Mar 31; renewal opp in Proposal, flat on 120 TB.",
   "changing": "New Dallas office opening in Q1; IT lead says file shares there are 'the plan'.",
   "risks": "Risk: Egnyte came up as an option for Dallas. Opening: MC at 88% of licensed before Dallas.",
   "next": "Get the slowness ticket closed and a capacity quote for Dallas before the Jan budget meeting.",
   "do_first": "Get the Revit slowness ticket closed this week: it colours the March renewal more than price does. Then: send the Dallas capacity quote; book the Q1 EBR.",
   "situation": "Acme is a 400-person engineering firm on CloudFS (the Panzura file system) across three offices, 120 TB licensed, with Search & Audit added last year.\n\nThe renewal ends Mar 31; the renewal opp is in Proposal, flat, and Dallas (a fourth office opening in January) would add about 20 TB. Egnyte came up as an option for Dallas.\n\nOpen: Dana owes a decision on the Dallas quote after the Jan 12 budget meeting; we owe a fix on the Revit slowness ticket, six weeks old. My read: the ticket colours the renewal more than the price does."}},
 "reason": "Oct 28 call changed the renewal picture.", "sources": [{"kind": "meeting", "id": "<meeting_id>", "title": "Acme sync", "date": "2026-10-28"}]}
```

**A History line** on an existing storyline:
```json
{"action": "add", "record": "history", "company": "Acme Engineering",
 "fields": {"bulletId": "<bullet_id>", "onDate": "2026-10-28", "text": "IT lead confirmed the Dallas office opens Feb 1.",
   "source": {"type": "meeting", "refId": "<meeting_id>", "title": "Acme sync"}},
 "reason": "New date for the Dallas project.", "sources": [{"kind": "meeting", "id": "<meeting_id>", "title": "Acme sync", "date": "2026-10-28"}]}
```

**A storyline change** (any of `kind`, `status`, `watchUntil`, `outcome`, `nowText`, `text`). `text` and a new storyline need a CodyNote source (a meeting or document); the tool drops them otherwise. Kind, status, watch and Now may come from anything, including HubSpot or email. Helps / Hurts / Signals to watch lines and Open questions on an existing storyline are `line` and `tick` records (`cards.md`):
```json
{"action": "change", "record": "bullet", "company": "Acme Engineering", "id": "<bullet_id>",
 "fields": {"status": "resolved", "outcome": "worked", "nowText": "Snapshot alerts stopped after the 8.7.1 upgrade on Oct 20; IT confirmed on the Oct 28 call."},
 "reason": "They said it's fixed.", "sources": [{"kind": "meeting", "id": "<meeting_id>", "title": "Acme sync", "date": "2026-10-28", "quote": "No alerts since the upgrade"}]}
```
Watch: `{"status": "watch", "watchUntil": "2027-01-15", "nowText": "..."}`. Back to open: `{"status": "open"}`.

**A new storyline** (section `projects` for a project or opportunity, `issues` for an issue or risk):
```json
{"action": "add", "record": "bullet", "company": "Acme Engineering",
 "fields": {"section": "projects", "kind": "opportunity", "text": "Dallas office on CloudFS",
   "nowText": "Opening Feb 1 with about 40 users; IT is choosing between CloudFS and Egnyte.",
   "history": [{"onDate": "2026-10-28", "text": "IT lead raised the Dallas office and the Egnyte option.",
     "source": {"type": "meeting", "refId": "<meeting_id>", "title": "Acme sync"}}],
   "lines": {
     "helps": [{"text": "Another site on their existing ring: one file system across offices, no new silo.", "date": "2026-10-28"}],
     "hurts": [{"text": "IT likes that Egnyte needs no hardware in Dallas.", "date": "2026-10-28"}],
     "watch": [{"text": "The Jan budget meeting decides Dallas storage.", "date": "2026-10-28"}],
     "open_questions": [{"text": "Who decides for Dallas, and by when?"}]}},
 "reason": "New office raised on the call.", "sources": [{"kind": "meeting", "id": "<meeting_id>", "title": "Acme sync", "date": "2026-10-28"}]}
```

**A company fact** (only a field the brief says the owner hasn't typed; `change` only for one Claude filled earlier):
```json
{"action": "add", "record": "fact", "company": "Acme Engineering",
 "fields": {"field": "employees", "value": "about 1,200", "sourceTitle": "Acme Engineering: About us", "sourceUrl": "https://example.com/about", "asOf": "2026-10-28"},
 "reason": "Blank; from their website.", "sources": [{"kind": "other", "title": "Acme Engineering: About us", "url": "https://example.com/about", "date": "2026-10-28"}]}
```
`field: "other"` needs a `label`, one of: parent company, ticker, ENR rank, fiscal year end, stock exchange, former name (CodyNote refuses others; a project belongs to a storyline).

**A Ring's environment** (`id` = the Ring's `ring_id`; only the fields the material gave):
```json
{"action": "change", "record": "ring", "company": "Acme Engineering", "id": "<ring_id>",
 "fields": {"cloudfsVersion": "8.5.1", "nodeCount": 4, "objectStore": "Azure Blob (East US)", "hypervisor": "VMware ESXi 8", "haNotes": "HA pair at each site"},
 "reason": "Their version and setup, from the Oct 28 call.", "sources": [{"kind": "meeting", "id": "<meeting_id>", "title": "Acme sync", "date": "2026-10-28", "quote": "we're on 8.5.1 on all four nodes"}]}
```

**Email and people (0.29.0):** `email.md` has when email is read (the owner's word only), the search recipe and the three new records: `email` (one reference per email, with the `email_sync` watermark), `follow_up` (a waiting tag from sent mail) and `attendee` (a meeting attendee matched to a contact); a contact's `role` goes on a `contact` change. Email can be the source of a History line, Now, status, fact, contact or item.

## People (0.31.0)

Every contact and every teammate who appears on an account has a **profile**: the person card in `cards.md` (Sentiment to Panzura, Background, Personality as one box each; Interests and small talk, Sore spots and Observations as lines; Notes the owner's). Observations are the evidence, one dated, sourced line per thing noticed in a call or email; the other boxes are built from them. `codynote_get_customer` returns `people` (`person_id`, `kind` contact or teammate, the boxes, the newest 5 Observations, `last_met_on`, `last_emailed_on`); `codynote_read` (`person_id`, or `name`) the whole card with line ids, dates and sources, their meetings and email references.

- **Profile lines and boxes:** `box` / `line` records with `card: "person"`, `id` = the `person_id` (the contact's or teammate's id) and the keys in `cards.md`. Each carries a source with a short quote. A person's `sentiment` box may be suggested (with a quote); the company's sentiment never.
- **Persona:** a `contact` change with `fields: {role}`, one of `champion`, `economic_buyer`, `technical_buyer`, `decision_maker`, `executive_sponsor`, `coach`, `user`, `blocker`, `admin`, `billing`, `partner`, `other`. Contacts only: a teammate has no persona.
- **Reports to:** a `contact` change with `fields: {reports_to_contact_id}`, another contact on the account.
- **Speakers:** OneNote transcripts say "Speaker 1", "Speaker 2". `codynote_read` on the meeting serves the text with names where the owner (or an approved suggestion) named them, and `speakers`: each label with `name` (null when unnamed), `person_id`, `turns` and the `first` things it said. Name a label only when the transcript makes it plain ("Thanks, Dana" right after Speaker 2 spoke; "This is Dana from IT"); the reason quotes that line. `contact_id` for a contact, `teammate_id` for a teammate, `is_owner: true` for the owner. The stored transcript is never rewritten.

```json
{"action": "change", "record": "speaker", "company": "Acme Engineering", "meeting_id": "<meeting_id>",
 "fields": {"label": "Speaker 2", "name": "Dana Whitfield", "contact_id": "<contact_id>"},
 "reason": "Sam answers Speaker 2 with \"Thanks, Dana\" at 00:14:32.", "sources": [{"kind": "meeting", "id": "<meeting_id>", "title": "Acme sync", "date": "2026-10-28", "quote": "Thanks, Dana, that helps"}]}
```

```json
{"action": "add", "record": "line", "company": "Acme Engineering", "card": "person", "id": "<contact_id>",
 "fields": {"key": "observations", "text": "Said she signs off on the renewal and wants the Dallas quote before the Jan budget meeting.", "date": "2026-10-28"},
 "reason": "What Dana owns, from the Oct 28 call.", "sources": [{"kind": "meeting", "id": "<meeting_id>", "title": "Acme sync", "date": "2026-10-28", "quote": "the renewal is my sign-off"}]}
```

## Day to day (0.32.0)

**A joint QBR + Roadmap** is a meeting of kind `qbr_roadmap`: one session that counts as **both** the last QBR and the last roadmap. `overview.last_qbr_shown` and `last_roadmap_shown` both follow it, and `codynote_changes` counts from it for "since the last QBR" and for `since_last: "roadmap"`. Read it for both parts: the review and the asks (the QBR), what Panzura showed and how they took it (the roadmap). As `codynote_get_customer` lists it in `meetings`:
```json
{"meeting_id": "<meeting_id>", "held_on": "2026-10-21", "title": "Acme Q3 QBR + Roadmap", "kind": "qbr_roadmap"}
```

**Contacts no longer there** (`active`, on every contact; `true` unless the owner or an approved suggestion says otherwise): someone who retired, left or changed role. `codynote_get_customer` lists them last in `contacts` with `active: false` and names them in `inactive_people`; `people` (the profiles), Who's who and the email search (`addresses`, the derived `domains`) leave them out. `codynote_read` still opens them, and matching one to an old meeting (`attendee`, `speaker`) is fine. When a call or email says a contact retired, left or changed role, suggest a `contact` change with `fields: {active: false}` and the quote that says so; when it names their replacement, a new contact too (`{"action": "add", "record": "contact", "fields": {"name": "Priya Shah", "title": "IT Director"}}`). Someone back again: `{active: true}`.
```json
{"action": "change", "record": "contact", "company": "Acme Engineering", "id": "<contact_id>",
 "fields": {"active": false},
 "reason": "Lee retired at the end of September; Dana named Priya Shah as the new IT lead.", "sources": [{"kind": "email", "id": "<message_id>", "title": "Re: FY27 renewal quote", "date": "2026-10-27", "quote": "Lee retired last month; Priya now runs IT"}]}
```

**Deals in my name** (`in_my_name`, on every deal; `true` unless the owner says otherwise): whether the deal was closed in the owner's name in HubSpot. Only deals in the owner's name count on the quarter (`codynote_quarter_deals`) and GRR, on any account, including one the owner doesn't manage (`is_my_account: false`, `owner_teammate`: covering for that teammate). `false` is the rare co-sell closed in someone else's name; `codynote_get_customer` still lists it. A `deal` add or change may carry it (`in_my_name` or `inMyName`), from HubSpot's deal owner:
```json
{"action": "change", "record": "deal", "company": "Acme Engineering", "id": "<deal_id>",
 "fields": {"in_my_name": false},
 "reason": "HubSpot's deal owner is Jordan Lee, not you: a co-sell in his name.", "sources": [{"kind": "hubspot", "title": "Acme FY27 expansion", "url": "https://app.hubspot.com/contacts/1/record/0-3/2", "date": "2026-10-28"}]}
```

## Managed accounts (0.34.0)

An account can be **managed by** another of the owner's accounts: an MSP, a Panzura partner, itself an ordinary account. A managed account keeps its own deals, items, meetings, contacts and Rings; nothing rolls up and nothing moves. `codynote_get_customer` on a managed account gives `managed_by` (`{company_id, name}`) and `managed_by_people`: the MSP's contacts with their profiles, read from the MSP and marked `from: "<MSP>"`, those still there first. They are people on the managed account too: Who's who includes them, and attendees and speakers are matched against them as well as `people`. On an MSP it gives `manages_count` and `manages`: one line per managed account (health, sentiment, next renewal and its ARR, open and waiting items, last touch, `needs_attention` and `why`). `codynote_book` carries `managed_by` on each account and `manages_needing_attention` on an MSP; `codynote_quarter_deals` carries `managed_by` per deal, and the quarter's totals are unchanged.

- An MSP person on a managed account's call: a `speaker` may name them (their `contact_id`), and their Observations and boxes go under the managed account (`card: "person"`, `id` = their `person_id`). An `attendee` link takes the account's own contacts only. Their contact record (title, email, active) is changed under the MSP. Never add them as a new contact of the managed account.
- Setting who manages an account: a `company` add or change with `fields: {managed_by: "<the MSP's CodyNote name>"}` (or `managed_by_company_id`; `null` clears it), from HubSpot's parent or partner association or the owner's word. The MSP must already be in CodyNote; Only one level: CodyNote refuses the company itself, a managing account that is itself managed, and a company that already manages others.

## One pass: building on your own suggestions (0.46.0)

**Send everything in one pass. Never stop to ask the owner to approve some suggestions first and say go.** The owner: "It should be able to make suggestions on top of suggestions and them land in the right order."

- **Same call:** put `ref` (a short name: `c1` for a contact, `d1` a deal, `i1` an item, `s1` a storyline, `r1` a Ring) on the add, and that name wherever its id goes: `id` of a person's `box` or `line` (`card: "person"`), `fields.contact_id` (attendee, speaker), `fields.reports_to_contact_id`, `deal_id` of a license line, `fields.renews`, `ring_id`, the `id` of an item's or storyline's `box` / `line`, `fields.bullet_id`. Put the add before what points at it.
- **A later call:** every answer lists `suggestion_ids` (`index`, `id`, `label`) for the adds it saved. Give that id wherever the record's id goes, the same way. A person a waiting `link` puts on this account is named by their own `person_id`, as usual.
- **Email:** an email from someone you suggested as a contact (same call, or still waiting) is taken; its card says it also adds the contact.
- CodyNote checks each change against what the waiting suggestions would make, and approving one adds what it builds on first, in the same click (one Undo). Rejecting the contact greys what builds on it.
- **No duplicates:** the same add sent again (a contact or deal of the same name, still waiting on the account) is not saved twice: the answer's `existing` gives the waiting one's `id`, and a `ref` on it names that one.

## People across accounts (0.35.0)

**One person, one card.** A contact has a home account and can be **linked** to other accounts (a partner's account executive who covers three of the owner's accounts, an IT lead shared by two sister companies). A linked person is listed on every account they are on: `codynote_get_customer`'s `people` and `contacts` list them after the account's own with `linked: true` (`contacts` also gives `home_company`), and anyone on more than one account carries `accounts` (`[{company_id, company, home}]`, home first); `codynote_read` on a person returns `accounts` too, and its name candidates say `also_on`. Attendees, speakers and profile lines take a linked person on any account they are on.

- **Look up before you add.** Before suggesting a new contact, search for them: `codynote_read` with their `name` (across every account), or `people` on the account. Someone who is already in CodyNote on another account is **linked**, never added again: `{"action": "link", "record": "contact", "id": "<person_id>", "company": "Fabrikam Clinics", "reason": "...", "sources": [...]}` (or `fields: {company_id}` for the account). Add a contact only when nobody matches.
- **The guard.** CodyNote checks every contact add against everyone it has: the same email, or the same name at the same partner org, turns the add into a link to that person (the reason says why), and one already on this account is refused (change that contact instead). If a source shows it really is a different person with the same name or email, send the add with `fields: {..., "new_person": true}`: it stays an add and the owner sees the lookalikes. The same name alone stays an add, with the lookalikes shown to the owner.
- **Copies already there.** `codynote_lookalikes` with `kind: "people"` (and `company` for one account) lists the groups of contacts that look like copies of one person: `strength` strong (same email, or same name at the same partner org) or maybe (same name only: check the titles, calls and emails before suggesting anything), the copy CodyNote would `keep`, and each copy's accounts. Suggest one **merge** per group: `{"action": "merge", "record": "contact", "id": "<the copy to keep>", "fields": {"merge_ids": ["<the other copies>"]}, "reason": "...", "sources": [{"kind": "other", "title": "same email on Contoso Health and Fabrikam Clinics"}]}`. On approval everything the copies held (meetings, speaker names, profile lines, notes, accounts) moves to the one kept, and Undo puts it back. Never merge a teammate, a copy already merged, or two people the owner marked "not the same person" (CodyNote refuses those).
- A merged copy opens with `codynote_read` as `merged_into` (the survivor): read and suggest on the survivor.

## No duplicate items (0.36.0)

**One thing, one item.** Before suggesting a new action item, read the account's open items (`codynote_list_action_items` with the company and `mine_only: false`, or `open_items` on `codynote_get_customer`). Something already open, in any wording ("Dana needs the pricing" when "Pricing: Symphony quote for Dana" is open), gets an update of that item instead (0.53.0, recipe `item.pick`: fold first), a Context line: `{"action": "add", "record": "line", "company": "Contoso Health", "card": "item", "id": "<item_id>", "fields": {"key": "context", "text": "...", "date": "2026-10-27"}, "reason": "...", "sources": [...]}`, plus a new `next_step` box when the call moved it.

- **The check.** CodyNote compares every `add action_item` on an account with its open items, by shared words (no meaning model, so fold first yourself: recipe `item.pick`). A matching add becomes an update of the open item (0.53.0): a Context line and a Next step, the reason says "Looks like <item> (open since <date>)", and the result's `turned` lists each one; its other boxes, due date and priority are not carried over, so suggest them on the item if they add something. A match with a waiting suggestion, or with another add in the same call, is refused: send one add, or change the waiting one. One only close to an open item stays an add, and the owner sees the lookalikes.
- **Really new?** If a source shows it is a different thing from the open item it looks like, send the add with `fields: {..., "new_item": true}`: it stays an add and the owner sees the lookalikes.

## Tidy (0.52.0: one `merge`)

**Two open items, one task.** `codynote_lookalikes` with `kind: "items"` and the account's `company_id` lists the pairs of its open items that look alike, closest first: `keep` (the item CodyNote would keep, `keep_reason` says why), `fold` (the other), `strength` (`same` or `near`) and `hold` when the owner should decide. Fold a pair only when both items plainly say one task, with **one** suggestion (recipe `item.fold`): Claude proposes, the owner approves, one Undo puts everything back.

```json
{"action": "merge", "record": "action_item", "company": "Acme", "fields": {"keep_id": "<keep>", "fold_id": "<fold>"}, "reason": "Dana's Symphony pricing, asked twice: one task said two ways.", "sources": [{"kind": "other", "title": "same item twice on Acme"}]}
```

On approval the kept item gets the other's Context lines (origin and dates kept), its follow-ups and sub-items, and a History row with the other's title and boxes; the other goes to Trash marked "Folded into …". The connector refuses a pair that is not two open items of the same account, the same item, or a parent and its sub-item, and a merge with no `reason`; a pair that does not look alike by words is saved on your reason alone and the reply's `notes` says so (the owner reads both).

- **Never fold** a pair with `hold` (different owners, or both hold Notes, sub-items, follow-ups or documents) unless the owner says so, a `near` pair that doesn't plainly say the same thing, or two items the tool doesn't list (a different product or number keeps them apart) without a reason that says what makes them one.
- `fold`'s due date and priority aren't carried over: when its due date is sooner or its priority higher, say so in the reply. Its Notes are the owner's: never copied anywhere (the app moves what moves).

## Corrections (0.47.0): suggest the fix, never "fix by hand"

When something in CodyNote is wrong or stale, suggest the fix; the owner approves it like anything else. Read `write:<record>` for the full shape.

- **One of your own card lines is wrong** (`origin: "claude"`): `{"action": "supersede", "record": "line", "card": "item" | "storyline" | "person", "id": <card id>, "fields": {"line_id", "text"?, "date"?}}` with a source; the old text stays in Earlier versions. History stays add-only (add a new History line instead).
- **A box is right but its date is old:** a `box` change with the same `text`, a new `as_of` and the source that says it again.
- **A meeting's date, title or kind is wrong:** `{"action": "change", "record": "meeting", "id": <meeting_id>, "fields": {"held_on"?, "title"?, "kind"?}}` with a Fathom, meeting, email or HubSpot source; on approval the lines that cite it are re-dated too.
- **Rings:** two that are one → `{"action": "merge", "record": "ring", "id": <the Ring to keep>, "fields": {"merge_ids": [...]}}`; what a Ring is for → `fields.role` (production, dr, archive, test); a real second Ring → `fields.new_ring: true` on the add.
- **Deals:** a current renewal a later Closed Won renewal replaced → a `deal` change with `replaced_by` (that renewal's id) and a HubSpot source; a deal that looks won → `{"action": "add", "record": "signal", "id": <open deal id>, "fields": {"signal": "looks_won", "proof"}}` (the owner clicks Won).
- **Items:** done → `status: "done"` with `done_proof {kind, id, title, url?, date}`; a reply came → `{"action": "change", "record": "follow_up", "fields": {"item_id"}}` with the reply as source; owner, deal, parent → `teammate_id`, `deal_id`, `parent_id`.
- **HubSpot as a source:** `{"kind": "hubspot", "id": "<object type>:<id>", "title", "url"}` (like `ticket:123`) on any record, History line (`source: {"type": "hubspot", ...}`) or card line; not on an Overview bullet's wording.
- **What only HubSpot can fix** (a deal's owner, a duplicate company record, a monthly amount, a stage, an outdated company record): the call's `hubspot_fix: [{kind: owner | duplicate_company | monthly_amount | stage | outdated_record, deal?, company?, note}]`, not prose; CodyNote returns it checked and stores nothing. Put that list in the reply.
- **What the owner typed stays the owner's** (a box with `set_by: "owner"`, their lines, Notes, a fact they set): never rewritten; say in the reply what looks wrong and why.

## Never

- Suggest the **company's sentiment** (the owner's; a person's sentiment box is fine, with a quote), or mark a deal **Closed Won / Closed Lost** (a `looks_won` signal at most; say it in the reply).
- Reword or drop a **History** line, or change a **fact the owner typed** (if research disagrees, say so in the reply).
- More than one **read** per scan, or a read line over 200 characters.
- Carry fallbacks for an older CodyNote (each skill checks `codynote_ping` once at its start instead, and asks the owner to update CodyNote when it is too old).

If CodyNote drops a record or a field anyway, keep the rest and say what and why in one line.
