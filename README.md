# JACCMS Nigeria

**Joint Agencies Case & Crime Management System**

A national security platform that unifies case and crime record-keeping across **land borders**, **seaports and waterways**, **airports**, and **Nigerian airspace**.

JACCMS is the joint records, evidence, and case-file system for the agencies that police movement of people and goods into, out of, and through Nigeria. It replaces fragmented paper files and siloed digital records with a single, real-time, auditable national record — then, after the first corridor is stable, adds the targeting layer that advanced-country border agencies already run as standard.

The acronym originally stood for Joint *Aviation & Customs* Case Management System. The product is no longer airport-only. **Airports are one port-of-entry type.** Land, water, and airspace are first-class domains in the schema, the app, and the rollout.

> **Status:** Greenfield — documentation and roadmap in place. Implementation starts at Phase 1.  
> **Development / staging host:** [Railway](https://railway.app)  
> **Production host:** Nigeria-resident cloud or government-certified DC for classified investigation data (see [Hosting and data residency](#hosting-and-data-residency)); Railway only if a sovereign-data waiver is granted.  
> **Design language:** Nigerian green & white (`#008751` / `#FFFFFF`)

---

## Operating domains

Every incident is logged against a **mode** and a **location type**. The same person, conveyance, and seizure can appear in more than one domain.

| Domain | What it covers | Typical locations | Typical conveyances |
|--------|----------------|-------------------|---------------------|
| **Land** | Official land borders, green/unofficial crossings, inland checkpoints, truck parks, highway interdiction | Seme-Krake, Idiroko, Jibia, Ilela, Mfun, inland Customs/NIS posts | Vehicles, trailers, containers, pack animals, pedestrians |
| **Water** | Seaports, free-trade terminals, inland waterways, coastal and offshore interdiction | Apapa, Tincan, Onne, Calabar, Warri; NIWA channels; coastal patrol | Vessels (IMO/name/flag), barges, fishing boats, containers |
| **Air (airports)** | Passenger and cargo ports of entry, airside/landside, airport access control | MMIA Lagos, NAIA Abuja, Kano, Port Harcourt, other international and designated airports | Aircraft, ULDs, passenger baggage, air cargo |
| **Airspace** | Sovereignty and general aviation: unauthorized aircraft, illegal landing strips, drones/UAS, restricted-area overflight | FIR/sectors, unlicensed strips, oil-field helipads, no-fly zones | Aircraft, helicopters, UAS — often *no* airport of entry |

Advanced-country analogues are multi-modal by default (US CBP air/land/sea + ATS; UK Border Force air/maritime/rail; Australia ABF). An airport-only case file would leave the same syndicate free to move to Seme or Apapa.

---

## What we are building

JACCMS is two products sharing one backend:

| Surface | Audience | Purpose |
|---------|----------|---------|
| **Executive web portal** | Commandants, targeting analysts, legal/prosecutors, super admins | National dashboard by mode, global search, watchlist hits, transfer approvals, audit trail, 24/7 ops view |
| **Offline-first mobile app** | Field officers on land, water, airport, and airspace patrol | Incident logging, evidence capture, document/watchlist check, biometric sign-off, operation with no network |

The system must remain usable when officers have **no network** — airside blackspots, green-border patrols, creeks, and ship-side — then reconcile to the national record when connectivity returns.

JACCMS is **not** a trade Single Window (that is NCS **B'Odogwu** / the National Single Window), **not** an e-gate (NIS / FAAN), **not** a port community system (NPA / terminals), and **not** an air-traffic or naval C2 system (NAMA / Navy). It is the **joint crime and case file** those systems should feed when a hit, seizure, or offence occurs.

### Key capabilities (v1 — Lagos corridor)

Software is **national and multi-modal from Phase 1**. First operational corridor is Lagos: one airport, one seaport cluster, one land border, plus airspace case types anywhere they are reported.

- **One national case file** across land, water, airports, and airspace
- **Location + conveyance model** — port of entry, GPS, vehicle / vessel / aircraft / container — not an “airport” field with a comment
- **Real-time case tracking** from incident to prosecution bundle
- **Offline-first mobile app** for field officers (blackspots are the default)
- **Executive web portal** with a 24/7 operations view **filterable by mode**
- **Person master / entity resolution** — one canonical person across agencies, modes, and incidents
- **Watchlist hits at scan** — local wanted list plus INTERPOL SLTD, Nominals, TDAWN, and Stolen Motor Vehicles (not outbound alerts only)
- **Immutable audit trail** using hash-chaining, including **search audit** (who queried whom)
- **Digital evidence lite** — SHA-256 of every exhibit at capture, legal hold, court-readable chain of custody
- **Seizure register** — classification, valuation, store location, disposal (the SEACATS / WCO nCEN pattern)
- **Inter-agency transfer management** with biometric sign-off
- **RBAC** with granular permissions across agencies **and** location assignments
- **Universal Incident Number (UIN)** — cryptographically unique case ID
- **External identity checks** — NIMC (NIN), NIS (passport), INTERPOL databases
- **Scanner / alert ingest** — bag-screen, cargo scan, or PLACI/manifest hit can open a case
- **NDPA 2023** data-protection baseline (DPIA, retention, biometric lawful basis)

### Key capabilities (Phase 6 — after the Lagos corridor is stable)

Capabilities that advanced-country **border agencies** treat as table-stakes, scheduled **after** the corridor so v1 does not become a second B'Odogwu or a full CBP ATS:

- **National Targeting Centre** covering air, land, and sea (Nigeria analogue of CBP National Targeting Center / UK National Border Targeting Centre)
- **API / PNR / iAPI** for commercial air (ICAO Annex 9 Standard 9.7)
- **Maritime advance data** — cargo manifests, IMO FAL, AIS correlation (not a second Navy C2)
- **Land advance data** — truck/cargo declarations referred from B'Odogwu; optional ANPR later
- **PLACI / ACAS** air-cargo pre-load targeting (NCAA signed a PLACI MoU in August 2026)
- **Airspace referrals** from NCAA / NAMA / NAF (flight-plan vs observed; illegal-strip GEOINT from NCS)
- **Rules-based risk engine** with analyst override
- INTERPOL **FIND** at the primary line and on the field device
- 1:N biometric search (AFIS / facial gallery) against the person master
- Live **WCO nCEN** exchange of non-nominal seizure data
- Read-only fusion with **B'Odogwu** cargo declarations (do not duplicate the Single Window)
- Full digital evidence management and DPP / court e-bundle

### Participating agencies

| Agency | Domain | Role in JACCMS |
|--------|--------|----------------|
| **NCS** — Nigeria Customs Service | Land, water, air | Seizures, contraband, customs offences; nCEN-compatible seizure records; GEOINT on strips and crossings. Trade clearance stays on B'Odogwu. |
| **NIS** — Nigeria Immigration Service | Land, water, air | Travel-document identity, SLTD/passport hits, immigration offences. First-class agency, not only an external passport API. |
| **NDLEA** | Land, water, air | Narcotics cases, controlled substances, passenger/cargo profiling intake |
| **NPF** — Nigeria Police Force | All | Criminal investigation, prosecution handoff, NCB/INTERPOL coordination, marine police |
| **FAAN AVSEC** | Airports | Airside incidents, access control, aviation security, scanner events |
| **NCAA** | Airports + airspace | Regulatory oversight, civil aviation security, PLACI, general-aviation compliance |
| **NPA** — Nigerian Ports Authority | Water | Seaport security referrals, terminal access, port incidents (operator role, analogue of FAAN) |
| **NIMASA** | Water | Maritime safety/security, vessel identity, coastal incidents |
| **Nigerian Navy** | Water | Interdiction and maritime-security case create/read (not naval C2) |
| **NIWA** | Water (inland) | Inland-waterway incidents and referrals |
| **NAF** — Nigerian Air Force | Airspace | Unauthorized aircraft, intercept/referral, strip watch (restricted create) |
| **DSS** | All | Read / alert on national-security hits (no field-create in v1) |
| **NAQS / Port Health** | Land, water, air | Case *type* and referral only in v1 (quarantine / public-health seizures) |

Field-create in v1: NCS, NIS, NDLEA, NPF, FAAN AVSEC, NPA port security, NIMASA. Navy, NAF, NCAA, DSS as specified above.

### Pilot and rollout

Software is multi-modal from day one. Operations roll out by **corridor**, not by rewriting the product.

1. **v1 corridor — Lagos (all four domains)**
   - **Air:** Murtala Muhammed International Airport (MMIA)
   - **Water:** Apapa and/or Tincan Island Port
   - **Land:** Seme-Krake border post
   - **Airspace:** case types live nationally (illegal strip, unauthorized aircraft, UAS) logged by NCS / NCAA / NAF — no extra “site”
2. **Air expansion:** NAIA Abuja, Mallam Aminu Kano, Port Harcourt, remaining international airports
3. **Water expansion:** Onne, Calabar, Warri, other NPA ports; NIWA channels
4. **Land expansion:** Idiroko, Jibia, Ilela, Mfun, and designated inland checkpoints; green-border patrol units
5. **Phase 6:** National Targeting Centre (all modes), then remaining posts

---

## Why this exists (Nigeria vs advanced-country borders)

Advanced-country border agencies do not treat “logging an incident at the airport” as the whole job, and they do not run a different case file per mode. The United States, United Kingdom, Australia, Canada, and the EU Schengen system run **three layers, for air, land, and sea**, that Nigeria currently splits across paper, email, B'Odogwu, and agency silos.

| Layer | What advanced countries run | Nigeria today | JACCMS response |
|-------|-----------------------------|---------------|-----------------|
| **1. Pre-arrival targeting** | CBP ATS (passengers *and* cargo, all modes) + ACAS air cargo + ACE manifests; UK Border Force API/PNR + maritime targeting + Target Operating Model; ICAO Annex 9 **requires** an API system | NCAA PLACI MoU (Aug 2026) is new and **air-cargo only**. NDLEA has asked for API/PNR. No national targeting centre. Land and sea still largely post-arrival. | **Phase 6**, all modes. v1 reserves schema and API stubs. |
| **2. Primary-line / patrol hit** | Every document and many plates/IMO numbers query INTERPOL **SLTD**, Nominals, TDAWN, **SMV**; US TECS; Schengen SIS II. Hit in seconds. | Nigeria can *upload* SLTD (Project TERMINUS) and NCB exists, but field devices at airports, Seme, and Apapa do not run FIND. Officers still profile with spreadsheets. | **v1:** watchlist service on every MRZ/NIN/plate scan. **Phase 6:** FIND on the device. |
| **3. Case, evidence, seizure, court** | US **SEACATS**; UK CONNECT / HOLMES + Digital Case File + DEMS; WCO **nCEN**. Searching a person is audited. One case file whether the seizure was in the air, at the dock, or on the road. | Paper files, email between agencies, FAAN 6-monitor scanners with no shared case, B'Odogwu for *trade* not crime, NSW stalled. A seizure at MMIA does not automatically show at Seme or Apapa. Insider collusion is documented at airports **and** ports. | **v1 core.** Person master, multi-modal locations, seizure register, hashed exhibits, search audit, inter-agency transfer, prosecution bundle. |

**What Nigeria already has (do not rebuild):**

- **B'Odogwu** — NCS unified *customs processing* (declarations, duty) for all modes. JACCMS reads hits / referrals; it does not clear cargo.
- **National Single Window** — stalled; out of scope.
- **FAAN Orion scanners**, NCS cargo scanners, port/land scanners — hardware exists; JACCMS must *ingest the event*.
- **NCS GEOINT / Geoportal** — already used for strips, fords, and border tracks (WCO, 2025). JACCMS takes the *case*, not the GIS stack.
- **NIMC, NIS passport systems, INTERPOL NCB Abuja, SLTD uploader** — identity sources.
- **NAMA / NCAA ATM, Navy C2, NPA TOS** — operational control systems. JACCMS consumes referrals only.
- **NCASP 2024** — aviation-security programme; JACCMS is the records tool, not the screening equipment.

**What JACCMS must do better than a typical Western airport RMS:** offline-first on **green borders, creeks, airside, and ship-side**. Treat blackspots as the default in every domain.

Sources for the comparison (public): CBP ACE / ATS / ACAS and SEACATS; ICAO Annex 9 Chapter 9; WCO nCEN / CEN and NCS GEOINT write-up; INTERPOL I-24/7, SLTD, SMV, FIND; UK Border Target Operating Model; NCAA PLACI MoU (27 Aug 2026); FAAN scanner rollout; NDLEA remarks on API/PNR and a Kenya-style coordination centre; NITDA National Digital Cloud Policy (classified / law-enforcement investigation data is Level 4, in-country).

---

## System architecture

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│                         EXTERNAL SYSTEMS (APIs)                             │
│  NIMC  │  NIS  │  INTERPOL NCB (SLTD/Nominals/TDAWN/SMV)  │  B'Odogwu      │
│  NCAA PLACI / API-PNR (Phase 6)  │  AIS / manifest (Phase 6)  │  nCEN       │
│  NCS GEOINT / NAMA / NAF referrals (airspace)  │  NPA / scanner ingest     │
└─────────────────────────────────────────────────────────────────────────────┘
                                      ▲
                                      │ HTTPS
┌─────────────────────────────────────┴───────────────────────────────────────┐
│                         API GATEWAY & IAM                                   │
│     Rate limiting · JWT · RBAC · search-audit · NDPA purpose limitation     │
└─────────────────────────────────────┬───────────────────────────────────────┘
        ┌──────────────┬──────────────┼──────────────┬──────────────┐
┌───────▼──────┐ ┌─────▼─────┐ ┌──────▼──────┐ ┌─────▼─────┐ ┌──────▼──────┐
│ CORE SERVICES│ │ PERSONS & │ │  WATCHLIST  │ │  SYNC     │ │ AUDIT       │
│ locations,   │ │ ENTITY    │ │  + HITS     │ │  ENGINE   │ │ + LEDGER    │
│ conveyances, │ │ RESOLUTION│ │             │ │           │ │             │
│ incidents,   │ │           │ │             │ │           │ │             │
│ exhibits,    │ │           │ │             │ │           │ │             │
│ seizures,    │ │           │ │             │ │           │ │             │
│ transfers    │ │           │ │             │ │           │ │             │
└───────┬──────┘ └─────┬─────┘ └──────┬──────┘ └─────┬─────┘ └──────┬──────┘
        └──────────────┴──────────────┼──────────────┴──────────────┘
                                      │
                    ┌─────────────────┼─────────────────┐
              ┌─────▼─────┐    ┌──────▼──────┐   ┌──────▼──────┐
              │ PostgreSQL│    │    Redis    │   │ Object store│
              │ + PostGIS │    │             │   │ (encrypted) │
              └───────────┘    └─────────────┘   └─────────────┘
```

- Services are **containers** (portable: Railway for dev/staging; Nigeria-resident production).
- Evidence media is encrypted **before** write. SHA-256 is stored on the exhibit record at capture.
- The **web app** is Next.js. The **mobile app** talks to the API Gateway over HTTPS.
- Phase 6 targeting is a **separate service** behind the same gateway, not a rewrite of incidents.
- **PostGIS** stores every incident and patrol track, not only airport pins.

### Hosting and data residency

Law-enforcement investigation files, biometric templates, and watchlist hits are **classified / national-security data** under the NITDA National Digital Cloud Policy (Level 3–4) and the **Nigeria Data Protection Act 2023**. Level 4 data must be hosted in Nigeria (on-premise, collocated private DC, or government-certified private cloud), with encryption keys held in Nigeria.

| Environment | Host | Constraint |
|-------------|------|------------|
| Development | Railway | Synthetic / anonymised data only |
| Staging | Railway **or** Nigeria cloud | Prefer Nigeria if production will be in-country; no live watchlists |
| Production | Nigeria-resident, government-certified | Default. Railway production only with a written sovereign-data waiver from the competent authority (ONSA / NITDA / MDA) |

The stack stays portable: NestJS containers, PostgreSQL, Redis, S3-compatible object storage. Do not take a Railway-only dependency without an equivalent on the Nigeria target.

---

## Technical stack

| Layer | Technology |
|-------|------------|
| **Frontend (web)** | React 18, Next.js, Tailwind CSS, Recharts, MapLibre |
| **Backend** | Node.js (NestJS) — default for this repo; Go (Fiber) only if the team chooses it later |
| **Mobile** | Flutter (Dart), SQLite / WatermelonDB, background sync |
| **Database** | PostgreSQL 15 + PostGIS (TimescaleDB optional) |
| **Cache / queue** | Redis 7 (sessions, pub/sub, job queues) |
| **Object storage** | S3-compatible (Railway Volume in dev; Nigeria object store in production) |
| **Authentication** | JWT + OIDC (Keycloak, or managed Auth0 / FusionAuth) |
| **Biometrics / ID** | Native plugins for fingerprint scanners, camera MRZ, optional document-authenticator (UV/IR); plate/container OCR where hardware exists |
| **DevOps** | GitHub Actions, Railway CLI (dev/staging), portable container deploy to Nigeria cloud |
| **Security** | AES-256 at rest, TLS 1.3 in transit, OWASP ZAP, Trivy, NDPA DPIA, HashiCorp Vault if required |

### Backend choice for this repository

**NestJS (Node.js)** is the default backend so we can share TypeScript types with the Next.js portal, reuse existing Railway patterns from other StudioProjects, and ship Phase 2 faster. This can be revisited before Phase 2 if the team prefers Go.

---

## Role-based access (RBAC)

| Role | Access | Web | Mobile | Offline write |
|------|--------|-----|--------|---------------|
| **Super Administrator** | Full system management, audit logs, API keys, watchlist admin | Yes | No | No |
| **Top Executive (Commandant)** | Analytics, global search, transfer approvals, sign-offs, ops view — scoped by agency and optionally by domain | Yes | No | No |
| **Targeting Analyst** (Phase 6; stub in v1) | Watchlists, hit queue, risk rules, pre-arrival workbench (all modes) | Yes | No | No |
| **Field Officer** (NCS / NIS / NDLEA / NPF / AVSEC / NPA / NIMASA) | Create/update incidents at assigned locations, evidence, biometrics, scan-to-hit | No | Yes | Yes |
| **Airspace / maritime specialist** (NAF / Navy / NCAA) | Create/read in their domain; no land-border create unless assigned | Yes | Yes | Yes |
| **Legal / Prosecutor / Disclosure officer** | Read-only case files, hashed exhibits, prosecution-bundle export | Yes | No | No |
| **Integrity / Audit officer** | Search-audit, privileged-action review, dual-control exceptions | Yes | No | No |
| **DSS liaison** | Read / alert on flagged national-security cases | Yes (restricted) | No | No |
| **External Agency (INTERPOL NCB)** | Automated hits and limited case details (API only) | API only | No | No |

Permissions are enforced at the API with scopes such as `incident:create`, `exhibit:transfer:approve`, `watchlist:query`, `person:read`, `search:audit`, plus **location and mode constraints** (a Seme officer does not receive Apapa duty-roster cases by default). **Every person/document/plate/IMO search is itself an audited event.**

---

## Core domain entities

| Entity | Purpose |
|--------|---------|
| `users`, `roles`, `permissions` | Identity and RBAC |
| `agencies` | NCS, NIS, NDLEA, NPF, FAAN AVSEC, NCAA, NPA, NIMASA, Navy, NIWA, NAF, DSS |
| `locations` | Ports of entry and operating areas. `kind`: `airport`, `seaport`, `land_border`, `inland_checkpoint`, `waterway`, `coastal`, `airspace_sector`, `airstrip`, `green_border`. PostGIS geometry required. |
| `conveyances` | Vehicle (plate, VIN), vessel (IMO, name, flag), aircraft (registration, type), container (ISO), UAS. Linked across incidents. |
| `persons` | **Canonical identity** (NIN, passport, biometric template ids). Incidents link here. |
| `incidents` | Master case record, keyed by **UIN**. Required: `mode` (`land` \| `water` \| `air` \| `airspace`), `location_id`, optional `conveyance_id`. |
| `suspects` | Role of a `person` on an incident |
| `witness_statements` | Signed narrative; SHA-256 at submission |
| `exhibits` | Seized items and media; SHA-256 at capture; chain of custody |
| `legal_holds` | Freeze exhibit/case against retention deletion |
| `seizures` | Lot: HS/commodity, quantity, estimated value, store, disposal state (nCEN-shaped) |
| `agency_transfers` | Inter-agency handoff with biometric / OTP sign-off |
| `watchlist_entries` | Local wanted / lookout; INTERPOL cache metadata (not a shadow NCB) |
| `watchlist_hits` | Result of a scan or targeting rule, linked to person **or** conveyance + incident |
| `scanner_events` | Ingest from screening equipment, PLACI, or manifest referral |
| `search_audits` | Who searched which identifier, when, why |
| `audit_logs` | Append-only hash-chained ledger |
| `sync_queue` | Offline operations from field devices |
| `modus_operandi` | Repeatable concealment / route tags (including mode-switch: air→land, sea→land) |

`sync_queue` remains device-side; server entities above are the national record.

An incident at MMIA, a seizure of the same container at Apapa, and a later stop of the same trailer at Seme are **three incidents, one person master, one conveyance**, linkable in Person 360 / Conveyance 360.

---

## Design system (green & white)

Inspired by the Nigerian national colours.

| Token | Value | Use |
|-------|-------|-----|
| Primary | `#008751` Deep forest green | Brand, primary actions, online state |
| Secondary | `#00D68F` Emerald | Success, synced, highlights |
| Warning | `#F59E0B` Amber | Offline mode, pending sync |
| Danger | `#DC2626` Crimson | Alerts, failed sync, critical cases, **watchlist hit** |
| Background | `#FFFFFF` / `#F8FAFC` | Surfaces |

Mobile connectivity banners:

- **Online** — emerald — `● ONLINE – Real-Time Server Link`
- **Offline** — amber — `▲ OFFLINE MODE – Saving to Encrypted Local Vault`
- **Sync pending** — orange badge — `[OFFLINE QUEUED]`
- **Synced** — green badge — `[SYNCED TO SERVER]`
- **Hit** — crimson — `[WATCHLIST HIT]` — never colour-only; icon + text. Offline scans queue the query and show `HIT CHECK QUEUED`.

Every screen shows **mode + location** (e.g. `LAND · SEME`, `WATER · APAPA`, `AIR · MMIA`, `AIRSPACE · STRIP`). Field-officer UI uses large touch targets suitable for gloves, rain, and vehicle cabs.

---

## Documentation

| File | Contents |
|------|----------|
| [ROADMAP.md](./ROADMAP.md) | Phases 1–5 (Lagos multi-modal corridor) plus Phase 6 (national targeting), security, hosting |
| [APPROVALS-AND-INTEGRATIONS.md](./APPROVALS-AND-INTEGRATIONS.md) | Sponsoring MDA, NITDA/NDPC/ONSA approvals, and how to get NIMC, NIS, INTERPOL NCB, B'Odogwu APIs |

---

## Repository layout (planned)

```text
CrimeRecord/
├── README.md
├── ROADMAP.md
├── apps/
│   ├── web/          # Next.js executive portal
│   └── mobile/       # Flutter field-officer app (all modes)
├── services/
│   ├── api-gateway/
│   ├── auth/
│   ├── locations/    # ports of entry, sectors, geometry
│   ├── conveyances/  # vehicle / vessel / aircraft / container
│   ├── incidents/
│   ├── persons/
│   ├── exhibits/
│   ├── seizures/
│   ├── transfers/
│   ├── watchlist/
│   ├── sync/
│   └── audit/
├── packages/
│   └── shared/       # OpenAPI types, UIN helpers, design tokens
└── deploy/
    ├── railway/      # dev/staging
    └── nigeria/      # production compose / terraform stubs
```

This layout will be created during Phase 1. Do not treat it as present until scaffolding lands.

---

## Security baseline

- TLS 1.3 in transit; AES-256 at rest (DB, object store, and mobile SQLCipher)
- Column-level encryption for NIN, passport numbers, and biometric templates
- JWT scopes at every policy enforcement point, including **location/mode**
- Certificate pinning on mobile
- Encrypted biometric **templates only** — never raw fingerprint images on device
- SHA-256 of every exhibit and witness statement at capture; legal hold before retention expiry
- Immutable audit hash chain; **search audit** on every identity or conveyance query
- Dual control (two-person) for watchlist add/remove and for Super Admin override of custody
- Secrets in environment / Vault; encryption keys for production stay in Nigeria
- NDPA 2023: DPIA before go-live, retention schedule, lawful basis for biometrics, NDPC registration as a controller of major importance
- Daily backups; restore drills required; production backups stay in-country

---

## Timeline

**~28 weeks** from architecture to Lagos corridor go-live (MMIA + Apapa/Tincan + Seme + airspace case types), then **~12 weeks** for Phase 6 national targeting (starts only when the corridor is stable). See [ROADMAP.md](./ROADMAP.md).

| Phase | Duration | Outcome |
|-------|----------|---------|
| 1 – Architecture & UI | 4 weeks | Multi-modal ERD (locations, conveyances, persons, watchlists, seizures), OpenAPI, design system, hosting decision, DPIA start |
| 2 – Web portal & backend | 8 weeks | National dashboard by mode, person/conveyance 360, watchlist, seizure register |
| 3 – Mobile & offline sync | 8 weeks | Field app for land/water/air/airspace, hardware, hashed exhibits, sync |
| 4 – Security & testing | 4 weeks | Pen-test passed, NDPA/residency review, UAT at MMIA + Apapa + Seme |
| 5 – Production | 4 weeks | Go-live Lagos corridor, MDM, training |
| 6 – Targeting parity | 12 weeks | National Targeting Centre (API/PNR, maritime manifest/AIS, land referrals, airspace ingest, FIND) — **after** corridor |

---

## Current next step

Two tracks start together (engineering will otherwise finish with no live identity or watchlist):

1. **Engineering — Phase 1, Task 1:** Domain analysis and database schema (ERD) for the v1 entities in [Core domain entities](#core-domain-entities), with `locations.kind` and `incidents.mode` covering **land, water, air, and airspace** from the first freeze — plus a written **hosting decision** (Railway vs Nigeria-resident production) signed off against NITDA Level 3/4.
2. **Government — Week 0:** Name the **sponsoring MDA**, draft the concept note, and open adapter interfaces for NIMC / NIS / INTERPOL NCB on **fixtures**. Live APIs are MDA-to-MDA. See [APPROVALS-AND-INTEGRATIONS.md](./APPROVALS-AND-INTEGRATIONS.md).

Do not start feature code until the ERD, OpenAPI skeletons, and hosting decision exist. Do not freeze an airport-only schema. Do not call commercial KYC aggregators as if they were NIMC or NIS.
