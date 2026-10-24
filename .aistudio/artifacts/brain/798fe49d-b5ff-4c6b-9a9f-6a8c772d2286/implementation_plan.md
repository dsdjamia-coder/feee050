# Implementation Plan: Remove Redundant 'Default' Label from Monthly Fee Cards

## 1. Requirement & Problem Analysis
- **Current Behavior**:
  - In Admin Portal -> **Fee Settings** (`scripts/pages/admin-page.js` -> `renderFeeItemsManagement`), the section heading already states:
    `Default Months (2024-2025)`
  - However, every month card (January, February, ..., December) repeatedly displays a pill badge:
    `<span class="... shrink-0">Default</span>`
  - Additionally, if any fee item title contains "Default", it causes repetitive clutter.
- **Target Behavior**:
  - Remove the repeated `Default` badge from every monthly fee card.
  - Clean any redundant "Default" prefix from the month title.
  - Display only the clean month name (e.g. `June (2024-2025)` or `June`) with its calendar icon in a clean, modern card layout.

---

## 2. Proposed Changes

### File: `scripts/pages/admin-page.js`
- **Location**: `renderFeeItemsManagement` -> `renderMonthlyItemCard` (lines 8219–8227).
- **Changes**:
  1. Strip any "Default" prefix from `item.name` using `(item.name || '').replace(/^default\s*[-:—–]?\s*/i, '').trim()`.
  2. Remove the `<span ...>Default</span>` badge element from the card markup.
  3. Keep the card clean and responsive with the calendar icon and prominent month name.

---

## 3. Risks & Safety
- **Compatibility**: Does not modify database fields or change fee keys. Strictly a presentation update in the Admin UI.
- **UI Consistency**: All text remains in English.
- **Verification**: Run `npx tsc --noEmit` and `compile_applet` to ensure zero compilation or syntax errors.
