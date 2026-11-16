# Panzura guide (public information)

Ready-made context on Panzura's products, what customer problems they answer, proof, and competitors. Built from panzura.com and public sources (checked 2026-10-26). Panzura's own claims are marked "Panzura says". The owner's notes, contracts and CodyNote's connector override this guide where they differ (for example the owner's product list and pricing units). This is public context only: no customer data, no internal roadmap.

## The footprint: one ring, many add-ons

- **CloudFS** is one global file system per **ring**: one authoritative copy in the customer's object storage (deduplicated, compressed, AES-256 encrypted, customer-held keys), served at each site by **nodes** (VMs) over SMB/NFS (native S3 since 8.6). Older customers say "filer" (the old Freedom line).
- **Global file locking** is the signature feature: one editor at a time, or part of a file in apps like Revit; everyone sees the latest save from any site. Panzura says metadata syncs every 60 seconds (its "60-second RPO").
- Editions, all priced per TB of **managed capacity (MC)**: **Collab** (many offices editing interlinked files, e.g. Revit/CAD), **NAS** (multi-site sharing in one namespace), **Archive** (rarely touched data, retention). Panzura says the modes can be applied per project.
- MC is bought as 25 TB minimum, then 5 TB steps (since 8.5.1); the console's managed capacity graph tracks **peak monthly use** against purchased capacity, so spikes matter at renewal.
- Add-ons follow the footprint: Data Services modules follow TB, Cloud Mirroring follows rings, Nexus follows the data put in front of Copilot, Edge follows remote users. Symphony stands alone.

## Customer problem → Panzura answer

| What the customer says or shows | Panzura answer | Needs first |
|---|---|---|
| Slow file opens across offices; "file is locked / someone else has it"; version conflicts | CloudFS Collab (global locking, local cache, Prewarm since 8.7, pinning) | CloudFS |
| Separate NAS/NetApp filers per site, hardware refresh coming, backups to tape | CloudFS NAS or Collab replacing site filers | — |
| Old project data filling expensive storage | CloudFS Archive, cloud storage tiering | — |
| Running near or past licensed TB; growth from new offices or acquisitions | MC increase, co-termed with the renewal; Managed Capacity Optimization (8.7) can trim counted MC | CloudFS |
| "Where did that file go?", "who deleted / moved it?", audits, compliance | **Search & Audit** (Panzura Data Services Standard: global search incl. deleted/moved files, audit trail, one-click restore) | CloudFS |
| Ransomware worry, cyber-insurance questions, a peer firm got hit, mass deletes | **Threat Control** (was Detect and Rescue): per-user behavior baselines, detects exfiltration and mass deletion, can disable a compromised account; SIEM sync (8.7) | **Search & Audit / PDS Standard** |
| Remote or mobile users, VPN pain, sharing files with outside partners | **Edge**: VPN-free access from browser, Windows, Mac, phones, keeps NTFS/AD permissions, external sharing with passwords/2FA/expiry; AutoCAD viewer, Teams | CloudFS; a Windows server to host it |
| One cloud or region as a single point of failure; fear of a deleted bucket | **Cloud Mirroring**: writes every block to a second object store, automatic failover (roughly doubles their cloud storage bill, paid to the cloud provider) | CloudFS **8.1+** |
| Wants Copilot to use their project files safely | **Nexus** (GA Apr 2026): connects CloudFS to Microsoft 365 Copilot, permission-aware, real-time | CloudFS; the customer's own **Copilot licenses** |
| Data scattered across SharePoint, Box, old file servers; permission sprawl; needs to know what they have before moving | **Symphony** (from the Moonwalk acquisition): scans and catalogs 500+ file types, reports, fixes inherited permissions, moves/tiers data | Nothing (can come before CloudFS) |
| Many sites to rebuild after an outage; DR worries | Instant Node (8.5, node rebuilt from the cloud in minutes), Continuous Data Availability (8.6), Global/Local HA | CloudFS version |
| Data must stay in a country or region | Geofencing (8.6) | CloudFS 8.6+ |

## Recent releases (talking points; check panzura.com/blog for newer)

- **8.5 (Feb 2025):** Instant Node, Regional Store, Okta SSO. **8.5.1:** 5 TB increments, managed capacity graph.
- **8.6 (Nov 2025):** geofencing, native S3, AD-integrated quotas, Continuous Data Availability.
- **8.7 (Mar 2026):** adaptive snapshot retention, admin file-lock release, Prewarm, Threat Control with User Behavior Intelligence, SIEM sync, Managed Capacity Optimization.
- **8.7.1 (Aug 2026):** ring-wide health API, 15-minute scheduled snapshots, stuck locks auto-release after 7 days, 10 security fixes. Before upgrading: usernames must include the domain; Linux CIFS clients may need `noserverino`.
- **Panzura Express (Aug 2026):** fixed-price single-site package sold through partners; an Express customer outgrowing one site moves to Collab or NAS.
- **Leadership:** Karthik Ramamurthy CEO since Apr 2026; Guy Churchward Executive Chairman since Jun 2025.

## Proof: customer stories by situation

All are Panzura-authored; cite the link.
- **AEC / engineering, many offices:** Milwaukee Tool, CAD opens from 40+ minutes to seconds ([story](https://panzura.com/customers/milwaukee-tool-case-study)); Mead & Hunt, 40+ offices, filers retired at 30+ sites, about $250K IT savings over two years, two-minute threat detection with Threat Control ([story](https://panzura.com/customers/panzura-workspot-help-drive-mead-hunts-rapid-growth-simplify-infrastructure)); AFRY, about 100 offices and 18,000 engineers on Azure ([story](https://panzura.com/customers/afry-case-study)); M&S Engineering, file silos removed ([story](https://panzura.com/customers/ms-engineering-case-study)).
- **Growing by acquisition:** McMillan Pazdan Smith brings acquired firms in without long migrations ([story](https://panzura.com/customers/mcmillan-pazdan-smith-unifies-file-access-across-rapidly-expanding-architecture-practice)).
- **Ransomware recovery:** Illinois hospital restored clean data in under 15 minutes ([story](https://panzura.com/customers/illinois-hospital-thwarts-ransomware-attack)); a multi-office firm contained an attack in under 20 minutes ([story](https://panzura.com/customers/recovering-from-ransomware)).
- **Large / regulated:** a "$2 trillion" Wall Street bank, 10+ PB on private storage ([story](https://panzura.com/customers/2-trillion-wall-street-bank)); Skyscanner ([story](https://panzura.com/customers/skyscanner-books-panzura-for-its-journey-to-secure-reliable-unstructured-data-management)).
- **Moving to the cloud:** New Orleans Saints ([story](https://panzura.com/customers/new-orleans-saints-cloud-move)); Pret a Manger ([story](https://panzura.com/customers/pret-a-manger)); a global game maker ([story](https://panzura.com/customers/gamemaker-accelerates-project-delivery)).
- No public stories yet for Edge, Archive, Symphony or Nexus: say so rather than stretch one.

More: [panzura.com/customers](https://panzura.com/customers).

## Competitors (Panzura's positioning)

| Competitor | Panzura says | Page |
|---|---|---|
| **Nasuni** (closest like-for-like) | Immediate sync everywhere vs delays that grow with the number of sites | [compare](https://panzura.com/compare/nasuni) |
| **Egnyte** | Hybrid platform where the customer controls its data, cloud, keys and AI, vs Egnyte's SaaS | [compare](https://panzura.com/compare/egnyte) |
| **CTERA** | Performance, efficiency and total cost | [compare](https://panzura.com/compare/ctera) |
| **Azure Files** | 60-second global recovery points, immediate global sync, no storage lock-in, lower total cost | [compare](https://panzura.com/compare/azure-files) |
| **AWS FSx** | Instant global sync, 60-second restore points, hot-file speed on cheap storage tiers | [compare](https://panzura.com/compare/aws-fsx/) |
| **NetApp** (incl. Global File Cache) | Performance, resilience, one platform instead of several; a GFC migration path | [compare](https://panzura.com/compare/netapp), [GFC](https://panzura.com/migrate-netapp-gfc) |
| **PeerGFS** | Scales to 500+ sites vs a wall around 50 | [compare](https://panzura.com/compare/cloudfs-vs-peergfs) |
| **SharePoint / OneDrive** | Not a Panzura comparison page; position Nexus (Copilot on CloudFS files) and Symphony (moving and cleaning up data) instead | — |
| **Komprise, Hammerspace** | Symphony's competitors | — |

A competitor mention is a risk to flag, and also a chance: ask what prompted it.

## Old names → current names

Freedom → CloudFS; filer → node; Detect and Rescue / D&R / Detect → Threat Control; PDS Standard ≈ Search & Audit; Basic → Essentials (free Data Services tier); Cloud Mirroring = Mirroring ("dual cloud").
