# OPEN-ITEMS.md

Tracked concerns and open decisions for AuditionApp.  
Each item has a status and priority. New sessions should start here.

---

## Status legend
- `OPEN` — not yet addressed
- `RESOLVED` — decision made, documented
- `IN-PROGRESS` — actively being worked

## Priority legend
- `P0` — blocks agent development from starting
- `P1` — blocks a major feature or workstream
- `P2` — important but not immediately blocking

---

## Resolved

| # | Item | Resolution |
|---|------|------------|
| R1 | Oracle APEX not agent-friendly (no git-diffable artifacts) | **RESOLVED** — APEXlang (APEX 26.1) exports apps as structured `.apx` text files; fully git-diffable and AI-agent-ready. Toolchain: VS Code + SQL Developer extension + SQLcl. |

---

## Open — Technical Stack

| # | Priority | Item | Notes |
|---|----------|------|-------|
| T1 | P0 | **Mobile framework not selected** | Cross-platform research needed. Candidates must work well with ORDS REST APIs and fit low infrastructure budget. Flutter, React Native, and Expo are likely candidates. |
| T2 | P0 | **Cloud storage not selected** | Must be enterprise-grade, low-cost, free tier available. Candidates: Oracle Object Storage (already in OCI free tier), Cloudflare R2, Backblaze B2. Budget is high priority constraint. |
| T3 | P1 | **Testing framework not selected** | Preference for AI-assisted/AI-powered. Must cover APEX/PL/SQL layer and REST API layer. |
| T4 | P1 | **Agent dev environment not defined** | Agents need SQLcl + VS Code SQL Developer extension configured to export/validate/import APEXlang `.apx` files. Setup must be documented and reproducible. |
| T5 | P2 | **Communication/notification channel not confirmed** | Telegram is a candidate. Decision affects Notification entity implementation and alert triggers. |
| T6 | P2 | **Architecture diagram tool not confirmed** | Miro (3 free diagrams) is a candidate. Low priority — diagrams already exist in SVG/Excalidraw format in repo. |

---

## Open — Architecture

| # | Priority | Item | Notes |
|---|----------|------|-------|
| A1 | P0 | **REST API design not covered** | Oracle ORDS provides the REST layer. Needs: endpoint inventory, request/response contracts, authentication method for API consumers (mobile client + agents). Critical for mobile workstream. |
| A2 | P0 | **Authentication strategy not defined** | Options: APEX built-in auth, Oracle IDCS/OCI IAM, OAuth2 with ORDS. Must cover both web (APEX session) and API (token/JWT for mobile). Decision affects all login workflows. |
| A3 | P1 | **Application state machine not defined** | `application_status` values are undefined. Agents will invent them inconsistently without a formal state machine. Proposed: `DRAFT → SUBMITTED → UNDER_REVIEW → ACCEPTED / REJECTED / WITHDRAWN`. Needs confirmation. |
| A4 | P1 | **Video upload constraints not defined** | Missing: accepted formats, max file size per video, max total size per application, virus/content scanning requirement, upload timeout handling. |
| A5 | P1 | **Multi-tenancy / data isolation not specified** | Must confirm: can Organization A see Organization B's applicants or events? Row-level security policy needed in Oracle DB. |
| A6 | P1 | **Environment strategy not defined** | dev / staging / prod environments needed. Oracle Cloud Free Tier has limits — need to confirm if one APEX workspace can host multiple environments or if separate workspaces are required. |
| A7 | P1 | **CI/CD pipeline not defined** | How do agent-written APEXlang `.apx` files get validated and deployed? SQLcl scripting + pipeline (GitHub Actions?) needs to be designed. |
| A8 | P2 | **Admin role not defined** | Who manages the platform itself (approve organizations, handle abuse, manage users)? No admin persona in current requirements. |
| A9 | P2 | **Musician browse/search not specified** | "Browse auditions" is undefined. Needs: search by instrument, location, deadline, organization name? Filtering and pagination requirements. |
| A10 | P2 | **Post-review notification flow not specified** | After organization accepts/rejects an application, how is the musician notified? Email? Telegram? In-app only? |

---

## Open — Application Requirements

| # | Priority | Item | Notes |
|---|----------|------|-------|
| Q1 | P0 | **APPLICATION-REQUIREMENTS.md needs acceptance criteria** | Current document is one sentence. Agents need user stories with pass/fail criteria per persona (Musician, Organization, Admin). |
| Q2 | P1 | **Email verification flow not specified** | Is email verification required on registration for both personas? What happens to unverified accounts? |
| Q3 | P1 | **Organization approval workflow not specified** | Can any organization self-register and immediately publish events, or is there an admin approval step? |
| Q4 | P2 | **Permissions matrix not defined** | What actions can each role (Musician, Organization, Admin) perform? Needs a formal RBAC table. |

---

## Budget constraint (cross-cutting)

**Low infrastructure budget is a high priority.** All stack decisions must be evaluated against this.  
Current free-tier anchors:
- Oracle Cloud Free Tier (APEX + ORDS + Oracle DB + Object Storage)
- Jira Free (≤10 users)
- GitHub Free

Any candidate tool must have a credible free or very low-cost entry point before it is selected.

---

## How to use this file

1. Pick the highest-priority `OPEN` item.
2. Research or decide with the user.
3. Move to `Resolved` table with a one-line resolution summary.
4. Update the relevant `docs/` file (ARCHITECTURE.md, TECHNICAL-REQUIREMENTS.md, etc.) with the decision.
