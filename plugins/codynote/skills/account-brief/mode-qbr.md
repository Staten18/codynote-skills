# QBR mode (was QBR prep)

Account brief's QBR mode: "brief me on Acme for the QBR", "prep the Acme QBR", "QBR prep for Acme", "get me ready for the Acme business review / EBR / roadmap / QBR + Roadmap", or any of these "with email". From CodyNote (the account's read, storylines, company facts and each Ring's environment), HubSpot (tickets and logged emails), Fathom, recent news about the customer and Panzura's own solutions.

Get the owner ready for a QBR, business review or roadmap call: the quarter's story, what to say, what to ask and what to do first, built from the account's storylines, its stored read and facts, the customer's emails and tickets, outside news and what Panzura offers.

This mode only reads. Never save, suggest or change anything in CodyNote or HubSpot, and never send anything.

Read `account-picture.md` (next to this file) for the read, storylines, facts and the Ring environment fields, `cards.md` for the storyline and action item boxes, `work-order.md` for the order to think in, `panzura.md` for where Panzura fits (problem → product, what each needs first, customer stories, competitors), and with "with email", `email.md`.

## 1. Gather (everything before writing)

**The window.** A QBR or business review covers the time since the last QBR (`overview.last_qbr_shown`: the newer of the typed date and the newest QBR meeting); a roadmap call, since the last roadmap (`overview.last_roadmap_shown`); a joint QBR + Roadmap call, since the older of the two. A `qbr_roadmap` meeting (one joint QBR + Roadmap session, `account-picture.md` "Day to day") is the last QBR **and** the last roadmap: read it for both parts, the review and asks for Wins and Open issues, and what Panzura showed and how they took it for What's new from Panzura. With neither, the last 6 months. Its start is the *window start*.

**CodyNote:**
1. `codynote_get_customer` with the name and `detail: "full"`. If it matches more than one account, or none, ask which and stop. Note `overview` (company_id, last_qbr_shown, last_roadmap_shown, renewal date and ARR, both $/TB, owner, RSM, MC used vs licensed), `rings` (each Ring's CloudFS version, nodes, object store, hypervisor, HA and latest MC), **the read** (lines, health, date), the owner's **sentiment**, the **storylines** (with their Helps, Hurts, Signals to watch and Open questions), the **facts**, `products`, `environment_history`, `deals` and `deal_lines`, `open_items` (each with its Situation (0.52.0, read first) and Next step; Goal only on an old item with no Situation), `to_present`, `recently_completed`, the knowledge lists, `meetings`, `contacts`, `people` (each contact's and teammate's persona `role`, `sentiment`, `background`, `personality`, `interests`, `sore_spots`, newest `observations`, `last_met_on`); on an account managed by an MSP (one of the owner's accounts, a Panzura partner), `managed_by` and `managed_by_people` (the MSP's contacts with the same profiles, read from the MSP, each marked `from`); on an MSP, `manages` (one line per account it manages: health, sentiment, next renewal and ARR, open and waiting items, last touch, `needs_attention` and `why`). For anyone who will be in the room and has more on their card than `people` shows, `codynote_read` with their `person_id`.
2. `codynote_changes` with the company and `since` = window start.
3. What happened to the storylines in the window, and which were resolved, with `codynote_query` (put the window start in place of `<window_start>`, as YYYY-MM-DD):
   ```sql
   select s.bullet_id, s.kind, s.status, s.outcome, s.resolved_at, s.text, s.now_text, h.on_date, h.text as happened
   from v_storylines s left join v_bullet_history h on h.bullet_id = s.bullet_id and h.on_date >= '<window_start>'
   where s.company_id = '<company_id>'
     and (s.status <> 'resolved' or s.resolved_at >= '<window_start>' or h.on_date is not null)
   order by s.status, s.kind, s.bullet_id, h.on_date
   ```
4. Wins from items: `codynote_list_action_items` with the company and `status: "done"`; keep those completed since the window start (`is_present` first).
5. Products they don't own yet (retired ones left out):
   ```sql
   select p.name from products p
   where p.deleted_at is null and p.retired_at is null
     and p.id not in (select cp.product_id from company_products cp
                      where cp.company_id = '<company_id>' and cp.deleted_at is null)
   order by p.name
   ```
6. Meetings in the window: notes and next steps; for transcripts, `codynote_read` (the meeting's id from `codynote_map` or `meetings`) with `find` for concerns, projects, praise and asks.

**HubSpot** (read only; skip quietly if not connected), since the window start: support tickets (count, themes, which are open and for how long); logged emails, calls and notes (asks, promises, complaints or praise worth quoting, threads waiting on a reply); open deals (a close date, amount or stage that differs from CodyNote's is a mismatch to flag, not to resolve). Without "with email", email comes from HubSpot only.

**Outlook** (only when the owner said "with email"): the recipe in `email.md` over the window (`afterDateTime` = window start instead of the watermark; the briefing's `email` block for the addresses, domains and the owner's domains): the quarter's threads, asks and promises, praise and complaints worth quoting, threads waiting on either side (the waiting rule), new people. Read only: here they're agenda inputs, not suggestions, and the watermark stays; end with "Say 'catch up on email for <Company>' to save them to CodyNote."

**Fathom** (skip quietly if not connected): calls in the window that CodyNote lacks; read their summaries.

**The web** (about the last 6 months): the company's name (with city or industry if common). Growth or change (offices, acquisitions, big wins, hiring), leadership (a new CIO, the champion leaving), trouble (layoffs, closures, a security incident), IT signals (job posts naming cloud moves, Egnyte, Nasuni, SharePoint, AI projects), the latest earnings headline if public. Keep only what matters; note each link and date. Compare with the facts card: a newer headcount, revenue or acquisition goes under Before the meeting ("refresh the facts").

**Panzura:** check panzura.com's newsroom and blog for releases and news since the window start (public only), and whether any of it names the newest CloudFS version.

## 2. Think, in the work order

Walk the account in the order an update works it (`work-order.md`), the read last. This mode only reads: no scan, no suggestions, no coverage.

- **People:** who will be in the room (the invitees the owner named; else the contacts still there met in the window, plus the economic buyer and decision maker; never a contact with `active: false`, listed in `inactive_people`: they retired, left or changed role): what each cares about, how they feel about Panzura and why (their sentiment and the Observations behind it), their sore spots to steer around and an opener from their interests. Who matters now: new faces in the window, a champion leaving (news), a decision maker missing from the calls, someone whose sentiment turned. On a managed account the MSP's people often join: match the invitees and the window's attendees and speakers against `managed_by_people` as well as `people` (by name or address), and list the MSP's people who will be there, marked with the MSP (their sore spots and openers as for the account's own; never one with `active: false`).
- **Environment:** per Ring, the CloudFS version, nodes, object store, hypervisor and HA. The upgrade gap only against a newest version the material names (Panzura's site, a document, a call); otherwise "check the latest release". Empty fields are a question for the call, not a guess.
- **Capacity:** used vs licensed per Ring and in total; from `environment_history`, the TB-per-month trend and, if growing, roughly when they reach licensed vs the renewal date.
- **Deals:** the renewal and open opps; HubSpot mismatches.
- **An MSP's QBR** (the briefing has `manages`): the review covers the partnership and the accounts it manages together: their health and renewals side by side, which need attention and why, and what the MSP can do across them. Deals and items stay per account; never add the managed accounts' ARR to the MSP's without saying "with managed accounts".
- **Storylines:** what each did in the window (History); its Open questions not yet ticked (questions for the call); its Helps lines against `panzura.md` (what the product needs first, the closest customer story) and each open project and opportunity storyline without one; Hurts and Signals to watch lines worth defusing.
- **Action items:** wins (done in the window), past due and waiting; each open item's Next step for Before the meeting.
- **Facts:** the facts card against the news.
- **The read, last:** start from the stored read and its date: does the quarter bear it out? Say if you'd change the health ("my read"), and note the owner's sentiment if it differs. Then **the quarter's story** in a sentence or two, joining sources (news that explains a capacity jump, a ticket theme that matches an email complaint, a project storyline that fits a product they don't own), **what they should walk away believing**, and **the one ask** (a renewal heads-up, an MC increase, an expansion, an introduction, a reference).

Ground every judgment in a fact and name its source. Mark inferences "my read". Say what you couldn't see. Never invent news.

## 3. Write

In chat, in this order. Short lines; sources in a few words. Money "$84k", dates "Oct 9". Mark anything only for the owner's eyes **(internal)**: ARR, $/TB, discounts, health, sentiment, competitor worries.

```
**<Company> QBR prep** · window: since <date> (<last QBR | last roadmap | 6 months>) · renews <date> (internal: <ARR>)

**The read** (internal) · stored <date>: <Healthy | Watch | At risk>; your sentiment: <colour, date>
<3-5 sentences: the quarter's story, whether the stored read still holds, what they should walk away believing, the one ask. "My read" where inferred.>

**Environment** (from the Rings)
- <Ring>: CloudFS <version> (<newest named in <source>: <gap> | check the latest release>) · <n> nodes · <object store> · <hypervisor> · HA <notes>
- Capacity: <used> of <licensed> TB (<%>) · <+n TB/month since <date>> · <reaches licensed around <month>, before | after the <renewal date> renewal>
- Last QBR: <date> (<typed | the <date> QBR meeting | the <date> joint QBR + Roadmap | none on record>)

**Who will be in the room** (internal)
- <name> (<persona>; <MSP> for one of the MSP's people) · <positive | neutral | negative, why> · cares about: <from their Observations> · avoid: <their sore spots> · open with: <an interest>
- With you: <teammates joining: name (what they'll cover)>
- No longer there: <names from `inactive_people`, one line; leave out when none>

**Managed accounts** (an MSP only, internal): one line each: <account> · <health> · renews <date> (<ARR>) · <open> / <waiting> items · last touch <date> · <why it needs attention, or fine>; then one line across them.

**About them** (for the "about you" slide): <industry · HQ · employees · offices · revenue · recent acquisitions, from the facts card with dates; note anything stale or missing>

**Outside news**
- <date> · <headline> (<link>) — <why it matters for this QBR>
(3-5 items, or "Nothing that stands out in the last 6 months.")

**Draft agenda**
1. **Wins since last QBR** — <storylines resolved as worked, finished items, resolved tickets, praise from emails (HubSpot, or Outlook with email)>
2. **Your environment and capacity** — <from the Environment block: capacity used vs licensed, trend, when they reach it; version, upgrade gap or "check", and what's new since their release; nodes, object store, HA worth confirming>
3. **Open issues** — <open and watched issue storylines, open tickets and items: where each stands (Now), promises made>
4. **Your projects and where Panzura fits** — <project storyline → Panzura answer → why it suits them → customer story link>
5. **Opportunities** — <opportunity storylines and products they don't own that fit a project> (internal framing; pick what to say)
6. **What's new from Panzura** — <public releases and news since the window start> · [Your roadmap slides]
7. **Next steps and renewal** — <what to agree; renewal timing>

**Questions to ask them**
1. <sharp question from a gap: a quiet storyline, a storyline's unticked Open question, a product bought but not set up, a news item, a competitor mention>
...(3-6)

**Before the meeting**
- [ ] <an open item's Next step that should happen first, fill a blank in the Environment (ask for it, or check the version), close a past-due item, invite a missing decision maker, refresh an old capacity reading, chase an email thread, refresh the read or the facts>
```

In Who will be in the room, leave out what a profile doesn't have, and say "no profile yet" for someone with none ("profile the people at <Company>" builds them). Leave out an agenda item with nothing to say; dropped or irrelevant storylines aren't wins. Don't paste raw lists; pick what serves the meeting. End with one line: "Want this as a document or slides?" If the stored read is older than the newest meeting, add: "Say 'refresh <Company>' or 'wrap up my <Company> call' to refresh the read first."
