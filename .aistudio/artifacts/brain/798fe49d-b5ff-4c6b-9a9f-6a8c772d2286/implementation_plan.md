# Implementation Plan: Parent Details (Father/Mother) in Existing Groups Table

## 1. Requirement & Problem Analysis
- **Current Behavior**:
  - In Existing Groups (`#student-groups-list`), the member student table currently displays a column labeled `Guardian / Phone`.
  - It resolves parent info using `student.fatherName || student.parentPhone || student.phone || '-'`.
  - The actual student database schema stores parent details as Father (`father`, `mobile` / `fatherMobile`) and Mother (`mother`, `motherMobile`).
- **Target Behavior**:
  - Display Father's name and phone number by default.
  - If Father's info (name & phone) is not present, automatically fall back to Mother's name and phone number.
  - Display a clear role badge (`Father` or `Mother`) alongside the name and phone number.
  - Column header updated to `Parent (Father / Mother)`.
  - Update group search filter to support matching Father's name/phone and Mother's name/phone.

---

## 2. Proposed Changes

### File: `scripts/pages/admin-page.js`
- **Location**: `renderStudentGroups` (lines 6395–6515).
- **Changes**:
  1. For each student row in `memberRows`:
     - Extract `fatherName = student.father || student.fatherName || student.profile?.father || ''`.
     - Extract `fatherPhone = student.mobile || student.fatherMobile || student.parentPhone || student.phone || student.profile?.fatherMobile || ''`.
     - Extract `motherName = student.mother || student.motherName || student.profile?.mother || ''`.
     - Extract `motherPhone = student.motherMobile || student.profile?.motherMobile || ''`.
     - Priority logic: If `fatherName || fatherPhone`, show Father's name & phone. Else if `motherName || motherPhone`, show Mother's name & phone.
     - Display with formatted name and phone number (e.g. `[Father] Ahmed · 9876543210`).
  2. Change table header column from `Guardian / Phone` to `Parent (Father / Mother)`.
  3. Include Father and Mother names and phone numbers in the `sTerm` search matcher so searching either parent quickly finds the group.

---

## 3. Risks & Verification
- **Compatibility**: Read-only display adjustment using existing student fields (`father`, `mobile`, `mother`, `motherMobile`).
- **UI Integrity**: Keeps clean layout with responsive text wrapping and font-mono styling for phone numbers. All UI text in English.
- **Verification**: Run `npx tsc --noEmit` and `compile_applet`.
