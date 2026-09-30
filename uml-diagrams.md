# UML Diagrams — Student Attendance Monitoring System (SAMS)

**Group:** BSIT 3A · **Members:** Angela Rose R. Gramatica, Honey Lee Sibugan

These diagrams are written in Mermaid, so GitHub draws them automatically when you open this file.

## 1. Use Case Diagram

Who uses the system and what they can do. (Mermaid has no built-in use case diagram, so this uses a flowchart: rounded boxes are users, ovals are use cases.)

```mermaid
flowchart LR
  I("Instructor")
  S("Student")
  C("Coordinator")
  A("Administrator")

  UC1(["Sign in / sign out"])
  UC2(["Record attendance"])
  UC3(["Edit attendance record"])
  UC4(["View own attendance"])
  UC5(["View attendance history"])
  UC6(["View flagged students"])
  UC7(["Manage users, classes, enrollment"])
  UC8(["Set absence threshold"])
  UC9(["Receive absence warning"])

  I --> UC1
  S --> UC1
  C --> UC1
  A --> UC1
  I --> UC2
  I --> UC3
  I --> UC5
  S --> UC4
  S --> UC9
  C --> UC5
  C --> UC6
  C --> UC9
  A --> UC7
  A --> UC8
```

## 2. Class Diagram

The main classes, their key fields, and how they relate. Matches §2.2 and §5 of the design specification.

```mermaid
classDiagram
  class Student {
    +studentId
    +name
    +email
    +status
  }
  class Instructor {
    +instructorId
    +name
    +email
  }
  class Coordinator {
    +coordinatorId
    +name
    +email
  }
  class Administrator {
    +adminId
    +name
    +email
  }
  class ClassSection {
    +classId
    +title
    +instructorId
    +schedule
  }
  class Enrollment {
    +studentId
    +classId
  }
  class AttendanceRecord {
    +studentId
    +classId
    +date
    +status
    +recordedBy
    +lastEditedBy
    +lastEditedAt
  }
  class Notification {
    +notificationId
    +studentId
    +type
    +sentAt
  }
  class Setting {
    +absenceThreshold
  }

  Instructor "1" --> "0..*" ClassSection : teaches
  Student "1" --> "0..*" Enrollment : has
  ClassSection "1" --> "0..*" Enrollment : has
  Student "1" --> "0..*" AttendanceRecord : has
  ClassSection "1" --> "0..*" AttendanceRecord : has
  Instructor "1" --> "0..*" AttendanceRecord : records
  Student "1" --> "0..*" Notification : receives
  Coordinator "1" --> "0..*" Notification : receives
  Administrator ..> Setting : sets
```

## 3. Sequence Diagram — Recording Attendance

The same workflow as §4 of the design specification.

```mermaid
sequenceDiagram
  actor Instructor
  participant UI as Attendance Recording screen
  participant Auth as AuthService
  participant Att as AttendanceService
  participant DB as Database
  participant Notif as NotificationService

  Instructor->>UI: Sign in
  UI->>Auth: Verify credentials and role
  Auth-->>UI: Signed in as Instructor
  Instructor->>UI: Select class and date
  UI->>Att: Request roster
  Att->>DB: Load enrolled students
  DB-->>Att: Roster
  Att-->>UI: Roster
  Instructor->>UI: Mark each student Present or Absent, then Submit
  UI->>Att: Send completed record set
  Att->>DB: Check for existing record (same class and date)
  alt Duplicate found (REQ-10)
    Att-->>UI: Reject, show error
  else No duplicate
    Att->>DB: Save records (recordedBy, timestamp)
    Att->>Att: Recalculate absence counts
    opt Absences reach threshold (REQ-06)
      Att->>Notif: Trigger warning
      Notif->>DB: Log notification
      Notif-->>Att: Sent to student and coordinator
    end
    Att-->>UI: Success confirmation
  end
```
