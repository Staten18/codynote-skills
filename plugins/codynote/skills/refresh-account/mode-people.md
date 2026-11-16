# People mode (was Profile people)

Refresh account's people mode: "profile the people at Acme", "who's who at Acme", "what do I know about Dana", "update Dana's profile", "profile Sam" (a teammate). From every call transcript and email reference in CodyNote: merge copies of one person, name the unnamed speakers where the call makes it plain, then a few dated, sourced Observations for each person (at most 6, only what changes how the owner deals with them) and the boxes they support; reply with a who's who. Teammates who speak on the calls are profiled too.

Who the people on an account are, how they feel about Panzura, what to talk about and what to avoid: built from what they said on calls and wrote in email, one observation at a time, for the owner to approve in CodyNote › Claude suggestions. Only when the owner asks; nothing here runs on its own.

Nothing is saved except **suggestions** through `codynote_suggest_changes`. This mode never reads Outlook: email comes from the references CodyNote has indexed (email mode, "catch up on email for <Company>", for what people wrote since).

`cards.md` (next to this file) has the person card's boxes, their limits, the person rules and the `box` / `line` shapes; `work-order.md` the People step this mode works; `account-picture.md` "People" the `speaker` shape and the personas.

**For field X, follow its recipe** (`field-guide.md` next to this file, one line per field; `codynote_recipe(id)` has the whole recipe: where to look in order, what counts, the format, what never to do). Never stop for an approval: the owner approves each suggestion in CodyNote.

## 1. Find the account and open the scan

1. **One person named** ("what do I know about Dana", "update Dana's profile", "profile Sam"): `codynote_read` with `name`. Several match: ask which and stop. A contact's account is theirs; a teammate's is the account the owner named, else the one of their newest customer meeting (say which, and that "profile Sam at <Company>" works the others).
2. `codynote_get_customer` with the account and `detail: "full"`. If it matches more than one account, or none, ask which and stop. Keep `overview.company_id`, `people` (contacts and teammates on the account: `person_id`, `kind`, persona `role`, `sentiment`, the boxes, the newest Observations, `last_met_on`, `last_emailed_on`), `contacts` (ids, titles, `active`), `meetings` (ids, dates, titles), and `inactive_people`: contacts no longer there (`active: false`: retired, left or changed role). They are not in `people` and are not profiled, though naming one as a speaker in an older call is still fine. Only when the owner names one of them ("what do I know about Lee") read their card.
3. **Copies first:** `codynote_lookalikes` with `kind: "people"` and the company. Each group is one person typed more than once (on this account, or on this and others: a partner's account executive typed on every account they cover). Strong groups (same email, or same name at the same partner org) get a `merge` in step 3; a "maybe" group (same name only) only when the calls or emails show it is one person, else it goes under Heads-up. Profile the copy you would keep (`keep`, unless another has the owner's notes or more of the card): its Observations land on the survivor either way. People linked to this account from another (`linked: true` in `people`) are on this account too: profile them here like the account's own.
4. `codynote_get_update_brief` with the company. Keep its `scan_id` (on both `codynote_suggest_changes` calls), `cards` and what the owner rejected or removed (never suggest those again). If it says nothing is new, carry on: profiling reads every call and email on the account, not only the new ones, and leaves the new material for the next update.

## 2. Read (no suggestions yet)

1. **What is known:** `codynote_read` (`id`: the `person_id`) for each person on the account (only the one named, when one was): every line of every box with its id, date and source, their meetings and email references. Nothing already on a card is suggested again; a box with `set_by: "owner"` is never rewritten.
2. **The calls:** `codynote_read` on each meeting with a transcript (`codynote_map` lists them with their transcript words), newest first. It returns the text with names where speakers are named, and `speakers`: every "Speaker N" label with `name` (null when unnamed), `person_id`, `turns` and the `first` things it said. On an account with many calls, read the newest six whole and the rest with `find` for each person's name (what they said, and what others said about them: "Dana's been here twelve years").
3. **The email:** each person's email references (date, subject, people, gist, message id) from `codynote_read` on the person (step 2.1); the account's newest 20 are `emails` in `codynote_get_customer`. A gist is a line, not the email: an observation from it says only what the gist says.

## 3. Suggest, in the People step's order (one `codynote_suggest_changes` call with `scan_id`)

Every record has `company`, a one-line `reason` and `sources` (the meeting with `date` and a short `quote` of what the person said, or the email `{kind: "email", id: <message_id>, title, date}`). Give each change a `confidence` (`high`, `medium` or `low`: how sure you are), and copy every quote exactly as it is in the source: CodyNote looks for a meeting's quote in its transcript and marks one it can't find "quote not found". Profile lines and boxes are `box` / `line` records with `card: "person"` and `id` = the person's `person_id` (the contact's or teammate's id). Put them in this order:

0. **Copies merged** (step 1.3): one `merge` per group, `id` = the copy to keep, `fields.merge_ids` = the others, and a source line that says why they are one person ("same email on ⟨account⟩ and ⟨account⟩"). The owner approves each; everything the copies held moves to the one kept (`account-picture.md`, "People across accounts"). For example: `{"action": "merge", "record": "contact", "company": "Contoso Health", "id": "<person_id>", "fields": {"merge_ids": ["<copy_id>"]}, "reason": "Morgan Reyes was typed on Contoso Health and on Fabrikam Clinics: one person.", "sources": [{"kind": "other", "title": "same email on Contoso Health and Fabrikam Clinics"}]}`
1. **Speakers.** Each label with `name: null` that the transcript makes plain gets a `speaker` record: `meeting_id`, `fields: {label: "Speaker 2", name, contact_id}` (`teammate_id` for a teammate; `is_owner: true` for the owner, who is never profiled). Plain means someone names them right after they speak ("Thanks, Dana" after Speaker 2), they introduce themselves ("This is Dana from IT"), or they're the only attendee who could have said it. The `reason` quotes that line. A guess from the topic is not plain: leave the label and name it under Heads-up. Words of an unnamed speaker go on a profile only when this same call names them.
2. **Observations: 3–6 per person per run, at most 6; fewer is fine, never pad.** The first run on an account gets the same cap: the best 6 is what the owner reads. Each is a `line` with `key: "observations"`, dated the day of the call or email, at most 300 characters, the quote in the source. Only what changes how the owner deals with them: what they decide or sign off on ("I sign off on renewals"), whom they defer to, how they like to communicate, what they praised about Panzura or complained about, anything personal they shared. Facts about them, never a judgment of character; nothing sensitive (health, religion, politics, family trouble).
   - **No environment or technology facts on a person.** A product, a protocol, a vendor, a project's setup belongs to a storyline or a Ring (call mode), never to a person card. If that is the only thing a person said, they get no card.
   - **What others said about a person is never its own observation.** It may inform the Personality box, marked "others say".
   - **Quotes:** the person's own words, cleaned up, under 15 words. Never a transcript fragment with filler.
   - **The test before every line:** "would I say this to the owner before their next call with this person?" No: drop it. Keep count of what you drop, per person, for the reply.
3. **The boxes the observations support**, only when they add something, each sourced to the observations behind it:
   - `sentiment` (a `box`): positive, neutral or negative to Panzura, and why, as of the latest call or email; only when it changed or is blank. This is the person's own. **Never** the company's sentiment (the owner's green / yellow / red).
   - `interests` and `sore_spots` (`line`s), **only when real:** something the owner would bring up or avoid on the next call, one line each, at most 3 interests and 3 sore spots per person per run, never a repeat of an observation. An interest says where it came up so the owner can say "you mentioned…"; a sore spot is what not to raise or what went wrong.
   - `background` and `personality` (`box`es, rewritten whole): only once two or more observations support them. Personality is a note of at most 2 sentences on how to work with them; background only when the title does not already say it.
   - **Partners** (a partner's people, persona `partner`): role, whom they replaced, how they work with the owner. Nothing else.
   - **Persona:** a `contact` change with `fields: {role}` (the personas in `account-picture.md`) when the material makes it plain. Contacts only.
   - **Reports to:** a `contact` change with `fields: {reports_to_contact_id}` when someone says it ("I'll run it by Lee, my boss").

**Teammates** (Panzura people on the calls: an SE, the RSM) are profiled the same way from what they say on the account's calls: Observations, Interests, Sore spots, Background, Personality. No persona and no sentiment for a teammate. Internal team calls are not read.

Someone who spoke but is neither a contact on this account nor a teammate: look them up first (`codynote_read` with their `name`, every account). Already in CodyNote on another account: a `link` (`{"action": "link", "record": "contact", "id": <their person_id>, "company": <this account>}`), never a second contact; their lines go in the same call, under this account, with their `person_id` (approving one adds the link first). Nobody matches: a `contact` add (name, title if said) with `ref: "c1"`, and their speaker name, Observations and boxes in the same call with `"c1"` where their `person_id` goes (`account-picture.md`, "One pass"). Never wait for the owner to approve the contact first.

## 4. Finish the scan (`close_scan`)

A last `codynote_suggest_changes` call with `scan_id`, a `summary`, `changes: []` and `close_scan: {company, coverage}`, where **`coverage`**: `people` is the honest count of what you suggested (`"nothing new"` when every card already had it), every other step `"skipped: profiling only"` (the new material stays new for the next update) and email `"not asked"`. No Knowledge bullets; nothing for the account's read.

```json
{"read_material": "skipped: profiling only", "email": "not asked", "people": 14, "account": "skipped: profiling only", "deals": "skipped: profiling only", "storylines": "skipped: profiling only", "items": "skipped: profiling only", "facts": "skipped: profiling only", "knowledge": "skipped: profiling only", "read": "skipped: profiling only"}
```

**Last, before the reply (0.53.0): the cleanup.** Bring this account's open items that do not meet the rules up to them: a "Topic: what exactly, how much, for whom" title, a three-part Situation (The ask / Where it stands / Why it matters), a next step who / what / when, folds, follow-ups. Follow `mode-items.md` (its steps 2 and 3, rounds of at most 12 cards, one `codynote_suggest_changes` call per round without `scan_id` once the scan is closed). This is the account-by-account cleanup; there is no separate cleanup skill. Items that already pass get nothing; say what changed in the reply.

## 5. Reply

Show each person as they will be once the suggestions are approved; a suggested change reads "Neutral → Positive (suggested)".

```
**Who's who at <Company>** · <n> calls and <n> emails read
| Name | Persona | Sentiment | One line |
|---|---|---|---|
| Dana Whitfield | Champion | Positive | Driving the Dallas move; wants the quote before the Jan budget. Ask about her trail running. |
| Sam Ortiz (Panzura SE) | n/a | n/a | Ran the sizing; Dana trusts his numbers. |
No longer there: <names from `inactive_people`, one line; leave out when none>
**Left out** (one line per person: how many lines were left out and why; leave out when none were):
- Dana Whitfield: 12 setup facts (storylines, not the person), 3 things others said about her
Suggested: <n> copies merged, <n> speakers named, <n> observations, <n> boxes (<2 sentiment, 3 interests, 1 sore spot, 1 persona, 1 reports to>). Review in CodyNote › Claude suggestions.
**Checked:** <people 14 · everything else skipped: profiling only>

**Heads-up:** <only if it matters: a speaker you couldn't name (meeting, label, why); someone who isn't a contact yet; a "maybe" pair of copies you left alone (same name only, nothing shows it is one person); a contact who turned negative; a sentiment or box the owner typed that the calls now contradict.>
```

For one person ("what do I know about Dana"), instead of the table: their persona, sentiment, background, personality, interests (as openers), sore spots, last met and emailed, and their three newest observations with dates, then the Suggested and **Checked:** lines. Nothing new to suggest: say the card is current and send `coverage` with people `"nothing new"`. If CodyNote drops a record, keep the rest and say what and why.
