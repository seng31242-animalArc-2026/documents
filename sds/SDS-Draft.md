# Software Design Specification
## Veterinary Society Management System
**SENG 31242 - System Design Project**  
**Project:** Animal Arc  
**Group:** Logic Lords (Group 6)  
SE/2022/021 · SE/2022/028 · SE/2022/033 · SE/2022/050

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [System Overview](#2-system-overview)
3. [Architectural Design](#3-architectural-design)
4. [Detailed Design](#4-detailed-design)
5. [User Interface Design](#6-user-interface-design)
6. [Security Design](#7-security-design)
7. [Design Constraints](#8-design-constraints)

---

## 1. Introduction

### 1.1 Purpose

This Software Design Specification (SDS) describes the complete technical design of the Veterinary Society Management System (VSMS). It translates the approved Software Requirements Specification (SRS) into a concrete, implementable design ready to hand off for development in SENG 34213. The document covers system architecture, technology justification, MongoDB data models, class specifications, sequence diagrams for all major use cases, UI wireframes, security strategy, and design constraints.

### 1.2 Scope

VSMS is a web-based platform for veterinary societies to manage street animals and household pets, coordinate volunteer and supervisor doctors, process money and blood donations, track medicine stock, and manage animal adoptions. Five actor types use the system: Admin, Supervisor Doctor, Volunteer Doctor, Pet Owner, and Donor.

### 1.3 References

| Ref | Document |
|-----|----------|
| [SRS] | Software Requirement Specification v1.0 - VSMS |
| [ADR] | Architectural Decision Records – documents/architecture/decisions/ |
| [UML] | UML 2.x Notation Standard |
| [OWASP] | OWASP Top 10 Web Application Security Risks (2021) |
| [Mongoose] | Mongoose ODM Documentation v8.x |

---

## 2. System Overview

The VSMS follows a layered web application architecture consisting of a React single-page application (SPA) frontend communicating with a Node.js/Express REST API backend. Persistent data is stored in a MongoDB document-oriented database, providing flexible schema support for managing animal records, treatment histories, donations, adoption requests, and emergency alerts. An auxiliary notification service handles real-time updates and alerts through WebSocket communication. The system is deployed on a cloud-based VPS environment with Nginx acting as a reverse proxy and load-balancing gateway. The architecture prioritizes scalability, maintainability, performance, and role-based access control across all user roles, including administrators, volunteers, veterinary supervisors, donors, and adopters.

---

## 3. Architectural Design

### 3.1 Architecture Pattern

The VSMS adopts a Layered (N-Tier) Architecture with three primary tiers. Within the Application tier the codebase follows the MVC pattern — Mongoose Models define schemas and data access, Express Controllers handle HTTP routing and validation, and Views are rendered by the React SPA client-side. This pattern was selected over microservices because the team size (4–5 members) and 8-week design horizon favour a monolith with clear internal service boundaries that can be decomposed later.

| Tier | Contents |
|------|----------|
| Presentation | React SPA, Nginx reverse proxy |
| Application | Node.js / Express API, modular service layer |
| Data | MongoDB Atlas (primary store), Redis (cache + pub/sub) |

### 3.2 Architecture Diagram

![alt text](diagrams/architecture/system-architecture.png)

### 3.3 Component Diagram

![alt text](diagrams/component/Component-diagram.png)

### 3.4 Deployment Diagram

![alt text](diagrams/deployment/Deployment-diagram.png)

### 3.5 Technology Stack

#### Frontend

| Technology | Version | Justification |
|------------|---------|---------------|
| React | 18.x | Component model suits role-specific dashboards; large ecosystem |
| TailwindCSS | 3.x | Utility-first; no unused CSS in production builds |
| Axios | 1.x | Promise-based HTTP; interceptors for JWT auto-refresh |
| Socket.IO client | 4.x | Pairs with server; handles WebSocket reconnection |
| React Router | 6.x | Client-side routing with role-based route guards |

#### Backend

| Technology | Version | Justification |
|------------|---------|---------------|
| Node.js | 20 LTS | Single-language stack; non-blocking I/O suits real-time notifications |
| Express.js | 4.x | Minimal, well-documented; team familiarity |
| Mongoose ODM | 8.x | Schema validation, middleware hooks, populate for references |
| Socket.IO | 4.x | Real-time bidirectional events with Redis adapter |
| bcrypt | 5.x | Industry-standard password hashing (cost factor 12) |
| jsonwebtoken | 9.x | JWT signing and verification |
| Joi | 17.x | Request body validation schemas |
| Winston | 3.x | Structured logging with daily rotation |

#### Database

| Technology | Version | Justification |
|------------|---------|---------------|
| MongoDB | 7.x | Primary document store; flexible schema for animal/treatment records |
| Redis | 7.x | JWT blacklist; Socket.IO pub/sub; dashboard KPI caching (TTL 60s) |

#### External Services

| Service | Purpose |
|---------|---------|
| SendGrid (SMTP) | Transactional email — password reset, donation acknowledgement, alerts |
| PayHere | Sri Lanka payment gateway for money donations; webhook confirms payment |
| AWS S3 / Local FS | Animal photo upload and retrieval via presigned URLs |

#### DevOps

| Technology | Purpose |
|------------|---------|
| Docker + Compose | Four containers: nginx, app, mongo, redis — environment parity dev/prod |
| GitHub Actions | CI lint, test, build on every push to main |
| Ubuntu VPS 22.04 | Hosting; full control; team has Linux experience |

---

## 4. Detailed Design

### 4.1 Class Specifications

#### User `[users collection]`

Represents all system actors via role-based single-collection inheritance.

| Attribute | Type | Notes |
|-----------|------|-------|
| `_id` | ObjectId | Auto-generated primary key |
| `fullName` | String | Max 100 chars |
| `email` | String | Unique index; used for login and notifications |
| `passwordHash` | String | bcrypt hash cost 12; never returned in responses |
| `role` | String | Enum: `ADMIN`, `SUPERVISOR_DOCTOR`, `VOLUNTEER_DOCTOR`, `PET_OWNER` |
| `isActive` | Boolean | Soft-delete flag |
| `createdAt` | Date | Auto-set by Mongoose timestamps |
| `updatedAt` | Date | Auto-updated by Mongoose timestamps |

| Method | Returns | Description |
|--------|---------|-------------|
| `authenticate(email, pwd)` | String | Validates credentials; returns signed JWT pair |
| `resetPassword(token, newPwd)` | Boolean | Validates reset token; updates hash |
| `deactivate()` | void | Sets `isActive=false`; adds token to Redis blacklist |
| `toPublicProfile()` | Object | Returns safe subset excluding `passwordHash` |

---

#### Animal `[animals collection]`

Represents a street animal or household pet registered in the system.

| Attribute | Type | Notes |
|-----------|------|-------|
| `_id` | ObjectId | Auto generated |
| `name` | String | Nullable for unnamed street animals |
| `species` | String | e.g. Dog, Cat, Bird |
| `breed` | String | Optional |
| `estimatedAge` | Number | In months; estimated for street animals |
| `gender` | String | Enum: `MALE`, `FEMALE`, `UNKNOWN` |
| `type` | String | Enum: `STREET`, `HOUSEHOLD` |
| `status` | String | Enum: `REGISTERED`, `UNDER_TREATMENT`, `RECOVERED`, `ADOPTED`, `DECEASED` |
| `photoUrl` | String | S3 key or local path; nullable |
| `ownerId` | ObjectId | ref: User — null for street animals |
| `registeredAt` | Date | Auto-set on creation |

| Method | Returns | Description |
|--------|---------|-------------|
| `register(dto)` | Animal | Validates and saves new animal document |
| `updateStatus(status)` | void | Updates status with `updatedAt` timestamp |
| `linkOwner(userId)` | void | Associates pet owner; validates `role=PET_OWNER` |
| `getActiveCases()` | Case[] | `Case.find({animalId, status:{$ne:'CLOSED'}})` |

---

#### Case `[cases collection]`

Tracks a medical case from opening through closure. One animal may have many sequential cases.

| Attribute | Type | Notes |
|-----------|------|-------|
| `_id` | ObjectId | Auto generated |
| `animalId` | ObjectId | ref: Animal |
| `supervisorId` | ObjectId | ref: User — fixed for case lifetime |
| `symptoms` | String | Initial symptom description |
| `condition` | String | e.g. Critical, Stable, Recovering |
| `priority` | String | Enum: `P1_CRITICAL`, `P2_HIGH`, `P3_MEDIUM`, `P4_LOW` |
| `status` | String | Enum: `OPEN`, `IN_TREATMENT`, `COMPLETED`, `CLOSED` |
| `openedAt` | Date | Auto-set on creation |
| `closedAt` | Date | Nullable; set when closed |

| Method | Returns | Description |
|--------|---------|-------------|
| `open(animalId, supervisorId, dto)` | Case | Creates case; validates animal exists |
| `assignVolunteer(userId)` | void | Sets current volunteer; `supervisorId` is immutable |
| `close(notes)` | void | Sets status `CLOSED` and `closedAt=now` |

---

#### Treatment `[treatments collection]`

Records a single treatment phase administered by one volunteer doctor.

| Attribute | Type | Notes |
|-----------|------|-------|
| `_id` | ObjectId | Auto generated |
| `caseId` | ObjectId | ref: Case |
| `volunteerId` | ObjectId | ref: User |
| `phase` | Number | Sequential phase number within a case (1, 2, 3…) |
| `diagnosis` | String | Doctor's diagnosis notes |
| `treatmentNotes` | String | Procedure performed and observations |
| `handoverNotes` | String | Instructions for next volunteer; null if final phase |
| `medicinesUsed` | Array | Embedded: `[{medId: ObjectId, qty: Number}]` |
| `startedAt` | Date | When phase began |
| `completedAt` | Date | Nullable; set on phase completion |

| Method | Returns | Description |
|--------|---------|-------------|
| `record(dto)` | Treatment | Validates and saves treatment document |
| `addHandover(notes)` | void | Sets `handoverNotes`; triggers next-doctor notification |
| `deductMedicines()` | void | Calls StockService for each item in `medicinesUsed` |

---

#### Medicine `[medicines collection]`

Represents a medicine or supply tracked in inventory.

| Attribute | Type | Notes |
|-----------|------|-------|
| `_id` | ObjectId | Auto generated |
| `name` | String | Generic name, max 150 chars |
| `category` | String | e.g. Antibiotic, Analgesic, Vaccine |
| `unit` | String | e.g. tablet, ml, vial |
| `currentStock` | Number | Current quantity in stock |
| `reorderLevel` | Number | Alert threshold |
| `expiryDate` | Date | Nullable; earliest batch expiry |
| `updatedAt` | Date | Auto updated by Mongoose timestamps |

| Method | Returns | Description |
|--------|---------|-------------|
| `deduct(qty)` | void | `$inc:{currentStock:-qty}`; throws if insufficient |
| `restock(qty)` | void | `$inc:{currentStock:+qty}`; logs restock event |
| `isLowStock()` | Boolean | Returns `currentStock <= reorderLevel` |

---

#### Donation `[donations collection]`

Records a money or blood donation from a registered donor.

| Attribute | Type | Notes |
|-----------|------|-------|
| `_id` | ObjectId | Auto generated |
| `donorId` | ObjectId | ref: User |
| `type` | String | Enum: `MONEY`, `BLOOD` |
| `amount` | Number | Nullable for blood donations |
| `bloodType` | String | Nullable for money donations |
| `status` | String | Enum: `PENDING`, `VERIFIED`, `REJECTED` |
| `notes` | String | Optional donor notes |
| `submittedAt` | Date | Auto-set on submission |
| `verifiedAt` | Date | Nullable; set by admin |
| `verifiedBy` | ObjectId | ref: User (admin); nullable |

| Method | Returns | Description |
|--------|---------|-------------|
| `submit(dto)` | Donation | Validates and creates donation document |
| `verify(adminId)` | void | Sets `VERIFIED`; sends acknowledgement email |
| `reject(reason)` | void | Sets `REJECTED`; notifies donor |

---

#### Notification `[notifications collection]`

Persisted notification record, also emitted in real time via Socket.IO.

| Attribute | Type | Notes |
|-----------|------|-------|
| `_id` | ObjectId | Auto generated |
| `recipientId` | ObjectId | ref: User |
| `type` | String | Enum: `TREATMENT_UPDATE`, `EMERGENCY`, `ASSIGNMENT`, `STOCK_LOW` |
| `title` | String | Max 80 chars |
| `body` | String | Full notification body |
| `isRead` | Boolean | Default false |
| `createdAt` | Date | Auto-set |

| Method | Returns | Description |
|--------|---------|-------------|
| `create(dto)` | Notification | Saves document; emits via Socket.IO to `user:<id>` room |
| `markRead(id)` | void | Sets `isRead=true` |
| `getUnread(userId)` | Notification[] | Returns all unread for a user |

---

#### Adoption `[adoptions collection]`

Tracks the full lifecycle of an animal adoption application.

| Attribute | Type | Notes |
|-----------|------|-------|
| `_id` | ObjectId | Auto generated |
| `animalId` | ObjectId | ref: Animal |
| `applicantId` | ObjectId | ref: User |
| `status` | String | Enum: `PENDING`, `UNDER_REVIEW`, `APPROVED`, `REJECTED`, `COMPLETED` |
| `homeAssessmentDate` | Date | Nullable until scheduled |
| `assessmentNotes` | String | Admin field notes from home visit |
| `approvedBy` | ObjectId | ref: User (admin); nullable |
| `submittedAt` | Date | Auto-set on application |
| `completedAt` | Date | Nullable; set when animal handed over |

| Method | Returns | Description |
|--------|---------|-------------|
| `apply(animalId, applicantId)` | Adoption | Creates `PENDING` application |
| `scheduleVisit(date)` | void | Sets `homeAssessmentDate`; notifies applicant |
| `approve(adminId)` | void | Sets `APPROVED`; `Animal.findByIdAndUpdate status=ADOPTED` |
| `reject(reason, adminId)` | void | Sets `REJECTED`; notifies applicant with reason |

---
![alt text](diagrams/class/class-diagram.png)

### 4.2 Sequence Diagrams

*(Source draw.io files and SVG exports are in `documents/sds/diagrams/sequence/`)*

#### SD-01 Register Animal [UC-01]
![alt text](diagrams/sequence/SD%2001%20-%20Register%20Animal(UC%2001).png)

#### SD-02 Book Appointment [UC-02]
![alt text](diagrams/sequence/SD%2002%20-%20Book%20Appoiment(UC%2002).png)

#### SD-03 Create Case [UC-03]
![alt text](diagrams/sequence/SD%2003%20-%20Create%20Caset(UC%2003).png)

#### SD-04 Assign Supervisor Doctor [UC-04]
![alt text](diagrams/sequence/SD%2004%20-%20Assign%20Supervisor%20Doctor(UC%2004).png)

#### SD-05 Generate Report & Dashboard [UC-05]
![alt text](diagrams/sequence/SD%2005%20-%20Generate%20Report%20and%20Dashboard(UC%2005).png)

#### SD-06 Assign Volunteer Doctor [UC-06]
![alt text](diagrams/sequence/SD%2006%20-%20Assign%20Volunteer%20Doctor(UC%2006).png)

#### SD-07 Review & Approve Programs [UC-07]
![alt text](diagrams/sequence/SD%2007%20-%20Review%20and%20Approve%20Program(UC%2007).png)

#### SD-08 Conduct Treatment [UC-08]
![alt text](diagrams/sequence/SD%2008%20-%20Conduct%20Treatment(UC%2008).png)

#### SD-09 Update Case & Handover Notes [UC-09]
![alt text](diagrams/sequence/SD%2009%20-%20Update%20case%20&%20handover(UC%2009).png)

#### SD-10 Send Notification [UC-10]
![alt text](diagrams/sequence/SD%2010%20-%20Send%20Notification(UC%2010).png)

#### SD-11 Manage Medicine Stock [UC-11]
![alt text](diagrams/sequence/SD%2011%20-%20Manage%20Medicine%20Stock(UC%2011).png)

#### SD-12 Close Case [UC-12]
![alt text](diagrams/sequence/SD%2012%20-%20Close%20Case(UC%2012).png)

#### SD-13 Send Donation [UC-13]
![alt text](diagrams/sequence/SD%2013%20-%20Send%20Donation(UC%2013).png)

---

### 4.3 State Machine Diagrams


#### 1. Animal Intake
![alt text](diagrams/state%20machine/ST%2001%20-%20Animal%20Intake.png)

#### 2. Adoption Request
![alt text](diagrams/state%20machine/ST%2002%20-%20Adoption.png)

#### 3. Emergency Alert
![alt text](diagrams/state%20machine/ST%2003%20-%20Emergency.png)

#### 4. Donation Campaign
![alt text](diagrams/state%20machine/ST%2004%20-%20Donation.png)

#### 5. Medicine Stock
![alt text](diagrams/state%20machine/ST%2005%20-%20Medicine%20Stock.png)

#### 6. Blood Donation Request
![alt text](diagrams/state%20machine/ST%2006%20-%20Blood.png)

### 4.4 ER Diagram

![alt text](diagrams/ER/ER%20Diagram.png)

**Entities and key attributes:**

| Entity | Key Attributes |
|--------|---------------|
| Animal | `animalId`, name, species, breed, age, type, status, ownerId, dateRegistered |
| User | `userId`, name, email, password, role, phone |
| DonationCampaign | `campaignId`, title, targetAmount, currentAmount, status |
| Medicine | `medicineId`, name, quantity, expiryDate |
| EmergencyAlert | `alertId`, animalId, location, severity, status, description, createdAt |
| Treatment | `treatmentId`, animalId, volunteerId, medicine, description, treatmentDate |
| AdoptionApplication | `applicationId`, animalId, applicantId, status, applicationDate |
| Donation | `donationId`, campaignId, donorId, amount, donationDate |

**Relationships:** Animal generates EmergencyAlert; User receives/performs Treatment; User applied_for AdoptionApplication; User submits Donation; DonationCampaign makes/contains Donation/Medicine.

---

## 6. User Interface Design

### 6.1 Wireframe Descriptions

*(All wireframe source files are in `documents/sds/wireframes/`. Figma prototype: https://www.figma.com/design/3aRVb2OpZpIykqfoJUlqST/Healthcare-Dashboard-UI--Community-?node-id=0-1&t=ip9aLNfqyGt243dc-1)*

#### WF-01 Log In (All users)
- Animal Arc paw logo and "Animal Arc" heading centered at top of card
- "Veterinary Society Management" subtitle in muted text below heading
- EMAIL labelled text input field
- PASSWORD labelled masked input field with "Forgot Password?" link inline on the right
- Full-width "Sign in" primary button
- "Don't have an account? Sign up" link below button — navigates to WF-02
- Role is determined from JWT payload after sign-in; no role selector shown on this screen
- Inline error banner appears above email field on failed credentials

#### WF-02 Sign Up (All new users)
- Animal Arc paw logo and "Animal Arc" heading centered at top of card
- "Veterinary Society Management" subtitle in muted text below heading
- NAME labelled text input field
- EMAIL labelled text input field with format validation
- PASSWORD labelled masked input field
- CONFIRM PASSWORD labelled masked input field with inline match validation
- ROLE labelled dropdown or segmented selector
- Full-width "Sign up" primary button
- "Already have an account? Log in" link below button — navigates back to WF-01
- Successful sign-up redirects to Dashboard (WF-03) after account creation

#### WF-03 Dashboard (Volunteer / Supervisor / Admin)
- Top navigation bar: Animal Arc logo on the left, user avatar on the right
- Three KPI stat cards in a row: Active Rescued, My Assigned, Emergencies
- MY ACTIVE CASES section: vertical list of case cards, each showing animal initial avatar (coloured circle), animal name and case ID, condition description, and a colour-coded status badge (Treatment → amber, Stable → green, Monitored → blue)
- Blue navigation arrows on left and right sides of the case list for pagination
- Navigation flow connections visible to Animal Profile, Animals list, Sign Up, Adoption, and Emergency Alert screens

#### WF-04 Animal Profile (Volunteer / Supervisor)
- Back arrow and breadcrumb: "Case #038 Dog"
- "Active treatment" status badge and "+ Add update" button in page header
- Left column Animal details card:
  - Real animal photo displayed
  - Metadata table: Species, Breed, Est. age, Intake date, found at, Condition tag
- Right column Care team panel:
  - Supervisor section: avatar initials, doctor name, "Assigned supervisor All updates" label
  - Volunteers on case: row of three circular avatar bubbles with initials
  - Current medicines section: medicine name, dose, and frequency per active medicine
- TREATMENT TIMELINE section (full width below):
  - Vertical chronological list; each entry shows date/time, volunteer name, note text, and an entry-type badge
  - Badge types: Improving → green, Handover note → blue, Supervisor instruction → purple, Intake → gray
  - Entries with unread status shown with an open circle indicator on the left

#### WF-05 Animals List (Volunteer / Supervisor / Admin)
- Top bar: search input and Filter button on the left; "+ Register animal" primary button top-right
- Left filter sidebar: FILTER BY STATUS pills (All, Emergency, Active, Recovered, Adopted with counts); FILTER BY TYPE pills (Dog, Cat, Other with counts)
- Main content: responsive image grid of animal cards (3 columns visible)
- Each animal card: actual animal photo, colour-coded status badge overlaid on photo, animal name and case ID, short condition note, "View case" button
- Status badge colours: Emergency → red, Treatment → amber, Stable → green, Monitored → blue, Recovered → dark gray, Adoption ready → teal
- Clicking "View case" navigates to WF-04 Animal Profile
- "+ Register animal" button navigates to WF-06 Animal Intake Form

#### WF-06 Animal Intake Form (Volunteer)
- Page heading "Register new animal" with back arrow
- Two-column form layout:
  - Left column: SPECIES * dropdown, BREED text input, ESTIMATED AGE text input, SEX dropdown, FOUND LOCATION * text input, RESCUE DATE & TIME * datetime picker
  - Right column: IMAGE upload area (drag-and-drop or file picker with preview), CONDITION multi-line textarea, ASSIGNED TO volunteer picker dropdown
- "Is this an emergency?" toggle at bottom: Yes (red "alert on-call team") / No
- CANCEL button and "Submit" primary button at bottom-right
- Selecting Yes on the emergency toggle fires a real-time alert to the on-call supervisor
- Form validates all required fields (* marked) before submission

#### WF-07 Emergency Alert (Volunteer / Supervisor)
- Page heading "Emergencies" with notification bell icon
- Active emergency alert cards (red bordered):
  - Card 1: "Road accident Dog (Case #042)" — ACTIVE badge, time-ago label, description, location, admitted-by name, notified count; "I'm responding" button and "View case" button
  - Card 2: "Critical Cat (Case #039)" — ACTIVE badge, condition deteriorated note, supervisor approval required label; "Review & approve" and "View case" buttons
- RESOLVED (LAST 7 DAYS) section below active cards:
  - Each resolved item: green tick icon, case name, resolution date, responder name, response time in minutes
- Active cards refresh automatically via WebSocket; new alerts push to top of list
- "I'm responding" updates case assignment and collapses the card for other volunteers
- "Review & approve" opens an inline approval modal for supervisors

#### WF-08 Medicine Stock (Volunteer / Admin)
- Page heading "Medicine stock management" with Volunteer / Admin role badge
- "+ Add stock" primary button top-right
- Four KPI cards in a row: Total items (34, white), Low stock (3, red), Expiring soon (2, amber), Used this week (12, teal)
- Amber warning banner below KPIs: medicine name, days until expiry, tablets remaining, recommended action text
- "Search medicines…" text input and Filter button
- Medicine list rows: pill icon, medicine name, category and expiry date, color-coded stock-level badge, quantity text, "Use" button
- Thin color-coded progress bar beneath each medicine row showing stock level relative to maximum capacity
- Badge colors: Green → sufficient, Amber → low, Red → critical / out of stock
- "Use" button opens a quantity entry modal; on confirm, stock decremented and a treatment log entry created automatically

#### WF-09 Blood Donation Management (Volunteer / Admin / Public)
- Page heading "Blood Donation Management" with "+ Register donor" primary button top-right
- Search input "Search by species, blood type, location…" with Species and Location dropdown filters
- BLOOD DONOR REGISTRY list:
  - Each row: blood-drop icon, pet name with breed, age and sex, blood type, owner name and contact phone, location, Availability badge (Available → green, Busy until [month] → amber), "Contact" button
- REGISTER A BLOOD DONOR inline form section below the registry:
  - PET NAME, SPECIES, BLOOD TYPE (if known), OWNER CONTACT, LOCATION inputs
  - "Register donor" submit button
- Registered donors are immediately searchable and contactable for emergency matching

#### WF-10 Donation Management — Volunteer/Admin view
- Page heading "Donation & Campaigns" with role badge
- Three KPI cards: LKR raised this month, Active campaigns count, Pending verification (red)
- ACTIVE CAMPAIGNS section: each campaign card shows name, description, LKR raised, goal amount, green progress bar, "View donors" button and "Record donation" button
- RECENT DONATIONS list: donor name, donation amount, fund name, date, verified (green) or Pending (amber) status badge
- All data accessible to Volunteer role for recording and viewing purposes

#### WF-10b Donation Management — Admin Panel view (Admin only)
- Identical layout to WF-10 with additional admin-only controls:
  - "+ New campaign" primary button top-right
  - "Manage campaign" edit button on each campaign card
  - Pending verification amount highlighted in red KPI card
  - "Verify" and "Reject" action buttons on each recent donation row
  - Full donor list export accessible via "View donors" modal

#### WF-11 Adoption (Admin / Public)
- Page heading "Adoption" with "+ Create listing" primary button top-right
- Four KPI cards: Listed for adoption, Applications, Pending review, Adopted (total)
- AVAILABLE FOR ADOPTION left column:
  - Each listing card: paw placeholder photo, Ready badge, case ID, species and age, trait tags (e.g. Friendly / Vaccinated / Good with people), "Edit listing" button and "{n} apps" application count button
- PENDING APPLICATIONS right column:
  - Each row: applicant avatar initials, applicant full name, "Applying for Case #0XX" label, submission date, "Review" button
- "Review" button opens applicant detail modal with home type, pet experience, declaration, Approve and Reject action buttons
- Approving sets `Animal.status = ADOPTED` in database and notifies applicant

#### WF-12 MyStats / Volunteer Statistics (Volunteer / Admin)
- Page heading "Volunteer Statistics" with month selector dropdown top-right (e.g. Month: May 2026)
- Left panel: best volunteer of the month highlighted card — Name, Cases count, Hours total
- Right panel MY STATS for the selected month: four tiles — Cases, Hours, Updates, Emergencies
- LEADERBOARD section (full width below):
  - Ranked rows: rank number with color indicator, avatar initials circle, volunteer name, cases + hours + updates summary
  - Rank #1 has a trophy icon
  - Current logged-in user's row highlighted with blue border

#### WF-13 Public Portal (Donors / Adopters / Public — no login required)
- Shield/paw logo in top navigation bar
- Top navigation links: Home, Adopt, Donate, Blood donors, Stories, Contact
- Hero section: "EVERY RESCUED LIFE COUNTS" headline, society description subtitle, three primary CTA buttons: "Adopt an animal", "Make a donation", "Register blood donor"
- SUPPORT A CAMPAIGN section (right of hero): campaign name, description, LKR raised and goal, progress bar, "Donate now" button
- ANIMALS AVAILABLE FOR ADOPTION grid (below hero): each card shows actual animal photo, species and age, two trait tags, "Apply" button
- SUCCESS STORIES section: animal name and recovery headline, timeline summary, outcome (e.g. adopted)
- "Apply" navigates to adoption application form — no account required
- "Donate now" navigates to payment form with PayHere integration
- "Register blood donor" navigates to WF-09 Blood Donor Registry

### 6.2 Navigation Flow

![alt text](image.png)

---

## 7. Security Design

### 7.1 Authentication

| Mechanism | Detail |
|-----------|--------|
| Password hashing | bcrypt with cost factor 12. Never stored or logged in plain text |
| Access token | JWT signed with HS256, expires in 15 minutes |
| Refresh token | Opaque token, 7-day expiry, stored in HTTP-only cookie (not localStorage) |
| Token revocation | Revoked tokens stored in Redis with TTL matching remaining validity |
| Password reset | Time-limited (30 min) single-use token sent via email |

### 7.2 Authorization – Role Based Access Control (RBAC)

Every Express route is protected by `authMiddleware` which:
1. Verifies the JWT signature and expiry
2. Checks the token is not in the Redis blacklist
3. Attaches `req.user` with `{userId, role}`

| Route | Allowed Roles |
|-------|---------------|
| `POST /api/animals` | `ADMIN`, `PET_OWNER` |
| `POST /api/cases` | `ADMIN` |
| `PATCH /cases/:id/assign` | `SUPERVISOR_DOCTOR` |
| `POST /api/treatments` | `VOLUNTEER_DOCTOR` |
| `PATCH /cases/:id/close` | `SUPERVISOR_DOCTOR`, `ADMIN` |
| `POST /api/donations` | `DONOR` |
| `PATCH /donations/:id/verify` | `ADMIN` |
| `POST /api/adoptions` | `PET_OWNER` |
| `GET /api/reports/dashboard` | `ADMIN` |

### 7.3 Input Validation

- All request bodies validated with **Joi schemas** before any controller logic runs
- **Mongoose schema validation** as a second layer (type, required, enum, minlength)
- File uploads: MIME type whitelist (`image/jpeg`, `image/png`), size limit 5 MB, filename sanitised
- No raw query string interpolation — all MongoDB queries use parameterised Mongoose calls

### 7.4 OWASP Top 10 Mitigations

| Risk | Mitigation |
|------|------------|
| A01 Broken Access Control | RBAC on every route; ownership check in service layer |
| A02 Cryptographic Failures | HTTPS (TLS 1.2+) at Nginx; bcrypt for passwords; secrets in env vars only |
| A03 Injection | Mongoose parameterised queries; Joi validation; no eval or concatenated queries |
| A07 Auth Failures | Rate limit: 5 login attempts per 15 min per IP; lockout at 10 failures |
| A09 Logging Failures | Winston logs all auth events, role changes, case closures, stock deductions |

---

## 8. Design Constraints

### 8.1 Browser Support

| Browser | Min Version | Notes |
|---------|-------------|-------|
| Chrome | 110+ | Primary development target |
| Firefox | 110+ | Full support |
| Safari | 16+ | WebSocket and CSS grid verified |
| Edge | 110+ | Chromium-based; same as Chrome |
| Mobile Safari / Chrome | iOS 15+ / Android 10+ | Responsive layout required |

### 8.2 Database Constraints

| Constraint | Detail |
|------------|--------|
| Index strategy | Indexed: `users.email` (unique), `cases.animalId`, `treatments.caseId`, `notifications.recipientId` |
| Soft delete | Users deactivated via `isActive=false`; no cascade deletes to preserve case history |
| Session atomicity | Treatment recording + medicine stock deduction wrapped in MongoDB session to prevent partial writes |
| Supervisor immutability | `cases.supervisorId` set once and never updated — enforced in service layer |
| Collection size | No hard cap; monitored via MongoDB Atlas metrics |

### 8.3 Hosting Assumptions

| Assumption | Detail |
|------------|--------|
| VPS spec | Minimum 2 vCPU, 4 GB RAM, 40 GB SSD (Ubuntu 22.04) |
| Docker | Docker Engine 24+, Docker Compose v2 |
| Concurrent users | Designed for 200 concurrent users; Redis adapter readies Socket.IO for horizontal scaling |
| Backup | Daily MongoDB Atlas automated backup; weekly manual export to S3 |
| Uptime target | 99% availability; Docker health checks with auto-restart |
| SSL | Let's Encrypt certificate via Certbot; auto-renews every 90 days |
| Secrets management | All secrets in `.env` file; never committed to Git; example in `.env.example` |

### 8.4 API Constraints

| Constraint | Detail |
|------------|--------|
| Pagination | All list endpoints paginated (default 20 items, max 100) |
| Rate limiting | 100 requests/min per IP globally; 5 login attempts/15 min per IP |
| Payload size | Request body limit 1 MB (JSON); 5 MB multipart for file uploads |
| CORS | Restricted to production domain + `localhost:3000` in development |