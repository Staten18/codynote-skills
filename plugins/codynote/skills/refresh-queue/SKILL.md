---
name: refresh-queue
description: Work the refreshes the owner queued in CodyNote with Queue for Claude, now, in this chat, instead of waiting for the hourly pass (the owner is a Panzura CSM) - takes the oldest queued run, does exactly what its prompt says for that one account, tags what it suggests with the run and reports it done or failed, then the next. Or lists what is queued. Everything it makes waits in CodyNote for the owner's approval. Use when the owner says "work the queue", "run the queue", "do what's queued", "what's queued", "what's in the queue", or similar.
---

# Work the queue

The owner queues refreshes in CodyNote with **Queue for Claude**; Claude Desktop's hourly pass takes up to three of them before the book run. This skill does the same on demand, in this chat. `queue-run.md` (next to this file) says how one queued run is worked.

**For field X, follow its recipe** (`field-guide.md` next to this file, one line per field; `codynote_recipe(id)` has the whole recipe). Never stop for an approval: the owner approves each suggestion in CodyNote.

1. `codynote_ping(skills: "0.55.0")` once (the `skills` value tells CodyNote which skills Claude has). Its `version` must be **0.55.0** or later: if it is older, say "Update CodyNote to 0.55.0 or later, then ask again" and stop. Call only the tools in its `tools`.
2. **"What's queued"** (only a look): `codynote_query` with `select c.name as company, r.label, r.status, r.queued_at, r.started_at from refresh_runs r join companies c on c.id = r.company_id where r.deleted_at is null and r.status in ('queued', 'running') order by r.queued_at`. Reply with one line per run (account, scope, queued or being worked, since when) and stop. Nothing is taken.
3. **"Work the queue"**: `codynote_refresh_queue_next(pass: "chat")`. Its `action`:
   - **`run`**: work it as `queue-run.md` says, then ask for the next run (at most 3 in this chat, none after a whole-account run). When runs are left (`left`), say how many and that a new chat ("work the queue") or the next hourly pass takes them.
   - **`none`**: nothing queued. Say "Nothing is queued in CodyNote." and stop.
4. Reply with one line per run (account, scope, what was suggested or why it failed), and that the suggestions wait in CodyNote › Claude suggestions (the Queue for Claude page's **Open the review** shows one run's).
