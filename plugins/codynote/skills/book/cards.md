<!-- Generated from src/shared/cards.ts by scripts/gen-skill-shared.ts. Do not edit: run `pnpm gen:skills`. -->

# Cards and their boxes (CodyNote 0.30.0 and later)

Every card (an action item, a storyline) has named boxes. The connector refuses a suggestion that breaks these rules and says which box; the update brief prints the same list. Three shapes: **one box** (text, rewritten whole), **lines** (dated, sourced, only ever added) and **checklist** (lines that get ticked done, never removed). "Required" binds Claude on a new card it suggests, never the owner.

## Rules

- A text box Claude set may be rewritten by Claude; one the owner typed is never replaced by Claude (the owner can clear it).
- Lines are only added. A checklist line is ticked with the source that answered it, never removed.
- Every box or line Claude suggests carries a source (a meeting, document, link or email id from the brief).
- Notes is the owner's own: the old details, unchanged. Claude may offer a `split` of Notes into the boxes once per card; Notes keeps its text either way.

## The action item card

- Summary (summary): The executive summary: one or two plain sentences on what this is and where it stands, no greeting. The same words open Send this. [one box, rewritten whole; optional; at most 200 characters; recipe item.summary]
- Next step (next_step): The very next action, and who else is involved. Rewritten as it moves. [one box, rewritten whole; required on a new card; at most 600 characters; recipe item.next_step]
- Done when (done_when): The concrete end state: what is true when this is finished. [one box, rewritten whole; required on a new card; at most 600 characters; recipe item.done_when]
- Situation (situation): The ask / Where it stands / Why it matters, in plain words: three short labeled paragraphs. [one box, rewritten whole; required on a new card; at most 1200 characters; recipe item.situation]
- Context (context): What was said, by whom, on what date; numbers and deadlines. One line per thing; add a line, never rewrite one. [lines, only added; optional; at most 300 characters; recipe item.context]
- Goal (goal): What finishing this achieves for them and why it matters, in a sentence. Kept from before 0.52.0: shown until a Situation is written, which says it. [one box, rewritten whole; optional; at most 600 characters; recipe item.goal]
- Notes (notes): The owner's own notes (the item's details from before 0.30.0, unchanged). Claude never writes here; it may offer to split them into the boxes above. [one box, rewritten whole; the owner's alone; recipe item.notes]

## The storyline card

- Now (now): Where it stands today, 1-3 sentences (at most 600 characters), rewritten whole; the app dates it. [one box, rewritten whole; required on a new card; via record bullet (now_text); at most 600 characters; recipe storyline.now]
- History (history): One past-tense sentence per thing that happened, dated the day it happened, sourced. Only ever added. [lines, only added; optional; via record history; at most 300 characters; recipe storyline.history_text]
- Helps (helps): Where Panzura helps with this, one line each, sourced (the call, or linked research). [lines, only added; optional; at most 300 characters; recipe storyline.helps]
- Hurts (hurts): Where this works against Panzura or the account, one line each, sourced. [lines, only added; optional; at most 300 characters; recipe storyline.hurts]
- Signals to watch (watch): What to keep an eye on: a date, a decision, a person, a competitor. One line each, sourced. [lines, only added; optional; at most 300 characters; recipe storyline.signals_to_watch]
- Open questions (open_questions): What we still need to find out, one per line. Tick one when the material answers it, with the source; never remove it. [checklist; optional; at most 300 characters; recipe storyline.open_questions]
- Notes (notes): The owner's own notes (the storyline's details from before 0.30.0, unchanged). Claude never writes here; it may offer to split them into the lists above. [one box, rewritten whole; the owner's alone; recipe storyline.notes]

## The person card

- Sentiment to Panzura (sentiment): Positive, neutral or negative, and why, in one line as of the latest call or email. Rewritten whole; the history is in Observations. [one box, rewritten whole; optional; at most 300 characters; recipe person.sentiment]
- Background (background): Career, how long at the company, what they own, where they came from. [one box, rewritten whole; optional; at most 600 characters; recipe person.background]
- Personality (personality): How they come across on calls and in email, and how to work with them. [one box, rewritten whole; optional; at most 600 characters; recipe person.personality]
- Interests and small talk (interests): Hobbies, family they mention, teams, places, things to ask about next time. One per line, with where it came up. [lines, only added; optional; at most 300 characters; recipe person.interests]
- Sore spots (sore_spots): What not to raise, what went wrong before, what frustrates them. One per line, sourced. [lines, only added; optional; at most 300 characters; recipe person.sore_spots]
- Observations (observations): The evidence: one line per thing noticed in a call or email, dated and sourced (what they said or did). Everything above is built from these. [lines, only added; optional; at most 300 characters; recipe person.observations]
- Notes (notes): The owner's own notes on the person (the contact's notes, unchanged). Claude never writes here. [one box, rewritten whole; the owner's alone; recipe person.notes]

### Person rules (every skill that adds to a person card)

- **Observations: 3–6 per person per run, at most 6; fewer is fine, never pad.** Refresh account's call and email modes add two or three at most, as part of a wider update. Only what changes how the owner deals with them: what they decide or sign off on, whom they defer to, how they like to communicate, what they praised about Panzura or complained about, anything personal they shared.
- **No environment or technology facts on a person.** A product, a protocol, a vendor, a project's setup belongs to a storyline or a Ring (call mode), never to a person card. If that is the only thing a person said, they get no card.
- **What others said about a person is never its own observation.** It may inform the Personality box, marked "others say".
- **Interests and sore spots only when real:** something the owner would bring up or avoid on the next call, one line each, at most 3 interests and 3 sore spots per person per run, never a repeat of an observation.
- **Boxes only when they add something:** sentiment, and a personality note of at most 2 sentences; background only when the title does not already say it.
- **Partners:** role, whom they replaced, how they work with the owner. Nothing else.
- **Quotes:** the person's own words, cleaned up, under 15 words. Never a transcript fragment with filler.
- **The test before every line:** "would I say this to the owner before their next call with this person?" No: drop it.
- **The first run on an account** gets the same cap: the best 6 is what the owner reads.
- **People mode's reply** says, per person, how many lines were left out and why ("12 setup facts: storylines, not the person").

## Suggestion shapes (codynote_suggest_changes)

```json
{"record": "box", "action": "change", "card": "item", "id": "<item id>", "fields": {"key": "next_step", "text": "Send the sizing sheet to Dana by Friday"}, "sources": [...], "reason": "..."}
{"record": "line", "action": "add", "card": "storyline", "id": "<bullet id>", "fields": {"key": "helps", "text": "Global file locking covers the Denver and Austin studios", "date": "2026-10-20"}, "sources": [...], "reason": "..."}
{"record": "tick", "action": "change", "card": "storyline", "id": "<bullet id>", "fields": {"key": "open_questions", "line_id": "<line id>"}, "sources": [...], "reason": "answered on the Oct 20 call: ..."}
{"record": "split", "action": "change", "card": "item", "id": "<item id>", "fields": {"boxes": {"situation": "...", "done_when": "...", "next_step": "..."}, "lines": {"context": [{"text": "...", "date": "2026-10-20"}]}}, "sources": [...], "reason": "from the owner's notes"}
{"record": "action_item", "action": "merge", "company": "Acme", "fields": {"keep_id": "<item id kept>", "fold_id": "<item id folded into it>"}, "reason": "Dana's pricing, asked twice", "sources": [...]}
```

A new action item (`record: "action_item", action: "add"`) carries its boxes in `fields.boxes` (situation, done_when, next_step required; **goal is retired since 0.52.0**: the connector refuses it with "retired in 0.52.0: write situation", and existing Goals stay as they are) and its first Context lines in `fields.lines.context`. The Situation (recipe `item.situation`) is the story in three labeled parts, each its own paragraph of 1–3 sentences, ≤ 1,200 in all: "The ask:" (what was asked or promised, by whom, with their role), "Where it stands:" (what has happened, who has the next move), "Why it matters:" (the renewal, deadline or storyline it bears on); jargon explained once, "My read:" marks Claude’s own sentence; rewritten whole (a `box` change with `as_of`) as things change. The title (recipe `item.text`, 0.53.0) is "Topic: what exactly, how much, for whom", ≤ 90, no leading "do this" verb; one item per topic (recipe `item.pick`). Two open items that are one thing are folded with one `merge` (0.52.0, recipe `item.fold`): `fields.keep_id` stays, `fields.fold_id` goes into it, `reason` says why they are one; on approval the kept item gets the other's Context lines, follow-ups, sub-items and a History row, the other goes to Trash marked "Folded into …", and one Undo puts it all back. A new storyline (`record: "bullet", action: "add"`) carries `now_text`, its first History lines and, in `fields.boxes` / `fields.lines`, any Helps / Hurts / Signals to watch / Open questions it has.
