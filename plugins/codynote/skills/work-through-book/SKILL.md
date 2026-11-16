---
name: work-through-book
description: Work through the owner's whole book of accounts in CodyNote, unattended, one account per conversation (the owner is a Panzura CSM) - first works up to three refreshes the owner queued in CodyNote (Queue for Claude), then asks CodyNote which account is next (finishing one a stopped run left part way first), imports it from a blank slate or refreshes it with email, records each step, marks it done and stops. Everything it makes waits in CodyNote for the owner's approval. Use when the owner says "work through my book", "update all my accounts", "keep going through the book", and for the hourly scheduled task's prompt "Work through my book in CodyNote".
---

# Work through the book

The owner's words: "give Claude a goal to update all the accounts using that skill and the other skills as needed. I will come back and approve everything afterward." CodyNote keeps the run (Settings › Claude › Work through the book) and the queue (the Queue for Claude page); an hourly scheduled task in Claude Desktop sends this skill's prompt. Each pass: **the queue first, then exactly one book-run account**, never two: a conversation that runs out of room part way is finished by the next hour's.

**For field X, follow its recipe** (`field-guide.md` next to this file, one line per field; `codynote_recipe(id)` has the whole recipe: where to look in order, what counts, the format, what never to do). Never stop for an approval: the owner approves each suggestion in CodyNote.

1. `codynote_ping(skills: "0.55.0")` once (the `skills` value tells CodyNote which skills Claude has). Its `version` must be **0.55.0** or later: if it is older, say "Update CodyNote to 0.55.0 or later, then ask again" and stop. Call only the tools in its `tools`.
2. **The queue first** (refreshes the owner asked for with Queue for Claude): `codynote_refresh_queue_next()` (pass `hourly`, the default). Its `action`:
   - **`run`**: work it as `queue-run.md` (next to this file) says, then ask for the next run: at most 3 a pass, none after a whole-account run. What is left waits for the next hour.
   - **`none`**: nothing queued: go on to step 3.
3. `codynote_book_run()` (with `start: true` only when the owner asked to start a new run in this conversation). Its `action`:
   - **`finish`**: the last run stopped part way on this account (no activity for 70 minutes or more). Do what `do` says: for an import, the Import a customer skill's "continue the <Company> import" (`codynote_import_start` resumes it from `holds`); for a refresh, the Refresh account skill's refresh with email, picking up after `last_step`.
   - **`wait`**: another conversation is still on an account. Say "A run is still working on <Company>; nothing to do this hour." and stop.
   - **`start`**: the next account. `kind` `import` (no meetings or people yet; deals an earlier deals refresh brought in are fine): the **Import a customer** skill ("import <Company> from scratch"). `kind` `refresh`: first the **Refresh account** skill's verify mode ("check <Company>": what is done, still open or no longer true, with proof), then its refresh ("refresh <Company> with email"), in one pass.
   - **`done`**: every account is finished. Say so with what waits for approval (`waiting`), and stop.
   - **`none`**: no run. Say "No book run: start one in CodyNote (Settings › Claude › Work through the book)." and stop (after any queued runs of step 2).
   - When step 2 worked a whole-account run, skip the book-run account this pass unless the answer is `finish` or `wait` (say so in the reply): the next hour takes it.
4. Run that skill for that one account, without stopping to ask the owner (they approve everything afterward): new contacts and what builds on them go in the same pass (`account-picture.md`, "One pass"); what is wrong gets its fix suggested and HubSpot-side cleanup goes in `hubspot_fix` (`account-picture.md`, "Corrections"), never a "fix by hand" list. After each of its steps, `codynote_book_run_step(company, step)` with a few words ("verify: 14 checks", "meetings: 12 staged", "deals", "email"), so a later run knows where to pick up.
5. At the end, `codynote_book_run_done(company, note)` with one line of what was made ("import ready: 214 lines", "refresh: 9 cards"). A source the skill can't do without is missing (no HubSpot, no Fathom for an import with meetings to pull): `codynote_book_run_done(company, note: "<why>", skipped: true)`. Waiting approvals never hold the run up.
6. Stop. Reply in three lines: the queued runs worked (or none) and the book-run account with what was done, what waits for approval (`waiting`), and that the next hour takes the next account.
