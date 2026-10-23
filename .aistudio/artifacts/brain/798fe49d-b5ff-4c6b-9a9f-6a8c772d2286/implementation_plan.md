# Granular Data Deletion & System Purge Architecture

A robust, enterprise-grade data purge capability integrated within the **Admin Security & Settings** page (`#security-settings-page`), allowing the Super Admin to selectively wipe specific functional modules or perform a complete institutional reset while permanently protecting Super Admin credentials and root access.

---

### User Review & Critical Decisions

> [!IMPORTANT]
> The following architectural parameters were confirmed via clarification:
> - **Authentication & Authorization**: Dual-factor security verification requiring the Super Admin's current password (re-authenticated via Firebase Auth) plus typing a strict confirmation phrase (`DELETE PERMANENTLY`).
> - **Pre-Deletion Backup**: Optional backup prompt with a single-click action to export an offline JSON snapshot before any records are permanently erased.
> - **Category Organization**: Structured "Danger Zone" in Security Settings featuring a **Select All** master toggle alongside individual granular checkboxes for each module.
> - **Super Admin Protection Invariant**: Root Super Admin credentials, authentication documents (`settings/adminAuth`), and current active administrative session are strictly immutable and protected against deletion.

---

## 1. Overview & Core Concept

### What It Does
Provides an authorized, granular deletion engine in the Admin Portal under **Security Settings / Danger Zone**. An administrator can target specific data categories (e.g., clearing only student records, wiping fee payment histories, or clearing exam results) or perform a full system reset.

### Target Audience & Persona
- **Solely Super Admin**: Standard staff, operations admins, viewers, and result operators are strictly prohibited from viewing or triggering this interface (`security.manage` permission required, further restricted to `currentAdminRole === 'super_admin'`).

### Key Value
- Enables seasonal database resets, academic turnover, and test data cleanup without manually wiping collections in the Firebase Console or risking institutional lockout.

---

## 2. User Experience & Visual Design

### UI Layout: "Danger Zone" in Security Settings
A dedicated card positioned at the bottom of `#security-settings-page` styled with a subtle crimson border (`border-rose-300 bg-rose-50/20`), distinct from routine operations:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ ⚠️  Institutional Data Purge & Reset (Danger Zone)       [Super Admin Only] │
│ Permanent data removal tool. Select categories to delete.                   │
├─────────────────────────────────────────────────────────────────────────────┤
│  [✓] Select All Categories                                                  │
│                                                                             │
│  ┌─────────────────────────────┐  ┌─────────────────────────────┐           │
│  │ [ ] Students & Enrollments  │  │ [ ] Fee Payments & History  │           │
│  │     Active, Alumni, TC logs │  │     Collection records      │           │
│  └─────────────────────────────┘  └─────────────────────────────┘           │
│  ┌─────────────────────────────┐  ┌─────────────────────────────┐           │
│  │ [ ] Academic Results        │  │ [ ] Staff & Faculty Records │           │
│  │     Marks, exams & schemas  │  │     Excludes Super Admin    │           │
│  └─────────────────────────────┘  └─────────────────────────────┘           │
│  ┌─────────────────────────────┐  ┌─────────────────────────────┐           │
│  │ [ ] Student Groups (Sibling)│  │ [ ] Website Notices & Gallery│          │
│  │     Family discount links   │  │     Public circulars & media│           │
│  └─────────────────────────────┘  └─────────────────────────────┘           │
│  ┌─────────────────────────────┐  ┌─────────────────────────────┐           │
│  │ [ ] Academic Years Ladder   │  │ [ ] System Audit Trail Logs │           │
│  │     Resets custom years     │  │     Leaves audit checkpoint │           │
│  └─────────────────────────────┘  └─────────────────────────────┘           │
│                                                                             │
│  [Download JSON Backup]                     [Purge Selected Data (Red Btn)] │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Deletion Execution Workflow

```
[Click "Purge Selected Data"]
             │
             ▼
┌──────────────────────────────────────────────┐
│ Prompt: "Download Backup Before Proceeding?" │
│ [Download Backup & Proceed]  [Skip Backup]   │
└──────────────────────────────────────────────┘
             │
             ▼
┌───────────────────────────────────────────────────────────────────────────┐
│ ⚠️  Confirm Permanent Data Deletion                                       │
│                                                                           │
│ You are about to permanently delete:                                      │
│ • Students & Enrollments (124 records)                                    │
│ • Fee Payments & History (480 records)                                    │
│                                                                           │
│ Super Admin Account: admin@institute.com (Protected - Will NOT be deleted)│
│                                                                           │
│ 1. Current Super Admin Password:                                          │
│    [••••••••••••••••••••••••]                                             │
│                                                                           │
│ 2. Type "DELETE PERMANENTLY" to confirm:                                  │
│    [DELETE PERMANENTLY      ]                                             │
│                                                                           │
│ [Cancel]                              [Confirm & Execute Permanent Delete]│
└───────────────────────────────────────────────────────────────────────────┘
             │
             ▼
┌───────────────────────────────────────────────────────────────────────────┐
│ Deletion in Progress...                                                   │
│ [████████████████░░░░░░] 60%                                              │
│ Deleting payments in batches of 400...                                    │
└───────────────────────────────────────────────────────────────────────────┘
             │
             ▼
[Audit Checkpoint Recorded] -> [In-Memory State Cleared] -> [Success Toast]
```

---

## 3. Granular Modules & Firestore Collections Mapping

| Module Name | Targeted Firestore Collections & Documents | Preservation Rules |
| :--- | :--- | :--- |
| **Students & Enrollments** | `students` collection, `academicYears/{yearId}/enrollments` subcollections | Photo data deleted with documents. |
| **Fee Payments & Collections** | `payments` collection | All transaction receipts and fee records wiped. |
| **Academic Results** | `results`, `examResults`, `resultSchemas` collections | All published and draft marks wiped. |
| **Staff & Faculty** | `staff` collection, `staffAccess` collection | **Super Admin document (`settings/adminAuth`) is completely skipped and preserved.** |
| **Student Groups** | `studentGroups` collection | Sibling linking and group fee overrides wiped. |
| **Website Content & Directory** | `settings/content` (notices, gallery), `publicDirectory` collection | Resets public circulars, event images, and directory entries. |
| **Academic Years** | `settings/academicYears` | Resets custom years; initializes clean default active year. |
| **Audit Trail Logs** | `auditLogs` collection | Wipes prior entries; creates single new checkpoint entry recording purge details. |

---

## 4. Technical Architecture & Safety Implementation

### A. Batch Deletion Primitive (Avoiding 500-Op Firestore Limits)
```javascript
// Safely chunk collection deletes into 400-op atomic batches
async function purgeCollectionInBatches(collectionPath) {
    const snap = await getDocs(collection(db, collectionPath));
    const allRefs = snap.docs.map(d => d.ref);
    const BATCH_SIZE = 400;
    for (let i = 0; i < allRefs.length; i += BATCH_SIZE) {
        const batch = writeBatch(db);
        allRefs.slice(i, i + BATCH_SIZE).forEach(ref => batch.delete(ref));
        await batch.commit();
    }
    return allRefs.length;
}
```

### B. Super Admin Password Re-Authentication
Before executing any batch write, the system re-authenticates the current Super Admin credential:
```javascript
const credential = EmailAuthProvider.credential(auth.currentUser.email, enteredPassword);
await reauthenticateWithCredential(auth.currentUser, credential);
```
If the password fails or the entered phrase does not match `DELETE PERMANENTLY`, execution terminates immediately.

### C. State Refresh & Audit Trail
Following completion:
1. An audit event `data.purge` is committed to `${BASE_PATH}/auditLogs` recording:
   - Admin email & timestamp
   - Array of purged categories
   - Total records deleted per category
2. Local arrays (`students`, `allPayments`, `studentGroups`, `allStaff`, etc.) are reset to clean zero-state.
3. Relevant dashboard metrics and views update immediately without full page reloads.

---

## 5. Testing & Verification Plan

1. **Permission Check**: Verify that users logged in with `operations_admin`, `finance_admin`, or `viewer` cannot see or interact with the Danger Zone purge card.
2. **Super Admin Preservation**: Perform a test purge selecting "Select All" and verify:
   - Super admin login remains valid in Firebase Auth.
   - `settings/adminAuth` remains untouched.
   - User does not get logged out or stranded without admin credentials.
3. **Password Validation Test**: Attempt purge with an incorrect password -> Expect error message and no database mutations.
4. **Phrase Verification Test**: Attempt purge with typos in confirmation phrase -> Verify execute button remains disabled.
5. **Partial / Granular Selection Test**: Select only "Academic Results" -> Verify students and payments remain 100% intact while result collections are cleared.
6. **Backup Test**: Click "Download Backup & Proceed" -> Verify complete `.json` file downloads before deletion begins.

---

## 6. Implementation Status & Verification Results

- [x] **Danger Zone UI in `#security-settings-page`**: Added in `/admin.html` with category checkboxes, master "Select All" toggle, live selection counter badge, and Super Admin badge.
- [x] **Pre-Purge Backup Modal**: Added `#purge-backup-modal` allowing optional JSON snapshot export before deletion.
- [x] **Dual-Factor Confirmation Modal**: Added `#purge-confirm-modal` requiring live Super Admin password re-authentication via `reauthenticateWithCredential` and typing `DELETE PERMANENTLY`.
- [x] **Real-Time Progress Modal**: Added `#purge-progress-modal` displaying transaction progress bar, percentage, live execution log stream, and final completion confirmation.
- [x] **Purge Controller & Batch Logic**: Implemented in `/scripts/pages/admin-page.js` with 400-doc transactional chunking (below Firestore's 500 limit).
- [x] **Super Admin Invariant Protection**: Guaranteed root preservation for `${BASE_PATH}/settings/${ADMIN_AUTH_DOC_ID}`, `auth.currentUser`, and Super Admin access credentials.
- [x] **Audit Checkpoint & Memory Reset**: Writes single checkpoint to `${BASE_PATH}/auditLogs` upon purge and refreshes all active view components.
- [x] **Compilation & Lint Verification**: Ran `compile_applet` and `lint_applet` (`tsc --noEmit`); both passed with 0 errors.

