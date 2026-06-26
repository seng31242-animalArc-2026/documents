# Internal Design Review Meeting – Animal Ark SDS

**Project:** Animal Ark Veterinary Society Management System  
**Course:** SENG 31242 – System Design Project  
**Group:** Logic Lords – Group 06  
**Meeting Type:** Internal Design Review  
**Date:** 2026-06-26  
**Repository:** `documents`  
**File Path:** `documents/meetings/internal-design-review-2026-06-26.md`  
**Related Issue:** `#44 Conduct internal design review and log findings as issues`

---

## 1. Purpose of the Meeting

The purpose of this internal design review meeting was to review the Software Design Specification (SDS) of the Animal Ark Veterinary Society Management System before final submission. The team reviewed the SDS against the approved SRS, system design guideline, architecture decisions, diagrams, database model, UI wireframes, interface design, security design, and non-functional design decisions.

This review was conducted to confirm that the SDS is complete, consistent, understandable, and ready for final submission.

---

## 2. Participants

| Name | Student Number | Role |
|---|---|---|
| W. M. A. K. Wijesundara | SE/2022/033 | Team Lead |
| J. A. U. Saubhagya | SE/2022/021 | Requirements & Diagram Lead |
| D. D. N. De Silva | SE/2022/028 | Documentation Lead |
| H. M. J. K. Wijerathna | SE/2022/050 | Research & Prototype Lead |

---

## 3. Documents and Artefacts Reviewed

| Artefact | Location / Reference | Review Status |
|---|---|---|
| Final SRS | `documents/srs/` | Reviewed |
| SDS Draft / Final SDS | `documents/sds/` | Reviewed |
| Architecture Design | SDS Section 3 | Reviewed |
| Technology Stack | SDS Section 3.5 | Reviewed |
| Class Diagram and Class Specifications | SDS Section 4.1 and 4.2 | Reviewed |
| Sequence Diagrams | SDS Section 4.3 | Reviewed |
| State Machine Diagrams | SDS Section 4.4 | Reviewed |
| ER Diagram and MongoDB Collections | SDS Section 4.5 and 4.6 | Reviewed |
| UI Wireframes | SDS Section 5 | Reviewed |
| Navigation Flow | SDS Section 5.2 | Reviewed |
| External and Internal Interfaces | SDS Section 6 | Reviewed |
| Security Design | SDS Section 7 | Reviewed |
| Non-Functional Design Decisions | SDS Section 8 | Reviewed |
| Design Constraints | SDS Section 9 | Reviewed |

---

## 4. Review Checklist

| Review Area | Result | Notes |
|---|---|---|
| SRS alignment | Accepted | The SDS design modules are aligned with the approved SRS functional areas. |
| System overview | Accepted | The system overview explains the Animal Ark platform, major modules, and relationship to SRS requirements. |
| Architecture pattern | Accepted | The layered N-tier architecture with MVC structure is suitable for the project scope and team size. |
| Technology stack | Accepted | React, Node.js, Express, MongoDB, Redis, Socket.IO, and related tools are justified according to system needs. |
| Class design | Accepted | Main domain classes, attributes, methods, and relationships are included. |
| Sequence diagrams | Accepted | Major use cases are represented using sequence diagrams. |
| State machine diagrams | Accepted | Stateful workflows such as animal intake, adoption, emergency alerts, donation campaign, medicine stock, and blood donation are represented. |
| Database design | Accepted | MongoDB collections and entity relationships are documented clearly. |
| UI wireframes | Accepted | Major screens and navigation flow are documented for the main user roles. |
| Interface design | Accepted | External and internal interfaces are described, including payment, email, storage, WebSocket, REST API, and internal modules. |
| Security design | Accepted | Authentication, authorization, validation, and OWASP mitigation decisions are included. |
| Non-functional design | Accepted | Performance, scalability, maintainability, reliability, security, usability, availability, auditability, portability, and accessibility decisions are documented. |
| Formatting and terminology | Accepted | The document uses consistent project terminology and formal report style. |

---

## 5. Discussion Summary

The team reviewed whether the SDS provides enough technical detail for the next development phase. The discussion focused on whether each SRS requirement area is represented in the design document and whether the design decisions are suitable for a four-member student team.

The team agreed that the SDS provides a complete design baseline for the Animal Ark system. The layered architecture was considered appropriate because it separates the presentation layer, API layer, service layer, data layer, and external integrations while keeping implementation manageable. The team also confirmed that the selected technology stack supports role-specific dashboards, treatment tracking, emergency alerts, medicine stock management, donation handling, adoption workflows, blood donor registration, public portal access, and reporting.

The diagrams and UI descriptions were reviewed for coverage. The class diagram, sequence diagrams, state machine diagrams, ER diagram, wireframe descriptions, navigation flow, REST API summary, and security design were considered sufficient for final submission. No new unresolved correction items were identified during this internal review.

---

## 6. Decisions Made

| Decision ID | Decision | Reason |
|---|---|---|
| D-01 | Accept the layered N-tier architecture with MVC structure inside the application tier. | It supports maintainability, clear separation of concerns, and manageable development for the team. |
| D-02 | Continue with React for the frontend and Node.js/Express for the backend. | This stack supports reusable dashboards, REST API development, and fast team implementation. |
| D-03 | Use MongoDB with Mongoose as the main database design approach. | The system needs flexible records for animal cases, treatment notes, photos, donations, and adoption data. |
| D-04 | Use Redis and Socket.IO for real-time notification support. | Emergency alerts, handover updates, and stock warnings require fast dashboard updates. |
| D-05 | Keep the current SDS module separation: Animal/Case, Treatment/Handover, Emergency, Inventory, Donation, Blood Donor, Adoption, Reports, Notification, and Public Portal. | This structure maps clearly to the SRS functional areas and improves maintainability. |
| D-06 | Accept the current class diagram and class specifications as the design baseline. | The main entities, attributes, methods, and relationships are represented adequately. |
| D-07 | Accept the sequence diagrams for the major use cases. | The diagrams show the required interaction flow between users, UI, APIs, services, and database components. |
| D-08 | Accept the state machine diagrams for stateful workflows. | The diagrams represent important lifecycle changes for animals, adoption requests, emergency alerts, donation campaigns, medicine stock, and blood donation requests. |
| D-09 | Accept the MongoDB collection-based data model. | The documented collections cover users, animals, cases, treatments, medicines, donations, notifications, and adoptions. |
| D-10 | Accept the UI wireframe descriptions and navigation flow. | The UI design covers login, signup, dashboards, animal profile, intake form, emergencies, medicine stock, donation, adoption, volunteer statistics, and public portal screens. |
| D-11 | Accept the interface design section. | The SDS documents external interfaces and internal module interfaces clearly enough for implementation planning. |
| D-12 | Accept the security design strategy. | The SDS includes JWT authentication, refresh tokens, RBAC, input validation, secure file upload handling, and OWASP Top 10 mitigations. |
| D-13 | Accept the non-functional design decisions. | The design maps major quality requirements such as performance, scalability, maintainability, reliability, security, usability, availability, auditability, portability, and accessibility to concrete design decisions. |
| D-14 | No additional follow-up GitHub correction issues are required from this review. | The team did not identify any new unresolved design correction item during the review. |
| D-15 | Approve the SDS for final submission after normal PR review and merge. | The reviewed SDS is considered complete and consistent with the required design document structure. |

---

## 7. Review Outcome

The internal design review was completed successfully. The team reviewed the SDS against the approved SRS and the expected SDS structure. The reviewed design covers the required areas: system overview, architecture, technology stack, detailed design, class diagram, sequence diagrams, state machine diagrams, database model, UI wireframes, navigation flow, interface design, security design, non-functional design decisions, and design constraints.

The team agreed that the SDS is acceptable for final submission. No new unresolved correction items were found during this review, so no additional follow-up GitHub issues were created.

---

## 8. Evidence for Issue Closure

This meeting note confirms that the internal design review was conducted and documented. It can be committed to the documents repository as evidence for closing Issue `#44`.

**Evidence path:** `documents/meetings/internal-design-review-2026-06-26.md`  
**Related SDS path:** `documents/sds/`  
**Related prototype path:** `prototypes/`  
**Related issue:** `#44`

---

## 9. Action for Repository

Commit this meeting note to the `documents` repository and link the commit or pull request in Issue `#44`.

Suggested commit message:

```text
docs(meetings): add internal design review notes

Documented the internal SDS design review meeting, reviewed artefacts, checklist results, and decisions made.

Closes #44
```
