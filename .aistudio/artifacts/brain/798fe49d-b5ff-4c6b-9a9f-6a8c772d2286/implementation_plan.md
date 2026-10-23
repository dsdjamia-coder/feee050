# Dedicated Toggle Switch & Mode-Locked Column Mapping Architecture

## 1. Overview & Requirements

Transform the Excel Edit Mode activation into a **modern iOS-style sliding Toggle Switch** and enforce strict mode locking:
1. **Real Sliding Toggle Switch**: Replace the button with a toggle switch component (track + sliding thumb + dynamic state label: `OFF` / `ON`).
2. **Strict Mode Separation**:
   - **When Switch is OFF (Default)**:
     - The preview table is in standard clean view.
     - Column mapping dropdowns are disabled and locked from accidental changes.
     - Table cells are strictly read-only.
     - The "+ Add Field" column button is hidden.
     - Guidance banner is hidden.
   - **When Switch is ON**:
     - Column header mapping dropdowns become fully interactive, allowing administrators to re-assign or skip columns.
     - In-table cell editing activates with `contenteditable="true"` and spreadsheet focus outlines.
     - The "+ Add Field" button displays.
     - The active guidance banner displays with real-time editing hints.

---

## 2. Technical Implementation Details

### A. Toggle Switch Component (`admin.html`)
Replace the push button `#excel-edit-mode-toggle-btn` with a Tailwind toggle switch:
```html
<label for="excel-edit-mode-switch" class="inline-flex items-center gap-2.5 px-3 py-1.5 bg-white border border-slate-300 hover:border-slate-400 rounded-lg cursor-pointer shadow-xs select-none transition">
    <span class="text-xs font-bold text-slate-700 flex items-center gap-1.5">
        <i class="fas fa-sliders-h text-emerald-600"></i>
        <span>Edit & Mapping Mode</span>
    </span>
    <!-- Sliding Switch -->
    <div class="relative inline-flex items-center cursor-pointer">
        <input type="checkbox" id="excel-edit-mode-switch" class="sr-only peer">
        <div class="w-9 h-5 bg-slate-300 peer-focus:outline-none rounded-full peer peer-checked:after:translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[2px] after:left-[2px] after:bg-white after:border-slate-300 after:border after:rounded-full after:h-4 after:w-4 after:transition-all peer-checked:bg-emerald-600"></div>
    </div>
    <span id="excel-edit-mode-status" class="text-[11px] font-extrabold uppercase tracking-wider text-slate-500">OFF</span>
</label>
```

### B. Mode Control in JavaScript (`scripts/pages/admin-page.js`)
1. **Switch Event**:
   - Listen to `change` event on `#excel-edit-mode-switch`.
   - Call `setExcelEditMode(switchElement.checked)`.
2. **`setExcelEditMode(enabled)` Behavior**:
   - Updates `isExcelEditMode = enabled`.
   - Updates status label text (`OFF` vs `ON`) and color (`text-slate-500` vs `text-emerald-700`).
   - Toggles guidance banner.
   - Enables or disables all `.excel-col-map-dropdown` elements (`select.disabled = !isExcelEditMode`).
   - Toggles `.excel-add-col-btn` visibility.
   - Toggles `contenteditable="true"` on `.excel-cell` elements.
3. **Table Render (`renderExcelPreviewTable`)**:
   - When rendering headers, if `!isExcelEditMode`:
     - Dropdowns render with `disabled` and clean locked styling.
     - "+ Add Field" button rendered as hidden.
   - When `isExcelEditMode`:
     - Dropdowns render enabled and prominent.
     - "+ Add Field" button rendered as visible.

---

## 3. Affected Files

1. **`admin.html`**:
   - Replace button with sliding toggle switch structure.
2. **`scripts/pages/admin-page.js`**:
   - Wire `#excel-edit-mode-switch` change event.
   - Enforce mode lock on header dropdowns, cells, and add-field buttons.

---

## 4. Risks & Mitigations

- **Risk**: User accidentally changes a column mapping while scrolling.
  - **Mitigation**: With the switch OFF by default, all dropdowns are strictly disabled and locked against accidental clicks.
- **Risk**: Toggle state desyncing after cancel/reset.
  - **Mitigation**: Reset explicitly sets `excelEditModeSwitch.checked = false` and invokes `setExcelEditMode(false)`.

---

## 5. Verification & Testing Plan

1. **Visual Switch Test**:
   - Verify that the toggle switch animates smoothly between OFF and ON.
2. **Locking Test (When Switch is OFF)**:
   - Verify that table cells cannot be edited or typed into.
   - Verify that column header dropdowns are disabled and cannot be changed.
   - Verify that "+ Add Field" is hidden.
3. **Activation Test (When Switch is ON)**:
   - Verify that column dropdowns can be changed and re-map the data instantly.
   - Verify that cells can be edited and dynamically re-validate rows.
   - Verify that "+ Add Field" is clickable.
4. **Build & Lint Verification**:
   - Run `compile_applet` and `lint_applet` to confirm zero errors.

---

## 6. Implementation Status & Verification Results

- [x] **Sliding Toggle Switch UI (`admin.html`)**: Replaced push-button with a sliding toggle switch (`#excel-edit-mode-switch`) with smooth sliding thumb and dynamic status indicator (`OFF` / `ON`).
- [x] **Strict OFF State Locking (`scripts/pages/admin-page.js`)**:
  - When switch is OFF: Preview table renders in standard clean view.
  - Column mapping dropdowns are disabled and locked (`disabled` attribute + `pointer-events-none opacity-60`).
  - Table cells are completely read-only (`removeAttribute('contenteditable')`).
  - `+ Add Field` button is hidden.
  - Subtitle shows `"Standard View (Turn ON switch above to edit cells or remap headers)"`.
- [x] **Active ON State Capabilities (`scripts/pages/admin-page.js`)**:
  - When switch is toggled ON: Column mapping dropdowns unlock and allow full re-mapping.
  - Table cells activate spreadsheet editing (`contenteditable="true"` with cursor and green focus rings).
  - `+ Add Field` button becomes visible and active.
  - Live guidance banner displays.
- [x] **Compilation & Lint Verification**: Both `compile_applet` and `lint_applet` (`tsc --noEmit`) succeeded with 0 errors.
