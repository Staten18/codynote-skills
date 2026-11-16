# Email (CodyNote 0.29.0 and later)

How the skills read the owner's Outlook email for one account and what they suggest from it. Email is a **source, not a copy**: CodyNote keeps a one-line reference per email (never the body) and a watermark per company (`emails_through`: email read up to then). `codynote_get_update_brief`'s `email` block is the live rulebook: where its `how` and this file differ, follow the brief. A read-only skill (Account brief's QBR mode) takes the same block, without `how`, from `codynote_get_customer`'s `email`.

## When

**Only on the owner's word**: Refresh account's email mode ("catch up on email for Acme", "sync Acme's email", "read Acme's email since last time"), or "with email" added to "refresh Acme", to its call mode or to Account brief's QBR mode. Never otherwise: no other skill or step reads Outlook, and nothing runs on its own.

Email comes through the **Outlook connector** in Claude Desktop (`outlook_email_search`, `read_resource`). If it isn't there, say "The Outlook connector isn't connected in Claude Desktop, so I can't read email" and skip the email part.

## The search recipe

From the brief's `email` block: `emails_through` (the watermark; none yet: the last 90 days, and say so), `addresses` (the customer contacts' still there: a contact marked inactive, `active: false`, has retired, left or changed role, so their address is not searched and makes no derived domain), `domains` (the account's), `owner_domains` (the owner's own, e.g. panzura.com) and, when there are any, `partner_addresses` (a reseller's or integrator's people, and people linked to this account from another one: search those only with a query about this account, never alone). The message id is the id in `mail:///messages/{id}`.

Run `outlook_email_search` once per line below (never a domain in `owner_domains`, never a personal service like gmail.com):

1. Inbound, per domain in domains: sender: <domain>, afterDateTime: <emails_through>, limit: 25.
2. Inbound, per contact address in addresses: sender: <address>, afterDateTime: <emails_through>, limit: 25.
3. Sent and copied, per contact address in addresses: recipient: <address>, afterDateTime: <emails_through>, limit: 25.
4. Every page: while the last result has nextOffset (or nextCursor), call again with it, until there is none. Results are not in date order, so read every page before the watermark moves.
5. Never pass folderName (refused with a date) or order: "oldest" (refused with recipient). Dedupe on the message id.
6. **Skip** calendar invites, acceptances and cancellations, bounces ("Undeliverable", "Delivery has failed") and auto-replies ("Automatic reply", "Out of office"). Don't index them; they still count for the watermark.
7. **Bodies only when needed:** read one with `read_resource` only when its subject or preview says the thread matters (a decision, a date, a problem, a number, a new person, a promise). Use the message's **own text** only, never the quoted history below it.

The owner's own messages are those from an address in `owner_domains` (or an Exchange address starting `/O=`). If the brief's `domains` say `derived`, name them in the reply so the owner can correct them on the company.

## What to send (one `codynote_suggest_changes` call)

- **One `email` record per kept email in the window**, worth it or not, so the index is complete: `matchedBy: "contact"` when a contact's address is on it, else `"domain"`; a **gist** of one line (200 characters at most) in your words; `folder` `inbox` or `sent`; `webLink` only an https link the connector gave.
- **`email_sync`** at the top of the call: `emails_through` = the **newest `sentAt` read** (skipped ones included), even when nothing was worth keeping. Only a full sync (steps 1-4 over every domain and address) moves it; a narrower search (call mode's) sends no `email_sync`.
- **What the emails support**, with the email as the source (`sources: [{kind: "email", id: <messageId>, title: <subject>, date}]`; a History line's `source: {type: "email", refId: <messageId>, title: <subject>}`), in the shapes of `account-picture.md`:
  - a **History** line on a storyline (dated the day the email was sent), a **Now** rewrite when its state changed, `watch` / `resolved` when the email says so;
  - a new storyline for something new (if CodyNote drops it for wanting a meeting or document source, say "worth a storyline" in the reply);
  - a contact who **retired, left or changed role** (an "I'm leaving" note, "Lee retired last month"): a `contact` change `{active: false}` with that line as the quote, and the replacement they name as a new contact (`account-picture.md`, "Day to day");
  - **facts** (blanks only), **contacts** (new people from the account's domain, a new title; look a new person up first with `codynote_read` and their `name`, and `link` someone already on another account instead of adding them again: `account-picture.md`, "People across accounts") and their **role** (persona: `champion`, `economic_buyer`, `technical_buyer`, `decision_maker`, `executive_sponsor`, `coach`, `user`, `blocker`, `admin`, `billing`, `partner`, `other`) when the email makes it plain, and **profile lines** for each person who wrote: an Observations `line` (`card: "person"`, `id` = their `person_id`) per thing noticed, dated the day it was sent, with a short quote of their own text, and the boxes it supports (a sentiment, an interest, a sore spot; `cards.md`);
  - **action items** for asks and promises, with their boxes (`cards.md`).
- **Never** the company's sentiment or a deal Closed Won / Lost. A read only when the emails change the account materially, and then in the final call's `close_scan` with `coverage`, never here.

### Waiting tags from sent mail

A thread (same `conversationId`) is **waiting on them** when: the last message is the owner's, someone outside `owner_domains` is on it, nothing later came back, it's **older than 5 days**, and it isn't an auto-reply. For each:
- an open item it matches (from `codynote_get_customer` `open_items`; fold first, recipe `item.pick`) → a `follow_up` on it: verb `sent`, the person, `occurredOn` = the sent date, `emailId` = its message id, `setWaiting: true`;
- none matches → a new action item with `status: "waiting"`, the title "Topic: what exactly, how much, for whom" (recipe `item.text`) and the follow-up "waiting on <name, role> since <date>", `due_date` empty, and its boxes (Situation in three labeled parts, Done when, and as Next step who / what / when: the reply you're waiting for, with the day you expect it; never a Goal). Asks of the owner and the owner's promises in an email go through `item.pick` too: an open item on the topic gets the update, not a new item.

### The look-back rule

When a sync (or a call with email) **opens a new storyline**, look back once for older mail on its topic: one `outlook_email_search` with `query: "<topic in 2-4 words>"` and `sender: <domain>`, and once per key contact with `query` and `recipient: <address>`, **no date**. Attach what turns up as `email` records (if not already indexed) and History lines dated when each was sent. Don't move the watermark for these.

## Privacy

Never paste a body into a suggestion, a gist or the reply; quote at most a short line of the message's own text in a source's `quote`. Never forward, send, reply to, move, flag or delete email; a follow-up email is only ever a draft in the chat.

## Shapes

A sync for Acme Engineering (made-up names; `example.com` is Acme's domain):
```json
{"summary": "Acme email since Oct 14: 2 emails indexed, 1 History line, 1 waiting tag, 1 role.",
 "email_sync": {"company": "Acme Engineering", "emails_through": "2026-10-27T16:42:00Z"},
 "changes": [
  {"action": "add", "record": "email", "company": "Acme Engineering",
   "fields": {"messageId": "<message_id_1>", "conversationId": "<conversation_id_1>", "subject": "Dallas office timeline",
     "fromAddress": "dana.ruiz@example.com", "fromName": "Dana Ruiz", "to": [{"address": "you@panzura.com", "name": "You"}],
     "sentAt": "2026-10-27T16:42:00Z", "folder": "inbox", "webLink": "https://outlook.office365.com/owa/?ItemID=<message_id_1>",
     "gist": "Dallas opens Feb 1; IT wants a capacity quote before the Jan budget meeting.", "matchedBy": "contact"},
   "reason": "Email in the window.", "sources": [{"kind": "email", "id": "<message_id_1>", "title": "Dallas office timeline", "date": "2026-10-27"}]},
  {"action": "add", "record": "email", "company": "Acme Engineering",
   "fields": {"messageId": "<message_id_2>", "conversationId": "<conversation_id_2>", "subject": "FY27 renewal quote",
     "fromAddress": "you@panzura.com", "fromName": "You", "to": [{"address": "lee.park@example.com", "name": "Lee Park"}],
     "sentAt": "2026-10-20T15:05:00Z", "folder": "sent", "gist": "Sent the FY27 renewal quote; asked Lee to confirm the TB by Oct 31.", "matchedBy": "contact"},
   "reason": "Email in the window.", "sources": [{"kind": "email", "id": "<message_id_2>", "title": "FY27 renewal quote", "date": "2026-10-20"}]},
  {"action": "add", "record": "history", "company": "Acme Engineering",
   "fields": {"bulletId": "<bullet_id>", "onDate": "2026-10-27", "text": "Dana confirmed by email that the Dallas office opens Feb 1.",
     "source": {"type": "email", "refId": "<message_id_1>", "title": "Dallas office timeline"}},
   "reason": "New date for the Dallas project.", "sources": [{"kind": "email", "id": "<message_id_1>", "title": "Dallas office timeline", "date": "2026-10-27", "quote": "We open Feb 1"}]},
  {"action": "add", "record": "follow_up", "company": "Acme Engineering",
   "fields": {"itemId": "<item_id>", "verb": "sent", "person": "Lee Park", "occurredOn": "2026-10-20",
     "note": "FY27 renewal quote: no reply in 8 days.", "emailId": "<message_id_2>", "setWaiting": true},
   "reason": "You emailed Lee on Oct 20 and he hasn't replied.", "sources": [{"kind": "email", "id": "<message_id_2>", "title": "FY27 renewal quote", "date": "2026-10-20"}]},
  {"action": "change", "record": "contact", "company": "Acme Engineering", "id": "<contact_id>",
   "fields": {"role": "champion"},
   "reason": "Dana is driving Dallas onto CloudFS and asked for the quote.", "sources": [{"kind": "email", "id": "<message_id_1>", "title": "Dallas office timeline", "date": "2026-10-27"}]}
 ]}
```

**People on a call** (not email, same release): a meeting's attendee matched to a contact, one per person the notes name and the contacts hold (`codynote_read` on the meeting gives the attendees; `codynote_get_customer` the contacts):
```json
{"action": "add", "record": "attendee", "company": "Acme Engineering",
 "fields": {"meetingId": "<meeting_id>", "contactId": "<contact_id>", "nameText": "Dana Ruiz"},
 "reason": "Dana was on the Oct 28 call.", "sources": [{"kind": "meeting", "id": "<meeting_id>", "title": "Acme sync", "date": "2026-10-28"}]}
```
