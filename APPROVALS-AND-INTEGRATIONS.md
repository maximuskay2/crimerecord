# JACCMS — Government approval and existing-system APIs

This is the **how** for live identity, watchlist, and cargo systems. Engineering Phases 1–5 can proceed on **fixtures and sandboxes**. Production, INTERPOL, NIMC, NIS, and B'Odogwu will not turn on until the steps in this file exist on paper.

Engineering plan: [ROADMAP.md](./ROADMAP.md)  
Product: [README.md](./README.md)

**Reality check:** NIMC, NIS, INTERPOL I-24/7, and B'Odogwu do not issue API keys to a private GitHub repo. They issue access to a **Federal Public Institution** (MDA) that owns the processing purpose, signs an MoU, and is accountable under NDPA 2023. JACCMS must have a **sponsoring MDA** before any live API is possible.

Work this track **in parallel** with the 28-week build. Do not wait until Phase 5 to start letters.

---

## 1. What you can do this week (no government key required)

Build as if the APIs exist. Do not wait for Abuja to write code.

| Do now | Do not do |
|--------|-----------|
| Define **adapter interfaces** in OpenAPI (`NimcClient`, `NisPassportClient`, `InterpolNcbClient`, `BodogwuReferralClient`, `ScannerIngestClient`) | Call commercial KYC aggregators (Dojah, Youverify, etc.) as if they were NIMC/NIS for law-enforcement cases |
| Implement each adapter with a **fixture / sandbox** backend | Store live NINs in Railway development |
| NIMC: apply for the **NVS / NINAuth developer sandbox** with a written request to the DG (even as a vendor supporting an MDA) | Scrape passport or NIN portals |
| Write the **concept note + one-page hosting decision** the MDA will attach to every letter | Promise “INTERPOL integration” in a sales deck without NCB in the room |
| Identify the **sponsoring MDA** and a named counterpart (ICT + Legal + operations) | Treat StudioProjects as the data controller of investigation files |

Commercial identity APIs are consent-based KYC products. JACCMS lookups are **law-enforcement searches**. Wrong legal basis, wrong audit trail, wrong retention. Sandbox-only, if at all.

---

## 2. Who must own JACCMS in government

Pick an owner first. Everything else is an MoU from that owner.

| Option | Why it works | Risk |
|--------|----------------|------|
| **A. Joint programme, ONSA-chaired steering committee** (recommended) | Multi-agency (NCS, NIS, NDLEA, NPF, FAAN, NPA, NCAA, NAF, Navy). ONSA already owns CNII policy and national-security coordination. | Slower to stand up; needs a Terms of Reference and a secretariat MDA |
| **B. NCS as lead MDA**, others as participating agencies | Customs already runs B'Odogwu, nCEN, GEOINT, and operates land/sea/air. CG NCS is a natural operational owner of seizures. | Interior/Aviation may resist if it looks “Customs-only” |
| **C. Ministry of Interior (NIS + NPF)** as lead | NIS is already the **Passenger Data Single Window** for API/PNR (Civil Aviation Act 2022 s.42; NCAA 2024). Interior Minister has stated iAPI is live. | Seizures and B'Odogwu sit in NCS/Finance, not Interior |

**Recommended structure**

```text
Steering committee (ONSA chair or co-chair)
    ├── Lead operational MDA (propose: NCS) — budget, NITDA ITPC, NDPC controller
    ├── Co-lead for identity (NIS) — passport, API/PNR single window, SLTD
    ├── Participating: NDLEA, NPF (NCB), FAAN, NCAA, NPA, NIMASA, Navy, NAF, DSS (read)
    └── Technical vendor (StudioProjects or successor) — processor under NDPA, not controller
```

The **data controller** is the lead MDA (and each agency for data it contributes). The software vendor is a **data processor**. Put that in every MoU.

Until a Comptroller-General, Permanent Secretary, or NSA-level person will sign “this is our system”, stop treating API access as an engineering ticket.

---

## 3. Approval stack (what “government approval” actually is)

There is no single “JACCMS licence”. There is a stack. Run them as a pack, not one after another where you can overlap.

```text
1. Political / operational mandate
   Concept note → sponsoring MDA + inter-agency ToR → (if capital) FEC/ministry vote)

2. ICT and procurement (the system may be built)
   NITDA IT Project Clearance → BPP procurement (if government-funded)
   → vendor on NITDA ICT service-provider register (51% Nigerian) if you are the implementer
   → software testing / Class A certification before go-live (NITDA 2026)

3. Data protection
   NDPA DPIA (file with NDPC before live biometrics/watchlists)
   → NDPC registration as DCPMI (MDA = Extra-High Level)
   → processor agreement with the vendor

4. Sovereignty / security
   Hosting decision vs NITDA National Digital Cloud Policy Level 3–4
   → ONSA/CNII if designated Critical National Information Infrastructure
   → Galaxy Backbone / government-certified DC for production

5. System-to-system access (the APIs)
   Bilateral MoU + NDA + technical annex per owner (NIMC, NIS, NCB, NCS, NCAA, …)
```

NITDA IT Project Clearance is **mandatory** for Federal Public Institution IT projects (SGF Circular SGF/6/S.19/T/65 of 18 April 2006; Guidelines 2026). Typical path: FPI writes NITDA → portal login within ~48 hours → submit project → clearance target **21 working days** if the pack is complete. Projects **₦100 million and above** need a Licensed IT Projects Clearance Compliance Assurance Firm and lifecycle QA.

If the government is buying the build, BPP rules apply. If the vendor builds at its own cost as a partnership, you still need NITDA clearance **on the MDA side** and a signed partnership instrument — otherwise no MDA will connect you to NIMC or INTERPOL.

---

## 4. Sequence (parallel to the 28-week build)

| When (engineering) | Government / API action | Exit artefact |
|--------------------|-------------------------|---------------|
| **Week 0–2** (Phase 1 start) | Confirm sponsoring MDA. Submit **concept note** (4–8 pages) + architecture one-pager + hosting decision. Request a Joint Steering Committee ToR. | Letter of intent or minute from CG/PS/ONSA |
| **Week 2–6** | Draft **inter-agency MoU** (NCS, NIS, NDLEA, NPF, FAAN, NCAA, NPA). Start **NITDA ITPC** pack. Appoint MDA DPO. Start **DPIA**. | MoU in legal review; ITPC submitted |
| **Week 4–8** | **NIMC:** letter to DG/CEO for NVS and/or NINAuth (MDA, not the vendor). NDA + NVS forms. Sandbox credentials. | Sandbox username; NDA signed |
| **Week 6–12** | **NIS:** MoU for passport verification + agreement that JACCMS **consumes** the NIS Passenger Data Single Window (do not build a second airline API). | Technical annex + staging endpoint or “manual verify until API” |
| **Week 8–16** | **INTERPOL NCB Abuja:** MoU for Direct Access to I-24/7 information systems (same pattern as EFCC 2022). NPF/IGP and NCB Legal. FIND/MIND is a **later** technical project with INTERPOL GS. | MoU; query SOP; no live FIND yet |
| **Week 10–18** | **NCS ICT:** B'Odogwu **read-only referral** (hit/declaration id → JACCMS `scanner_events`). Scanner vendors: event feed MoU (FAAN/NCS/NPA). | Interface spec; fixture XML/JSON |
| **Week 12–20** | NDPC: file DPIA; register MDA as DCPMI. NITDA: Class A software test path if required. Confirm production host in Nigeria. | DPIA acknowledgement; NDPC number |
| **Before Phase 5 go-live** | ITPC certificate (or equivalent). Processor agreement. Live keys in Nigeria vault. Pilot SOPs signed by site commandants (MMIA, Apapa, Seme). | Go-live board sign-off |
| **Phase 6** | NCAA/NIS legal instrument already exists for API (CAA 2022 s.42). MoU to **receive** API/PNR *from NIS*, PLACI from NCAA programme, nCEN from WCO via NCS. | Targeting data-sharing annex |

Approvals are slower than code. If letters slip, **keep shipping on fixtures**. Do not block Phase 2 on NIMC.

---

## 5. Existing systems — how access actually works

### 5.1 NIMC (NIN) — v1, required

**What exists (public):**

- **NIMC Verification Service (NVS)** — SOAP, VPN, multi-level access (Level 1 least data … Level 5 fullest). Verify by NIN, NIN+fingerprint, demographics; search by fingerprint or demographics. [nimc.gov.ng/nimc-verification-service-api](https://nimc.gov.ng/nimc-verification-service-api)
- **NINAuth** — API + QR consent flow; Presidency has directed MDAs to use NINAuth for government verification. Better for *consent* services than for covert investigation queries. [ninauth.nimc.gov.ng](https://app.ninauth.nimc.gov.ng/)
- NIMC Act 2026: NIMC is root CA / DPI; NIN is foundational identity. Inter-MDA exchange is an explicit purpose.

**How to get it**

1. Sponsoring MDA writes the **DG/CEO, NIMC** (not a tweet, not a vendor email).
2. Execute **NDA / MoU**.
3. Complete **NVS forms**; agree access **level** (law enforcement typically needs more than Level 1; justify in the letter — investigation exemption vs NDPA).
4. Administrators created; VPN or NINAuth `client-id` / `client-secret`.
5. Sandbox first: “Submit a written request to the Director General” for the developers’ sandbox; credentials emailed when approved.
6. Service fee is **negotiable by access level** (NVS public page). Budget it in the MDA, not as a surprise.

**JACCMS design**

- Backend-only. Mobile never holds NIMC credentials.
- Prefer **NVS** (or a law-enforcement profile of NINAuth) for officer-initiated lookup; do not force a QR consent flow on a suspect at Seme.
- Store: verification timestamp, match/no-match, **level of data returned**, search-audit id. Do not keep a second copy of the national identity database.
- Blind index / HMAC for NIN at rest (already in the ERD).

**Letter to attach:** concept note, NDPA purpose (investigation / border control), DPIA excerpt, list of roles who may query, audit commitment.

### 5.2 NIS (passport, API/PNR window) — v1 identity; Phase 6 passenger data

**What exists (public):**

- NIS is the **Passenger Data Single Window** for API (and related passenger data) under Civil Aviation Act 2022 **section 42**; NCAA confirmed this in 2024 ICAO facilitation material. Interior has stated **iAPI is live** and NIMC↔NIS is integrated.
- Nigerian e-passport CSCA is on the **ICAO PKD** (July 2026) — document authentication is a PKD/chip problem, not a REST API you invent.
- There is **no public “NIS Passport API”** for third parties comparable to NVS. Private KYC vendors offering “NG passport verify” are not an NIS government integration.

**How to get it**

1. MoU: **CG Immigration** ↔ lead MDA (copy Ministry of Interior).
2. Ask for two things, in this order:
   - **v1:** passport-number verification (exists / biodata / watch-status) against the NIS document store — technical annex with NIS ICT.
   - **Phase 6:** JACCMS as a **downstream consumer** of the NIS PIU (API/PNR hits and referrals). Do **not** ask airlines to send PNR to JACCMS. That would violate the single-window Standard and pick a fight with NIS.
3. Until the API exists, v1 fallback: MRZ parse on device + officer confirmation + NIMC NIN if Nigerian + queued NCB check.

**JACCMS design**

- `NisPassportClient.verify({ number, issuingCountry, dob })`  
- Phase 6: `NisPiuClient.consumeHit(referral)` — JACCMS does not become a second PIU.

### 5.3 INTERPOL NCB Abuja — v1 query; Phase 6 FIND

**What exists (public):**

- NCB sits in the **Nigeria Police Force**. IGP (2026) is extending I-24/7 to border posts (Project GEMINI / SLTD).
- **Direct Access MoU** is the established pattern: EFCC signed with NCB in 2022 for I-24/7 databases (Nominals, SLTD, TDAWN, SMV, etc.).
- Nigeria was first in Africa to auto-upload SLTD (USNCB Project TERMINUS, NIS + NCB).
- **FIND/MIND** (front-line simultaneous national + INTERPOL query) is a separate technical deployment with INTERPOL General Secretariat. IGP has said expansion to borders is a priority — that is an NCB/INTERPOL project JACCMS should **plug into**, not replace.

**How to get it**

1. Lead MDA + **IGP / NCB Abuja** MoU: “Direct Access to and Use of INTERPOL Information Systems” (mirror the EFCC instrument; NCB Legal will have a template).
2. Nominate a **single technical gateway** (JACCMS backend in Nigeria). INTERPOL rules: no mobile app talking to Lyon; no caching a shadow copy of Nominals.
3. SOP: who may query, hit-handling (do not screenshot), dissemination limits (INTERPOL Rules on the Processing of Data).
4. FIND on devices = Phase 6 + INTERPOL GS + NCB. v1 = **backend query via NCB** + signed **local hot-list digest**.

**JACCMS design**

- `InterpolNcbClient.query({ sltd | nominal | tdawn | smv }, identifiers)`  
- Persist `watchlist_hits` metadata only (query time, hit/no-hit, NCB reference), not the foreign notice file unless NCB says you may.
- UI must never show `INTERPOL CLEAR` offline.

### 5.4 NCS B'Odogwu — Phase 6 read; v1 stub

**What exists:** NCS unified customs processing (declarations, duty) for all modes. Not a crime RMS. NSW is separate and stalled.

**How to get it**

1. Internal NCS ICT + Legal (if NCS is lead MDA this is a **directorate interface spec**, not a political MoU).
2. Ask only for **outbound referrals**: declaration id, scan hit, consignment key, officer id, timestamp → JACCMS `scanner_events`.
3. JACCMS **never writes** a declaration.

If NCS is *not* the lead MDA, you still need an MoU NCS ↔ lead MDA before B'Odogwu will emit events.

### 5.5 FAAN / NCS / NPA scanners — v1 ingest stub, live if vendor allows

FAAN Orion-class machines already fan out to six agency monitors. That is video, not an API.

**How to get it**

1. Site commandant + FAAN/NPA ICT + scanner OEM (Smiths, Nuctech, etc.): **secondary XML/JSON alarm export** or a simple “refer” button that POSTs to JACCMS ingest.
2. If OEM will not open a feed in time, officers **create the incident from the mobile app** (v1 still works). Do not block go-live on the OEM.

### 5.6 NCAA PLACI / API legislation — Phase 6

- API legal basis already exists (CAA 2022 s.42). NIS owns the window.
- NCAA PLACI MoU (August 2026) is **air cargo pre-load** with private vendors — JACCMS should **join as a government consumer**, not bid to become the PLACI system.
- Phase 6 letter: NCAA + NIS + NCS + lead MDA: “JACCMS receives referrals, does not collect carrier data itself.”

### 5.7 WCO nCEN — schema in v1, live export Phase 6

NCS already trains on nCEN. Export mapping is an **NCS enforcement** task. No new political approval if NCS is on the steering committee; just a technical mapping workshop with WCO/NCS nCEN admins.

### 5.8 NAMA / NAF airspace — Phase 6 referral

Do not ask for radar. Ask for a **structured referral form** (time, sector/lat-lon, registration if known, source). MoU with NCAA/NAMA/NAF as applicable. NCS GEOINT (illegal strips) is the same pattern: case, not GIS.

---

## 6. Documents you must produce (templates of substance, not letterhead art)

Have Legal (MDA) put these on MDA letterhead. The vendor may draft; the MDA signs.

| Document | Owner | Used for |
|----------|--------|----------|
| **Concept note** (4–8 pages): problem, agencies, Lagos corridor, what JACCMS is *not* (not B'Odogwu, not e-gates), 28-week plan, data classification | Vendor + MDA ICT | Every first meeting |
| **Inter-agency ToR / MoU** | Lead MDA Legal | Steering committee |
| **NDPA processor agreement** | Lead MDA + vendor | NDPC, NIMC, NCB |
| **DPIA** | MDA DPO (+ vendor input) | NDPC **before** live biometrics/watchlists |
| **NITDA ITPC pack** (FORM ITC / portal): scope, specs, local content, security, hosting | MDA | ITPC certificate |
| **Hosting decision** (Level 3/4, keys in Nigeria) | MDA ICT + CISO | NITDA + ONSA |
| **NIMC letter to DG** + NDA | MDA | NVS / NINAuth |
| **NIS MoU + technical annex** | MDA ↔ NIS | Passport + later PIU |
| **NCB Direct Access MoU** | MDA ↔ IGP/NCB | INTERPOL |
| **Pilot SOP** (hit handling, dual control, no screenshots) | Operations | Phase 5 UAT/go-live |
| **Go-live board minute** | Steering committee | Phase 5 exit |

---

## 7. Concept note — outline to write in Phase 1 week 1

Keep it short enough a CG will read it.

1. **Problem:** paper and email between NCS, NIS, NDLEA, AVSEC, NPA at the same seizure; no shared UIN; no person 360 across MMIA, Apapa, and Seme.
2. **What we will run:** joint case file, offline field app, hashed evidence, watchlist-at-scan, seizure register.
3. **What we will not run:** B'Odogwu, e-gates, airline API collection, Navy C2, NAMA radar.
4. **Corridor:** MMIA + Apapa/Tincan + Seme + airspace case types.
5. **Owner:** proposed steering committee and lead MDA.
6. **Data:** controller = MDA; processor = vendor; NDPA; in-country hosting for classified files.
7. **APIs requested:** NIMC NVS sandbox then production; NIS passport verify; NCB query; later FIND, B'Odogwu referral, NIS PIU.
8. **Ask:** letter of intent, named ICT/Legal counterparts, permission to file NITDA ITPC and NIMC sandbox in the MDA’s name.

---

## 8. Engineering rules while approvals are in flight

1. **One client module per external system**, behind the gateway. Swap fixture → sandbox → production with config, not a rewrite.
2. **No production secrets** on Railway. Sandbox keys in env for development only; live keys in the Nigeria vault after Phase 4.
3. **Fail closed on watchlists.** If NCB is down, show `CHECK UNAVAILABLE`, not `NO HIT`.
4. **Log every external call** into `search_audits` (who, which system, identifier HMAC, result class).
5. **Contract tests** against recorded NVS/NCB fixtures so a spec change in Abuja does not silently break the field app.
6. If an API is delayed, **v1 still ships**: MRZ + local watchlist + manual “pending NIMC/NCB” badge.

---

## 9. Who to put in the first meeting

Minimum viable room (Lagos corridor):

- NCS: ICT + Enforcement (or CG’s office)
- NIS: ICT + Border Management
- NPF: NCB Abuja
- FAAN AVSEC (MMIA)
- NPA (Apapa) or designated port security
- NDLEA (optional week 1, required before narcotics workflow)
- ONSA or ministry observer if you can get one
- MDA Legal + DPO
- Vendor: product owner + architect (no live demo of “hacking” government APIs)

Leave INTERPOL General Secretariat, NIMC DG, and NCAA DG for **follow-up letters**, not the first coffee.

---

## 10. Immediate checklist (this week)

- [ ] Name the **sponsoring MDA** and a human counterpart
- [ ] Draft the **concept note** from §7
- [ ] Freeze OpenAPI adapters: NIMC, NIS, NCB, B'Odogwu, scanner ingest (fixtures)
- [ ] Start NIMC sandbox request **in the MDA’s name** (letter to DG)
- [ ] Start hosting decision (NITDA Level 3/4) — already a Phase 1 exit criterion
- [ ] Do **not** integrate a commercial KYC API into the incident path
- [ ] Do **not** tell anyone the mobile app will “have INTERPOL” before NCB has signed

When the concept note and MDA counterpart exist, the next concrete filings are: inter-agency ToR, NITDA ITPC, NIMC NDA, NCB MoU. Engineering continues on fixtures until each annex lands.
