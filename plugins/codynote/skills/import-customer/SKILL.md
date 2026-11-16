---
name: import-customer
description: Import one customer into CodyNote from a blank slate for the owner (a Panzura CSM) - the account and every company field, every Fathom meeting with its transcript, the people and partners with profiles, the email references, the deals with their license lines, the account picture (Rings, environment, MC, storylines, facts), the last meeting's action items filled in and checked (closed when proven done, duplicates merged), then the overview and the read - all as one import the owner reviews once and approves with Approve all. Resumable across conversations. Use when the owner says "import Acme from scratch", "set up Acme from scratch", "blank slate Acme", "import a new customer Acme", "continue the Acme import" (any customer).
---

# Import a customer from a blank slate

One run turns an empty account into a full one. Every step's suggestions go into **one import** in CodyNote; nothing is written until the owner opens it (Claude suggestions › Import ready) and clicks **Approve all**. Do not stop between steps to ask: work through all eight, then finish. HubSpot, Outlook and web search are read-only and may be used at every step, not only where named. Never send anything, never set the owner's sentiment of the account.

The shared files next to this one: `account-picture.md` (every record's shape), `cards.md` (the boxes and the person rules, with their caps), `email.md` (the email reference rules), `panzura.md` (the products), and `skus.md` (the SKU to product table, step 5).

**For field X, follow its recipe** (`field-guide.md` next to this file, one line per field; `codynote_recipe(id)` has the whole recipe: where to look in order, what counts, the format, what never to do). Never stop for an approval: the owner approves each suggestion in CodyNote.

## 0. Check, then open the import

1. `codynote_ping(skills: "0.55.0")` once (the `skills` value tells CodyNote which skills Claude has). Its `version` must be **0.55.0** or later: if it is older, say "Update CodyNote to 0.55.0 or later, then ask again" and stop. Call only the tools in its `tools`.
2. Which connectors are here: the **Fathom** connector (`list_meetings` or `search_meetings`; without it, `codynote_fathom_missing` lists the recordings), **HubSpot** (`search_crm_objects`, `get_crm_objects`), **Outlook** (`outlook_email_search`) and web search. Say in one line which are missing and what you will skip (no Fathom: no meetings, so no people from calls and no step 7; carry on with the rest).
3. Read the contract once: `codynote_describe("write:import")`, `"write:rules"`, and `"write:<record>"` for every record you will suggest: company, meeting, contact, attendee, speaker, line, box, email, deal, license_line, product_status, ring, mc_reading, fact, bullet, history, action_item, read. Suggest a value for **every field** those topics list where a source supports one; a field nothing supports stays empty and goes in the reply under **Left empty** with why. Never fill a field from a guess.
4. `codynote_import_start(company)`. Deals already there (an earlier deals refresh brought them in) are fine: `deals_already` says how many, and step 5 adds only the deals and license lines not there yet (a line on an existing deal names its `deal_id`). It refuses an account that already has meetings or people: say "<Company> isn't a blank slate; say "refresh <Company>" instead" and stop (a refresh also goes in one pass). **Resumed** (`resumed: true`, the owner said "continue the <Company> import", or a new conversation): `holds` lists the lines per step, the meetings staged (ref and date) and the refs made so far. Continue from the first step with nothing, and the meetings not staged yet; read a staged meeting back with `codynote_read(import_id, id: <ref>, find)` when you need it. Keep `import_id` for every call.

Every staging call is `codynote_suggest_changes({import_id, changes})`: the usual records and rules, no `scan_id`. An add that others will point at carries `ref` (a short name: `p1` for a person, `d1` a deal, `r1` a Ring, `s1` a storyline, `i1` an item); put that name wherever an id goes (`id`, `deal_id`, `ring_id`, `meeting_id`, `fields.contact_id`, `fields.bullet_id`, `fields.renews`, a source's `id`). Stage what a ref names before anything that points at it. Each change carries `step` (1 to 8) and a `confidence`. Give each change a `confidence` (`high`, `medium` or `low`: how sure you are), and copy every quote exactly as it is in the source: CodyNote looks for a meeting's quote in its transcript and marks one it can't find "quote not found". What CodyNote drops comes back with why: fix it and send it again, or list it in the reply.

```json
{"record": "contact", "action": "add", "ref": "p1", "step": 3, "fields": {"name": "Jordan Example", "title": "IT Director", "email": "jordan@example.com"}, "reason": "On the Oct 20 call and in HubSpot", "sources": [{"kind": "meeting", "id": "m3", "title": "Example weekly", "date": "2026-10-20", "quote": "I sign off on the renewal"}], "confidence": "high"}
{"record": "line", "action": "add", "card": "person", "id": "p1", "step": 3, "fields": {"key": "observations", "text": "Signs off on the renewal himself", "date": "2026-10-20"}, "reason": "Said on the call", "sources": [{"kind": "meeting", "id": "m3", "title": "Example weekly", "date": "2026-10-20", "quote": "I sign off on the renewal"}]}
{"record": "license_line", "action": "add", "deal_id": "d1", "step": 5, "fields": {"product": "Collab", "quantity": 50, "unit": "TB"}, "reason": "HubSpot line item", "sources": [{"kind": "hubspot", "title": "Example - 2027 Renewal", "url": "https://app.hubspot.com/contacts/1/deal/2"}]}
```

## 1. Account

HubSpot's company record, the company's website and the web. A company not in CodyNote: `{"record": "company", "action": "add", "fields": {"name": ...}}` first, before anything else. One that is (empty): a `change` on it (its id from `codynote_import_start`). Every company field `write:company` lists (account group, clouds, managed by an MSP already in CodyNote...), then the facts card (`record: fact`: revenue, employees, offices, hq, industry, founded, website, acquisitions), each with its source link and date. Note the account's email domains (HubSpot's domain, the contacts' addresses): steps 2 and 4 search by them.

## 2. Meetings, from Fathom (oldest first)

List every recording whose attendees are from the account's domains, all dates (`list_meetings` or `search_meetings` by attendee domain; also by company name to catch a personal address; `codynote_fathom_missing(company)` gives each one's `recording_id` too). Then **oldest first, one at a time**, stage it with `codynote_import_meeting(import_id, ref: "m1", "m2"..., recording_id, kind?)`: CodyNote fetches the title, date, attendees, summary and transcript from Fathom itself. **Never carry a summary or transcript through the conversation** (don't fetch them to pass on). An answer with `may_be_cut_off`: check it (read the staged meeting's end with `codynote_read(import_id, id: <ref>, find)` against Fathom's recording) and say in the reply which meeting may be cut off. A recording CodyNote refuses as already in CodyNote: skip it and list it. Then read what later steps need from each staged meeting with `codynote_read(import_id, id: <ref>, find)` (who spoke and said what about themselves, the environment, deals, open questions, the action items of the newest one), not the whole transcript. A meeting's kind other than the title shows (a QBR, a roadmap): `{"record": "meeting", "action": "change", "id": "m4", "fields": {"kind": "qbr"}}`.

A long history can fill the conversation: when room runs short, finish the meeting you are on and stop **without** `codynote_import_finish` (the import stays unfinished, so the owner is never shown half of it), and tell the owner to say "continue the <Company> import" (a new conversation picks up from `holds`; in a book run, the next hour does).

## 3. People and partners

From the meetings' attendees and speakers, HubSpot's contacts and their roles, Outlook (who writes), and the web (title, LinkedIn). Before adding anyone, look them up (`codynote_read` with their `name`): someone already in CodyNote on another account (a partner, usually) is linked, not duplicated (`account-picture.md`, "People across accounts"). Each contact with every field `write:contact` lists that a source supports (title, email, phones, partner org, linkedin_url, role, reports_to via its ref). The account's people on each meeting: `attendee` (`meeting_id` and `contact_id` by ref). A pasted "Speaker N" named only where the call makes it plain. Then profiles under the person rules in `cards.md`: at most 6 observations, 3 interests and 3 sore spots per person, boxes only when they add something; what was left out goes in the reply.

## 4. Emails

Only when Outlook is here. `email.md`'s rules over the account's whole mailbox history (its domains and its contacts' addresses): one `email` reference per email worth keeping, and `email_sync` with how far you read. Emails are staged in the import like everything else.

## 5. Deals (HubSpot)

The one deal rule, in code (0.48.0): read every deal on the company in HubSpot, every line item on them and the owners, all as HubSpot returns them, and pass them to `codynote_hubspot_deals(company, raw: {deals, line_items, owners})` (each line item nested under its deal as `line_items`, or with `deal_id`). CodyNote picks the deals (every open deal and every Closed Won deal still in term, whoever owns it), works out each line's product and ARR, `renews`, `opp_type`, stage, forecast and `in_my_name`, and matches what CodyNote already has. Stage its `changes` as they are (deals by ref, their lines after them), never your own arithmetic; then a `product_status` "purchased" per product on a deal in term. Its `flags` go in the reply's heads-up (`amount_monthly` and `owner_mismatch` also in `hubspot_fix`; a new SKU is confirmed once by the owner under Settings › Products). Never Closed Won or Closed Lost as a stage.

## 6. The account picture

From the meetings, emails and HubSpot: each Ring and its environment (`write:ring`: CloudFS version, nodes, object store, hypervisor, HA, as said) and its MC readings; the storylines (`record: bullet`, Projects and Issues, with a ref), each with its History lines (dated the day it happened, sourced), Open questions, Helps / Hurts / Signals to watch, Now and status; the Tech stack, Workloads and Use cases bullets; every remaining company field and fact.

## 7. The last meeting's action items

Only the newest meeting's items; older meetings' are not added. First read the account's open items the owner typed (`codynote_list_action_items` with the company and `mine_only: false`). Then **sort them with recipe `item.pick`** (fold first; the transcript and both of Fathom's lists as one pool): a task said twice in the meeting is one item (the other wording as a Context line on it), and one that matches an owner's open item is an update of that item (a Context `line` and a Next step), not a new one (`codynote_lookalikes(kind: "items")` and `account-picture.md`, "No duplicate items"; CodyNote also turns a same-item add into that update and says so in `turned`; a match with a waiting suggestion or another add in the same call is refused). Waiting-on follow-ups count from any source. More than about five new items from one meeting means fold harder. Each item: `meeting_id` (its ref), a title "Topic: what exactly, how much, for whom" (recipe `item.text`, at most 90, no leading "do this" verb), a Situation in three labeled parts, "The ask:", "Where it stands:", "Why it matters:" (recipe `item.situation`; never a Goal, which the connector refuses), Done when, a Next step as who / what / when (naming a day or "this week"), a due date only when one was said or implied; a teammate's task with `teammate_id`, its deal with `deal_id` (an id or ref), a sub-item under its item with `parent_id`. Then check each against what happened since: HubSpot tickets and logged emails, Outlook threads, and the notes. **Proven done:** add it with `status: "done"` and `done_proof: {kind: meeting | document | email | hubspot, id, title, url, date}` (the meeting's ref, an email's id, or a HubSpot record as `<object type>:<id>` with its link; date: the day it shows done). **Not proven:** leave it open, with what was checked as a Context line.

## 8. Overview and the read, last

From everything above: the overview bullets and storylines not yet staged, then the read as the import's last change: `{"record": "read", "action": "add", ...}` per `write:read` (the five lines, a health, 200 characters each). This is the one place a read goes in as a change; anywhere else CodyNote refuses one. Never the owner's sentiment.

Then `codynote_import_finish(import_id, hubspot_fix)`. **HubSpot-side cleanup** you noticed (a deal's owner, a duplicate company record, a monthly amount, a stage, an outdated company record) goes in `hubspot_fix: [{kind: owner | duplicate_company | monthly_amount | stage | outdated_record, deal?, company?, note}]`, never as prose: CodyNote returns it checked (and stores nothing); put that list in the reply.

## Reply

```
**<Company>: import ready for your review** (CodyNote › Claude suggestions › Import ready)
1 Account: <n> · 2 Meetings: <n> (<oldest> to <newest>) · 3 People: <n> (<n> linked) · 4 Emails: <n> · 5 Deals: <n> · 6 Picture: <n> · 7 Items: <n> (<n> closed, <n> merged) · 8 Overview and read
**Closed:** "<item>": <proof>
**Left empty:** <field>: <why> (one line each)
**Left out:** <per person: n lines and why>; <recordings already in CodyNote>; <what CodyNote couldn't place>
**Couldn't reach:** <connectors missing, and what that skipped>
**Fix in HubSpot:** <the hubspot_fix list CodyNote returned, one line each; or "nothing">
Open CodyNote to review and approve.
```
