# ARCHITECTURE.md

## 1. System overview

AuditionApp is a music audition submission platform with two primary actor groups:
- **Organizations**: register, log in, publish audition events, and review applicants.
- **Musicians**: register, log in, browse audition opportunities, apply, and upload audition videos.

The initial web application target is **Oracle APEX on Oracle Cloud Free Tier**. Mobile will be researched separately and should align with the data model and workflows defined here.

---

## 2. Data model for the system

### 2.1 Core entities

#### Organization
Represents an audition-hosting organization.

Key fields:
- `organization_id`
- `organization_name`
- `contact_name`
- `email`
- `password_hash` or external auth identity
- `phone`
- `website`
- `status`
- `created_at`
- `updated_at`

#### Musician
Represents an applicant / prospective performer.

Key fields:
- `musician_id`
- `full_name`
- `email`
- `password_hash` or external auth identity
- `phone`
- `primary_instrument_or_voice`
- `city`
- `country`
- `bio`
- `created_at`
- `updated_at`

#### Audition Event
Represents an opportunity published by an organization.

Key fields:
- `audition_event_id`
- `organization_id` (FK)
- `title`
- `description`
- `location`
- `deadline`
- `status`
- `audition_count_min`
- `audition_count_max`
- `created_at`
- `updated_at`

#### Audition Requirement
Represents structured requirements for a specific audition event.

Key fields:
- `requirement_id`
- `audition_event_id` (FK)
- `requirement_type`
- `requirement_text`
- `display_order`

Examples:
- repertoire rule
- number of submissions required
- resume required
- headshot required
- age restriction

#### Application
Represents a musician applying to a specific audition event.

Key fields:
- `application_id`
- `musician_id` (FK)
- `audition_event_id` (FK)
- `application_status`
- `submitted_at`
- `reviewed_at`
- `review_notes`

#### Video Submission
Represents one uploaded audition video attached to an application.

Key fields:
- `video_submission_id`
- `application_id` (FK)
- `title`
- `composer_or_work`
- `duration`
- `storage_url`
- `upload_status`
- `notes`
- `created_at`

#### Notification (optional supporting entity)
Used for Telegram or future messaging notifications.

Key fields:
- `notification_id`
- `recipient_type`
- `recipient_id`
- `channel`
- `message_body`
- `delivery_status`
- `created_at`
- `sent_at`

---

## 3. Relationship summary

- One **Organization** can create many **Audition Events**.
- One **Audition Event** can have many **Audition Requirements**.
- One **Musician** can create many **Applications**.
- One **Application** belongs to one **Musician** and one **Audition Event**.
- One **Application** can have many **Video Submissions**.
- One **Organization** reviews many **Applications** through its **Audition Events**.

---

## 4. Workflows

## 4.1 Organization workflows

### 4.1.1 New organization registration
1. Organization opens the organization-facing front end.
2. Organization chooses **Register**.
3. Organization enters business/contact details.
4. System validates required fields and uniqueness of email.
5. System creates the organization account.
6. Organization receives confirmation and is routed to the dashboard.

### 4.1.2 Existing organization login
1. Organization opens the organization-facing front end.
2. Organization enters email and password.
3. System authenticates credentials.
4. If valid, system routes to the organization dashboard.
5. If invalid, system shows a friendly login error and retry path.

### 4.1.2 Manage audition events
Organizations manage their events from a dashboard listing active, draft, and closed auditions.

#### 4.1.2.1 Add new
1. Organization chooses **Add New Audition Event**.
2. Organization enters title, description, deadline, location, and requirements.
3. Organization defines submission limits (`audition_count_min`, `audition_count_max`).
4. System validates inputs.
5. System saves the event and publishes it according to selected status.

#### 4.1.2.2 Review applicants
1. Organization opens an audition event.
2. System displays all musician applications for that event.
3. Organization reviews profile details and uploaded videos.
4. Organization records notes and status changes.
5. System stores the review outcomes for later follow-up.

## 4.2 Musician workflows

### 4.2.1 New user registration
1. Musician opens the musician front end.
2. Musician chooses **Register**.
3. Musician enters identity/profile details.
4. System validates email uniqueness and required fields.
5. System creates the musician account.
6. Musician lands on the dashboard and can browse auditions.

### 4.2.2 Existing users login
1. Musician opens the musician front end.
2. Musician enters email and password.
3. System authenticates credentials.
4. System routes the musician to the dashboard.

### 4.2.3 Apply for audition
1. Musician browses available audition events.
2. Musician opens an audition detail page.
3. Musician reviews requirements and submission limits.
4. Musician clicks **Apply**.
5. System creates an application record.
6. Musician is routed to the application workspace for uploads.

### 4.2.4 Upload video
1. Musician opens an existing application.
2. System shows how many videos are required and the maximum allowed.
3. Musician uploads one or more videos.
4. System stores metadata and cloud storage references.
5. System blocks uploads beyond the maximum limit.
6. System marks the application ready when minimum requirements are satisfied.

---

## 5. Suggested logical architecture

### Front end
- Oracle APEX web application for the initial release.
- Separate musician-facing and organization-facing navigation paths.

### Application/data layer
- Oracle APEX + Oracle database objects for business logic, forms, reports, and workflows.
- PL/SQL validations for application limits and workflow status.

### File/video storage
- External low-cost cloud object storage with free tier for initial testing.
- Store only references/metadata in the primary application database.

### Notifications/integrations
- Telegram for alerts and operational messaging.
- Jira for task tracking.
- Miro for architecture diagrams and collaborative planning.

---

## 6. Key design constraints

- The system must support both **musician** and **organization** personas cleanly.
- Audition submission rules must be configurable per event.
- Video uploads must be limited according to event-specific rules.
- Architecture should stay simple enough for AI-assisted development while remaining extensible.
