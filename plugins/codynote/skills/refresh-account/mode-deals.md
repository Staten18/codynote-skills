# Deals mode (Refresh deals, Import from HubSpot)

Refresh account's deals mode: "refresh Acme's deals", "update Acme's deals from HubSpot", "are Acme's deals current?", and bringing a company in: "import Acme from HubSpot", "bring Acme in from HubSpot" (one account). CodyNote's deals and license lines brought in line with HubSpot by the one deal rule, in code: Claude reads HubSpot, CodyNote does the arithmetic. A company not in CodyNote yet is added first, with its Managed by and product statuses. Everything is a suggestion the owner approves in CodyNote › Claude suggestions; never stop to ask first. HubSpot is read-only.

**For field X, follow its recipe** (`field-guide.md` next to this file, the Deals tab; `codynote_recipe(tab: "deals")` has the whole recipes). Never stop for an approval: the owner approves each suggestion in CodyNote.

`codynote_hubspot_deals` matches line items to products (the owner's confirmed SKUs, then the rules in `skus.md`, next to this file); `panzura.md` says what the products are.

**Not brought in** (the owner handles these): contacts, HubSpot tasks, meetings and Fathom calls, Overview bullets, Rings and MC readings. Never mark a deal Closed Won or Closed Lost. If the owner asks for a contact anyway (a partner's account executive on the deal, say): look them up first (`codynote_read` with their `name`: every account, not just this one). Already in CodyNote on another account: suggest a `link` (`{"action": "link", "record": "contact", "id": <their person_id>, "company": <this company>}`), never a second contact; add one only when nobody matches (`account-picture.md`, "People across accounts"). No action items unless the owner asks for one (say, to chase a mismatch); then read the account's open items first (`codynote_list_action_items` with the company and `mine_only: false`): one already open for it gets a Context `line` instead (`account-picture.md`, "No duplicate items"); a new one carries its boxes (`cards.md`).

## 1. Read CodyNote, then HubSpot (read-only)

**CodyNote:** `codynote_get_customer` with the name. Not found: it's a new company (step 3.1 adds it). Found: keep its `overview.company_id`, `deals` and `deal_lines`, and its product statuses.

**HubSpot:** find the company (several match: ask which and stop). Then every deal associated with it, **every** one (open, won and lost: CodyNote decides which belong), with all its properties (deal name, pipeline, deal stage and its label, deal type, close date, amount, the **deal owner** `hubspot_owner_id`, forecast category, and any contract start / end / term properties); every line item on those deals with all its properties (name, `hs_sku`, quantity, price, discount, amount, billing frequency, `hs_term_in_months`, start date); and the owners (`search_owners`: id, first and last name, email). Pass them as HubSpot returns them: no sums, no conversions. Do not trust a deal's amount: the line items are the licensing. Also any parent or partner company HubSpot associates with it (for Managed by, step 3.1).

## 2. Let CodyNote apply the rule

`codynote_hubspot_deals(company, raw: {deals, line_items, owners})` (a new company: CodyNote answers that the account isn't there; suggest the company, step 3.1, and its deals come once the owner approved it). Each line item must reach its deal: nest it under the deal as `line_items`, or give it `deal_id`. CodyNote answers with:

- `deals`: the ones that belong (every open deal and every Closed Won deal still in term, whoever owns it: `in_my_name` false when the HubSpot deal owner isn't the owner's owner id in Settings › Connections), each with its lines (product, unit, quantity, ARR, whether it counts in ARR), `renews`, `opp_type`, stage and forecast by the owner's map, and `match` (the CodyNote deal it is) with what `differs`;
- `skipped` (Closed Lost, Closed Won past its term), `replaced` (current renewals a later Closed Won one renewed);
- `flags`: say each in the reply; `amount_monthly` and `owner_mismatch` also go in `hubspot_fix`; `unseen_sku` means the owner confirms that SKU once under Settings › Products (CodyNote noted it); `no_owner_id` means the owner's HubSpot owner id isn't in Settings › Connections yet;
- `changes`: ready to send.

Never work out which deals belong, an ARR, a product or `renews` yourself: if something looks wrong, say it in the reply. Never stop to show a SKU table.

## 3. Suggest

One `codynote_suggest_changes` call (no `scan_id`: this isn't an update), in this order. Give each change a `confidence` (`high`, `medium` or `low`: how sure you are), and copy every quote exactly as it is in the source: CodyNote looks for a meeting's quote in its transcript and marks one it can't find "quote not found".

1. **The company, when it's new** (or fields HubSpot has that differ on an existing one; never remove anything): `action: "add"`, `fields: {name, account_group: "current"}` plus what HubSpot knows that CodyNote has (the tool lists allowed fields if one is wrong). A new company's deals come once it is approved: say so in the reply ("then say "refresh Acme's deals"").
   - **Managed by**: when HubSpot associates the company with a parent or partner company that manages it (an MSP, a Panzura partner: the association label, or a partner / parent company property) and that company is already in CodyNote (`codynote_query`: `select name from companies where deleted_at is null and name like '%<name>%'`), add `managed_by: "<its CodyNote name>"` to the company's fields (on an existing company, only when its `managed_by` differs). If the partner isn't in CodyNote, or HubSpot doesn't make clear it manages the account (a reseller only, a sister company), say so in the reply and don't suggest it. The managed account keeps its own deals.
2. **The deals and their lines:** `changes` exactly as CodyNote gave them.
3. **Product statuses:** each product on a deal in term gets `product_status` "purchased", unless CodyNote already has it purchased or implemented.

**HubSpot-side cleanup** (a deal's owner looks wrong, a duplicate company record, an amount that is monthly, a stage that is off, an outdated company record) goes in the same call's `hubspot_fix: [{kind: owner | duplicate_company | monthly_amount | stage | outdated_record, deal?, company?, note}]`, not in prose; CodyNote returns it checked and stores nothing. A dropped change: keep the rest and say what and why. A `replaced_by` waiting on a new deal comes on the next refresh, once the owner approved it.

**Last, before the reply (0.53.0): the cleanup** (an account with items only). Bring this account's open items that do not meet the rules up to them: a "Topic: what exactly, how much, for whom" title, a three-part Situation (The ask / Where it stands / Why it matters), a next step who / what / when, folds, follow-ups. Follow `mode-items.md` (its steps 2 and 3, rounds of at most 12 cards, one `codynote_suggest_changes` call per round without `scan_id` once the scan is closed). This is the account-by-account cleanup; there is no separate cleanup skill. Items that already pass get nothing; say what changed in the reply.

## 4. Reply

```
**<Company> deals refreshed from HubSpot** (or **<Company> imported from HubSpot**; as suggestions, approve in CodyNote)
In term: <deals, ARR from lines> · Open: <deals, stage, renews>
Changed: <deal: what differs> · Skipped: <deal: why>
In someone else's name: <deal> (HubSpot owner <name>): not counted on your quarter or GRR
Managed by: <MSP> (from HubSpot's <association>), or "HubSpot shows <partner>, not in CodyNote: add it?"
Heads-up: <each flag in a line: monthly amount, new SKU to confirm, services not in ARR, stages that didn't map>
Fix in HubSpot: <the hubspot_fix list CodyNote returned, one line each; or "nothing">
For you (a new company): contacts, Rings and MC readings, implemented products.
```
