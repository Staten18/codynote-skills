# One queued run (Queue for Claude)

How the **refresh-queue** and **work-through-book** skills work a run the owner queued in CodyNote with **Queue for Claude** (on an account's header, a tab's menu, one field, or a card's menu). Each run's prompt is complete on its own: the same words Copy the prompt copies for that scope.

When `codynote_refresh_queue_next` answers `action: "run"`:

1. **Do exactly what its `prompt` says**, for that one account (`company`), as if the owner had pasted it: it names where to start, the recipes and the write tool. Pass `refresh_run_id` (the run's `run_id`) on every `codynote_suggest_changes` call of the run, and use its `company_id` wherever a tool takes the company.
2. **A run whose prompt starts "Wrap up" a meeting** (scope kind `meeting`, its record the meeting's id; the app queues it when the owner pastes a meeting) is **call mode for that one meeting** (the refresh-account skill): read the meeting, sort its action items with recipe `item.pick` (the transcript and both Fathom lists as one pool, fold first), then the cleanup of the account's open items, then the item checks (verify mode with `scope: "items"`, every source in order CodyNote → Fathom → Outlook → Teams → HubSpot, a source that can't be read skipped and said, a Summary where one is missing or stale). Nothing else.
3. **At the end** (with the run's `attempt`), `codynote_refresh_queue_done(run_id, summary)` with one line of what you suggested and what you could not find; when a source it needs is missing or it can't be done, `codynote_refresh_queue_failed(run_id, error)` with why.
4. **Then the next run**, the same way: **at most 3 runs in one conversation**, and none after a run whose scope is the whole account (`The whole account`, or a check of everything): those are as big as a whole account, and the conversation would run out of room. What is left (`left`) waits for a new chat or the next hourly pass.

**Wrapped up a meeting in chat without the queue?** `codynote_refresh_queue_done(meeting_id, summary)` closes that meeting's still-queued wrap-up (`nothing_queued` when there was none).
