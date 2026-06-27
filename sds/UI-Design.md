## 5.3 UX Decisions and Rationale

The Animal Ark user interface is designed to support fast, role-specific access to the most important veterinary society workflows. Since volunteers, supervisor doctors, administrators, donors, adopters, and public visitors use the system for different purposes, the interface uses role-based dashboards and protected navigation paths. This reduces screen clutter and ensures that each user group sees only the functions relevant to their responsibilities.

### Role-Based Navigation

After login, the system redirects users to a dashboard based on their authenticated role. Volunteer doctors are directed to animal intake, treatment timeline, handover notes, medicine stock usage, and emergency alert functions. Supervisor doctors are directed to case review, emergency approval, active cases, and treatment oversight. Administrators are directed to donation verification, adoption management, medicine stock management, user management, and reporting. Public users can access the public portal, adoption listings, donation campaigns, and blood donor registration without using restricted internal workflows.

This decision improves usability because users do not need to search through unrelated functions. It also supports security because restricted workflows are not shown to unauthorised users.

### Dashboard-Centred Workflow

The dashboard acts as the main entry point after authentication. It displays important case counts, active rescued animals, assigned cases, and emergency status. From the dashboard, users can move quickly to animal profiles, animal registration, emergency alerts, adoption, donation, and reporting screens.

This design supports quick decision-making for volunteers and supervisor doctors because active case information is visible immediately after login.

### Treatment Timeline as a Critical Workflow

The treatment timeline screen is one of the most important user interface flows in the Animal Ark system. Animal care is handled by rotating volunteer doctors, so treatment history, handover notes, supervisor instructions, and medicine usage must be visible in chronological order.

The Animal Profile screen includes a full treatment timeline showing date, time, volunteer name, treatment note, handover note, and entry type. Colour-coded badges are used to distinguish improving updates, handover notes, supervisor instructions, and intake records. This helps the next volunteer understand the animal’s current condition quickly without reading unrelated records.

The treatment timeline design reduces communication gaps between volunteer shifts and supports safer continuity of care.

### Emergency-First Design

Emergency alerts are designed with high visual priority using clear alert cards, status badges, responder actions, and direct links to the related animal case. New emergency alerts are pushed to users in real time through the notification flow.

This design decision is important because emergency animal cases require quick response from volunteers and supervisor doctors.

### Consistent Visual Cues

The UI uses consistent badges, cards, filters, buttons, and status colours across major screens. Animal status, treatment status, adoption readiness, emergency state, and medicine stock level are shown using labels and colour-coded indicators.

This improves learnability because users can recognise the same interaction patterns across different modules.

### Public Portal Simplicity

The public portal is separated from internal dashboards. Public visitors can view adoption animals, donation campaigns, success stories, and blood donor registration options without accessing internal case management features.

This keeps the public experience simple while protecting sensitive veterinary and society management data.

### Prototype and Wireframe References

The UI design is based on the approved prototype and wireframe exports. The major referenced screens include Login, Sign Up, Dashboard, Animal Profile, Animals List, Animal Intake Form, Emergency Alert, Medicine Stock, Blood Donation Management, Donation Management, Adoption, Volunteer Statistics, and Public Portal.

Prototype source and exports are maintained in the prototype repository and referenced from the SDS UI Design section.
