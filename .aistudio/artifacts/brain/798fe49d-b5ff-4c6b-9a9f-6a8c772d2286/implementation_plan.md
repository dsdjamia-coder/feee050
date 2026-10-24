# Implementation Plan: Full Student Details Modal for Groups & Concessions

## 1. Requirement & Problem Analysis
- **Current Behavior**:
  - In Existing Groups and Fee Concessions, student names are displayed as static text.
  - To view a student's full profile details (parents, Aadhaar, DOB, admission date, address, etc.), an admin has to navigate to Class Worksheet or Edit Student.
- **Target Behavior**:
  - In both **Student Groups** and **Fee Concessions**, clicking on any student's name opens a dedicated, comprehensive modal showing the student's complete information (matching the 15 fields entered in Add Student):
    1. Profile Photo
    2. Student Full Name
    3. Class
    4. ID/UID
    5. Admission Number (Adm. No.)
    6. Gender
    7. Date of Birth (DOB)
    8. Date of Admission
    9. Student Status (Active / Inactive / Alumni)
    10. Aadhaar Number
    11. Father's Name
    12. Father's Phone Number
    13. Mother's Name
    14. Mother's Phone Number
    15. Permanent Address
    + Fee & Group Info (Concession fee, Group monthly fee if applicable)

---

## 2. Proposed Changes

### File 1: `admin.html`
- Add a new full profile viewer modal: `#student-full-details-modal`
  - Responsive modal overlay with clean card styling.
  - Header: Photo preview, Student Name, Class and Adm. No. badges, Close button.
  - Details Grid: Organized sections for Academic Details, Personal Details, Parents & Contact, and Address.
  - Footer: Close button and quick navigation if needed.

### File 2: `scripts/pages/admin-page.js`
1. **Interactive Student Name Links**:
   - In `renderStudentGroups`: wrap student name with `.view-student-details-btn` and `data-student-id="${student.id}"`, styled with `hover:text-blue-600 hover:underline cursor-pointer`.
   - In `renderActiveConcessionsList` and `renderConcessionsList`: wrap student name with `.view-student-details-btn` and `data-student-id="${s.id}"`.
2. **Modal Controller Function (`openStudentDetailsModal`)**:
   - Retrieve student record from `students` array.
   - Populate photo, text fields, dates, contact numbers, and address into `#student-full-details-modal`.
   - Wire close triggers (`#student-details-modal-close`, overlay click, Escape key).
3. **Event Delegation**:
   - Handle clicks on `.view-student-details-btn` globally across the admin page.

---

## 3. Risks & Verification
- **Safety**: Non-destructive, read-only profile viewer. Does not overwrite or alter student records.
- **UI Consistency**: Follows existing Tailwind modal patterns; all UI labels and text in English.
- **Verification**: Run `npx tsc --noEmit` and `compile_applet`.
