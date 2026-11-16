# HubSpot SKUs → CodyNote products

How CodyNote (`codynote_hubspot_deals`) matches a HubSpot line item to one of the owner's products. Panzura's SKU names vary (editions, terms, bundles, old names) and the same product can have several SKUs. Match by what the line is (its name and description, read as a person would, not by exact words); when unsure, ask.

## Rules

| CodyNote product | Unit | HubSpot line items that mean it |
|---|---|---|
| Collab | TB | CloudFS Collaboration, Collab, "True Collaboration", Freedom Collaboration |
| NAS | TB | CloudFS NAS, Multi-site NAS, Freedom NAS |
| Archive | TB | CloudFS Archive, Active Archive, Freedom Archive |
| Search & Audit | TB | Search & Audit, Panzura Data Services (PDS) Standard, Search, Audit, Data Services |
| Threat Control | TB | Threat Control, Detect and Rescue, D&R, Detect |
| Edge | seat | Panzura Edge, Edge users / seats |
| Mirroring | ring | Cloud Mirroring, Mirroring, CMP license |
| Symphony | TB | Symphony, Symphony Data Insights / Enterprise |
| Nexus | TB | Nexus |

- **TAM, services and support** (0.48.0): TAM, professional services, migration or onboarding services, training, support or maintenance on its own are kept as license lines on a product CodyNote leaves out of ARR (Settings › Products), so the deal matches HubSpot.
- **Not a product** (left out; listed in the summary): hardware, Audit Retention add-ons, PDS Essentials (free).
- **Bundles** (e.g. "CloudFS Platform" or an Express package): split only if the line items list the parts; otherwise ask.
- A SKU whose name fits two products, or none: the rules' best guess (or none), flagged for the owner to confirm once in the app.

**Confirmed SKUs** live in CodyNote (Settings › Products): `codynote_hubspot_deals` notes each new SKU with these rules' guess, and the owner confirms it once, or maps it to another product. Nobody stops to ask.
