# Items mode (was Tidy items): the cleanup pass

Refresh account's items mode: "tidy Acme's items", "clean up Acme's items", "clean up duplicate items on Acme", "any doubled-up items on Acme?", "tidy items on Acme and Globex" (one account, or every account the owner names). Since 0.53.0 it is the **"bring items up to the rules" pass**, and it is also the last step of every other mode (`SKILL.md`, "The cleanup"): every open item on the account ends up reading cold (someone who missed the call knows what it is, why it matters and what is next), and items that are one thing are folded into one. The owner approves each card in CodyNote › Claude suggestions, in rounds; the reply shows what was suggested and what was left.

The owner's words (2026-11-05): item cards were "garbage", fragments that made no sense on their own. What fixes that, per item, in this order:

- **(a) a Situation** (recipe `item.situation`) in three labeled parts, each its own paragraph of 1 to 3 sentences, 1,200 characters at most: "The ask:" (what was asked or promised, by whom, with their role), "Where it stands:" (what has happened, who has the next move), "Why it matters:" (the renewal, deadline or storyline it bears on). Every id, host name, ticket or version number explained the first time. Written when the item has none; **rewritten** when the one it has fails the cold-read test (would someone who missed the call know what this is, why it matters and what to do next?). Never a Goal (retired; the connector refuses it; the Goals already there stay and feed the Situation).
- **(b) a title rewrite** (recipe `item.text`) when the title is not "Topic: what exactly, how much, for whom": at most 90 characters, the topic in the owner's own short style (the topic word of the owner's open item when there is one), no leading "do this" verb (that is the Next step), no bare label ("Ticket 4410", "Quote"), nothing that needs the card to make sense. A `change` with `text`; the old wording stays in History. **Rename an owner-typed title only with something on record behind it** (a Context line, a meeting, an email, a ticket that says what it is): "403 Error" with nothing behind it stays as it is. Never a rewrite that changes what the item is.
- **(c) a Next step** (recipe `item.next_step`) that names a day ("by Fri Nov 14") or "this week", when the item has none or the one it has names no day. Never a date no source gives: "this week" when the move is the owner's now.
- **(d) a storyline link** (recipe `item.bullet_id`) when the item plainly serves one of the account's storylines and has none.
- **(e) a fold** (recipe `item.fold`) for each pair of items that are one thing, as one `merge` suggestion. Folds come first (recipe `item.pick`: one topic, one item).
- **(f) follow-ups** (recipe `item.pick`): an ask the owner waits on, from any source (meeting, email, ticket), is a follow-up "waiting on <name, role> since <date>" on its item, with the item's status waiting when nothing else is open on it.

And when the account has a read: **the read's Do first and Situation** (recipes `read.do_first`, `read.situation`), suggested through a refresh of the read when they are missing or fail the same cold-read test (a read goes only in a scan: run call mode's steps 1.3 and 5 for it, with coverage saying every other step "nothing new").

Nothing is saved except **suggestions** through `codynote_suggest_changes`. For field X, follow its recipe (`field-guide.md` next to this file; `codynote_recipe(id)` has it whole). `cards.md` has the boxes and the `box` / `line` shapes; `account-picture.md`, "Tidy", the merge shape. Each card is the approval, so never stop to ask first.

## 1. Find the accounts

The account the owner named, or each one in a list ("tidy Acme and Globex"). For each, `codynote_get_customer` with the name; keep `overview.company_id`, `summary` (its Do first and Situation lines, or "not written yet"), `storylines` (ids and one lines, for the links) and the read. If a name matches more than one account, or none, ask which and stop. Several accounts: work them one at a time, in the order named.

## 2. Read (no suggestions yet)

1. **Every open item:** `codynote_list_action_items` with the company and `mine_only: false` (a teammate's item is on the account too), then `codynote_read` on each item for its card: Situation (or Goal on an old item), Done when, Next step, Context lines with dates, Notes (the owner's: read, never written), follow-ups, sub-items, `meeting_id`, `bullet_id`. For the Situation's sources, in the owner's order: the item's Context lines and the meeting it came from (`codynote_read` on the `meeting_id` with `find` on the item's words), the storyline it belongs to, the newest email reference naming its people (`codynote_get_customer` `emails`; the email itself only when the owner asked for email), the HubSpot ticket or deal the item names (read-only).
2. **Per item, decide (a) to (d) and (f):** which of them it needs. An item whose Situation, title and next step already pass gets nothing: say so in one line of the reply.
3. **Pairs:** `codynote_lookalikes` with `kind: "items"` and the `company_id`. For each pair, read both items and their Context lines and decide:
   - **Fold:** a `same` pair (or a `near` one that plainly says the same thing) that is one task said two ways, with no `hold`. Keep the item the tool names as `keep`: it holds what a fold would otherwise lose, else it is the fuller or older one.
   - **Leave:** two steps of one project, two people's tasks, a parent and its sub-item, a pair with `hold` (different owners, or both hold Notes, sub-items, follow-ups or documents: the owner decides those, so name them in the reply), or anything you're unsure of.
4. **The read:** when the account has one, does its Situation read cold, and does Do first name the one thing and why? "Not written yet" counts as failing.

## 3. Suggest, in rounds (never stop for approval)

Rounds keep the owner's review short: **at most 12 cards per round**, one `codynote_suggest_changes` call per round, no `scan_id` (this isn't an update). Order the rounds so the owner sees the whole account take shape: folds first (so no Situation is written for an item about to go), then, per item in the list's order, its Situation, title, next step and storyline link together. Every record has `company`, a one-line `reason` and a source (the meeting, email or ticket the Situation rests on; a fold's `{"kind": "other", "title": "same item twice on <Company>"}`). Give each change a `confidence` (`high`, `medium` or `low`: how sure you are), and copy every quote exactly as it is in the source: CodyNote looks for a meeting's quote in its transcript and marks one it can't find "quote not found".

- **Situation:** `{"action": "change", "record": "box", "card": "item", "id": "<item_id>", "fields": {"key": "situation", "text": "<two or three short paragraphs>", "as_of": "<the day of the newest thing in it>"}}`. A rewrite is the same shape (the old text stays in Earlier versions). Never over a Situation with `set_by: "owner"`: say what looks off in the reply instead.
- **Title:** `{"action": "change", "record": "action_item", "id": "<item_id>", "fields": {"text": "<Topic: what exactly, how much, for whom>"}}`, the reason naming what failed ("a bare label", "a step, not a topic", "the customer's task worded as the owner's") and what record is behind it.
- **The next step:** a `box` change with key `next_step` naming a day or "this week" (never over one with `set_by: "owner"`).
- **Storyline link:** `{"action": "change", "record": "action_item", "id": "<item_id>", "fields": {"bullet_id": "<storyline id>"}}`.
- **Fold:** one `merge` (`account-picture.md`, "Tidy"): `{"action": "merge", "record": "action_item", "fields": {"keep_id": "<keep>", "fold_id": "<fold>"}, "reason": "<why they are one task>"}`. Only the pairs you decided to fold; the owner rejects any they don't want, and one Undo puts a fold back.
- **The read** (when it fails): call mode's step 1.3 opens the scan and step 5 closes it with the read: health unchanged unless something says otherwise, the five short lines as they are when they still hold, `do_first` and `situation` written fresh, and `coverage` with every other step `"nothing new"`.

- **Summary** (0.54.0, when empty or stale): a `box` change with key `summary`, following recipe `item.summary`.

After each round, start the next; the owner approves in CodyNote at their own pace. If CodyNote drops a record, keep the rest and say what and why; a `notes` entry on a merge (the two do not look alike by words) goes in the reply.

**Then the item checks (0.54.0):** verify mode (`mode-verify.md`) with `scope: "items"`, every source in order (CodyNote → Fathom → Outlook → Teams → HubSpot), a source that can't be read skipped and said; its **Checked** line goes in the reply.

## 4. Reply

```
**Tidied <Company>** · <n> open items · <n> rounds, <n> cards waiting in CodyNote › Claude suggestions
Folds: "<fold text>" → "<keep text>" (<reason>)
Situations: <n> written, <n> rewritten (<the items, short>)
Titles: "<old>" → "<new>" (<what failed>) · Follow-ups: <n>
Next steps: <n> now name a day · Storylines: <n> linked
The read: Do first and Situation <suggested | already read cold | no read yet>
Already read cold: <the items that needed nothing, short>
Left: <one line per pair left, and why; items whose owner-typed boxes look off>
```

When `fold` had a sooner due date or a higher priority, say so: the item kept doesn't take it.
