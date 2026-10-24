# Implementation Plan: Cross-Month Fee Collection Window Support

## 1. Requirement & Problem Analysis
- **Current Behavior**:
  - In Admin Fee Settings (`scripts/pages/admin-page.js`), selecting `fee-collect-end-day` lower than `fee-collect-start-day` was artificially prohibited by a check:
    ```javascript
    if (endDay < startDay) { endDay = startDay; endSel.value = String(endDay); }
    ```
  - This prevented administrators from setting a recurring collection cycle that spans across month boundaries (e.g., starting on the 25th of every month and ending on the 5th of the following month).
- **Target Behavior**:
  - Allow administrators to select any start day (1 to 31) and any end day (1 to 31).
  - When `endDay < startDay` (e.g., Start: 25th, End: 5th), the system recognizes it as a cross-month collection window:
    - **Badge & Summary UI**: Displays "Every Month: 25th to 5th of next month" and explains "Regular monthly fee collection runs from the 25th of every month to the 5th of the following month."
    - **Drop-down Indicator**: In the End Day dropdown or helper note, indicate "(Next Month)" when spanning.
    - **Student Notification Logic (`scripts/pages/student-page.js`)**: Correctly calculate whether the fee is currently within the collection window or overdue across month boundaries without breaking single-month schedules.

---

## 2. Proposed Changes

### File 1: `scripts/pages/admin-page.js`
1. **Remove artificial clamp in `updateFeeScheduleLiveSummary`**:
   - Allow `endDay` to be less than `startDay`.
2. **Update Live Summary & Badge Text**:
   - Detect `isCrossMonth = endDay < startDay`.
   - If `isCrossMonth`:
     - Badge: `Every Month: {startDay} to {endDay} (Next Month)`
     - Summary text: `Regular monthly fee collection runs from the {startDay} of every month to the {endDay} of the following month.`
   - Adjust `days_before_end` calculation:
     - For cross-month, display clear notification timing (e.g., "Student fee alert notifications start {daysBefore} day(s) before the collection end date").
3. **Update `save-fee-config-btn` click handler**:
   - Remove `Math.max(startDay, ...)` constraint from `endDay`, permitting `endDay` to be saved as chosen (1 to 31).
4. **Update `populateFeeCollectionScheduleSelectors`**:
   - Populate options cleanly from 1 to 31, with 31 marked as "Last Day (28th–31st)".

### File 2: `scripts/pages/student-page.js`
1. **Update Fee Window & Overdue Calculation (`buildFeeNotificationAlerts`)**:
   - Remove `Math.max(startDay, ...)` on `rawEndDay`.
   - If `endDay < startDay`:
     - For the current month's fee, the window starts on `startDay` of this month and ends on `endDay` of next month.
     - Today is considered inside the collection window if:
       - Current date is in the billing month and `today >= startDay`, OR
       - Current date is in the immediate next month and `today <= endDay`.
     - Today is considered overdue for this fee if current date is past `endDay` in the subsequent month or beyond.
   - For same-month window (`endDay >= startDay`), preserve current reliable logic.

---

## 3. Verification & Safety
1. **Compilation / Type Check**: Run `lint_applet` (`tsc --noEmit`) and `compile_applet`.
2. **UI Verification**:
   - Test picking Start: 25th, End: 5th. Verify live badge and summary updates cleanly in English.
   - Test saving to Firestore config doc (`feeCollectionStartDay: 25, feeCollectionEndDay: 5`).
   - Test single-month (e.g. 1st to 10th) to ensure backwards compatibility.
