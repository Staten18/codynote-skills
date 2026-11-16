---
name: book
description: Show the owner (a Panzura CSM) where to look across their whole book of accounts, from CodyNote - which accounts need attention and why, renewals and QBRs coming up, the longest waits, what needs a look in CodyNote, and the two accounts to open first. Use when the owner asks "where should I look today?", "what needs attention?", "my book", "which accounts need me?", "morning look", or similar. Read-only.
---

# Where should I look

One screen across the owner's book: what needs them, what's coming, what they're waiting on, and where to start. Only when the owner asks; nothing here runs on its own.

This skill only reads. Never save, suggest or change anything, and never send anything. It never reads Outlook.

Read `account-picture.md` (next to this file), "The book and the lint", for the signals, the rules and the lint kinds.

**First, once:** `codynote_ping(skills: "0.55.0")` (the `skills` value tells CodyNote which skills Claude has). Its `version` must be **0.55.0** or later (an older CodyNote answers in one line, without `version` and `tools`): if it is older, say "Update CodyNote to 0.55.0 or later, then ask again" and stop. Call only the tools in its `tools`.

## 1. Gather

1. `codynote_book` with `only: "mine"` (`only: "all"` when the owner asked for the whole book): one row per account with its signals (touch, renewal, capacity, waiting, sentiment, risks, read, qbr; each with a tone and a reason) and `why` (the bad ones, using the owner's saved rules). An account needs attention when it has a `why`. Managed accounts (0.34.0): `managed_by` names the MSP that manages an account (an MSP is one of the owner's accounts, a Panzura partner), and an MSP carries `manages_count` and `manages_needing_attention` (the names of its managed accounts whose own rules fired; `managed_attention` says it in a line).
2. **Needs a look:** `codynote_query`: `select company_id, company, kind, title, detail, since from v_lint order by company, kind, since` (only the owner's accounts from step 1).
3. **Waiting:** `codynote_list_action_items` with `status: "waiting"` for the item text and who it's with, for the accounts with the oldest waits.
4. **Email not read lately:** the owner's current accounts whose email watermark is older than 30 days, or never set:
   ```sql
   select c.id as company_id, c.name as company, ls.emails_through
   from companies c left join v_last_scan ls on ls.company_id = c.id
   where c.deleted_at is null and c.account_group = 'current' and c.is_my_account = 1
     and (ls.emails_through is null or substr(ls.emails_through, 1, 10) < date('now', 'localtime', '-30 days'))
   order by ls.emails_through is null, ls.emails_through, c.name
   ```

## 2. Think

- **Needs attention:** each account with a `why`. Pick **the one thing to do** from its signals (no touch: a check-in with a reason; renewal and no opp: open the renewal opp or ask the RSM; MC high: a capacity conversation; read behind: refresh the read). "Sentiment never set" and "No read yet" are watch signals, not attention, unless a renewal is inside the window; accounts with only those go in one closing line ("N accounts have no read or sentiment yet").
- **MSPs and their accounts:** each account is judged by its own rules and listed as itself. Show a managed account under its MSP, indented ("  ↳ <Company> (via <MSP>)"), and the MSP with `manages_needing_attention` ("<MSP>: <account> needs attention") even when the MSP's own signals are fine. Several managed accounts needing attention at once may share a cause at the MSP: say so in My read. Never add the managed accounts' ARR to the MSP's.
- **Coming up:** renewals 0-120 days out (soonest first, ARR internal), and QBRs due: the `qbr` signal on watch (last QBR older than the owner's QBR rule, default 120 days). Accounts with no QBR on record: one line with a count.
- **My read:** the two accounts to open first, weighing renewal timing and ARR, how many bad signals, red or at risk, and how long a wait has sat. Say why in a sentence each.

Numbers come from the tools only. Missing data is a finding, not a guess: say "no read", "no sentiment", "no MC reading", "no renewal on file".

## 3. Write

One screen. Short lines; dates "Oct 9"; money "$84k"; days as numbers.

```
**Your book, <date>** · <n> accounts · <n> need attention

**Needs attention**
- **<Company>**: <why, short> → <the one thing to do>
- **<MSP>**: <its own why, or "fine itself"> · manages <n>: <managed_attention>
  - ↳ **<Managed company>** (via <MSP>): <why, short> → <the one thing to do>

**Coming up**
- <Company>: renews <date> (<n> days, <ARR>) · <open opp or "no opp">
- <Company>: last QBR <date> (<n> days ago)

**Waiting**
- <Company>: <item> · with <person> · **<n> days**

**Needs a look in CodyNote**
- <Company>: <title> (<what's wrong, short>) · <title> (...)

**My read:** open <Company> first (<why>), then <Company> (<why>).
No read or sentiment yet: <Company>, <Company>.
Email not read in 30+ days: <Company> (<date>), <Company> (+<n> never read).
```

Leave out a section with nothing in it. Keep Waiting and Needs a look to about five lines each (longest waits, accounts with most rows first) and say "+<n> more". The email line is optional: only the accounts that need attention or renew soonest (about five), with the rest as a count. End with: "Say 'brief me on <Company>' before a call, or 'wrap up my <Company> call' after one (or 'refresh <Company>' any time), to refresh its read."
