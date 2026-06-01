# PROGRESS.md

Latest status only. No discussion or rationale — see `OPEN-ITEMS.md` and the requirements docs for detail.

**Statuses:** `DECIDED` · `NEEDS-RESEARCH` · `NEEDS-DISCUSSION` · `AWAITING-INPUT` · `OPEN`

Last updated: 2026-05-31

---

## Product framing (cross-cutting)
AuditionApp is a **tamper-proof audition *recording* tool**. Its sole job is to let applicants record audition videos on a device and guarantee to camps/music institutions that the video was genuinely recorded and not enhanced or modified afterward. It is **not** an application-management system — the broader application process (camp website, fees, forms, medical records) lives outside this tool.

---

## Phase 0 — Foundational Decisions

| # | Item | Status | Latest |
|---|------|--------|--------|
| 1 | Product vision & scope | DECIDED | Tamper-proof recording tool for applicants to music camps/institutions; guarantees genuine, unmodified recordings. Not a full application manager. |
| 2 | Personas & roles | DECIDED | Applicant (musician), Camp Manager (org/camp rep), Admin (platform owner). |
| 3 | Glossary | DECIDED | **Camp Application** = the whole external camp process (fees, forms, medical records, etc.); **Audition** = the set of recordings for one camp managed by this tool; **Audition Recording** (a.k.a. Recording) = one video recorded by the Applicant on iOS/Android/macOS/Windows. Hierarchy: Camp Application ⊃ Audition ⊃ many Audition Recordings. **Event**: deprecated, do not use. Personas: Applicant, Camp Manager, Admin. (Diagram/ARCHITECTURE.md terminology rollout sequenced with item 17 — see notes.) |
| 4 | Auth & authorization strategy | NEEDS-DISCUSSION | Direction: APEX OAuth2 via ORDS; recording devices use ORDS API for metadata. **Applicant**: passwordless, email login + emailed OTP, identical on device & APEX app. **Camp Manager**: email or username + password + optional 2FA. **Admin**: username + password + mandatory 2FA. Exact tooling to be finalized before decision. |
| 4a | Tamper-proof / genuineness mechanism | DECIDED (direction) | Record-in-app only, NO file import. Cryptographic signing/hashing at capture time. Server-side timestamping. Liveness checks if not too complex. Core value proposition — needs dedicated detailed spec. |
| 5 | Multi-tenancy / data isolation | DECIDED | Strict isolation, top priority. Camp A must never see Camp B data. Row-level security required. |
| 6 | Mobile framework | NEEDS-RESEARCH | Critical architecture decision; research before deciding. Candidates: Flutter, React Native, Expo. |
| 7 | Cloud storage | NEEDS-RESEARCH | Research before deciding; APEX choice must NOT bias storage selection. POC ships WITHOUT cloud storage: signed recordings stay on device for manual upload to the camp site; only lightweight recording metadata goes to the APEX DB via ORDS (small calls vs. 0.5GB+ files). Architecture must still plan for cloud storage. |
| 8 | Notification channel | DECIDED | Admins: Telegram + Email. Camp Managers & Applicants: Email only. |

## Phase 1 — Business Requirements

| # | Item | Status | Latest |
|---|------|--------|--------|
| 9 | User stories + acceptance criteria | AWAITING-INPUT | Claude to propose a starter frame + initial suggestions; user will edit/extend. |
| 10 | RBAC / permissions matrix | AWAITING-INPUT | Claude to propose a starter frame from current info; user will edit/extend. |
| 11 | Registration, email verification, account states | DECIDED | No unverified accounts allowed. Account = email. Every login = email + emailed verification code (OTP). |
| 12 | Camp data-visibility & applicant roster (no approval step) | DECIDED | No org approval workflow. Camp Managers see applicant data only AFTER the Audition is Submitted. Before that, per applicant email they see only a status: "Never logged in", "Logged In, no recording started", "Recordings Started" (≥1 recording completed), "Audition is Submitted". "Never logged in" requires knowing applicant emails up front (applicants pay camp fees before starting) — plan for future Camp-system integration to pull the roster; **POC**: Camp Managers upload an Excel list of applicant emails, but any applicant may self-register an email even if not on the list. |
| 13 | Audition lifecycle state machine | DECIDED (POC self-upload pending) | Applicant-side states: "Not Started" (no recordings), "In Progress" (≥1 recorded), "Ready to Submit" (all recordings done + best version per piece selected), "Submitted". Camp-side view per item 12. Final design for the POC self-upload path is postponed. |
| 14 | Browse / search / filter | DECIDED | Admin: all searches/filters via Apex Interactive Report (out of box). Camp Manager: same but scoped to their own Camp's Auditions only (other orgs invisible); consider Apex Faceted Search. Applicant: minimal — simple report of pieces with drill-down into a piece to see all recording versions; same interface in the recording app. |
| 15 | Notification flows | POSTPONED | Deferred by user. |
| 16 | Non-functional requirements | DECIDED | **Recording UX**: must run flawlessly on every device, near-identical to native camera app. **Scalability**: infra easily scalable up/down (mostly cloud; less critical for POC). **i18n**: English for POC; structure supports future FR/DE/IT via Apex native i18n. **Privacy**: collect no private data to avoid regulation — min = email only, max = email + first/last name; US-only jurisdiction for POC. **Retention**: configurable, default 3 years, separate policies for video files vs. audition metadata (model must support both); POC shows no historical data to Applicants or Camp Managers, Admins get a full-history report; scheduled job deletes anything past retention. **History rollout:** POC = Admins only; later releases expose previous-year history to Applicants & Camp Managers in the **Apex app only** (never in the recording app). |

## Phase 2 — Technical & Architecture Requirements

| # | Item | Status | Latest |
|---|------|--------|--------|
| 17 | Finalized data model (DDL-level) | SEQUENCED-LATER | Tackled after all requirements are finalized. |
| 18 | REST API design (ORDS) | SEQUENCED-LATER | Same — after requirements are finalized. |
| 19 | Video upload pipeline & constraints | PARTIAL | Video is generated by our app, so it should be virus-free by design; still consider an out-of-box AV-scanning service as part of the cloud solution as a safeguard. Pipeline design deferred. |
| 20 | Storage integration design | SEQUENCED-LATER | Later discussion (tied to item 7). |
| 21 | Environment strategy | DECIDED | Three environments: **DEV** (dev team/agents), **UAT** (test full release deployment + manual & automated feature testing), **Prod** (live). |
| 22 | CI/CD pipeline for APEXlang | DECIDED (direction) | SQLcl + GitHub Actions. Evaluate **Liquibase** to avoid database state discrepancies. |
| 23 | Testing strategy & framework | AWAITING-CLAUDE-REC | Claude to recommend options; user will assess. (Also a research item.) |
| 24 | Observability, logging & audit | DECIDED | **Logging:** Apex/PLSQL use OraOpenSource Logger (github.com/OraOpenSource/Logger); recording-app logging TBD per chosen platform, pushed to Apex (likely via ORDS API). **Observability:** scheduled job scans Logger exceptions over a period and reports to Telegram for Admins. **Audit (Oracle):** every meaningful table audited on each change, per-column change history into a designated audit table (user has a trigger generator). **Audit (recording apps):** each recording-version completion and each best-version selection audited locally and sent to Apex. |
| 25 | Security requirements | NEEDS-RESEARCH | Important topic; more research needed. |

## Phase 3 — Configuration / Setup Scope

| # | Item | Status | Latest |
|---|------|--------|--------|
| 26 | Oracle Cloud account & tenancy | DECIDED (POC) | POC: likely three separate Free-Tier accounts (DEV / UAT / Prod); reconsider after POC. |
| 27 | Oracle DB + APEX workspace + ORDS | POSTPONED | Plus new requirement: put a **proper custom domain** in front of the Apex app so end users never navigate to the Oracle Cloud URL directly. |
| 28 | Object storage bucket setup | POSTPONED | Tied to item 7. |
| 29 | Agent dev environment | PARTIAL | SQLcl confirmed; everything else (VS Code extension, etc.) undecided. |
| 30 | GitHub configuration & branching | DECIDED | `main` = prod, protected, changes only via reviewed+approved PR. `dev` = completed development, protected, PR-only. Feature branches = unprotected, where agent/dev works; PR into `dev` when done. Release flow: features merge to `dev` → deploy UAT from `dev` → when prod-ready, deploy from `dev` to prod, then PR `dev`→`main`, merge, tag release on `main`. Hotfix flow: bug branch from `main` → PR direct to `main` → later `main`→`dev` PR to reflect prod fixes. |
| 31 | Task management (was Jira) | DECIDED (changed) | **Changed:** use GitHub Projects/Issues instead of Jira, at least for POC. |
| 32 | Telegram setup | DEFERRED | Details later. |
| 33 | Diagram tool (was Miro) | DECIDED (changed) | **Changed:** use Mermaid instead of Miro. |
| 34 | Mobile build/dev tooling & accounts | NEEDS-RESEARCH | — |
| 35 | Secrets & credentials management | DECIDED (POC) | POC: store on the local machines where the agent is installed. Later: likely HashiCorp Vault. |

## Phase 4 — Delivery Process & Conventions

| # | Item | Status | Latest |
|---|------|--------|--------|
| 36 | Implementation roadmap / milestones | AWAITING-CLAUDE-REC | Claude to recommend; user will decide. |
| 37 | Agent working agreement | OPEN | Not established yet; to be decided. |
| 38 | Document map | DECIDED | Final target trio: **APPLICATION-REQUIREMENTS.md** (all functional requirements), **ARCHITECTURE.md** (system architecture), **TECHNICAL-REQUIREMENTS.md** (all technical requirements). Everything else (PROGRESS.md, OPEN-ITEMS.md) eventually retired — but not now. |

---

## Notes
- **New concepts surfaced from other agents' `APPLICATION-REQUIREMENTS.md` — not yet placed in the item list, pending user direction:** per-piece recording-attempt limit set by Camp Manager; applicant private per-recording notes (never visible to camp); offline recording mode + wifi-only offload; external-microphone support; in-app tech-support and contact-camp sections; future AI performance-scan. 
- **Conflict resolved:** `APPLICATION-REQUIREMENTS.md` (#20/#21) vs. item 16 — POC = Admin-only history; later releases show previous-year history to Applicants & Camp Managers in the **Apex app only** (not the recording app). `APPLICATION-REQUIREMENTS.md` to be reconciled to this when that file is consolidated.
- **Diagram + ARCHITECTURE.md terminology rollout** (Musicians→Applicant, Organizations→Camp/Camp Manager, Application entity→Audition, drop "Event"): 105 occurrences across 4 files; sequenced as one coherent pass with the item-17 data-model rework rather than piecemeal now.
