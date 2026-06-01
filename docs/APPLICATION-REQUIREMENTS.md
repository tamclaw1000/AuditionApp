# APPLICATION-REQUIREMENTS.md

Functional requirements for **AuditionApp**.

> Scope of this document: **what** the system does (business/functional level).
> Technical "how" lives in `TECHNICAL-REQUIREMENTS.md`; data model, workflows and
> system architecture live in `ARCHITECTURE.md`. Live decision status for every open
> point is tracked in `PROGRESS.md`.

---

## 1. Purpose & scope

### 1.1 Main goal
AuditionApp is a **tamper-proof audition recording tool**. Its single purpose is to let
applicants record audition videos on their own device and guarantee to the receiving music
camp/institution that each video was **genuinely recorded in the app and not enhanced or
modified afterward**.

### 1.2 In scope
- Recording audition videos in a controlled, verifiable way across devices.
- Managing the set of recordings an applicant prepares for one camp (the **Audition**).
- Letting Camp Managers review submitted recordings and track applicant progress.
- Platform administration.

### 1.3 Out of scope
The broader **Camp Application** process — application forms, application fees, medical
records, acceptance decisions, scheduling — is **not** managed here. AuditionApp owns only the
audition-recording portion of that process.

---

## 2. Glossary

| Term | Meaning |
|------|---------|
| **Camp Application** | The whole external application process an applicant completes on the camp's side (fees, forms, medical records, recordings, etc.). Not managed by this tool. |
| **Audition** | The set of audition recordings an applicant prepares for one specific camp, managed together as a unit. Hierarchy: Camp Application ⊃ Audition ⊃ Audition Recordings. |
| **Audition Recording** (a.k.a. *Recording*) | One video recorded by the applicant on iOS / Android / macOS / Windows. |
| **Piece** | A required repertoire item; an Audition contains one or more pieces, each with one or more recording attempts. |
| **Submit / Submission** | The act of finalizing an Audition; after submission it becomes read-only and visible to the Camp Manager. |
| ~~Event~~ | **Deprecated** — do not use. |

---

## 3. Personas & roles

| Role | Description |
|------|-------------|
| **Applicant** | A musician recording auditions for one or more camps. |
| **Camp Manager** | A representative of a camp/institution who defines audition requirements and reviews submitted recordings for **their camp only**. |
| **Admin** | Platform owner; full cross-camp access, platform operations, history and abuse handling. |

---

## 4. Functional requirements

### 4.1 Accounts & authentication (functional behavior)
1. **No unverified accounts** are permitted; an account cannot be used until its email is verified.
2. **Applicant** — passwordless. The email address *is* the login; each sign-in sends a one-time
   code to that email which must be entered to authenticate. Behavior is **identical** in the
   Apex web app and in the recording apps.
3. **Camp Manager** — email or custom username + password, with **optional** second factor.
4. **Admin** — custom username + password with **mandatory** second factor on every sign-in.

*(Authentication technology — OAuth2 via ORDS, OTP delivery, 2FA — is specified in `TECHNICAL-REQUIREMENTS.md`.)*

### 4.2 Applicant roster & onboarding
1. A camp generally knows its applicants up front (applicants pay camp fees before starting),
   so the **"Never logged in"** state must be supported for known-but-inactive emails.
2. **POC:** a Camp Manager can upload an **Excel list** of applicant emails to seed the roster.
3. Any applicant may **self-register** an email and start an Audition even if their email is
   **not** on the camp's uploaded list.
4. *(Future)* Integrate with camp systems to pull the applicant roster automatically.

### 4.3 Recording & genuineness guarantee
1. Recordings are made **in-app only**; importing an externally produced video file is **not allowed**.
2. Each recording is **cryptographically signed / hashed at capture time** and carries a
   verifiable signature plus a tamper-evident watermark, so any later modification is detectable.
3. The server applies a **trusted timestamp** to each recording.
4. **Liveness checks** are desired where they do not add disproportionate complexity.
5. Camp Managers define, per piece, the **maximum number of recording attempts**; the limit applies
   to every piece. Remaining attempts per piece and the Audition status must be visible on every
   applicant screen.
6. Applicants may attach **private notes** to any recording attempt. These notes are **never**
   visible to the Camp Manager, even after submission.
7. The recording experience must feel essentially **like the native device camera app** (see §6 NFRs).

### 4.4 Audition lifecycle (applicant-side states)
| State | Meaning |
|-------|---------|
| **Not Started** | No recordings made yet. |
| **In Progress** | At least one recording made. |
| **Ready to Submit** | All required recordings made **and** the best version of each piece selected. |
| **Submitted** | Audition finalized. No new recordings; all data becomes read-only to the applicant; submitted versions become visible to the Camp Manager. |

1. Before submitting, the applicant **selects the best attempt** for each piece.
2. After submission, the Camp Manager sees **only the submitted version** of each piece — other
   attempts are never exposed.
3. *(POC caveat)* When applicants upload recordings to the camp manually (no cloud storage in the
   POC — see §6 / `TECHNICAL-REQUIREMENTS.md`), the final design of the submit/lock flow is
   **postponed.**

### 4.5 Camp Manager experience
1. **No camp-approval step** — a camp does not require platform approval to operate.
2. Before an Audition is **Submitted**, the Camp Manager can see **no recording content**. Per
   applicant email they see only a status:
   - **Never logged in** — email known (roster) but the applicant has never signed into the Apex
     app or a recording app.
   - **Logged In, no recording started** — signed in at least once, no recording yet.
   - **Recordings Started** — at least one recording completed.
   - **Audition is Submitted** — the Audition has been submitted.
3. After submission, the Camp Manager can review the submitted recording of each piece.
4. Camp Managers define per Audition: the **required repertoire / pieces**, the number of
   pieces/videos expected, the **attempts limit** per piece, and the **application deadline**.
5. A Camp Manager can **only ever** see data for their **own** camp (see §5.1).

### 4.6 Admin experience
1. Full access across all camps and applicants.
2. Access to a **complete history report** (all auditions across years) — see §4.9.
3. Platform operations: abuse handling, user/camp management.

### 4.7 Browse, search & filter
| Role | Capability |
|------|------------|
| **Admin** | All searches/filters (delivered via Apex Interactive Report, out of the box). |
| **Camp Manager** | Same breadth as Admin **but scoped strictly to their own camp's Auditions**; other camps' data is never available. Faceted search to be offered (Apex standard). |
| **Applicant** | Minimal — a simple report of pieces with **drill-down into a piece to see all recording versions**. The **same interface** appears in the recording app. |

### 4.8 Notifications
- **Admins:** Telegram **and** Email.
- **Camp Managers & Applicants:** Email only.
- *Detailed notification trigger flows are **postponed** (PROGRESS item 15).*

### 4.9 History & retention (user-facing behavior)
1. **POC:** previous-year history is visible to **Admins only**; Applicants and Camp Managers see
   no historical data.
2. **Later releases:** previous-year history becomes visible to Applicants and Camp Managers
   **in the Apex web app only** — never in the recording app.
3. Data retention is configurable (default 3 years) with **separate** policies for video files vs.
   audition metadata; data past its retention is deleted automatically. (Mechanism in
   `TECHNICAL-REQUIREMENTS.md`.)

### 4.10 Support
1. **Tech Support** section for Camp Managers to contact the platform/site support team.
2. **Contact Camp Support** section for Applicants to contact their camp's management.

---

## 5. Cross-cutting rules

### 5.1 Multi-tenant isolation (critical)
Strict data isolation between camps is a **top-priority** requirement. Camp A must **never** be
able to see any data belonging to Camp B. Applicants must never see other applicants' data.
*(Enforced via row-level security — see `TECHNICAL-REQUIREMENTS.md`.)*

### 5.2 Data minimization / privacy
Collect the **minimum** personal data to avoid heavier regulatory burden:
- **Minimum:** email only.
- **Maximum:** email + applicant first and last name.
- POC operates in the **US jurisdiction** only.

### 5.3 Internationalization
English for the POC; the structure must support additional languages (French, German, Italian)
in future, using Apex's native internationalization.

---

## 6. Platforms
1. **Web app:** Oracle Apex (Camp Manager & Admin management, applicant support and instructions).
2. **Recording apps:** iOS, Android, macOS, and Windows, all meeting the same recording and
   genuineness standards. A simplified submit/best-version-selection flow is available on the
   recording apps. The cross-platform framework is **under research** (PROGRESS item 6).
3. Recording apps must support: **external microphones**, an **offline mode** with deferred upload
   when connectivity returns, and a **Wi-Fi-only** offload option. A recording downloaded for
   personal use or manual camp submission still carries its watermark and signature.

---

## 7. Future functionality (kept in mind, not POC)
1. AI scan of recordings to identify strong/weak points and an overall performance grade.
2. Camp-configurable AI-scan templates (skills to emphasize).
3. Applicant-run generic AI scan on recording versions before submission.
4. Automatic camp-system integration to pull the applicant roster.
5. Cloud storage for recordings (POC ships without it; architecture must plan for it).

---

## 8. Open / pending decisions
The following functional areas are **not yet finalized** and are tracked in `PROGRESS.md`:
- Formal per-story **acceptance criteria** and the detailed **RBAC permissions matrix**
  (items 9 & 10) — to be drafted for review.
- Notification trigger flows (item 15).
- POC self-upload submit/lock flow (item 13).
- Mobile framework selection (item 6), which affects recording-app specifics.
