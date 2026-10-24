# Implementation Plan: Clean Parent Details & Upright Phone Icon

## 1. Requirement & Problem Analysis
- **Current Behavior**:
  - In Existing Groups (`#student-groups-list`), each member row rendered a badge pill (`[Father]` or `[Mother]`), which clutters the row.
  - The phone icon used `fa-phone-alt`, which in Font Awesome is tilted downward/inverted, appearing upside down to the user.
- **Target Behavior**:
  - Remove the inline `[Father]` / `[Mother]` badges from each student row.
  - Keep `Parent (Father / Mother)` clearly in the table header (`<th>`).
  - In each row cell, directly display whoever is available (Father if present, otherwise Mother) with their name and phone number.
  - Replace the tilted `fa-phone-alt` icon with standard upright `<i class="fas fa-phone text-[10px] text-gray-400"></i>`.

---

## 2. Proposed Changes

### File: `scripts/pages/admin-page.js`
- **Location**: `renderStudentGroups` (lines 6460–6550).
- **Changes**:
  1. Simplify the parent cell HTML:
     - Remove `<span ...>${parentRole}</span>` badge.
     - Display:
       ```html
       <div class="flex items-center gap-2 flex-wrap">
           <span class="font-bold text-gray-800">${escapeHtml(parentDisplayName || '-')}</span>
           ${parentDisplayPhone ? `<span class="text-gray-500 font-mono text-[11px] inline-flex items-center gap-1"><i class="fas fa-phone text-[10px] text-gray-400"></i> ${escapeHtml(parentDisplayPhone)}</span>` : ''}
       </div>
       ```
  2. Ensure header remains:
     `<th class="py-2 px-3">Parent (Father / Mother)</th>`

---

## 3. Risks & Verification
- **Visual Clarity**: Cleans visual clutter in the table, keeps phone icon properly upright.
- **Verification**: Run `npx tsc --noEmit` and `compile_applet`.
