# Design Specification — Student Attendance Monitoring System (SAMS)

**Course:** IT314 Software Engineering · **Module 4 Laboratory Activity**
**Group:** BSIT 3A · **Members:** Angela Rose R. Gramatica, Honey Lee Sibugan
**Version:** Draft 1 · **Date:** September 30, 2026

**Source artifacts**
- Requirements Document: [requirements.md](requirements.md)
- UML diagrams (use case, class, sequence): [uml-diagrams.md](uml-diagrams.md)
- Figma wireframes & prototype: https://www.figma.com/design/iTHFqFv4a05lcOhUcXsiJX

---

## 1. Overview

SAMS lets instructors record attendance per class session, and lets students, coordinators, and administrators view or manage that data according to their role. It replaces paper and spreadsheet tracking with one shared record and automatically notifies the relevant people when a student's absences cross a set threshold.

**Users and what they do**

| Role | Main goals |
|---|---|
| Instructor | Record attendance for a class session; correct their own past records |
| Student | View own attendance and absence count; receive absence warnings |
| Coordinator | View attendance across classes; review students flagged for excessive absences |
| Administrator | Manage user accounts, classes, and enrollment; set the absence threshold |

---

## 2. Architectural Design

SAMS uses a **three-layer architecture**: presentation, application, and data. Screens never touch data directly; every action goes through a service. This keeps validation rules (such as duplicate checks) in one place instead of being repeated on every screen.

### 2.1 Layers and components

| Layer | Component | Responsibility |
|---|---|---|
| Presentation | Login screen | Collect credentials, show auth errors |
| Presentation | Dashboard | Role-based landing page with links to allowed actions |
| Presentation | Attendance Recording screen | Mark and submit attendance for one class and date |
| Presentation | Attendance History screen | View past records (by student, class, or date range) |
| Presentation | Flagged Students screen | Coordinator view of students over the threshold |
| Presentation | Admin Management screens | Manage users, classes, enrollment, threshold |
| Application | AuthService | Verify credentials, start/end sessions, enforce role permissions |
| Application | AttendanceService | Load rosters, validate and save records, edit records, compute absence counts |
| Application | NotificationService | Send warnings when the absence threshold is crossed |
| Application | AdminService | CRUD for users, classes, enrollment, and settings |
| Data | Relational database | Persist all entities below |

### 2.2 Mapping to the UML class diagram

| UML class | Layer | Used by | Notes |
|---|---|---|---|
| Student | Data | AttendanceService, AdminService | Attributes match the class diagram in [uml-diagrams.md](uml-diagrams.md) |
| Instructor | Data | AuthService, AttendanceService | |
| Coordinator | Data | AuthService, NotificationService | |
| Administrator | Data | AuthService, AdminService | |
| ClassSection ("Class") | Data | AttendanceService, AdminService | Named ClassSection in the class diagram |
| Enrollment | Data | AttendanceService, AdminService | Links a student to a class |
| AttendanceRecord | Data | AttendanceService | Core transactional entity |
| Notification | Data | NotificationService | Created when the absence threshold is reached |
| Setting | Data | AdminService, AttendanceService | Holds the absence threshold |

### 2.3 Component connections

```
[Presentation screens] --> [AuthService | AttendanceService | NotificationService | AdminService] --> [Relational database]
                                         AttendanceService --> NotificationService (on threshold crossing)
```

**Rationale:** separating screens, logic, and storage means the UI can change (for example, a mobile version) without rewriting business rules, and business rules can be tested without a UI.

---

## 3. Interface Design

One entry per wireframe screen. Each answers: **purpose, contents, states, and what must be true before submission.**

### 3.1 Login screen
- **Purpose:** Let a registered user sign in.
- **Elements:** Username/ID field, password field, Sign In button, error message area.
- **States:** *Default* (empty form); *Loading* (button disabled, spinner while AuthService verifies); *Error* (invalid credentials, or account inactive); *Success* (redirect to Dashboard).
- **Validation:** Both fields required. Sign In is disabled until both are filled. Error messages must not reveal which field was wrong.
- **Requirement(s):** REQ-01, REQ-02, REQ-16

### 3.2 Dashboard
- **Purpose:** Give each role a starting point with only the actions they are allowed to use.
- **Elements:** Greeting and role, navigation menu, role-specific summary (e.g., instructor: today's classes; student: absence count; coordinator: number of flagged students).
- **States:** *Empty* (no classes assigned or no records yet, with a friendly message); *Loading* (skeleton placeholders); *Error* (summary failed to load, with Retry).
- **Validation:** None (read-only). Menu items are hidden for roles without permission.
- **Requirement(s):** REQ-02, REQ-11

### 3.3 Attendance Recording screen
- **Purpose:** Let an instructor mark each student present or absent for one class on one date.
- **Elements:** Class selector, date selector, student roster list, present/absent toggle per row, Submit button.
- **States:**
  - *Empty roster* — class has no enrolled students; Submit disabled and message shown.
  - *Loading* — roster is being fetched.
  - *Unsaved changes* — at least one toggle changed; warn before leaving the screen.
  - *Error* — save failed; keep the instructor's marks on screen and offer Retry.
  - *Success* — confirmation shown; record locked for this class and date (edits go through the edit flow).
- **Validation:**
  - Cannot submit if any student is unmarked.
  - Cannot submit a duplicate record for the same class and date (REQ-10).
  - Instructor can only record for classes they teach. REQ-07
- **Requirement(s):** REQ-07, REQ-08, REQ-09, REQ-10

### 3.4 Attendance History screen
- **Purpose:** View past attendance and, for permitted roles, correct records.
- **Elements:** Filters (class, student, date range), results table (student, class, date, status, recorded by, last edited by/at), Edit action on eligible rows.
- **States:** *Empty* (no records match the filters); *Loading*; *Error* (with Retry); *Success* (results shown).
- **Validation:** Students see only their own records. Editing records the editor and timestamp (REQ-12). [Define who may edit — see Open Questions.]
- **Requirement(s):** REQ-11, REQ-12, REQ-14

### 3.5 Flagged Students screen (Coordinator)
- **Purpose:** Show students whose absences meet or exceed the threshold.
- **Elements:** List of flagged students with absence count, class, and date flagged.
- **States:** *Empty* (no students flagged); *Loading*; *Error*.
- **Validation:** Read-only. Visible to coordinators only.
- **Requirement(s):** REQ-06, REQ-13

### 3.6 Admin Management screens
- **Purpose:** Create and maintain users, classes, enrollment, and the absence threshold.
- **Elements:** List views with Add/Edit/Deactivate actions; form for each entity; threshold setting field.
- **States:** *Empty* (no entries yet); *Loading*; *Error*; *Success* (saved confirmation).
- **Validation:** Required fields must be filled; IDs must be unique; threshold must be a positive whole number; a user with attendance history is deactivated, never deleted.
- **Requirement(s):** REQ-03, REQ-04, REQ-05

> **Checklist item:** every screen in the Figma file must appear above. Add a row here for any screen not yet listed (e.g., password reset, profile).

---

## 4. Interaction Design

### Key workflow: Recording attendance

**Actors/components:** Instructor, Attendance Recording screen (UI), AuthService, AttendanceService, Database, NotificationService.

| Step | Action | Handled by |
|---|---|---|
| 1 | Instructor signs in; credentials verified and role confirmed as Instructor | Login screen → AuthService |
| 2 | Instructor opens Attendance Recording screen from the Dashboard | UI |
| 3 | Instructor selects class and date; UI requests the roster | UI → AttendanceService |
| 4 | Service checks the instructor teaches this class and loads enrolled students | AttendanceService → Database |
| 5 | Roster displayed; instructor marks each student present or absent | UI |
| 6 | Instructor presses Submit; UI checks that no student is unmarked | UI |
| 7 | UI sends the completed record set to the service | UI → AttendanceService |
| 8 | Service checks for an existing record for the same class and date (REQ-10) | AttendanceService → Database |
| 9a | **Duplicate found:** service rejects the request; UI shows an error and no data is saved | AttendanceService → UI |
| 9b | **No duplicate:** service saves the records with `recordedBy` and timestamp | AttendanceService → Database |
| 10 | Service recalculates each absent student's total absences | AttendanceService |
| 11 | If a student's total reaches the threshold, service triggers a warning (REQ-06) | AttendanceService → NotificationService |
| 12 | NotificationService notifies the student and coordinator and logs the notification | NotificationService → Database |
| 13 | Success confirmation shown to the instructor | AttendanceService → UI |

**Failure paths:** save failure at step 9b keeps the on-screen marks and offers Retry; a notification failure at step 12 must not undo the saved attendance (it is logged and retried). [Confirm your group's decision.]

*Reference: Sequence diagram in [uml-diagrams.md](uml-diagrams.md); Figma prototype flow https://www.figma.com/design/iTHFqFv4a05lcOhUcXsiJX.*

---

## 5. Data Considerations

Key entities and fields relevant to the design. This is not a full schema.

| Entity | Key fields | Rules |
|---|---|---|
| **Student** | studentId, name, email, status (active/inactive) | studentId unique |
| **Instructor** | instructorId, name, email | instructorId unique |
| **Coordinator** | coordinatorId, name, email | |
| **Administrator** | adminId, name, email | |
| **Class** | classId, title, instructorId, schedule | Has one instructor; has many enrolled students |
| **Enrollment** | studentId, classId | A student may be enrolled in many classes |
| **AttendanceRecord** | studentId, classId, date, status, recordedBy, lastEditedBy, lastEditedAt | Unique by studentId + classId + date (REQ-10); status is Present or Absent; edits record editor and time (REQ-12) |
| **Notification** | notificationId, studentId, type, sentAt | Created when the absence threshold is reached (REQ-06) |
| **Setting** | absenceThreshold | Positive whole number; set by Administrator |

**Data rules**
- An attendance record cannot exist for a student not enrolled in that class.
- Records are never physically deleted; corrections are edits with an audit trail (REQ-15).
- Date cannot be in the future. [Confirm.]

---

## 6. Assumptions & Constraints

**Assumptions**
1. Authentication uses institution-issued credentials; SAMS does not manage its own identity system (carried over from the Appendix A requirements set).
2. Each class session is identified by class and date; there is at most one session per class per day.
3. Class rosters are maintained by administrators and are correct at the time attendance is taken.
4. Users have a modern browser and an internet connection.
5. Absence threshold is a single system-wide value, not set per class.

**Constraints / out of scope**
- No biometric or QR-code check-in.
- No integration with the institution's grading system.
- No full database design in this document (unless the instructor requests it).

**Open questions**
1. Who may edit a submitted record: only the original instructor, any instructor of that class, or coordinators too? Is there a time limit?
2. Does the threshold count absences per class or across all classes?
3. Are "excused" or "late" statuses needed, or only Present/Absent?
4. How are notifications delivered: in-app, email, or both?
5. What happens if a student is enrolled after attendance for that date was already recorded?
6. Should the Present/Absent default to a preset value, or stay unmarked to force a choice?

---

## Requirements Referenced

| ID | Requirement |
|---|---|
| REQ-01 | Users sign in with institution-issued credentials |
| REQ-02 | Access is restricted by role (Instructor, Student, Coordinator, Administrator) |
| REQ-03 | Administrators create, edit, and deactivate user accounts |
| REQ-04 | Administrators manage classes and student enrollment |
| REQ-05 | Administrators set the absence threshold |
| REQ-06 | The system notifies the student and coordinator when a student's absences reach the threshold |
| REQ-07 | Instructors can view the roster of the classes they teach |
| REQ-08 | Instructors record each student as present or absent for a class session |
| REQ-09 | A record cannot be submitted while any student is unmarked |
| REQ-10 | A duplicate record for the same class and date cannot be submitted |
| REQ-11 | Students can view their own attendance and absence count |
| REQ-12 | Edits to a record store who edited it and when |
| REQ-13 | Coordinators can view attendance across classes and the list of flagged students |
| REQ-14 | Attendance history can be filtered by class, student, and date range |
| REQ-15 | Attendance records are never deleted; corrections are edits with an audit trail |
| REQ-16 | Sessions end on sign-out or after a period of inactivity |

## Requirements Traceability

| Requirement | Addressed in |
|---|---|
| REQ-01, REQ-16 | §3.1 Login, §4 step 1 |
| REQ-02 | §2.1 AuthService, §3.2 Dashboard |
| REQ-03, REQ-04, REQ-05 | §3.6 Admin Management, §5 Setting |
| REQ-06 | §3.5, §4 steps 10–12, §5 Notification/Setting |
| REQ-07 | §3.3, §4 steps 3–4 |
| REQ-08, REQ-09 | §3.3, §4 steps 5–6 |
| REQ-10 | §3.3, §4 step 8, §5 AttendanceRecord |
| REQ-11 | §3.2, §3.4 |
| REQ-12 | §3.4, §5 AttendanceRecord |
| REQ-13 | §3.5 |
| REQ-14 | §3.4 |
| REQ-15 | §5 Data rules |

---

## Peer Review Checklist *(completed by the reviewing group)*

**Reviewed by:** [BSIT 3A] · **Date:** [9/30/2026]

| Check | Yes / Partial / No | Reviewer comments |
|---|---|---|
| Does the architecture clearly map to the approved UML/class structure? | | |
| Does every wireframed screen have a corresponding interface design entry? | | |
| Are validation rules and screen states (empty/loading/error) specified? | | |
| Is the key interaction/workflow traceable step by step? | | |
| Are assumptions and open questions clearly stated rather than left implicit? | | |

## Revision Log

| Feedback received | Change made |
|---|---|
| [Reviewer comment] | [What you changed and where] |

## Reflection *(Ask the Class)*
1. Where did writing the specification force a decision the wireframe alone didn't require?
2. What feedback from your peer reviewers changed something in your design?
3. If you handed this specification to a developer who had never seen your project, what would they still have to guess?
