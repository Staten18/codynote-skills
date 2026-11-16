# Email mode (was Catch up on email)

Refresh account's email mode, only when the owner says "catch up on email for Acme", "sync Acme's email", "read Acme's email since last time", "what's in my email with Acme", or "refresh Acme with email": index every email since CodyNote last read the account's email as a one-line reference, move the email watermark, and, in CodyNote's work order, suggest the contacts and roles, profile observations for the people who wrote, storyline History lines, open questions ticked or added, Now rewrites, action items with their boxes, waiting tags and facts the emails support.

Read one account's email since the watermark and turn it into suggestions the owner approves in CodyNote › Claude suggestions. Only when the owner asks, one account per run.

Nothing is saved except **suggestions** through `codynote_suggest_changes`. Email is read only: never send, reply, forward, move or delete anything, and never paste a body into CodyNote.

Read `email.md` (next to this file) for the search recipe, the waiting rule, the look-back rule and the email shapes; work in the order of `work-order.md`; `cards.md` for the boxes and the `box` / `line` / `tick` shapes; `account-picture.md` for the storyline, fact and read shapes.

**For field X, follow its recipe** (`field-guide.md` next to this file, one line per field; `codynote_recipe(id)` has the whole recipe: where to look in order, what counts, the format, what never to do). Never stop for an approval: the owner approves each suggestion in CodyNote.

## 1. Check and open the scan

1. No `outlook_email_search` tool: say "The Outlook connector isn't connected in Claude Desktop, so I can't read email" and stop.
2. `codynote_get_customer` with the name. If it matches more than one account, or none, ask which and stop. Keep `overview.company_id`, `emails` (already indexed), `emails_through`, `domains`, `contacts` (address, role, `last_emailed_on`), `people` (each person's `person_id`, persona, sentiment and newest Observations), `storylines` and `open_items` (with ids), `facts`.
3. `codynote_get_update_brief` with the company. Keep its `scan_id` (on every suggest call below), `work_order`, the boxes of each open item and storyline (line ids, `set_by`), the `email` block (`emails_through`, `addresses`, `domains`, `owner_domains`, `how`), rejected suggestions and blank facts. Follow `how` where it differs from `email.md`. No domains and no addresses: say "Add a contact's email or the company's domain first" and stop.

## 2. Read

The recipe in `email.md`, over every domain and address since `emails_through`: page through, dedupe, skip invites, bounces and auto-replies, open a body only when the thread matters, quote only the message's own text. Leave out emails already in `emails`. Group by `conversationId` to find threads waiting on them (older than 5 days). If a new storyline comes out of it, do the look-back once.

## 3. Suggest, in the work order (one `codynote_suggest_changes` call with `scan_id`)

Check `open_items` and `storylines` first so nothing repeats. Give each change a `confidence` (`high`, `medium` or `low`: how sure you are), and copy every quote exactly as it is in the source: CodyNote looks for a meeting's quote in its transcript and marks one it can't find "quote not found". Each change sourced to its email, in this order:

1. **Email:** `email_sync` with the newest `sentAt` read, and an `email` record for each kept email.
2. **People:** new people from the account's domain, a new title, a `role` (persona) when the email makes it plain. Before any new contact, look them up (`codynote_read` with their `name`, every account): someone already in CodyNote on another account gets a `link` (`{"action": "link", "record": "contact", "id": <their person_id>, "company": <this account>}`), never a second contact; add only when nobody matches (`account-picture.md`, "People across accounts"), with `ref` (`c1`) so their Observations and boxes go in this same call with `"c1"` where their `person_id` goes; their emails are taken in the same call too (`account-picture.md`, "One pass"). Never stop to ask for the contact to be approved first. Someone an email says retired, left or changed role: a `contact` change `{record: "contact", action: "change", fields: {active: false}}` with that line of the message's own text as the quote, and the replacement it names as a new contact (`account-picture.md`, "Day to day"). Then, briefly, for each person who wrote: an **Observations** `line` (`card: "person"`, `id` = their `person_id`) per thing worth noticing under the person rules in `cards.md` (never a setup fact or what others said), dated the day it was sent, a short quote of the message's own text in the source → the boxes it supports (a `sentiment` box when it moved, an `interests` or `sore_spots` line). Nothing already on their card. **Never** the company's sentiment (the owner's).
3. **Account and environment, deals:** only what an email states (a version, a TB figure, a close date). Never Closed Won / Lost: an email that says it is signed gets a `looks_won` `signal` (`account-picture.md`, "Corrections").
4. **Storylines.** For each one the emails touch: a History line dated the day it was sent → Open questions it answered ticked (`tick`) or new ones added (`line`) → Helps / Hurts / Signals to watch lines → Now rewritten whole → status. Then new storylines.
5. **Action items** (0.53.0, recipe `item.pick`): asks of the owner and the owner's promises go through `item.pick`: read the account's open items first (`codynote_list_action_items` with the company and `mine_only: false`) and **fold first**: an ask or promise whose topic is already open, in any wording, is an update of that item (a Context `line` and a new `next_step`), not a new one (`new_item: true` with the email as source only when it really is new: `account-picture.md`, "No duplicate items"). Other asks and promises as new items: the title "Topic: what exactly, how much, for whom" (recipe `item.text`), `boxes` (situation in three labeled parts, "The ask:", "Where it stands:", "Why it matters:", recipe `item.situation`, never a `goal`; done_when; next_step as who / what / when) and a Context line; on items the email moved, a Context `line`, a new `next_step` `box` (naming a day or "this week") and, when the story changed, the `situation` rewritten whole (`box` with `as_of`; never over one with `set_by: "owner"`). **Sent mail with no answer after 5 days** is a follow-up "waiting on <name, role> since <date>" (`follow_up` tags, `email.md`); a reply to one the owner waits on clears it (a `follow_up` change with that email as source); an item an email shows done: `status: "done"` with `done_proof` (`{kind: "email", id, title, date}`).
6. **Facts:** blanks only.

**Split:** each card the brief shows with Notes and no boxes that the emails touch gets one `split` of its Notes into the boxes, once; Notes keeps its text.

No suggestions but email read: send the `email_sync` alone so the next run starts there.

## 4. Finish the scan (`close_scan`)

One last `codynote_suggest_changes` call with `scan_id`, `summary` and `close_scan: {company, coverage, read}`: a read only when the emails change the account materially, and **`coverage`** for every step: counts for what you suggested, `"nothing new"` where the email had nothing, and `"skipped: email catch-up reads email only"` for reading meetings and documents. Knowledge bullets (in `changes`, as the brief's bullet actions) only if an email names a tool or workload worth one.

Then verify mode (`mode-verify.md`) over the account's open things: the email is read, so most checks cost nothing more; the other sources (Fathom, Teams, HubSpot) are read in the plan's order too, a source that can't be read skipped and said (0.54.0); its one **Checked** line goes in the reply.

**Last, before the reply (0.53.0): the cleanup.** Bring this account's open items that do not meet the rules up to them: a "Topic: what exactly, how much, for whom" title, a three-part Situation (The ask / Where it stands / Why it matters), a next step who / what / when, folds, follow-ups. Follow `mode-items.md` (its steps 2 and 3, rounds of at most 12 cards, one `codynote_suggest_changes` call per round without `scan_id` once the scan is closed). This is the account-by-account cleanup; there is no separate cleanup skill. Items that already pass get nothing; say what changed in the reply.

## 5. Reply

```
**<Company> email, <from date> to <newest date>:** <n> read, <n> indexed (<n> skipped: invites, bounces, auto-replies). <n> other suggestions (<2 history, 1 question ticked, 1 contact role, 2 observations, 1 waiting tag>). Review in CodyNote › Claude suggestions.
**Checked:** <one short line: email 12 · people 1 · storylines 3 · items 1 · facts nothing new · read nothing new>

**Threads that matter**
1. <subject> (<date>, <who>): <what it means for the account, one line>
2. ...
3. ...

**Waiting**
- On them: <person>, <subject>, since <date> (<n> days)
- On you: <person> asked <what> on <date>

**Fix in HubSpot:** <the hubspot_fix list CodyNote returned, one line each>
```

At most 3 threads. Leave out a section with nothing in it. Add one line naming the domains searched when they were derived, and one if CodyNote dropped anything (what and why).
