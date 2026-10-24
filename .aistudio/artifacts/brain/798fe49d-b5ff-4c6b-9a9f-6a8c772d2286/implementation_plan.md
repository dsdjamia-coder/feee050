# Implementation Plan: Table Layout & Search/Filter for Student Groups & Concessions

## 1. Requirement Overview
1. **Student Groups Search & Filter**:
   - In "Existing Groups" (`#manage-groups-page`), add a real-time search box (by student name, adm no) and a class filter dropdown.
   - Show dynamic matching count (e.g. "Showing X of Y groups").
2. **Student Groups Table Format (Grouped Table)**:
   - Replace the current badge/card view with a clear, professional grouped table structure.
   - Each group will be presented with:
     - Group Header: Group number, total member count, monthly fee badge (₹/month), and Edit/Delete buttons.
     - Members Table: Columns for `#`, `Student Name`, `Class`, `Admission No`, and `Phone/Guardian`.
3. **Active Concessions Table Format**:
   - Replace the card grid in Active Concessions (`#active-concessions-list`) with a clean, responsive table.
   - Table columns: `#`, `Student Name & Guardian`, `Class`, `Adm No`, `Special Monthly Fee (₹)`, and `Actions` (Edit & Remove).

---

## 2. Proposed Changes

### File 1: `admin.html`
1. **Student Groups Section (`#manage-groups-page`)**:
   - Above `#student-groups-list`, add a Search & Filter toolbar:
     - Search input: `#grp-search-input` (Search student name, adm no...)
     - Class dropdown: `#grp-class-filter` (All Classes filter)
     - Status counter: `#grp-count-badge`
   - Adjust `#student-groups-list` layout to host responsive grouped tables.
2. **Fee Concessions Section (`#fee-concessions-page`)**:
   - Replace grid wrapper around `#active-concessions-list` with a responsive table container:
     - `table` with sticky-friendly styled headers: `#`, `Student Name`, `Class`, `Adm No`, `Special Monthly Fee`, `Actions`.
     - `tbody id="active-concessions-table-body"` for rows.

### File 2: `scripts/pages/admin-page.js`
1. **Student Groups Rendering (`renderStudentGroups`)**:
   - Populate `#grp-class-filter` options using active classes.
   - Read `#grp-search-input` and `#grp-class-filter`.
   - Filter groups where any member student matches the search term or class.
   - Render each group as a structured block containing a clean tabular view of its member students.
   - Wire input/change event listeners for `#grp-search-input` and `#grp-class-filter`.
2. **Active Concessions Rendering (`renderActiveConcessionsList`)**:
   - Change rendering from card blocks to `<tr>` table rows.
   - Include row indices (`#`), formatted names, badges for class and admission number, bold currency badge for concession amount, and action buttons (`Edit`, `Remove`).
   - Preserve existing event delegations (`.concession-open-btn`, `.concession-delete-btn`).

---

## 3. Risks & Verification
- **Functional Integrity**: Edit and delete actions for both groups and concessions must retain exact event delegation and ID mapping.
- **Responsive Layout**: Tables will have `overflow-x-auto` to ensure seamless scrolling on mobile and tablet devices.
- **Language Requirements**: All UI elements, headers, and buttons in English only; user communication in Malayalam.
- **Verification**: Run `npx tsc --noEmit` and `compile_applet`.
