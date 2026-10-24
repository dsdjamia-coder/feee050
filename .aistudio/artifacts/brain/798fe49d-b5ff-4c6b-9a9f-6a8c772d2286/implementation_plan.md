# Implementation Plan: Excel Bulk Student Upload Refinement

Refine the bulk student Excel/CSV upload system in the Admin Portal to match the manual "Add Student" entry fields, prevent duplicate column mappings with high-visibility red indicators, apply vibrant high-contrast status colors, empower the "+ Add Field" button, and display blank cells cleanly without dashed placeholders.

## User Requirements & Problem Analysis

1. **Exact 13 Manual Form Fields (No Duplicates)**:
   - Previously, `EXCEL_SCHEMA_FIELDS` contained 14 fields with separate `ID / UID` and `Samastha Reg ID`. In the Madrasa/school database and manual entry form, UID and Samastha Registration ID refer to the same identifier (`uid`).
   - `Class` is already selected prior to upload via the target class dropdown, and `Photo` cannot be uploaded via spreadsheet.
   - The mapping schema must match the exact 13 manual entry input boxes from `admin.html`:
     1. Student Name * (`name`)
     2. ID/UID (`uid`)
     3. Father's Name (`father`)
     4. Mother's Name (`mother`)
     5. Gender * (`gender`)
     6. Admission Number (Adm. No.) (`adm`)
     7. Date of Birth (`dob`)
     8. Date of Admission (`admDate`)
     9. Student Status (`status`)
     10. Aadhaar Number (`adhaar`)
     11. Father's Phone Number (`mobile`)
     12. Mother's Phone Number (`motherMobile`)
     13. Permanent Address (`address`)
     + Option for `-- Skip / Ignore --` (`none`).

2. **Duplicate Mapping Prevention with Solid Red Warning**:
   - If an administrator selects a schema field that is already assigned to another column, both columns will be flagged with solid vibrant red styling (`bg-red-600 text-white font-bold`, red border, and a prominent badge `DUPLICATE MAPPING`).
   - A warning banner will explain the collision, and the upload buttons ("Upload All" / "Upload Valid Only") will be safely disabled until duplicate assignments are resolved.

3. **Vibrant & Bold Color Palette (കടും നിറങ്ങൾ)**:
   - Replace pale/washed-out pastel backgrounds (`bg-rose-50`, `bg-amber-50`, `bg-sky-50`) with rich, high-contrast colors:
     - **Fatal Errors / Missing Name**: Vivid red highlight (`bg-red-100 border-2 border-red-500 text-red-950 font-bold`, with badge `bg-red-600 text-white font-black`).
     - **Missing / Skipped Optional Fields**: Deep vibrant amber/yellow highlight (`bg-amber-100 border border-amber-400 text-amber-950 font-semibold`, badge `bg-amber-600 text-white font-bold`).
     - **Existing Student in Class**: Bold sky/blue highlight (`bg-blue-100 border border-blue-400 text-blue-950 font-semibold`, badge `bg-blue-600 text-white font-bold`).
     - **Valid Student**: Crisp emerald badge `bg-emerald-600 text-white font-bold`.

4. **Interactive "+ Add Field" Feature**:
   - When the "+ Add Field" button is clicked in Edit Mode, it will open a clean dropdown/modal menu showing available unmapped fields.
   - Selecting a field immediately inserts a new column into the spreadsheet, binds it to the chosen schema field, and allows the admin to type values directly into the table cells.
   - If all 13 fields are already mapped, it politely informs the user that all schema fields are present.

5. **Clean Blank Cell Rendering**:
   - Remove `--` placeholders from empty cells. Blank cells will render as completely clean empty strings `""`, allowing clean viewing and natural cell text editing.

---

## Proposed Changes

### 1. `scripts/pages/admin-page.js`
- **Schema & Aliases**:
  - Update `EXCEL_SCHEMA_FIELDS` to exactly 13 fields matching the manual form input labels word-for-word.
  - Update `aliasMap` to merge `samastha`, `samasthaid`, `regno`, `uid`, `id` into the single target key `uid`.
  - Exclude `samastha` as a standalone field in validation, payload building, and CSV template.
- **Duplicate Mapping Detection**:
  - Add `getDuplicateMappings()` helper to inspect `excelColumns` and count occurrences of mapped fields.
  - Highlight colliding column headers in solid red (`bg-red-600 text-white border-red-700`).
  - Block submission when duplicates exist and display a prominent warning banner.
- **Vibrant Styling**:
  - Upgrade table row and cell CSS classes from pale pastels to bold high-contrast colors.
  - Update row status badges to rich solid colors.
- **"+ Add Field" Interactive Menu**:
  - Provide a modal/popup menu with unmapped fields upon clicking "+ Add Field".
  - Insert chosen column into `excelColumns` and initialize in all `excelWorkingRows`.
- **Blank Cells**:
  - Render empty string `""` instead of `<span ...>--</span>`.

### 2. `admin.html`
- Ensure bulk upload CSV template download reflects the updated 13 fields.

---

## Verification Plan

1. **Compilation & Linting**:
   - Run `compile_applet` and `lint_applet` (`tsc --noEmit`) to verify 0 syntax or type errors.
2. **Schema Verification**:
   - Confirm dropdown contains exactly the 13 fields from the manual form plus `-- Skip / Ignore --`.
3. **Duplicate Detection Test**:
   - Map two columns to "Student Name *" or "Father's Name".
   - Confirm both headers turn solid red with duplicate warning tag and upload buttons are disabled.
   - Change one column to `-- Skip / Ignore --` or another field; verify red alert clears and upload re-enables.
4. **Visual Contrast Test**:
   - Confirm fatal error cells/rows are solid high-contrast red.
   - Confirm blank optional cells are deep amber.
   - Confirm blank cells have no `--` marks.
5. **Add Field Test**:
   - Click "+ Add Field" in edit mode; confirm unmapped fields dropdown appears and newly added column works seamlessly.
