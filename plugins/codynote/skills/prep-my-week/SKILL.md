---
name: prep-my-week
description: Prep the owner (a Panzura CSM) for the customer meetings on their Outlook calendar this week, next week or one day, from CodyNote - for each account meeting who is coming (persona and sentiment), how old the account's read is, open and waiting items with their ages, days to the renewal, the risk flags and the one thing to get out of the call; then items due with no meeting behind them and renewals inside 90 days with nothing booked. Use when the owner says "prep my week", "what's my week look like", "prep next week", "prep Thursday", "prep my day", or similar. Read-only.
---

# Prep my week

One screen for the days ahead: which customer meetings are coming, and what the owner needs to walk into each with. Only when the owner asks.

This skill only reads. Never save, suggest or change anything in CodyNote (no suggestions, no `scan_id`), never send anything, and never accept, decline or change a calendar event. It reads the Outlook calendar, never Outlook email: email comes from what CodyNote has indexed.

Read `account-picture.md` (next to this file) for the read, the signals and the personas; `panzura.md` for what is worth raising.

**First, once:** `codynote_ping(skills: "0.55.0")` (the `skills` value tells CodyNote which skills Claude has). Its `version` must be **0.55.0** or later (an older CodyNote answers in one line, without `version` and `tools`): if it is older, say "Update CodyNote to 0.55.0 or later, then ask again" and stop. Call only the tools in its `tools`.

## 1. The window

Count from today (the owner's local date).
- "prep my week", "what's my week look like": this week, Monday to Friday; mid-week, today to Friday. On a Saturday or Sunday, the coming Monday to Friday.
- "prep next week": next Monday to Friday.
- A weekday ("prep Thursday"): that one day, the next one coming (today if it is today).
- "prep my day", "prep today", "prep tomorrow": that one day.

## 2. Gather (read everything before writing anything)

1. **The calendar:** the Outlook connector's `outlook_calendar_search` with `query: "*"`, `afterDateTime` = the window's first day, `beforeDateTime` = the day after its last day, `order: "oldest"`, `limit: 25`; follow `nextOffset` to the end. Keep each event's subject, start (as given, with its time zone), organizer and attendees (email and name). Skip cancelled events, all-day events and events with no one else invited (holds, focus time, out of office). If the connector isn't there, start the reply with "The Outlook calendar isn't connected in Claude Desktop, so this is from CodyNote alone." and go to step 4: the tail is the whole reply.
2. **Sort each meeting.** The owner's domain is panzura.com (and any other the owner names); personal domains (gmail.com, outlook.com…) never match an account.
   - **Internal:** every attendee on the owner's domain. Listed, not prepped.
   - **An account meeting**, matched in this order:
     1. By attendee email domain: one `codynote_query` for the whole window with every outside domain. An account's own domains first (`companies.domains_json`, the ones the owner confirmed); an account with none confirmed counts its customer contacts' domains:
        ```sql
        select c.id as company_id, c.name as company, lower(j.value) as domain
        from companies c, json_each(c.domains_json) j
        where c.deleted_at is null and lower(j.value) in ('<domain>')
        union
        select v.company_id, v.company, v.domain
        from v_contacts v join companies c on c.id = v.company_id
        where c.deleted_at is null and c.domains_json is null and v.active = 1
          and v.kind = 'customer' and coalesce(v.role, '') <> 'partner' and v.domain in ('<domain>')
        ```
     2. No domain match: by attendee name in `v_contacts`, on every account they are on (`v_contact_accounts`, one row per person per account):
        ```sql
        select contact_id, company_id, company, name, kind, role from v_contact_accounts
        where active = 1 and lower(name) in ('<attendee name>')
        ```
     3. Still none: an account named in the subject.
     A meeting that matches two accounts (a partner who covers both, a name on two accounts): the one the subject names, else list it under Other meetings with both names.
   - **Other meetings:** no match and no customer in the subject. One line each, not prepped.
3. **Each account with a meeting** (once, however many meetings it has):
   - `codynote_get_customer` with the account: the read (health and date), the newest meeting, renewal date and ARR, open items (`days_overdue`, `due_date`) and waiting ones (`days_waiting`), `people` (persona `role`, the person's own `sentiment`) and, on a managed account, `managed_by_people`, open risk storylines, `needs_a_look` and `signals` (its `why`).
   - **The book:** one `codynote_book` with `only: "all"` for the whole window (not one per account): each account's `why`, `days_to_renewal` and open opps, for the tail as well.
   - Match the attendees to `people` by email, then name. Someone not in CodyNote: "new: <name or address>".
4. **The tail:**
   - **Due with no meeting:** `codynote_list_action_items` with the `due` bucket that holds the window (`"this_week"` runs from today to Sunday, `"next_week"` the Monday to Sunday after; a weekend "prep my week", or a weekday in next week, uses `"next_week"`), keeping only the items due inside the window, plus `due: "past_due"` unless the owner asked for next week. Leave out items on an account with a meeting in the window (they are in its prep).
   - **Renewing within 90 days, nothing booked:** from the book rows, the owner's current accounts (`is_my_account`) with `days_to_renewal` 0 to 90 and no meeting in the window. For each (at most 8), one `outlook_calendar_search` with `attendee` = the account's domain, from today to the renewal date, `limit: 1`; a meeting found: leave the account out. Without the calendar, list them all and say "calendar not checked".

## 3. Think before writing

- **Get out of this call:** one line per account meeting, grounded in what you found: the read's Next line, a past-due or long-waiting item, a renewal without an opp, a risk, a person who turned negative. Mark an inference "my read".
- Prep only from CodyNote and the calendar. Say what you could not see (no read yet, no profiles, an attendee not in CodyNote).

## 4. Write

One screen, about 30 lines for a normal week. Short lines. Dates "Nov 3", times as the calendar gives them, money "$84k", waits "12 days". Oldest meeting first; an account with two meetings shows both times on one header.

```
**Your week: Mon Nov 2 – Fri Nov 6** · <n> account meetings · <n> internal · <n> other

**Tue Nov 3, 10:00 · Acme Engineering** · Acme weekly sync
Coming: Dana Whitfield (Champion, positive) · Lee Park (Economic buyer, no sentiment yet) · new: pat@acmeeng.com
Read: Watch, 41 days old (older than the Oct 20 call) · renews Jan 15 (77 days, $84k) · flags: renewal with no opp, risk open (Dallas cutover)
Open: Dallas quote: 3 sizing options for Dana (due Nov 4) · Waiting on Dana: sizing sign-off, 9 days
Get out of this call: Dana's yes on the Dallas quote before the Jan budget meeting.

**Internal:** Mon 9:00 CSM Team Meeting · Thu 14:00 Forecast review
**Other meetings:** Wed 11:00 Intro call (no account matched)

**Due with no meeting:** Contoso Health: send the QBR deck (due Nov 5) · Fabrikam Clinics: renewal quote (**past due**, 3 days)
**Renewing within 90 days, nothing booked:** Fabrikam Clinics (Dec 1, 32 days, $52k)
```

For one day the header reads `**Thursday, Nov 5**`. Leave out a line or section with nothing in it ("flags" when none, Internal when there are none). No account meetings: say so in one line, then the tail. Don't paste the read, storylines or deals: that is Account brief. End with one line: "Say 'brief me on <Company>' for the full picture of one."
