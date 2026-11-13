# Make My Marriage (MMM)
## Product Requirements Document (PRD)

| Document field | Value |
|---|---|
| Product | **Make My Marriage** (MMM) |
| Version | **1.1 — Aligned Product Baseline** |
| Status | **Approved product baseline with later decisions incorporated** |
| Date | **12 November 2026** |
| Product type | Free, responsive, India-first wedding-management web application |
| Intended readers | Product owner, designers, frontend and backend engineers, QA, and future contributors |
| Scope | All 14 agreed functional modules; staged delivery is permitted |

> **Revision note (12 November 2026):** This aligned PRD supersedes PRD v1.0 where newer product decisions differ. In particular, **a Paid expense may have a selected registered organizer OR a manually named external payer**. Event cancellation preserves the existing family invitation/RSVP and notifies eligible accepted families; Vercel, Cloudflare R2 and Resend are the implementation platform choices in the companion system design.

> **Document principle:** This PRD defines *what the product must do*, not a mandatory technology stack or microservice architecture. Requirements explicitly marked **Open decision** are not additional agreed features. They are details to settle before implementing their affected workflows.

---

## Contents

1. [Product overview](#1-product-overview)
2. [Goals, principles and scope](#2-goals-principles-and-scope)
3. [Users, roles and authorization](#3-users-roles-and-authorization)
4. [Key concepts and business rules](#4-key-concepts-and-business-rules)
5. [User journeys](#5-user-journeys)
6. [Functional requirements](#6-functional-requirements)
7. [Navigation and screen inventory](#7-navigation-and-screen-inventory)
8. [Non-functional requirements](#8-non-functional-requirements)
9. [Cross-module edge cases](#9-cross-module-edge-cases)
10. [Release plan and priorities](#10-release-plan-and-priorities)
11. [Metrics and launch readiness](#11-metrics-and-launch-readiness)
12. [Dependencies, constraints and risks](#12-dependencies-constraints-and-risks)
13. [Open implementation decisions](#13-open-implementation-decisions)
14. [Acceptance and sign-off checklist](#14-acceptance-and-sign-off-checklist)
15. [Glossary](#15-glossary)

---

# 1. Product overview

### 1.1 Problem statement

Indian weddings often involve multiple ceremonies (e.g., Haldi, Mehndi, Sangeet, the main wedding and reception), many relatives and organizers, scattered task lists, payments made by different people, vendors, and large guest lists. Planning is frequently split across conversations, spreadsheets and shared documents. Invitations and post-event photos create additional coordination work.

### 1.2 Product vision

**Make My Marriage** is a single, straightforward workspace for planning an Indian wedding and a simple guest-facing experience for invitations, RSVP, a wedding website and photo sharing. The platform is designed for couples, families and their chosen organizers—not for professional financial accounting or a complex vendor marketplace.

### 1.3 Value proposition

- **One place to organize:** Events, tasks, guests, vendors and an expense ledger connect to the same wedding.
- **Simple collaboration:** Admins and managers can all carry out wedding-planning work; only admins manage organizer membership.
- **Easy guest participation:** Families can RSVP using a unique invitation link; guests never need to create accounts.
- **Event-wise invitations without event-wise RSVP complexity:** A family can be invited to selected ceremonies, but gives one answer covering all ceremonies on its invitation.
- **Memories collected easily:** Event albums accept guest photos through links and QR codes, with private gallery-sharing links.
- **Free to use:** No checkout, subscriptions or payment processing.

### 1.4 Primary platform and context

- Responsive **web application** usable on phones and desktops; no native app required.
- **India-first** examples, currency (INR), wedding customs and typical event structures.
- Guest pages should work particularly well on mobile, including low-friction RSVP and photo upload.
- Interface language for v1 is English. Other languages are **not committed** in v1.
- Confirmed implementation providers in the companion system design: **Next.js/TypeScript on Vercel**, **MongoDB Atlas/Mongoose**, **Cloudflare R2** for images, **Resend** for emails, **Google Places API** for vendor discovery. These selections do not change the functional scope.

# 2. Goals, principles and scope

### 2.1 Product goals

| ID | Goal | Desired observable outcome |
|---|---|---|
| G-01 | Make planning centrally manageable | An organizer can create a wedding and manage events, tasks, vendors and expenses in one workspace. |
| G-02 | Support lightweight collaboration | Multiple admins/managers work within one wedding without granular permissions. |
| G-03 | Simplify family invitations | A guest family receives one link listing its invited ceremonies and responds once. |
| G-04 | Make wedding information shareable | A public wedding website provides the couple's basic details and event schedule. |
| G-05 | Collect and share memories | Event-wise albums accept account-free guest uploads and support private sharing. |
| G-06 | Be practical to operate publicly | Guest-accessible links, email sending and file uploads have basic abuse prevention and reliability safeguards. |

### 2.2 Design principles

1. **Simple over sophisticated.** Avoid business features that require complex permissions, scheduling engines or financial bookkeeping.
2. **One wedding per user account.** A registered user is associated with at most one wedding at a time; each wedding can have multiple users.
3. **Equal operational access.** Admin and manager interfaces are effectively the same except for organizer membership management.
4. **Guests do not sign in.** Separate organizer authentication from guest links.
5. **Event-wise organization; family-wise response.** Events are distinct; invitations select events; RSVP is a single response per family invitation.
6. **Ledger, not transactions.** The app records whether an expense is paid and who paid it; it does not transfer money.
7. **Links grant limited access only.** Invitation, upload and gallery links are distinct and must not grant broader rights than their purpose.

### 2.3 Confirmed inclusions

The agreed functional scope consists of **14 modules**: authentication; wedding/organizer management; dashboard; events; tasks; expense tracker; guest management; digital invitations; RSVP/reminders; vendor management; vendor discovery; wedding website; event-based photo galleries/QR uploads; wedding calendar. YouTube Live embed is included in the wedding website module.

### 2.4 Explicit non-goals

- No payment processing, checkout, subscriptions, fees, budget allocation, financial forecasts, partial payments or installments.
- No guest login, guest role, or vendor login/dashboard.
- No multiple wedding workspaces under the same registered account.
- No bride-side/groom-side budget or guest pools; no private expenses between organizers.
- No event-specific RSVP responses within one family invitation.
- No travel/accommodation management.
- No in-app notifications, real-time synchronized collaboration, or user-facing activity/audit log.
- No custom granular role permissions, complex task dependencies, task approvals or recurring tasks.
- No automatic WhatsApp sending; sharing opens WhatsApp with a prefilled message.
- No vendor booking, contracts, payment schedules or vendor-side transactions.
- No drag-and-drop website builder or self-hosted livestream infrastructure.
- No guest photo moderation queue: accepted guest photo uploads appear immediately.

# 3. Users, roles and authorization

### 3.1 Personas

**Admin:** A registered organizer who can plan the wedding and manage who else participates as an organizer. There may be multiple admins.

**Manager:** A registered organizer who can do all everyday wedding-management work, but cannot add/remove/promote/demote organizers.

**Guest / family representative:** An unregistered person who receives a family invitation, responds to RSVP, views the public site, and may receive album upload/view links.

### 3.2 Permission matrix

| Operation | Admin | Manager | Guest / anonymous visitor |
|---|:---:|:---:|:---:|
| View and edit wedding details | ✓ | ✓ | No |
| Create/edit/delete events and tasks | ✓ | ✓ | No |
| Create/edit/delete expense records; mark Paid | ✓ | ✓ | No |
| Create/edit/delete guest, vendor and invitation records | ✓ | ✓ | No |
| Send invitations and manual RSVP reminders | ✓ | ✓ | No |
| Choose site themes, edit site, manage photos | ✓ | ✓ | No |
| View wedding dashboard and calendar | ✓ | ✓ | No |
| Invite/remove managers | ✓ | No | No |
| Promote managers to admins; manage admin membership | ✓ | No | No |
| Open public wedding website | ✓ | ✓ | ✓ |
| Respond to their own family invitation | Link-based | Link-based | Via valid invitation link |
| Upload photos to an album | ✓ | ✓ | Via valid upload link/QR |
| View private gallery | ✓ | ✓ | Via valid gallery-view link |

**Safety rule:** A wedding must always have at least one active admin. A manager cannot make another user an organizer. An admin must not be able to use organizer-management functionality to violate the last-admin rule.

### 3.3 Account association

- A user who creates a wedding becomes its first admin.
- An invited registered user with no current wedding association can join as a manager.
- Each account can be associated with **one wedding at a time**; it does not provide a wedding switcher.
- One wedding can have multiple admins and managers.
- Invitation link holders must authenticate before becoming managers.
- If an organizer is removed, they immediately lose access to that wedding. Their prior payer attribution remains available as historical expense data (see BR-16).

# 4. Key concepts and business rules

### 4.1 Conceptual objects

This is **not a database schema**; it defines the product's major records.

| Concept | Meaning | Core relationships |
|---|---|---|
| User | Registered organizer identity | May belong to one wedding |
| Wedding | Main planning workspace | Has organizers, events, guests, expenses, etc. |
| Organizer membership | User's admin/manager status in a wedding | Belongs to user + wedding |
| Event | Haldi, Mehndi, Sangeet, ceremony, reception, etc. | Belongs to wedding; may reference venue, tasks, vendors, expenses, albums |
| Task | Work to be completed | Belongs to wedding; optional event and assignee |
| Expense | One fully paid or unpaid ledger item | Belongs to wedding; optional event/vendor; active organizer or manually named external payer if Paid |
| Vendor | Organizer-maintained service provider record | Belongs to wedding; may be associated with events and expenses |
| Guest family | Household-level invitation contact and size | Belongs to wedding; has invited events |
| Invitation | Unique link sent to one guest family | References selected wedding events and family |
| RSVP | One response for an invitation | Pending, Accepted or Declined; optional attending count |
| Wedding website | Public presentation of wedding information | Belongs to wedding; selected theme and optional YouTube embed |
| Photo album | Collection for an event | Belongs to wedding; associated with event |
| Upload/view link | Distinct capability URL | Grants restricted access to upload or view one album |

### 4.2 Confirmed business rules

| ID | Rule |
|---|---|
| BR-01 | One wedding association per registered account at a time; many organizer accounts can share a wedding. |
| BR-02 | Only two registered roles: **Admin** and **Manager**. No guest role exists. |
| BR-03 | Both roles have full operational access; admins alone manage organizer membership and role changes. |
| BR-04 | Multiple admins are allowed; the last active admin cannot be removed or demoted. |
| BR-05 | Every guest invitation belongs to one family contact and may list several selected events. |
| BR-06 | One RSVP (`Pending`, `Accepted`, `Declined`) covers **all** selected events on the invitation. |
| BR-07 | Accepted RSVPs provide an attendee count for the family; the same count is treated as expected attendance for every invited event. There is no per-event attendance refinement. |
| BR-08 | RSVP, wedding website viewing, and guest photo links do not require account creation. |
| BR-09 | Expenses are ledger entries only: `Paid` or `Unpaid`. No partial payment or payment transaction occurs. |
| BR-10 | A paid expense records **exactly one payer**: either an active same-wedding Admin/Manager chosen by an organizer (including self), or an **external person entered by name manually**. Only authenticated organizers may create or change expenses. |
| BR-11 | Unpaid expenses have no recorded payer. Expenses are visible to every active admin and manager. |
| BR-12 | Vendor agreed costs are informational; they are **not automatically included** in expense totals unless an expense entry exists. |
| BR-13 | The wedding website is public to anyone who has its URL, and its published event schedule may show all events. A family's invitation independently shows only the events selected for **that family**. Event-wise invitation does not make an event confidential on the public website. |
| BR-14 | Photo galleries are private-link-accessible; public wedding pages must not automatically expose gallery-view URLs. |
| BR-15 | Guest uploads go to a chosen event album and appear immediately after successful validation/storage, without manual approval. |
| BR-16 | When a recorded organizer payer loses membership, existing paid expenses retain the payer's name snapshot and original reference; that removed member may not be selected for a new payer assignment. External payer names remain recorded as free text. No user-facing audit log is required. |
| BR-17 | No real-time updates: users see other organizers' changes on refresh, reopening a screen, or the next ordinary fetch. |
| BR-18 | The service is free for end users, with no chargeable product plans. Operational safeguards and fair-use limits may still be needed to control infrastructure costs. |

### 4.3 Common status values

- **Task:** `To Do` → `In Progress` → `Completed` (an organizer can change status, including moving backwards).
- **Expense:** `Unpaid`, `Paid`.
- **RSVP:** `Pending`, `Accepted`, `Declined`.
- **Event:** `Upcoming`, `Ongoing`, `Completed` (see [open decision OD-04](#13-open-implementation-decisions) about deriving versus explicitly editing the display status).
- **Invitation sending:** `Not Sent`, `Sending`, `Sent`, `Failed` where supported by the chosen email service; **Sent** is not proof the guest opened the email.

# 5. User journeys

## J-01. Create a wedding and collaborate

1. A new user registers and logs in.
2. They create a wedding and provide couple, date and location information.
3. The creator becomes Admin and sees the wedding dashboard.
4. Admin invites managers by email or copies a manager joining link.
5. Recipient registers/logs in, accepts the invitation and joins as Manager, if eligible.
6. Admin may promote a manager to Admin; the wedding may contain several admins.
7. Organizers use the same dashboard and all operational tools.

**Success:** A wedding workspace is active with members who can coordinate planning without unnecessary permissions.

## J-02. Plan an event and its tasks

1. Organizer creates a Sangeet event with date, times and venue.
2. They create tasks such as “Confirm DJ,” with optional assignee and due date.
3. Organizers update each task among To Do, In Progress, and Completed.
4. Dashboard and calendar show the relevant event/task summaries.

**Success:** Tasks and schedules are easy to locate and update.

## J-03. Record expenses

1. Organizer adds “Photographer — ₹50,000” as `Unpaid`.
2. When payment happens outside MMM, an organizer changes the record to `Paid`.
3. They choose **I paid**, select another active admin/manager, **or manually enter the name of someone outside MMM** who paid; they may optionally enter a paid date.
4. The expense ledger and aggregate paid/unpaid totals update.

**Success:** MMM records the fact of payment without handling money.

## J-04. Invite a family to selected functions

1. Organizer creates “Sharma Family,” primary contact email, and number of invitees.
2. Organizer selects Sangeet and Wedding (not Haldi).
3. The platform generates one unique invitation link and sends one email.
4. The family representative opens the link without authentication and sees only Sangeet and Wedding in that invitation.
5. They accept or decline once and, if accepting, provide one attendee count.
6. The RSVP is applied to both selected events. Organizers see the change.
7. If still Pending, an organizer can manually resend a reminder email.

**Success:** The family can respond in one step, and the response is reflected in wedding and event-level invited/expected counts, without per-event RSVP.

## J-05. Publish a wedding site

1. Organizer selects one of two or three predefined themes.
2. They configure headline, couple image, welcome message and published details.
3. The site shows the basic wedding schedule, venue information and optional YouTube Live player.
4. Organizer shares its public URL.

**Success:** Visitors can view published wedding information without signing in. Private invitations and gallery data remain separate.

## J-06. Collect event photos

1. Organizer creates an album for Mehndi.
2. The platform provides a Mehndi upload URL and corresponding QR code.
3. A guest scans the QR code and uploads images without a login.
4. The service checks files and stores accepted images.
5. Photos become visible in the album immediately; organizers can remove any photo.
6. Organizer distributes a **separate** private gallery-view link for browsing/download if enabled.

**Success:** Guests can contribute photos with minimal friction, while upload and viewing permissions remain distinct.

# 6. Functional requirements

**Priority legend:** `P0` = core wedding operation; `P1` = guest-facing coordination; `P2` = experience expansion. All are part of approved product scope. Priority is build order, not feature cancellation.

## FR-01 — Authentication and profile (P0)

| ID | Requirement |
|---|---|
| FR-01.1 | Register using name, valid email and password. |
| FR-01.2 | Sign in and sign out; preserve an authorized session securely. |
| FR-01.3 | Request a password reset through email; reset links are single-use/expiring. |
| FR-01.4 | View/edit basic profile: name, email and optional phone number, with appropriate verification if email changes. |
| FR-01.5 | Prevent a registered user from joining/creating a second wedding while already associated with one. |
| FR-01.6 | Protect organizer-facing pages and APIs from unauthenticated access. |
| FR-01.7 | Keep guest RSVP, public website and guest photo pages accessible without organizer login when their specific link is valid. |

**Acceptance criteria**
- Duplicate account emails cannot register twice; invalid email/password submissions show useful errors.
- Logged-out users cannot access protected wedding data by calling APIs directly.
- Password-reset links expire and cannot be replayed after use.
- A user already in wedding A cannot join wedding B from an emailed or copied link.

## FR-02 — Wedding setup and organizers (P0)

| ID | Requirement |
|---|---|
| FR-02.1 | Create a wedding with title, couple names, date or date range and primary venue/location. |
| FR-02.2 | Allow admins/managers to edit description, dates, location, cover image and contact details. |
| FR-02.3 | Wedding creator is automatically an Admin. |
| FR-02.4 | Admin can invite a manager via email. |
| FR-02.5 | Admin can create, copy, revoke and regenerate a unique manager-joining URL. |
| FR-02.6 | Joining via email or copied link requires registering or signing in. |
| FR-02.7 | Admin sees active admins/managers and outstanding invitations. |
| FR-02.8 | Admin may remove managers, promote managers to admin and manage other admins subject to last-admin protection. |
| FR-02.9 | Manager cannot access membership management operations or corresponding API actions. |
| FR-02.10 | Removed organizers immediately lose wedding access. |

**Acceptance criteria**
- Two or more admins can coexist and operate normally.
- Both invitation methods join eligible users as **Manager** by default.
- Membership actions are rejected if initiated by a manager.
- Removing/demoting the final admin is blocked with a clear message.
- A used, expired or revoked invitation behaves consistently with its configured rule; see OD-01.

## FR-03 — Wedding dashboard (P0)

| ID | Requirement |
|---|---|
| FR-03.1 | Show wedding heading, basic details and countdown to primary wedding date. |
| FR-03.2 | Show upcoming events and links to details, with timezone-aware relative countdown labels such as `2 days to go`, `1 day to go`, `Today`, and `Completed` where appropriate. |
| FR-03.3 | Show task totals by status and pending/overdue tasks. |
| FR-03.4 | Show expense grand total, amount marked Paid, amount Unpaid. |
| FR-03.5 | Show number of guest families, expected attendees and RSVP breakdown. |
| FR-03.6 | Offer quick access to key modules. |
| FR-03.7 | Use the same operational dashboard for Admin and Manager. |

**Acceptance criteria**
- Empty states work before events/tasks/guests are added.
- All totals are calculated from this wedding's data only.
- Saving a record and refreshing the dashboard updates the visible totals.
- Manager sees the same expense and guest summaries as Admin.

## FR-04 — Event management (P0)

| ID | Requirement |
|---|---|
| FR-04.1 | Add, edit, view and **archive (soft-delete)** wedding events; preserve linked history and do not cascade-delete related expenses, invitations, tasks or photos. |
| FR-04.2 | Store event name, optional description, start/end date-time, venue/address, optional cover image. |
| FR-04.3 | Present events in chronological order and show Upcoming/Ongoing/Completed. |
| FR-04.4 | Allow optional association of tasks, expenses, vendors and photo albums with events. |
| FR-04.5 | Provide an event details page with relevant linked information. |
| FR-04.6 | Make **all non-archived** wedding events visible on the public wedding website; archiving an event hides it there. |

**Acceptance criteria**
- Name and valid start/end date-time are required; end must not precede start.
- Events can exist without vendors, expenses, tasks or albums.
- Archiving does not delete or corrupt linked tasks, expense records, invitation event selections or albums. The preservation and cancellation rules in OD-03 are confirmed.
- Lists and calendar sort consistently using a chosen wedding time zone.

**Post-invitation event cancellation rules (confirmed):** When a future event is archived, it becomes cancelled; invitations still open using the same URL and show a polite message for the cancelled event. Other invited events stay active. The family's **one RSVP status and attendee count remain unchanged**. Generate one cancellation notification email for each family whose overall invitation RSVP was `Accepted` at cancellation time; do not send cancellation emails for Pending/Declined, or when archiving events that have already started or finished. If all invited events are cancelled, the invitation remains viewable but new acceptance is disabled. Email sending is asynchronous and retry-safe; event archival is recorded immediately.

**Relative event countdown (confirmed):** Display local-calendar-day labels on public wedding event cards and organizer event lists: `N days to go`, `1 day to go`, `Today`, `Happening Today`, or `Completed`. No stored daily countdown field or recurring daily update required.

## FR-05 — Task planner (P0)

| ID | Requirement |
|---|---|
| FR-05.1 | Create, edit, delete and list tasks with title, description and optional due date. |
| FR-05.2 | Optionally associate a task with an event. |
| FR-05.3 | Optionally assign task to an active organizer from this wedding. |
| FR-05.4 | Support priority Low/Medium/High. |
| FR-05.5 | Support exactly To Do, In Progress and Completed. |
| FR-05.6 | Filter by event, assignee, status and priority. |
| FR-05.7 | Mark incomplete tasks past due as overdue. |
| FR-05.8 | Show dated tasks in the wedding calendar. |

**Acceptance criteria**
- New tasks default to `To Do`.
- Any active admin or manager may update any task.
- Completed tasks do not count as overdue.
- Tasks without assignee or due date remain valid.
- Removed organizers cannot be assigned to new tasks; existing assigned tasks are handled per OD-05.

## FR-06 — Expense tracker (P0)

| ID | Requirement |
|---|---|
| FR-06.1 | Create, edit, delete and list expenses. |
| FR-06.2 | Record title, positive INR amount, category and optional description/notes. |
| FR-06.3 | Optionally connect an expense to an event and/or vendor. |
| FR-06.4 | Each expense is `Unpaid` or `Paid`; new expenses default to `Unpaid`. |
| FR-06.5 | Setting `Paid` requires exactly one payer: **Self** (current organizer), **another active wedding Admin/Manager** selected from a list, **or an external person whose name is manually entered**. |
| FR-06.6 | Store optional paid date and a payer identity snapshot. An organizer payer has a user reference plus preserved display name; an external payer has a saved name but no user reference. |
| FR-06.7 | Setting `Unpaid` clears the active payer/paid date fields in the current record. |
| FR-06.8 | Calculate total amount, sum of Paid entries and sum of Unpaid entries; total = paid + unpaid. |
| FR-06.9 | Filter/search by category, event, vendor, payer and status. |
| FR-06.10 | No partial payment amount, schedule, transaction processing, approval or budgeting controls. |

**Acceptance criteria**
- An unpaid item has no payer and contributes only to the unpaid total.
- A paid item has exactly **one valid organizer or external payer** and contributes its **full amount** to paid totals.
- **Only an authenticated Admin/Manager can mark an expense paid.** An outsider may be recorded as a payer **by entering their name**, but does not receive an account, role, or access. An inactive or cross-wedding organizer cannot be selected as an organizer-type payer.
- Users never enter payment card or bank account details for this module.
- Removing a member does not erase the historical payer name on existing Paid entries, and editing another field does not force an old payer to be active again.
- The form offers **I paid / Select organizer / Enter another name** and rejects blank manual names. Changing a Paid expense to Unpaid clears its payer.
- Vendor agreed-cost metadata is not double counted as an expense.

**Illustrative ledger**

| Expense | Amount | Status | Paid by |
|---|---:|---|---|
| Photographer | ₹50,000 | Paid | Organizer Rahul |
| Catering | ₹2,00,000 | Unpaid | — |
| Decoration | ₹75,000 | Paid | Rajesh Sharma (manual entry) |
| **Total** | **₹3,25,000** | **₹1,25,000 Paid / ₹2,00,000 Unpaid** | |

## FR-07 — Family guest management (P0)

| ID | Requirement |
|---|---|
| FR-07.1 | Add, edit, remove and search family guest records. |
| FR-07.2 | Store family label, primary representative name/email, optional phone, invited family size and optional notes. |
| FR-07.3 | Select one or more wedding events to which the entire family is invited. |
| FR-07.4 | View each family's invitation and RSVP states. |
| FR-07.5 | Display family count, invited headcount and expected headcount aggregates. |
| FR-07.6 | Filter families by invited event and RSVP status. |

**Acceptance criteria**
- A family invited to Sangeet + Wedding is **not** listed as invited to Haldi.
- A family can exist in the guest directory before an invitation is sent.
- An invitation cannot be sent without a usable contact email and at least one invited event.
- Counts distinguish **families** from **individual expected attendees**.

## FR-08 — Digital invitations (P1)

| ID | Requirement |
|---|---|
| FR-08.1 | Generate an unpredictable, unique family invitation URL. |
| FR-08.2 | Render couple/wedding details, recipient greeting and only events selected for that family; an archived **future** event remains visible there as cancelled, with a polite notice, and does not invalidate the invitation link or reset the overall RSVP. |
| FR-08.3 | Provide 2–3 predefined invitation visual designs. |
| FR-08.4 | Send the invitation to the primary contact by email. |
| FR-08.5 | Allow manual resend; expose sent/failed result when known. |
| FR-08.6 | Offer a WhatsApp share button that opens an external share flow with a prefilled message and invitation URL. |
| FR-08.7 | Keep invitation URL distinct from organizer joining URL and gallery URLs. |

**Acceptance criteria**
- A guest opens their invitation without logging in.
- The invitation's selected-event list never accidentally includes another family’s selection.
- Guessing sequential invitation identifiers must not expose other invitations.
- `Sent` means accepted by the sending provider, not necessarily delivered/opened/read.
- A failed send can be retried by an authorized organizer without duplicating the guest family.

## FR-09 — RSVP and reminders (P1)

| ID | Requirement |
|---|---|
| FR-09.1 | One RSVP record belongs to each family invitation; archiving one future invited event does not reset an already accepted/declined response or generate a new invitation. |
| FR-09.2 | Guests choose `Accepted` or `Declined`; unanswered is `Pending`. |
| FR-09.3 | If Accepted, ask for family attendee count from 1 through invited family size. |
| FR-09.4 | If Declined, count expected attendees as zero. |
| FR-09.5 | Permit the family representative to update their RSVP using the valid invitation link. |
| FR-09.6 | A single response covers **all events on that invitation**, with no per-event Yes/No. |
| FR-09.7 | Show pending/accepted/declined counts and event-wise invited/expected totals derived from family invitations. |
| FR-09.8 | Admins/managers may manually send email reminders to Pending families; show last attempted/successful reminder state as appropriate. |

**Acceptance criteria**
- Re-submitting the same RSVP updates the existing response rather than creating an additional RSVP.
- Accepted attendee count cannot exceed the number invited.
- Accepted→Declined or Declined→Accepted updates aggregates correctly.
- A family invited to three events receives **one** response flow.
- Reminder controls are only exposed to organizers; sending failure is not recorded as successful.
- No automatic reminder scheduler or app notification is needed.

## FR-10 — Vendor management (P2)

| ID | Requirement |
|---|---|
| FR-10.1 | Add, edit, delete and search vendor records. |
| FR-10.2 | Store name, category, phone/email, address, optional contact person, optional agreed cost and notes. |
| FR-10.3 | Associate a vendor with one or more events. |
| FR-10.4 | Optionally associate expenses with vendor records. |
| FR-10.5 | Do not provision vendor accounts or payment workflows. |

**Acceptance criteria**
- One photographer may serve multiple events.
- A vendor may exist without expenses or an agreed price.
- Updating a vendor's agreed cost does **not** modify expense totals.
- A removed vendor cannot cause historical expenses to disappear silently.

## FR-11 — Nearby vendor discovery (P2)

| ID | Requirement |
|---|---|
| FR-11.1 | Search vendors by service category and wedding/location query. |
| FR-11.2 | Display available business name, address, rating and contact information, if supplied by the provider. |
| FR-11.3 | Offer a link to view a business in Google Maps. |
| FR-11.4 | Allow organizers to add relevant information as a vendor record, subject to provider terms. |
| FR-11.5 | Handle quota, permission, networking and no-result errors with a useful message. |

**Acceptance criteria**
- Search results are scoped to the specified location/category.
- Missing ratings or phone numbers do not break cards/listings.
- External API unavailability does not block manual vendor record creation.
- No booking, purchasing or payment is performed from discovery.

## FR-12 — Public wedding website and livestream (P1; livestream P2)

| ID | Requirement |
|---|---|
| FR-12.1 | Provide each wedding with a unique, public website URL. |
| FR-12.2 | Display couple names, hero/cover image, welcome message, primary date and venue. |
| FR-12.3 | Display published event details, schedule and available map links. |
| FR-12.4 | Let organizers select from 2–3 predefined themes and preview before applying. |
| FR-12.5 | Show changes to public content after they are saved/published. |
| FR-12.6 | Optionally embed a valid YouTube Live player supplied by an organizer. |
| FR-12.7 | Never expose organizer-only records, guest lists, invitation tokens or gallery view tokens as part of public site payload. |

**Acceptance criteria**
- Anyone with the public URL can load the published website without an account.
- Switching between predefined themes does not lose saved wedding content.
- The embedded stream may be absent or unavailable without breaking the wedding website.
- A private family's invitation remains separate from the public event schedule.

## FR-13 — Event albums, private gallery and guest uploads (P2)

| ID | Requirement |
|---|---|
| FR-13.1 | Organizers create/edit/delete photo albums associated with wedding events. |
| FR-13.2 | Organizers upload one or more supported images and can delete images. |
| FR-13.3 | Generate an album-specific guest upload URL and downloadable/shareable QR code. |
| FR-13.4 | Anyone with a valid guest upload URL may upload without logging in. |
| FR-13.5 | Accept JPEG, PNG, WebP, HEIC/HEIF, enforce a **20 MB per-file maximum**, verify content server-side, and apply abuse/rate limits; there is **no fixed per-wedding photo-count quota initially**. |
| FR-13.6 | Make accepted guest uploads visible in the destination album **immediately** after successful processing (no approval queue). |
| FR-13.7 | Create **separate** private gallery-view URLs; a link may be disabled/replaced by organizers. |
| FR-13.8 | Gallery viewing requires a valid viewing link or organizer authorization; upload links do not automatically confer gallery-view access. |
| FR-13.9 | Permit downloads when enabled for a gallery; organizers can remove unwanted photos. |
| FR-13.10 | Show user-friendly feedback for upload progress, success and failure. |

**Acceptance criteria**
- A QR code for Mehndi opens the Mehndi upload page—not the Sangeet album.
- Guests do not see a login gate for the upload flow.
- A successfully processed upload appears in the album with no human approval step.
- A disabled/invalid upload or view URL is rejected.
- Files that are invalid, oversized or over usage limits are rejected without exposing internal errors.
- Upload access does not allow guest deletion, album management, or unauthorized browsing.
- Any organizer can remove inappropriate photos after publication.

**Important product/safety balance:** Immediate publication is an intentional product decision. Basic automated file security, spam/rate limits, and removal controls still apply. MMM does not promise that all user-submitted photo content has been reviewed before display.

## FR-14 — Wedding calendar (P1)

| ID | Requirement |
|---|---|
| FR-14.1 | Provide a monthly calendar with previous/next navigation. |
| FR-14.2 | Display wedding events on their dates. |
| FR-14.3 | Display task due dates on the calendar. |
| FR-14.4 | Open relevant event/task details when a calendar item is selected. |
| FR-14.5 | Filter calendar display by event and/or task type. |

**Acceptance criteria**
- Events and tasks display on the correct wedding-local dates.
- Tasks with no due date are not assigned arbitrary calendar dates.
- Empty calendars display a useful empty state.
- Both organizer roles see the same calendar data.

# 7. Navigation and screen inventory

The navigation does **not** need 14 different top-level items. A simple proposed information architecture:

```text
Public / guest experience
├── Public wedding website
├── Family invitation + RSVP
├── Guest album upload (link / QR)
└── Private gallery viewing (view link)

Authenticated organizer experience
├── Dashboard
├── Events
│   └── Event details (linked tasks, expenses, vendors, albums)
├── Tasks
├── Expenses
├── Guests
│   ├── Family details + event selection
│   └── Invitations + RSVP + reminders
├── Vendors
│   ├── My vendors
│   └── Discover nearby vendors
├── Calendar
├── Photos
│   └── Event albums + upload/view links
├── Wedding website
│   └── Theme, content and YouTube Live configuration
└── Settings
    ├── Wedding details
    ├── Profile
    └── Organizers (Admin only)
```

### 7.1 Usability expectations

- Forms should have explicit required fields, validation and save/cancel feedback.
- Guest RSVP and photo upload should be optimized for mobile browsers.
- Lists should provide search and meaningful empty states.
- Avoid exposing technical terms such as “tenant”, “RBAC” or “capability token” to end users.
- Avoid creating separate admin and manager dashboards; show role-sensitive **organizer management controls only**.

# 8. Non-functional requirements

Targets are initial, proposed standards for design and testing—not claims of achieved performance.

| ID | Area | Requirement / target |
|---|---|---|
| NFR-01 | Data isolation | Every organizer data operation must be restricted to the user's associated wedding server-side, including direct/API requests. |
| NFR-02 | Authentication | Passwords securely hashed; secure sessions or tokens; appropriate password-reset controls. |
| NFR-03 | Authorization | Role checks on backend operations, not only hidden UI elements. |
| NFR-04 | Link security | Invitation, joining, photo upload and gallery-view URLs use non-guessable tokens; revocation supported where required. |
| NFR-05 | Transport security | HTTPS for production; credentials, private links and API secrets protected. |
| NFR-06 | Upload defense | File type/content inspection, size limits, request limits, storage authorization and protection from abusive automated uploads. |
| NFR-07 | Privacy | Public pages do not expose organizer account data, family email/phone lists, or private gallery URLs. |
| NFR-08 | Responsiveness | Core functionality works at mobile widths as well as desktop; controls are touch-friendly. |
| NFR-09 | Accessibility | Labels, keyboard navigation, visible focus, sufficient contrast and understandable errors for common workflows; target WCAG 2.2 AA where feasible. |
| NFR-10 | Performance | Aim for typical organizer pages to become usable within ~3 s on representative supported connections; normal CRUD requests ~2 s excluding third-party service latency. Validate during testing. |
| NFR-11 | Upload UX | Show upload progress and actionable failures; do not claim success until the service confirms storage/processing. |
| NFR-12 | Email reliability | Distinguish request accepted/sent/failed states; enable manual resend; use retry-safe operations to avoid accidental duplicates. |
| NFR-13 | Concurrency | Concurrent edits must not corrupt data; identify a simple last-write-wins or conflict-detection policy before implementation. No real-time synchronization required. |
| NFR-14 | Date/time | Consistent storage and display of event times, task deadlines and website schedules using configured wedding time zone. |
| NFR-15 | Integrity | Financial totals and RSVP summaries must remain consistent with underlying records. |
| NFR-16 | Recoverability | Maintain recoverable database and photo-storage backup/recovery procedures appropriate for a public service. |
| NFR-17 | Observability | Internal operational logs/errors for troubleshooting; these are **not** a user-facing change/audit-history module. Do not log sensitive raw tokens. |
| NFR-18 | Abuse response | Organizers can remove inappropriate photo content and disable compromised upload/view links. |
| NFR-19 | Cost control | Rate limits, storage/email/provider quotas and alerts should keep the free service sustainable; thresholds to be specified before launch. |
| NFR-20 | Legal/privacy | Publish appropriate terms/privacy notices and define data retention/deletion and abuse-contact procedures before opening registration to the public. |

# 9. Cross-module edge cases

| Scenario | Expected product behavior |
|---|---|
| User with a wedding opens another wedding's joining link | Join blocked; existing wedding data unaffected. |
| Manager tries to add another manager/admin | Server rejects operation even if API is called directly. |
| Last admin tries to leave or be removed | Operation blocked until another active admin exists. |
| Invitation email fails | Invitation record remains; show failure/retry rather than implying guest received it. |
| Guest submits RSVP twice rapidly | One response is maintained; totals are not doubled. |
| Guest changes Accepted to Declined | Expected attendees become zero across all invited events. |
| Guest enters count larger than invitation size | Validation prevents submission. |
| Invited event is removed/changed after invitations were sent | Keep data consistent; reissue or update affected invitation per a documented policy (OD-03). |
| Organizer deletes a vendor tied to an expense | Preserve expense without orphaned data or silent deletion. |
| Paid-by organizer is removed | Historical payer name remains visible; cannot select them for new paid records. External names remain unchanged. |
| Two organizers edit the same expense/task | Prevent inconsistent stored state; implement chosen concurrency behavior (OD-09). |
| Guest tries upload URL as gallery-view link | Access rejected; privileges are separate. |
| Guest shares gallery-view link publicly | Anyone possessing the valid link may view; organizer can revoke/replace it; inform organizers of the sharing model. |
| Guest uploads invalid/oversized file | Explain rejection; keep valid previously uploaded files. |
| Guest uploads offensive photo | It may appear immediately by design; organizer can remove and disable links. |
| Photo upload provider or storage fails | Show failure; do not present a successful upload. |
| Wedding website has invalid YouTube URL | Other website content still works; player shows an appropriate fallback. |
| Vendor discovery API is unavailable | Manual vendor management remains available. |
| Calendar has no dated records | Show empty state rather than placeholder data. |

# 10. Release plan and priorities

The product includes all agreed modules. The phases below are **delivery sequencing**, not a different or reduced PRD.

### Phase P0 — Wedding-management foundation

- Authentication and organizer profiles.
- Wedding creation and organizer membership (multi-admin, email/link manager invitation).
- Dashboard, events, tasks, simple paid/unpaid expense ledger.
- Family guest records with event selection.

**Exit outcome:** Organizers can collaboratively plan a wedding and track core work in a protected workspace.

### Phase P1 — Guest coordination and sharing

- Invitation links/designs and email delivery.
- Single family RSVP across selected events; manual pending-RSVP reminders.
- Public themed wedding website and calendar.

**Exit outcome:** Organizers can share wedding information and collect guest responses without guest accounts.

### Phase P2 — Vendor and photo experience

- Vendor records and nearby vendor discovery.
- Event albums, guest QR uploads, separate private gallery links.
- Optional YouTube Live embed on the wedding website.

**Exit outcome:** The platform supports vendor coordination and post-/during-event photo collection.

### Suggested delivery practice

Deliver small end-to-end slices: create wedding → create event → create task → record expense → invite guest → RSVP → publish site → upload a photo. Features should not be considered complete solely because their UI exists.

# 11. Metrics and launch readiness

### 11.1 Product measures (optional internal metrics, not an analytics UI)

- Weddings created, and percentage completing basic wedding setup.
- Weddings with at least two organizers.
- Weddings with events, tasks and expenses recorded.
- Families invited, invitations successfully submitted to email provider, RSVP response rate.
- Average time for a guest to complete RSVP.
- Albums created and successful guest photo uploads.
- Public website publication/visits, if privacy-compliant analytics are added.
- Error rates for email, photo uploads and vendor searches.

### 11.2 Launch readiness checklist

- [ ] Core role and one-wedding-per-account rules are tested on the server.
- [ ] Two or more admins can exist; the last admin cannot be removed.
- [ ] Admin-only organizer-management tools are enforced.
- [ ] Expense paid/unpaid totals reconcile correctly; a Paid payer may be **active organizer or manual external name**.
- [ ] Family invitation lists selected events; one RSVP updates all selected-event summaries.
- [ ] Reminder sending correctly targets Pending families.
- [ ] Public wedding website cannot expose private guest, organizer or gallery link data.
- [ ] Upload links and gallery-view links are distinct, unguessable and revocable.
- [ ] Photo upload validation, limits and abuse-removal workflow are in place.
- [ ] Website and guest flows work on mobile.
- [ ] Provider failures and empty states are tested.
- [ ] Backups/recovery, support contact, privacy disclosure and retention/deletion policies are defined.
- [ ] Operating-cost thresholds for storage, email and API usage are set despite free pricing.

# 12. Dependencies, constraints and risks

### 12.1 External dependencies

- **Email service:** registration verification/reset as applicable, manager invitations, guest invitations and RSVP reminders.
- **Object storage and image delivery:** covers, album photographs, photo thumbnails and upload management.
- **QR generation:** album-specific QR codes representing upload URLs.
- **Vendor discovery:** potentially Google Places; assess current terms, billing and allowable storage of returned data before selecting a provider.
- **Map linking:** Google Maps or equivalent external links.
- **YouTube embeds:** external livestream playback subject to YouTube availability and embed settings.
- **Public hosting and database services:** availability, backup, domain and operational monitoring.

### 12.2 Material risks and mitigations

| Risk | Why it matters | Minimal mitigation |
|---|---|---|
| Free photo storage grows without bound | Public traffic and large photo libraries can create significant cost | Enforce 20 MB/file; no initial count cap, but add rate limits, cost alerts and abandoned-upload cleanup. Photos remain until organizer deletion. |
| Account-free upload links are abused | Malicious uploads/spam may become immediately visible | Strong tokens, throttling, validation, rapid deletion and link revocation. |
| Guest tokens are forwarded | A valid link grants access to that invitation/gallery capability | Avoid personal identifiers in URLs, support invalidation and explain link sensitivity. |
| External email quota/delivery failure | Invitations/reminders are essential guest functions | Provider status, manual retry and usage monitoring. |
| Vendor API usage costs or terms | Discovery results may be metered or subject to storage restrictions | Control query volume; review provider documentation/terms at implementation. |
| Event schedule changes after invitations | Previously sent details can become out of date | Show current details on the same link; if a future event is cancelled, preserve overall RSVP and email affected accepted families once. |
| Concurrent organizer edits | Shared records may be overwritten | Choose a simple consistent concurrency policy; test key races. |
| Publishing every event publicly | Some family-invited events might be intended as private | Clarify that invitations control invitations, not public site visibility; provide simple publication choice only if explicitly approved (OD-07). |

# 13. Open implementation decisions

These are **narrow design decisions**, not a request to expand the product's approved feature list. The **default recommendation** is included so design can proceed without multiplying choices.

| ID | Decision required | Proposed simple default |
|---|---|---|
| OD-01 | Expiration/revocation of organizer joining links and manager email invitations | Admin can revoke/regenerate; joining links expire after a configurable period; joining always requires login. |
| OD-02 | Whether an admin can remove another admin and whether an admin can leave voluntarily | Admins may manage other admins except the last one; block final-admin removal. |
| OD-03 | Deleting an event already linked to tasks, expenses, albums or issued invitations | **Resolved:** Archive/soft-delete; keep linked data. Archiving a future event marks it cancelled on affected invitations and queues one polite email per family accepted at cancellation time. Ongoing/past archival causes no cancellation emails. |
| OD-04 | Event `Upcoming/Ongoing/Completed` calculation | Derive from event start/end times by default rather than creating another manual workflow. |
| OD-05 | Task assignments for removed organizers | Clear active assignment while retaining the task; preserve paid expense payer identity separately. |
| OD-06 | Invitation link regeneration when a family/events are edited | Existing link reflects latest selected event list; allow organizers to resend an updated email and revoke links when needed. |
| OD-07 | Which event details appear on the public wedding website | **Resolved:** Public wedding website shows all non-archived events; unlike the family invitation, it is not guest-specific. |
| OD-08 | Photo quotas, gallery-link expiry/download settings | **Resolved for upload limits:** 20 MB per file, JPEG/PNG/WebP/HEIC/HEIF, no fixed photo-count cap initially; links have organizer-configurable expiry and revocation; photo records retained until organizer deletion. Download options remain an implementation detail. |
| OD-09 | Shared-edit conflict behavior | **Resolved:** Last saved write wins, with targeted field updates and backend validation; no realtime UI. |
| OD-10 | Wedding time zone and INR formatting | Default to Asia/Kolkata and INR for India-first release; display dates consistently. |
| OD-11 | Email invitation designs vs public website themes | 2–3 simple invitation designs; 2–3 website themes may share visual tokens but need not be the same templates. |
| OD-12 | Unpublishing the wedding website and account/wedding deletion | Support a safe organizer-controlled publishing/data-removal policy consistent with privacy expectations before public launch. |

# 14. Acceptance and sign-off checklist

The PRD is ready to move to UX flows and technical design when the product owner confirms:

- [x] **Business scope:** Wedding planning, family invitations, galleries, website and vendor discovery.
- [x] **Roles:** Only Admin and Manager, multiple admins, membership operations Admin-only.
- [x] **Account constraint:** One wedding association per user at a time.
- [x] **Expense semantics:** Simple full Paid/Unpaid ledger; payer may be a registered organizer **or a manually named non-member**.
- [x] **Invitations:** Selected events per family; one RSVP across all selected events; manual reminders; cancellation notice for future archived event with RSVP preserved.
- [x] **Guest experience:** No guest accounts.
- [x] **Website:** Public URL; predefined themes; optional embedded YouTube Live.
- [x] **Photos:** Event-based albums, QR/upload links, immediate visibility after automated verification, private view links, 20 MB per file, and no initial photo-count cap.
- [x] **Commercial model:** Free, no subscriptions or in-product payment flows.
- [ ] **PRD wording reviewed and approved by product owner.**
- [ ] **Open implementation decisions resolved as needed by each module.**

### After PRD approval

1. User-flow diagrams and low-fidelity wireframes.
2. Conceptual domain model and ER diagram.
3. API resource/contracts and role-authorization matrix.
4. Architecture decisions and data/storage/email integrations.
5. Implementation backlog with vertical slices and test cases.

---

# 15. Glossary

- **Wedding workspace:** The single wedding an organizer account belongs to.
- **Organizer:** Authenticated Admin or Manager.
- **Guest family:** A household or family unit managed through one primary recipient.
- **Event-wise invitation:** Selected ceremonies included on a family invitation.
- **Single-family RSVP:** One Yes/No reply for every event listed on that invitation.
- **Expense ledger:** A list of owed/paid costs, not an electronic payment service.
- **Payer:** Exactly one person recorded as having paid an expense: a selected active wedding organizer or a manually named person outside the platform.
- **Public wedding website:** A shareable site with published wedding details and an event schedule.
- **Private gallery link:** A hard-to-guess link giving only authorized viewing access to photos.
- **Guest upload link:** A separate hard-to-guess link allowing upload into a particular event album.
- **QR code:** An image encoding the event album's upload URL for easy phone access.
- **RSVP reminder:** A manual email initiated by an organizer for a family whose RSVP is Pending.

---

*End of aligned PRD v1.1 — Make My Marriage; prior payer-only rule is superseded.*
