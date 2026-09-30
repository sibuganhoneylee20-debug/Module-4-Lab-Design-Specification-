# Requirements Document — Student Attendance Monitoring System (SAMS)

**Group:** BSIT 3A · **Members:** Angela Rose R. Gramatica, Honey Lee Sibugan
**Version:** 1.0 · **Date:** September 30, 2026

## 1. Purpose and Scope

SAMS lets instructors record attendance per class session, and lets students, coordinators, and administrators view or manage that data according to their role. It notifies the right people when a student's absences reach a set threshold.

**In scope:** sign-in, role-based access, attendance recording and correction, attendance viewing, absence-threshold notifications, and administration of users, classes, and enrollment.

**Out of scope:** biometric or QR-code check-in, integration with the institution's grading system, and managing its own identity system (institution-issued credentials are used).

## 2. Users

| Role | Description |
|---|---|
| Instructor | Teaches one or more classes and records their attendance |
| Student | Enrolled in classes; views own attendance |
| Coordinator | Monitors attendance across classes and follows up on flagged students |
| Administrator | Maintains users, classes, enrollment, and system settings |

## 3. Functional Requirements

| ID | Requirement | Actor | Priority | Acceptance criteria |
|---|---|---|---|---|
| REQ-01 | Users sign in with institution-issued credentials | All | High | A valid user reaches the Dashboard; invalid credentials show an error that does not say which field was wrong |
| REQ-02 | Access is restricted by role | All | High | Each role sees only the screens and actions allowed for it |
| REQ-03 | Administrators create, edit, and deactivate user accounts | Administrator | High | A new account can sign in; a deactivated account cannot |
| REQ-04 | Administrators manage classes and student enrollment | Administrator | High | Enrolled students appear on the class roster |
| REQ-05 | Administrators set the absence threshold | Administrator | Medium | Threshold accepts only a positive whole number and applies to new notifications |
| REQ-06 | The system notifies the student and coordinator when a student's absences reach the threshold | System | High | A notification is created once when the threshold is reached, and is not lost if sending fails |
| REQ-07 | Instructors can view the roster of the classes they teach | Instructor | High | The roster lists all enrolled students; other instructors' classes are not available |
| REQ-08 | Instructors record each student as present or absent for a class session | Instructor | High | Each student in the roster has exactly one status saved for that class and date |
| REQ-09 | A record cannot be submitted while any student is unmarked | Instructor | High | Submit is rejected and unmarked students are indicated |
| REQ-10 | A duplicate record for the same class and date cannot be submitted | Instructor | High | A second submission for the same class and date is rejected and nothing is saved |
| REQ-11 | Students can view their own attendance and absence count | Student | High | A student sees only their own records |
| REQ-12 | Edits to a record store who edited it and when | Instructor | Medium | lastEditedBy and lastEditedAt are updated on every edit |
| REQ-13 | Coordinators can view attendance across classes and the list of flagged students | Coordinator | Medium | The flagged list shows students at or above the threshold |
| REQ-14 | Attendance history can be filtered by class, student, and date range | Instructor, Coordinator, Student | Medium | Results match all selected filters; an empty result shows a message |
| REQ-15 | Attendance records are never deleted; corrections are edits with an audit trail | System | High | No delete action exists for attendance records |
| REQ-16 | Sessions end on sign-out or after a period of inactivity | All | Medium | After sign-out or timeout, protected screens require signing in again |

## 4. Constraints and Assumptions

1. Authentication uses institution-issued credentials.
2. Each class has at most one session per day, identified by class and date.
3. Class rosters are maintained by administrators.
4. The absence threshold is one system-wide value.
5. Users access SAMS through a modern web browser.

## 5. Approval

| Name | Role | Date |
|---|---|---|
| Angela Rose R. Gramatica | Group member | |
| Honey Lee Sibugan | Group member | |
| Mitzi Clyde T. Conol | Instructor | |
