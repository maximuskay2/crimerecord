# JACCMS Nigeria — Full-Stack Developer Roadmap

**Joint Agencies Case & Crime Management System (JACCMS)**  
**Portable production edition** — Railway for development/staging; Nigeria-resident production for classified data

This roadmap is the implementation plan for the system described in [README.md](./README.md). Follow the phases in order. Each phase has a duration, deliverables, and concrete tasks.

**Mandate:** land borders, seaports and waterways, airports, and Nigerian airspace. Airports are one location type, not the product.

**Total duration to Lagos corridor go-live:** ~28 weeks (Phases 1–5)  
**National targeting (advanced-country layer, all modes):** +12 weeks after the corridor is stable (Phase 6)  
**Dev/staging host:** [Railway](https://railway.app)  
**Production host:** Nigeria-resident government-certified cloud or DC unless a sovereign-data waiver is granted (see §3.9)

The comparison that drove the additions is in [README.md — Why this exists](./README.md#why-this-exists-nigeria-vs-advanced-country-borders). Short version: v1 closes the **case / evidence / seizure / court-bundle** gap across **all modes** (US SEACATS, UK DEMS + Digital Case File, WCO nCEN). Phase 6 closes the **pre-arrival targeting** gap (CBP ATS, ICAO API/PNR, maritime manifests/AIS, land referrals, INTERPOL FIND). Do not pull Phase 6 into the corridor.

**Government and APIs are a parallel track**, not a Phase 5 surprise. NIMC, NIS, INTERPOL NCB, and B'Odogwu will not issue keys to a private repo. See [APPROVALS-AND-INTEGRATIONS.md](./APPROVALS-AND-INTEGRATIONS.md). Engineering uses fixtures until each MoU lands.

---

## 0. Parallel track — Government ownership, approvals, and APIs

Run this **beside** Phases 1–5. Do not stop coding for letters; do not go live without them.

| Window | Action | Exit artefact |
|--------|--------|---------------|
| Phase 1 weeks 1–2 | Name **sponsoring MDA**. Concept note. Joint steering ToR (ONSA-chaired recommended; NCS operational lead; NIS identity co-lead). OpenAPI **adapters** with fixtures. | Letter of intent or minute; `NimcClient` / `NisPassportClient` / `InterpolNcbClient` in OpenAPI |
| Phase 1–2 | NITDA **IT Project Clearance** pack. NDPA **DPIA** started. Processor agreement drafted. | ITPC submitted; DPO named |
| Phase 2 | **NIMC:** MDA letter to DG for NVS / NINAuth sandbox, then NDA + access level. **NIS:** MoU for passport verify; JACCMS will **consume** the NIS Passenger Data Single Window later, not collect airline PNR. | Sandbox credentials; NIS technical annex or documented manual fallback |
| Phase 2–3 | **NCB Abuja:** Direct Access MoU (EFCC 2022 pattern). Query SOP. FIND stays Phase 6. | MoU; backend-only query |
| Phase 3–4 | NCS **B'Odogwu** read-only referral spec. Scanner OEM ingest if available. File DPIA with NDPC; MDA DCPMI registration. | Interface spec; DPIA filed |
| Before Phase 5 | ITPC certificate (or equivalent). Live keys in Nigeria vault. Pilot SOPs at MMIA, Apapa, Seme. | Go-live board sign-off |

**Rules:** backend-only calls; fail closed (`CHECK UNAVAILABLE`, never `NO HIT` / `INTERPOL CLEAR` when the link is down); no commercial KYC aggregator on the incident path; vendor is **processor**, MDA is **controller**.

Phase 6 passenger data comes **from NIS** (already the API/PNR single window under Civil Aviation Act 2022 s.42). Do not build a second airline collection system.

---

## 1. Guiding principles

1. **National record is the source of truth** once an incident is synced. Offline copies are encrypted working copies, not independent ledgers.
2. **The person and the conveyance are canonical; the incident is an event.** One `persons` record per human. One `conveyances` record per vehicle / vessel / aircraft / container. Cases link to both. A container that flew into MMIA and left through Apapa is one conveyance, two incidents.
3. **Mode is a required field, not a comment.** Every incident has `mode` ∈ {`land`, `water`, `air`, `airspace`} and a `location` whose `kind` matches.
4. **Every mutation is audited. Every identity or conveyance search is audited.** Hash-chained, append-only. Querying a NIN, passport, plate, or IMO is itself a recorded action (UK PNC pattern).
5. **Scan means check.** MRZ / NIN / plate capture always queries the watchlist service (local + INTERPOL via NCB). A hit is a first-class object.
6. **Evidence is hashed at capture.** SHA-256 of the file (and of the witness statement) is stored before the record is considered complete. Legal hold beats retention.
7. **Field officers work without network.** Airside blackspots, green borders, creeks, and ship-side are the default operating condition, not an edge case. Watchlist checks queue when offline.
8. **Agencies share a case, not a login.** Inter-agency transfer is a first-class workflow with biometric or OTP sign-off — including NCS → NDLEA at a seaport, NIS → NPF at Seme, AVSEC → NCS at MMIA.
9. **Do not rebuild B'Odogwu, NPA TOS, NAMA ATM, or Navy C2.** JACCMS is the joint crime/case file. Those systems stay; JACCMS ingests referrals.
10. **Classified data stays in Nigeria.** Architecture is container-portable. Railway is the default for **development**. Production of investigation files, biometrics, and watchlists follows NITDA Level 3/4 unless waived.
11. **Green and white.** Visual identity follows Nigerian national colours (`#008751` / `#FFFFFF`).

---

## 2. Development phases

### Phase 1 — Architecture, data modelling, and UI design system

**Duration:** 4 weeks

**Deliverables**

- Approved system architecture document (portable containers; Railway for dev/staging; Nigeria-resident production path)
- Database schema (ERD) for all **v1** entities — **multi-modal from the first freeze** (locations, conveyances, persons, watchlists, seizures, search audits)
- Written **hosting decision** against NITDA National Digital Cloud Policy Level 3/4
- NDPA 2023 DPIA started (finish in Phase 4)
- UI/UX design system (green & white) for web and mobile, including mode/location chrome
- API specification (OpenAPI 3.0) and RBAC matrix (agency **and** location/mode)
- Initial Railway project with CI/CD
- Stub interfaces for Phase 6 (API/PNR, PLACI, maritime manifest/AIS, land referral, FIND, nCEN, airspace ingest) so the schema does not have to be rewritten
- Seed list of v1 locations: MMIA, Apapa, Tincan, Seme-Krake, plus placeholder airspace sectors / known strips

#### 1.1 Domain analysis & schema design

Define entities (v1 freeze):

- `users`, `roles`, `permissions`
- `agencies` (NCS, NIS, NDLEA, NPF, FAAN AVSEC, NCAA, NPA, NIMASA, Navy, NIWA, NAF, DSS)
- `locations`
  - `kind`: `airport` | `seaport` | `land_border` | `inland_checkpoint` | `waterway` | `coastal` | `airspace_sector` | `airstrip` | `green_border`
  - PostGIS `geometry` (point or polygon), parent location optional (terminal inside airport, berth inside seaport)
  - `mode` derived or stored: land / water / air / airspace
- `conveyances`
  - `type`: `vehicle` | `trailer` | `vessel` | `aircraft` | `container` | `uas` | `other`
  - identifiers: plate, VIN, IMO, vessel name, flag, aircraft registration, container ISO, flight number (event-level, may live on incident)
- `persons` (canonical identity: NIN, passport, DOB, nationality, biometric template ids, merge history)
- `incidents` (keyed by **UIN**; required `mode`, `location_id`; optional `conveyance_id`)
- `suspects` (person ↔ incident role)
- `witness_statements`
- `exhibits` (include `sha256`, `captured_at`, `capture_device_id`)
- `legal_holds`
- `seizures` (commodity, quantity, unit, estimated_ngn, store_location, disposal_state — **nCEN-shaped**)
- `agency_transfers`
- `watchlist_entries`, `watchlist_hits` (target may be person **or** conveyance)
- `scanner_events`
- `search_audits`
- `modus_operandi` (include `mode_switch` tags, e.g. sea→land)
- `audit_logs`
- `sync_queue` (device-side; server sees idempotent operations)

Phase 6 **stubs only** (tables or OpenAPI tagged `x-phase: 6`, not implemented):

- `api_pnr_messages`, `placi_filings`, `maritime_manifests`, `ais_positions`, `land_manifest_referrals`, `airspace_referrals`, `risk_scores`, `targeting_rules`

Use **PostgreSQL** with:

- **JSONB** for flexible agency-specific fields
- **PostGIS** for all locations, incident points, and patrol tracks (not airport-only)
- **TimescaleDB** optional for high-volume audit / search-audit / AIS stubs
- Unique indexes on `persons.nin`, `persons.passport_number + issuing_country`, `conveyances` natural keys (nullable, encrypted + blind index / HMAC for lookup)

Produce an ERD (dbdiagram.io or equivalent) and check it against the RBAC matrix before any application code.

**Entity-resolution rules (write them in the ERD notes):**

- Same NIN → same person
- Same passport number + issuing country → same person
- Same vehicle plate + issuing state (normalize) → same conveyance
- Same IMO → same vessel; same aircraft registration → same aircraft; same container ISO → same container
- Biometric template match is Phase 6 (1:N); v1 stores templates and matches 1:1 on the capturing device only
- Conflicts go to a supervisor merge queue — never silent merge

**Do not freeze an airport-only ERD.** If a reviewer cannot log a Seme vehicle stop, an Apapa container seizure, and an illegal-strip landing on the same schema, Phase 1 has failed.

#### 1.2 RBAC & permission model

- JWT with scopes, for example:
  - `incident:create`, `incident:read`, `incident:update`
  - `person:read`, `person:merge`
  - `conveyance:read`, `conveyance:write`
  - `watchlist:query`, `watchlist:manage`
  - `exhibit:upload`, `exhibit:transfer:approve`
  - `seizure:write`, `seizure:dispose`
  - `search:audit`
  - `audit:read`
  - `user:manage`
- **Constraints:** `agency_id`, `location_ids[]`, `modes[]` on the session. A Seme NCS officer creates `land` incidents at Seme; they do not see MMIA AVSEC duty cases unless transferred or granted.
- Keycloak deployed on Railway (dev), **or** a custom IAM microservice; production IAM in Nigeria
- Policy enforcement in the API gateway (PEP), not only in the UI
- Dual-control scopes: `watchlist:manage` and Super Admin custody override require a second authorised actor

#### 1.3 UI design system (green & white)

| Token | Hex | Role |
|-------|-----|------|
| Primary | `#008751` | Brand, primary buttons, online state |
| Secondary | `#00D68F` | Success, synced |
| Warning | `#F59E0B` | Offline, pending |
| Danger | `#DC2626` | Critical, failed, **watchlist hit** |
| Surface | `#FFFFFF` / `#F8FAFC` | Backgrounds |

Reusable components:

- **Web:** Tailwind config + React library (Button, Card, Badge, Table, Modal, Command palette, Hit banner, Person 360, Conveyance 360, **mode filter**)
- **Mobile:** Flutter `ThemeData`, type scale, glove-friendly touch targets (min 48dp); **location picker** (GPS-suggested + list) and **mode** on the first step of the incident form

Design:

- Executive dashboard (wide, analytics-first) with **mode tabs** (All / Land / Water / Air / Airspace) **and** a 24/7 operations strip (open hits, pending transfers, offline-device count)
- National map (MapLibre): land posts, seaports, airports, airspace/strip points
- Person 360 and Conveyance 360 (all incidents across modes)
- Mobile incident form (multi-step, offline-safe) with scan → hit → case path; first step is mode + location, not “airport terminal”
- Seizure register (not a photo gallery)

#### 1.4 API contract (OpenAPI 3.0)

Specify at least:

- Auth: login, refresh, MFA
- Locations: list assigned, nearby (PostGIS), CRUD (admin)
- Person: get-or-create by NIN/passport, merge, 360 view
- Conveyance: get-or-create by plate / IMO / registration / container ISO, 360 view
- Incident CRUD and search (UIN, NIN, passport, plate, IMO, name, mode, location)
- Watchlist: query, hit acknowledge, entry CRUD (dual control)
- Exhibit upload (returns `sha256`), chain-of-custody, legal hold
- Seizure CRUD and disposal workflow
- Transfer request / approve / reject
- NIN / passport verification proxy
- INTERPOL query proxy (NCB; never from the mobile app directly)
- Scanner-event ingest
- Prosecution-bundle export (read package: case JSON + hashed files + custody log)
- Sync: push offline queue, pull server changes
- Search-audit read (integrity officer)

Tools: Stoplight, SwaggerHub, or `openapi.yaml` in-repo.

Tag Phase 6 paths (`/v1/targeting/*`) as `x-phase: 6` so clients do not call them.

#### 1.5 Railway + portable deploy initialization

- Create the Railway project for **development**
- Add **PostgreSQL** (enable PostGIS via SQL)
- Add **Redis**
- Attach a **Volume** for evidence files in dev, or configure S3-compatible credentials
- Link GitHub for automated deploys
- Environments: `development` (Railway), `staging` (Railway or Nigeria), `production` (Nigeria-resident by default)
- Secrets as env vars (`DATABASE_URL`, `REDIS_URL`, JWT, external API keys)
- Push to `main` deploys staging; protect `production`
- Add `deploy/nigeria/` with a container compose / terraform **stub** so production cutover is not a surprise

**Phase 1 hosting decision (required exit artefact)**

Write one page:

1. Classification of JACCMS data (Level 3 vs 4)
2. Chosen production region / provider (NITDA-certified)
3. Where encryption keys live
4. Whether any waiver will be sought for Railway production
5. Backup locality

**Phase 1 exit criteria**

- [ ] ERD reviewed and frozen for v1 — reviewer can place land, water, air, and airspace incidents on the same schema
- [ ] `locations` seeded for MMIA, Apapa, Tincan, Seme-Krake, and at least one airspace sector / airstrip
- [ ] OpenAPI 3.0 published in-repo
- [ ] Design tokens and component inventory agreed (Hit banner, Person 360, Conveyance 360, mode filter)
- [ ] Hosting decision signed
- [ ] DPIA scoped (full document in Phase 4)
- [ ] Railway project has Postgres + Redis + empty service stubs
- [ ] CI runs lint/test on pull requests

---

### Phase 2 — Executive web portal & core backend services

**Duration:** 8 weeks

**Deliverables**

- Functional web app for top officials (national, filterable by mode)
- Backend services: auth, locations, conveyances, persons, incidents, exhibits, seizures, watchlist, transfers, audit
- Immutable audit log + search audit
- Services running on Railway **development**

#### 2.1 Backend services (NestJS modular monolith, split later if needed)

Each module must be deployable as its own service if traffic requires it.

| Service | Responsibility |
|---------|----------------|
| **Auth** | JWT issue, MFA, refresh, Keycloak/OIDC adapter; location/mode claims |
| **Locations** | Ports of entry and sectors; PostGIS; assignment to users |
| **Conveyances** | Get-or-create vehicle/vessel/aircraft/container; 360 view |
| **Persons** | Get-or-create, merge queue, 360 view, blind-index lookup |
| **Incident** | Create, update, search by UIN / NIN / passport / plate / IMO / mode |
| **Exhibit** | Media upload, SHA-256 at write, metadata, chain of custody, legal hold |
| **Seizure** | nCEN-shaped lot, valuation, store, disposal state machine |
| **Watchlist** | Local entries; query INTERPOL via NCB (SLTD, Nominals, TDAWN, SMV); persist hits; dual-control manage |
| **Transfer** | Inter-agency handoff workflow + biometric/OTP confirmation |
| **Audit** | Append-only hash chain for every mutation; search-audit stream |
| **Ingest** | Scanner events / PLACI / manifest referral stub → draft incident |
| **Sync** | Accept offline queues, emit pull deltas (skeleton in Phase 2; complete in Phase 3) |

**Universal Incident Number (UIN)**

Format: `UIN-YYYY-AGCY-XXXXXX`  
Example: `UIN-2026-NCS-A1B2C3`

- Year is UTC year of first capture
- Agency code is the originating agency (`NCS`, `NIS`, `NDL`, `NPF`, `AVS`, `NCA`, `NPA`, `NMS`, `NNV`, `NAF`, …)
- Suffix is a cryptographically random 6-character Crockford Base32 token
- Collision check at insert; never reuse a UIN
- Generated **on the device** when offline, then reserved on the server at sync (idempotent on client UUID)
- UIN does **not** encode mode; mode lives on the incident so a case can be reclassified (e.g. airspace strip → land seizure) without minting a new number

**Immutable audit log (hash chain)**

```text
audit_record {
  id
  entity_type
  entity_id
  action
  actor_id
  timestamp
  metadata          -- JSON snapshot of change
  previous_hash
  current_hash      -- SHA-256(previous_hash || canonical(payload))
}
```

- Inserts only. No `UPDATE` / `DELETE` on `audit_logs`.
- Enforce with table privileges and a database trigger that rejects mutations.
- Per-entity chain so parallel writes do not serialize the whole system.
- Verification job recomputes hashes and flags breaks.

**Search audit**

```text
search_audit {
  id, actor_id, timestamp, query_type, query_hmac, result_count, purpose_code, client_id
}
```

- `query_type` includes `nin`, `passport`, `plate`, `imo`, `aircraft_reg`, `container`, `name`
- Failed searches are logged too (anti-fishing)

#### 2.2 Executive web portal (Next.js)

- Login + MFA
- Dashboard: live stats (open cases, seizure value, pending transfers, **open hits**) **by mode**
- **24/7 ops strip:** unacknowledged hits, devices offline > N minutes, transfers waiting > SLA
- Global search: NIN, passport, UIN, plate, IMO, aircraft reg, container, name — every search writes `search_audits`
- **Person 360** and **Conveyance 360:** incidents, hits, seizures, transfers across agencies **and modes**
- Case detail: mode badge, location, timeline, exhibits (hash + custody), seizure lot, audit trail
- Hit queue (acknowledge, attach to case, dismiss with reason)
- Transfer approval queue (biometric or OTP)
- Seizure register (filter by commodity, agency, mode, disposal state)
- Audit trail viewer; Integrity role sees search-audit
- Prosecution-bundle export (zip: case JSON, exhibit hashes, custody PDF)
- Analytics (Recharts): crime patterns, seized-value trends, **mode-switch routes**
- Map (MapLibre): live incidents on a Nigerian land/sea/air map

#### 2.3 Migrations & seed data

- Prisma or TypeORM (pick one in week 1 of this phase and stick to it)
- Seed roles, agencies, **locations for the Lagos corridor**, test users, sample watchlist, sample persons **and conveyances**
- Fixture incidents: one land (Seme vehicle), one water (Apapa container), one air (MMIA passenger), one airspace (illegal strip)
- Run migrations as a **release command**

#### 2.4 Deploy to development

- Push to the development branch; builds backend + web
- Private networking: Postgres and Redis not public
- Health checks on every service
- Env vars: `DATABASE_URL`, `REDIS_URL`, `JWT_SECRET`, object-storage credentials
- INTERPOL / NIMC keys are **stubs** in development (recorded fixtures)

**Phase 2 exit criteria**

- [ ] Commandant can log in with MFA and filter the dashboard by Land / Water / Air / Airspace
- [ ] Person 360 shows an MMIA incident and a Seme incident for the same NIN as one person
- [ ] Conveyance 360 shows a container at Apapa and a later trailer stop as one container ISO
- [ ] Incident can be created, searched by UIN, and opened as a case file
- [ ] Watchlist query against fixtures returns a hit object (person or plate)
- [ ] Exhibit upload stores encrypted media, SHA-256, and a custody event
- [ ] Seizure lot can move from `detained` → `stored` → `disposed` with audit
- [ ] Transfer request appears in the approval queue
- [ ] Search for a NIN or plate writes a search-audit row
- [ ] Audit viewer shows a verifiable hash chain
- [ ] Seeded demo data covers NCS, NIS, FAAN AVSEC at MMIA, Apapa, and Seme

---

### Phase 3 — Mobile app & offline sync engine

**Duration:** 8 weeks

**Deliverables**

- Flutter field-officer app, offline-first, **all four modes**
- Background sync with conflict resolution
- MRZ + watchlist check; plate/container capture; biometric / optional document-authenticator
- SHA-256 of photos/files **on device** before queue
- App talks to the API Gateway

#### 3.1 Mobile UI (Flutter)

Screens:

- Login with MFA
- Home with connectivity banner, **assigned location**, and mode
- Quick actions: **Log New Incident**, **Scan Passport / NIN**, **Scan plate / container**, **Acknowledge Hit**
- Multi-step incident form: **mode + location** (GPS-suggested) → person/conveyance → exhibits → seizure → sign-off
- Recent logs with per-record sync status
- Settings: fingerprint-scanner pairing, document-reader pairing

Connectivity banners:

| State | Colour | Copy |
|-------|--------|------|
| Online | Emerald | `● ONLINE – Real-Time Server Link` |
| Offline | Amber | `▲ OFFLINE MODE – Saving to Encrypted Local Vault` |
| Sync pending | Orange badge | `[OFFLINE QUEUED]` |
| Synced | Green badge | `[SYNCED TO SERVER]` |
| Hit | Crimson | `[WATCHLIST HIT]` |
| Hit queued | Amber | `[HIT CHECK QUEUED]` |

Large touch targets for gloved use, rain, and vehicle cabs. Hit state is icon + text, never colour alone. Chrome always shows `MODE · LOCATION`.

**Happy path:** Scan MRZ or plate → watchlist query (or queue) → person/conveyance 360 summary if online → create incident pre-filled.

**Mode-specific capture (same form, extra fields):**

| Mode | Extra fields on the device |
|------|----------------------------|
| Land | Plate, VIN, trailer, crossing direction, GPS track |
| Water | IMO/name/flag, berth, container ISO, voyage |
| Air | Flight number, aircraft registration, terminal/zone |
| Airspace | Aircraft registration or UAS notes, strip/sector, how detected (visual, GEOINT, NAF referral) |

#### 3.2 Local database & encrypted store

- WatermelonDB **or** SQLite + sqflite (decide in week 1 of this phase)
- Encrypt with **AES-256** (SQLCipher or field-level encryption)
- Cached **assigned locations** and a signed watchlist **digest** for offline — not the full INTERPOL copy
- `sync_queue`: operation type (`create` / `update` / `watchlist_query`), payload, client UUID, timestamp, status, retry count

#### 3.3 Offline sync manager

On connectivity (`connectivity_plus`):

1. Push queued operations to the API (idempotent on client UUID)
2. Flush queued watchlist queries; raise local notifications on hits
3. Pull server changes (assignments, updates, hit acknowledgements) and merge locally
4. Retry with exponential backoff in the background

**Offline watchlist**

- Device holds a signed, encrypted **hot-list digest** (local wanted + recently circulated INTERPOL numbers **and plates**), refreshed when online
- Full INTERPOL query always goes through the backend when online
- Offline miss is not a clearance. UI copy: `Not on local hot-list — full check when online`

**Conflict resolution**

- Prefer **vector clocks** per record
- Last-write-wins is **not** allowed on exhibits, transfers, identity fields, conveyance identifiers, seizure disposal, or watchlist hits
- Conflicts surface to a supervisor queue on the web portal when automatic merge is unsafe
- Server is authoritative for UIN uniqueness, person/conveyance merge, transfer state, watchlist, and audit hashes

#### 3.4 Hardware integration

- **Passport MRZ:** `google_mlkit_text_recognition` + `mrz_parser`
- **Plate / container OCR:** ML Kit or vendor camera; manual override always available
- **Document authenticator (optional v1, recommended at airports and Seme):** UV/IR/white reader via vendor SDK. If hardware is not on the pilot devices, keep the interface and fall back to camera MRZ
- **Fingerprint scanners:** Futronic / SecuGen (or equivalent) via platform channels
- Store **encrypted templates only** — never raw fingerprint images on device
- After MRZ or plate parse: call watchlist query

#### 3.5 Device security

- Android Keystore / iOS Keychain for encryption keys
- Certificate pinning against the production (and staging) API certificates
- HTTPS only, custom domain (`api.jaccms.gov.ng` when available)
- Compute SHA-256 of each exhibit file **before** encrypting into the vault; send hash with the queued operation

**Phase 3 exit criteria**

- [ ] Officer can create a full incident with zero network, then sync without duplicates
- [ ] Same build supports land, water, air, and airspace (mode on step 1) — not four apps
- [ ] MRZ scan populates passport fields and queues or runs a watchlist check
- [ ] Plate / container capture creates or links a conveyance
- [ ] Online hit shows crimson banner and can attach to the new incident
- [ ] Offline “not on hot-list” does not claim INTERPOL-clear
- [ ] Fingerprint capture stores a template, not an image
- [ ] Exhibit photo has a SHA-256 that matches the server copy after sync
- [ ] Connectivity banners match the states above
- [ ] Conflict that cannot auto-merge appears on the executive portal

---

### Phase 4 — Security hardening, auditability, and integration testing

**Duration:** 4 weeks

**Deliverables**

- Pen-test report with critical/high findings closed
- Immutability verification tests
- NDPA DPIA completed; NDPC registration path started
- Hosting/residency review against the Phase 1 decision
- UAT sign-off at **three Lagos sites plus an airspace tabletop**
- All tests run against **staging**

#### 4.1 Encryption

- TLS 1.3 (SSL termination + valid custom-domain cert)
- At rest: PostgreSQL platform encryption **plus** column-level encryption for NIN, passport, plates where required, biometric templates
- Object storage encrypted at rest; application-level AES-256 before write
- Mobile SQLCipher
- Production encryption keys generated and stored in Nigeria
- Secrets only in env vars; Vault optional

#### 4.2 Penetration testing

- OWASP ZAP and Burp Suite against staging web + API
- Cover SQLi, XSS, CSRF, broken access control, IDOR, **search-oracle**, **cross-location IDOR** (Seme officer must not open unassigned Apapa cases)
- Mobile reverse-engineering attempt (MobSF)
- Prove watchlist:manage cannot succeed with a single actor
- Fix all **critical** and **high** findings before Phase 5

#### 4.3 Audit log integrity tests

- Mutating an audited entity always appends a hash-chain entry
- Searching a person or conveyance always appends a search-audit entry
- Direct SQL `UPDATE` on a case row is flagged by the verifier
- `audit_logs` rejects `UPDATE`/`DELETE`
- Re-hash an exhibit and prove mismatch is flagged

#### 4.4 Offline sync stress test

- Hundreds of simulated field devices (mix of land/water/air) creating offline records, then syncing together
- Include queued watchlist queries and exhibit hashes
- No data loss, no duplicate UINs, no silent person/conveyance merges, no unresolved silent conflicts

#### 4.5 NDPA / residency

- Finish DPIA: lawful basis for biometrics, retention per record class, data-subject rights vs investigation exemption, cross-border (INTERPOL) transfers
- Retention job: delete or anonymise according to schedule **unless** `legal_holds` is active
- Confirm production target meets NITDA Level 3/4; do not go live on Railway production without the waiver in writing

#### 4.6 UAT

- Staging isolated from production
- **MMIA:** NCS, FAAN AVSEC, NIS — airside blackspot + planted watchlist hit
- **Apapa or Tincan:** NCS, NDLEA, NPA, NIS — container seizure + agency transfer
- **Seme-Krake:** NCS, NIS — vehicle stop, plate hit, offline patrol track
- **Airspace tabletop:** NCS GEOINT / NCAA / NAF officer logs an illegal-strip landing from the web or mobile; case visible on the national map
- Capture feedback; fix blockers only (no Phase 6 scope)

**Phase 4 exit criteria**

- [ ] No open critical/high vulns
- [ ] Hash-chain and search-audit tamper tests pass
- [ ] Dual-control watchlist test passes
- [ ] Cross-location IDOR test passes
- [ ] Sync storm test passes
- [ ] DPIA signed; retention + legal-hold tested
- [ ] Hosting decision still valid (or updated in writing)
- [ ] Written UAT sign-off from NCS, AVSEC, NIS **and** a seaport + land-border lead

---

### Phase 5 — Production deployment and Lagos corridor rollout

**Duration:** 4 weeks

**Deliverables**

- Production environment live on the **approved** host
- Field devices provisioned with MDM at MMIA, Apapa/Tincan, and Seme
- Training materials per mode
- Go-live on the Lagos corridor, including airspace case types

#### 5.1 Production

- Separate production environment (not a promotion of staging data)
- Replicas: API gateway ≥ 2; core services 2 each; web 2
- Auto-scale on CPU / memory
- Private networking between services
- Custom domains + SSL: `api.jaccms.gov.ng`, `app.jaccms.gov.ng`
- PostgreSQL HA plan appropriate to load
- Redis persistence (AOF) if sessions/queues must survive restart
- Object-store versioning for exhibits; backup schedule documented **in-country**
- Observability: platform metrics/logs + Sentry; Grafana optional
- INTERPOL NCB and NIMC credentials from the live vault, not fixtures

#### 5.2 Mobile device management

- Enrol field tablets/phones in MDM (Miradore, ManageEngine, or equivalent)
- Restrict sideloading, enforce disk encryption, remote wipe on loss
- Pre-install JACCMS with a profile pointing at the **production** API URL
- Assign devices to **location** (MMIA / Apapa / Seme / patrol) so mode defaults are correct
- Hot-list refresh interval documented (e.g. every 15 minutes when online)

#### 5.3 Pilot launch — Lagos corridor

| Site | Mode | Agencies (minimum) |
|------|------|--------------------|
| MMIA | Air | NCS, FAAN AVSEC, NIS (NDLEA if devices available) |
| Apapa and/or Tincan | Water | NCS, NDLEA, NPA, NIS |
| Seme-Krake | Land | NCS, NIS |
| National (no extra site) | Airspace | NCS GEOINT, NCAA, NAF (web + selected mobile) |

- Run **in parallel** with legacy paper/digital process for a defined overlap window at each site
- Watch: sync success rate, API latency, error rate, UIN collisions (must be zero), watchlist false-dismissals, exhibit hash mismatches, **cross-site Person 360** (same NIN at two corridor sites)

#### 5.4 Training & support

- User manuals: web (commandants, legal, integrity) and mobile (field officers) with **mode-specific** annexes (airside, dockside, land-border patrol, airspace report)
- Hit-handling SOP (acknowledge, escalate, do not screenshot INTERPOL data)
- Live training sessions at each corridor site
- Helpdesk process and severity matrix

#### 5.5 Go-live & expansion

- Monitor for 2 weeks post go-live before the next corridor
- Expansion order (software already supports them; this is operations + devices):
  1. Air: NAIA Abuja → Kano → Port Harcourt
  2. Water: Onne → Calabar → Warri
  3. Land: Idiroko → Jibia → Ilela → designated inland posts
  4. Inland water: NIWA Lagos / Onitsha when NIWA cohort is ready
- Rollback / blue-green if a release fails; zero-downtime deploys required
- **Do not start Phase 6** until the Lagos corridor parallel-run report is accepted

**Phase 5 exit criteria**

- [ ] Production health checks green for 7 consecutive days
- [ ] Production data and keys confirmed in Nigeria (or waiver on file)
- [ ] MDM fleet enrolled at all three corridor sites; lost-device wipe tested on a spare unit
- [ ] Training completed for MMIA, Apapa/Tincan, and Seme cohorts
- [ ] Parallel-run report: JACCMS matches or exceeds legacy completeness **at each site**, including at least one genuine or drill watchlist path
- [ ] At least one Person 360 or Conveyance 360 spans two modes in live or drill data
- [ ] Expansion plan dated for the next air, land, and water post

---

### Phase 6 — National targeting parity (all modes)

**Duration:** 12 weeks  
**Start:** only after Phase 5 exit criteria. This is the CBP ATS / UK NBTC layer for **air, land, and sea**, plus airspace referrals. It is **not** the Lagos corridor pilot.

**Deliverables**

- National Targeting Centre workbench on the executive portal (mode tabs)
- Inbound **batch API** (ICAO/WCO/IATA PAXLST); iAPI if carriers can support it
- **PNR** ingest (ICAO Doc 9944 / PNRGOV) with NDPA-compatible retention
- **PLACI / ACAS** air-cargo pre-load messages (align with NCAA’s 2026 PLACI programme)
- **Maritime advance data:** cargo manifest / IMO FAL ingest; AIS correlation as a referral (not a Navy plot-table)
- **Land advance data:** B'Odogwu declaration/scan hit → `scanner_events`; optional ANPR later (do not build a national ANPR network here)
- **Airspace ingest:** NCAA / NAMA / NAF / NCS GEOINT referral → draft `airspace` incident
- Rules-based **risk engine** with analyst override (no unsupervised “AI deny board”)
- INTERPOL **FIND** (or NCB web-service equivalent) at primary line and on the field device when online
- 1:N biometric search against the person-master gallery (AFIS; facial if a lawful gallery exists)
- WCO **nCEN** export of non-nominal seizure data
- Read-only referral from **B'Odogwu** (all modes)
- Optional: body-worn camera / CCTV clip attach to exhibit (hash + legal hold)

#### 6.1 National Targeting Centre

- 24/7 queue: inbound flights, vessels, land consignments, API/PNR hits, PLACI/manifest do-not-load, INTERPOL hits, airspace referrals
- Analyst role (seeded in Phase 1, enabled here)
- Board / no-board (air) or load / do-not-load (cargo) is **advisory to NIS/NCS/carrier** unless a legal instrument says otherwise
- Every rule fire is an audit event and, if actioned, a `watchlist_hits` row

#### 6.2 API / PNR / iAPI (air)

- Passenger Data Single Window pattern (ICAO Annex 9 §9.1): carriers send once
- Legal basis required before go-live — engineering does not ship collection without it
- Retention: PNR is not a case file; default shorter than incident retention; DPIA addendum
- Match API document numbers to `persons` and `watchlist_entries`

#### 6.3 Maritime and land cargo

- Ingest pre-arrival cargo data; raise `scanner_events` / draft incidents for NCS/NDLEA
- Do-Not-Load style instruction is an **agency decision**, recorded on the seizure/incident
- AIS: correlate IMO on a `conveyance` with an incident or hit; do not replace Navy / NIMASA MDA
- Do not implement a second customs declaration system

#### 6.4 Airspace referrals

- Accept a structured referral (time, lat/lon or sector, registration if known, source: NAF / NCAA / NCS GEOINT / public)
- Create draft `airspace` incident; link aircraft conveyance if registration is known
- Illegal-strip `location.kind = airstrip` can be created by an authorised NCS/NAF user
- Do not ingest raw radar tracks into JACCMS

#### 6.5 Risk engine

- Deterministic rules first (document or plate on watchlist, repeat seizure MO, **mode-switch** pairs, route + commodity)
- Optional ML **score** as a feature, never as the sole board/no-board
- Analyst can suppress or escalate; suppressions expire

#### 6.6 FIND and 1:N biometrics

- Online field query to INTERPOL FIND via NCB; cache policy agreed with NCB
- AFIS 1:N against JACCMS gallery only (not a new national AFIS). Integrate NIMC/NIS galleries only with written MoU
- False-positive SOP: no arrest on biometric score alone

#### 6.7 nCEN and B'Odogwu

- Map `seizures` to nCEN national seizure / suspect / company / conveyance tables; Icomm export
- B'Odogwu: consume referral webhook or file drop; never write into B'Odogwu from JACCMS in this phase

**Phase 6 exit criteria**

- [ ] Targeting workbench processes a fixture **flight** (API + PNR) end-to-end
- [ ] Targeting workbench processes a fixture **vessel manifest** and a fixture **land declaration referral**
- [ ] PLACI fixture creates a draft NCS/NDLEA incident
- [ ] Airspace referral fixture creates a draft incident on the map
- [ ] FIND query from web and from mobile (online) returns a fixture hit
- [ ] nCEN export validates against WCO sample
- [ ] DPIA addendum for PNR/API (and maritime personal data) signed
- [ ] Legal instrument for carrier/manifest data collection is in force or explicitly waived by NCAA/NIS/NCS

---

## 3. Key technical designs

### 3.1 Universal Incident Number (UIN)

| Rule | Detail |
|------|--------|
| Format | `UIN-{YYYY}-{AGCY}-{TOKEN}` |
| Token | 6-char Crockford Base32, CSPRNG |
| Created | On device at first save (offline-safe) |
| Reserved | Server upsert by client UUID; reject duplicates of the UIN string |
| Display | Always shown on case header, printouts, and transfer forms |
| Immutable | Originating UIN never changes after first sync |
| Mode | Stored on the incident, **not** in the UIN |

### 3.2 Locations and modes

| `locations.kind` | Default `incidents.mode` |
|------------------|--------------------------|
| `land_border`, `inland_checkpoint`, `green_border` | `land` |
| `seaport`, `waterway`, `coastal` | `water` |
| `airport` | `air` |
| `airspace_sector`, `airstrip` | `airspace` |

GPS on the device **suggests** location; the officer confirms. Green-border and airspace incidents may have a GPS point that is not a named post — still required: mode + nearest location or a new `airstrip`/`green_border` point created under dual control.

### 3.3 Person and conveyance masters

- `persons.id` and `conveyances.id` are internal. Officers see NIN / passport / plate / IMO / photo, not extra public IDs.
- Get-or-create on every scan. Incidents store `person_id` and optional `conveyance_id`.
- Merge is dual-control (`person:merge`, `conveyance:merge`).
- Cross-mode: the same person at MMIA and Seme is one 360 view. The same container at Apapa and on a trailer inland is one conveyance. This is the cheap version of CBP TECS / UK PNC.

### 3.4 Watchlist and hits

```text
Scan (MRZ / NIN / plate / IMO / registration)
    → HMAC lookup on local entries
    → Backend query to INTERPOL NCB (SLTD, Nominals, TDAWN, SMV) when online
        → miss: log search-audit, continue case
        → hit: watchlist_hits row + crimson UI + optional auto-draft incident
```

- JACCMS is not a shadow NCB. Live INTERPOL records stay at NCB; JACCMS stores hit metadata, query time, and the case link.
- Dual control to add/remove **local** entries.
- Offline hot-list is a subset, signed, and never described as “INTERPOL clear”.

### 3.5 Immutable audit log

- SHA-256 hash chain **per entity**
- Payload canonicalisation (sorted JSON keys, no volatile whitespace) before hashing
- Verifier job (hourly on staging, every 15 minutes on production) walks recent chains
- Breaks raise a P1 alert; the record is frozen for investigation

### 3.6 Digital evidence lite (v1) vs DEMS (Phase 6)

v1 (corridor):

| Field | Rule |
|-------|------|
| Hash | SHA-256 of original bytes at capture (device) and at server write; must match |
| Custody | Append-only events: seized, bagged, stored, transferred, analysed, disclosed, disposed |
| Legal hold | Blocks retention deletion |
| Master / working | Server keeps master; mobile keeps encrypted working copy only |
| Disclosure | Prosecution-bundle zip with hashes; not a defence portal |

Phase 6 may add ACPO-style master/working copies, DESC-like defence share, and lab-referral workflow. Do not build that for the Lagos corridor.

### 3.7 Seizure register (nCEN-shaped)

States: `detained → examined → stored → (forfeited \| released \| destroyed \| auctioned \| transferred)`

Minimum fields: commodity/HS, quantity, unit, estimated value NGN, concealment method (`modus_operandi`), **conveyance**, **route (origin/transit/dest, including mode-switch)**, store location, photos (exhibits), linked persons/companies.

This is what advanced customs run as SEACATS / nCEN. It is not a photo album of bags.

### 3.8 Inter-agency transfer workflow

```text
Request (origin agency)
    → Pending (destination commandant queue)
        → Biometric or OTP sign-off
            → Accepted  → custody event + exhibit lock to destination
            → Rejected  → reason required, case remains with origin
```

- Both agencies retain read access to the shared case after transfer
- Write access follows the **current custodian** unless Super Admin dual-control override
- Every state change is an audit event and a chain-of-custody exhibit event
- Valid origin/destination includes NCS, NIS, NDLEA, NPF, AVSEC, NPA, NIMASA, Navy, NAF as assigned
- Transfer may also change **location** (Apapa warehouse → NCS HQ store) without changing agency

### 3.9 Offline sync conflict resolution

| Field class | Strategy |
|-------------|----------|
| Incident narrative / notes | Merge if non-overlapping; else supervisor queue |
| Mode / location | Server wins if already synced; else first verified write; reclassification is audited |
| Suspect identity (NIN, passport) | Server wins if already verified; else first verified write |
| Conveyance identifiers | Same as identity — no silent overwrite of IMO/plate |
| Person / conveyance merge | Server dual-control only |
| Exhibits | Never last-write-wins; append-only custody; hash mismatch is a P1 |
| Seizure disposal | Server finite-state machine only |
| Transfer state | Server finite-state machine only |
| Watchlist hits | Server authoritative; device may only acknowledge |
| GPS / timestamp | Keep both; display original capture time |

Idempotency key: client-generated UUID on every queued operation.

### 3.10 Hosting and data residency

| Class | Examples | Host |
|-------|----------|------|
| Level 1–2 | Public stats, anonymised analytics | Anywhere approved |
| Level 3 | Operational case metadata | Stored at rest in Nigeria |
| Level 4 | Investigation files, biometrics, watchlist hits, INTERPOL query logs | In Nigeria; keys in Nigeria |

Railway is correct for **development** and for staging **if** staging uses synthetic data. Production default is a NITDA-certified Nigerian cloud or government DC. Container portability is mandatory.

### 3.11 External systems

| System | Use | Phase | Notes |
|--------|-----|-------|-------|
| **NIMC** | NIN verification, basic biodata | 2 | REST from backend; keys in secrets |
| **NIS** | Passport verification; agency user of JACCMS | 2 | NIS is not “API only” |
| **INTERPOL NCB** | SLTD, Nominals, TDAWN, SMV query | 2 query / 6 FIND | Never from the mobile app directly |
| **FAAN / NCS / NDLEA / NPA scanners** | Referral event ingest | 2 stub / 5 live if vendor allows | Do not wait on a perfect SDK |
| **NCS GEOINT** | Illegal-strip / green-border referral | 5 tabletop / 6 ingest | Do not rebuild the GIS stack |
| **B'Odogwu** | Read-only cargo referral (all modes) | 6 | Do not duplicate declarations |
| **NCAA PLACI** | Pre-load air cargo | 6 | Align with 2026 MoU |
| **Airline API/PNR** | Pre-arrival passenger data | 6 | **Consume from NIS PIU** (CAA 2022 s.42 single window). Do not collect from carriers. |
| **Maritime manifest / AIS** | Pre-arrival cargo and vessel correlate | 6 | Referral, not Navy C2 |
| **NAMA / NAF** | Airspace referral | 6 | No raw radar in JACCMS |
| **WCO nCEN** | National seizure intelligence | 6 export | Schema shaped in v1 |
| **NIMC/NIS AFIS galleries** | 1:N biometrics | 6 | MoU required |

All third-party calls originate **only** from backend services.

### 3.12 File storage

- Dev: Railway Volume or S3-compatible
- Production: Nigeria S3-compatible or certified object store
- Application-level AES-256 encryption **before** write, regardless of platform encryption
- SHA-256 of plaintext original stored on `exhibits`
- Thumbnails generated server-side; originals never cached unencrypted on mobile beyond the encrypted local vault

### 3.13 Backups

- Daily backups for PostgreSQL and Redis
- Production backups remain in Nigeria
- Additional `pg_dump` to a second in-country location if policy requires it
- Restore drill at least once before go-live and quarterly after

### 3.14 Insider threat (Nigeria border reality)

Insider collusion is documented at airports **and** ports. JACCMS cannot vet staff (HR/DSS do that) but it can make collusion visible:

- Search-audit on every lookup (person **and** conveyance)
- Dual control on watchlist and custody override
- Location-scoped RBAC (no national fishing from a single post)
- Integrity officer role
- Unusual-search alerts in Phase 6 (volume, off-shift, same officer always “missing” hits)

Do not add a public citizen portal as a corruption control; that is a different product.

---

## 4. UI / UX guidelines

1. **National colours first.** Primary actions are forest green on white. Do not introduce a third brand colour. Crimson is reserved for hits, failures, and critical alerts.
2. **Mode is always visible.** `LAND · SEME`, `WATER · APAPA`, `AIR · MMIA`, `AIRSPACE · STRIP` on every case and on the mobile chrome.
3. **Status is always visible.** Connectivity and sync state on every mobile screen; case status, custodian, and open hits on every web case view.
4. **Search is a command, not a page.** Commandants search by NIN / passport / UIN / plate / IMO from a global bar. The search is audited.
5. **Gloves, rain, glare, vehicle cabs.** Mobile controls ≥ 48dp; high contrast; no colour-only status (pair colour with icon + text).
6. **Evidence is a chain, not a gallery.** Exhibit UI shows hash, custody events, and legal-hold state in time order.
7. **A person or conveyance is a 360, not a row on one case.** Cross-agency and **cross-mode** history is one click.
8. **Offline is calm, not alarming.** Amber, not red. Red is reserved for failed sync after retries, watchlist hits, and critical alerts.
9. **Least privilege in the chrome.** Field app has no admin navigation. Web has no incident-create for executives. Location assignment is visible.
10. **Audit is readable.** Hash values can be hidden behind “verify”; the human-readable actor/action/time is the default view.
11. **Never display “INTERPOL clear” on an offline device.**

---

## 5. Testing strategy

Executed against **staging** unless noted.

| Type | Tools | Scope |
|------|-------|-------|
| Unit | Jest (backend), Flutter test | Services, UIN, hash chain, RBAC, person/conveyance merge, sync merge, watchlist HMAC |
| Integration | Supertest, Cypress | API, web flows, mobile-backend sync, search-audit, location IDOR |
| End-to-end | Detox (mobile), Playwright | Login → scan → hit → create incident (offline) → sync → Person/Conveyance 360 across modes |
| Security | OWASP ZAP, Burp Suite, MobSF | Web, API, mobile binary, IDOR on persons and locations, dual-control bypass |
| Performance | k6 or Locust | 1000+ concurrent field sync requests |
| UAT | Manual | MMIA airside, Apapa dockside, Seme patrol, airspace tabletop, planted hit |
| Residency | Checklist | Production keys and backups in Nigeria |

---

## 6. Milestones and timeline

| Phase | Duration | Key deliverable |
|-------|----------|-----------------|
| 1 – Architecture & UI | 4 weeks | Multi-modal ERD (locations, conveyances, persons, watchlists, seizures), API specs, design system, hosting decision |
| 2 – Web portal & backend | 8 weeks | National dashboard by mode; person/conveyance 360; watchlist; seizure register |
| 3 – Mobile & offline sync | 8 weeks | One field app for land/water/air/airspace; scan-to-hit; hashed exhibits; sync |
| 4 – Security & testing | 4 weeks | Pen-test, NDPA DPIA, UAT at MMIA + Apapa + Seme |
| 5 – Production | 4 weeks | Go-live Lagos corridor; MDM; training |
| 6 – Targeting parity | 12 weeks | National Targeting Centre (all modes) — after corridor |

**~28 weeks** from Phase 1 start to Lagos corridor go-live.  
**~40 weeks** to national targeting parity, if Phase 6 starts immediately after a stable corridor.

---

## 7. Immediate work (Phase 1, week 1)

**Engineering**

1. Freeze the ERD for v1 entities listed in §1.1 — **including** `locations.kind` and `incidents.mode` for land, water, air, and airspace. Reject an airport-only schema.
2. Write `openapi.yaml` skeletons for auth, locations, conveyances, persons, incidents, watchlist, exhibits, seizures, transfers, sync, **and external adapters** (NIMC, NIS, NCB) on fixtures
3. Write the one-page **hosting decision** (NITDA Level 3/4)
4. Check Tailwind + Flutter token files against the colour table (add Hit banner and mode chrome)
5. Create the Railway **development** project and provision PostgreSQL + Redis (no public ingress); enable PostGIS
6. Scaffold the repo layout described in [README.md](./README.md)

**Government (same week — see [APPROVALS-AND-INTEGRATIONS.md](./APPROVALS-AND-INTEGRATIONS.md))**

7. Name the sponsoring MDA and a named ICT/Legal counterpart
8. Draft the 4–8 page **concept note** (problem, corridor, what JACCMS is not, API ask)
9. Do not integrate a commercial NIN/passport KYC API into the case path

Do not start Phase 2 feature work until Phase 1 exit criteria are met. Do not block Phase 2 on NIMC production keys — fixtures are enough to build.

---

## 8. Out of scope

### Do not pull into v1 (Phases 1–5)

- API / PNR / iAPI collection from carriers (Phase 6 + legal instrument)
- PLACI / ACAS risk engine (Phase 6; NCAA programme)
- Maritime AIS plot-table or Navy C2 (Phase 6 is referral/correlation only)
- National ANPR network (optional later; land v1 is officer-captured plates)
- INTERPOL FIND on the device as a live primary-line system (Phase 6; v1 is NCB query + hot-list)
- 1:N AFIS / facial gallery search (Phase 6)
- Live WCO nCEN Icomm (schema only in v1)
- B'Odogwu write-path or a second Single Window
- NPA terminal operating system, NAMA ATM/radar, NIMASA MDA rebuilds
- Public citizen portal
- Court e-filing / Judiciary integration (prosecution **bundle export** is in v1; filing is not)
- Full INTERPOL bidirectional case exchange
- Kubernetes / self-hosted cluster unless the Nigeria production target requires it
- On-device raw biometric image archives
- e-Gates, kiosks, Trusted Traveller, FAAN V-Pass (different products; JACCMS may later consume a referral)
- Unsupervised ML boarding or load denials
- Body-worn camera platform (optional attach in Phase 6, not a BWC vendor)

### Later national programmes (architecture must not prevent; do not staff here)

- Every remaining land post, creek, and airstrip on day one of the Lagos corridor (roll out devices and training after Phase 5)
- ECOWAS / AfCFTA automated case exchange
- Replacement of NIMC, NIS passport systems, NCS B'Odogwu, NPA TOS, or NAMA

The **schema and the app** cover land, water, air, and airspace from Phase 1. **Operations** still roll out by corridor. That is deliberate: a national data model with a Lagos first corridor, not an airport product with land bolted on later.
