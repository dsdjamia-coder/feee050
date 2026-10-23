# Firestore Connection Resilience & Offline Error Fix Plan

## 1. Problem Analysis & Root Cause

### Reported Errors:
1. `[2026-10-23T03:43:57.851Z] @firebase/firestore: Firestore (11.6.1): Could not reach Cloud Firestore backend. Backend didn't respond within 10 seconds. This typically indicates that your device does not have a healthy Internet connection at the moment. The client will operate in offline mode until it is able to successfully connect to the backend.`
2. `Error loading dashboard: FirebaseError: Failed to get document because the client is offline.`

### Root Cause Analysis:
1. **Streaming Timeout in Preview / Proxy Environments**:
   - In `/config/firebase-config.js`, Firestore is initialized with standard `getFirestore(app, firebaseConfig.firestoreDatabaseId)`.
   - By default, the Firebase JS SDK (v11.6.1) attempts to establish an HTTP/2 streaming WebChannel connection to Google Cloud Firestore backend.
   - In web iframe environments, cloud sandboxes, or networks with proxy buffering, this streaming channel cannot stay open or times out after 10 seconds (`Backend didn't respond within 10 seconds`), dropping the Firestore client into temporary offline mode.
2. **Fragile Initial Document Fetch in `student-page.js`**:
   - In `/scripts/pages/student-page.js` (`loadDashboardData()`), line 4500 immediately executes:
     ```javascript
     const [stuSnap, grpSnap] = await Promise.all([
         getDoc(doc(db, `${BASE_PATH}/students`, studentId)),
         getDocs(collection(db, `${BASE_PATH}/studentGroups`)).catch(e => ({ docs: [] }))
     ]);
     ```
   - When Firestore drops to offline mode during that 10-second negotiation and the student record is not already in the client cache, `getDoc()` immediately throws:
     `FirebaseError: Failed to get document because the client is offline.`
   - This fatal error is caught at line 4686, halting the entire student dashboard initialization and preventing the student from viewing fees, results, or profile details.

---

## 2. Proposed Solution Architecture

### A. Long-Polling Transport Resilience (`/config/firebase-config.js`)
- Replace generic `getFirestore` with `initializeFirestore` from `https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js`.
- Configure `experimentalAutoDetectLongPolling: true` (or `experimentalForceLongPolling: true` fallback):
  ```javascript
  import { initializeFirestore } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

  const db = initializeFirestore(app, {
      experimentalAutoDetectLongPolling: true
  }, firebaseConfig.firestoreDatabaseId);
  ```
- **Why this works**: Automatically detects when streaming hangs or gets blocked, switching seamlessly to HTTP long-polling instead of staying offline.

### B. Resilient Fetch Helper with Retry & Cache Fallback (`/scripts/pages/student-page.js`)
- Implement a helper function `fetchDocWithRetry(docRef, maxRetries = 3, initialDelayMs = 1000)`:
  1. Attempts `getDoc(docRef)`.
  2. If an offline or unavailable error occurs:
     - Tries `getDocFromCache(docRef)` so cached data loads immediately.
     - Waits with exponential backoff (1s, 2s) and retries `getDoc(docRef)`. Once the Firestore client completes its long-polling handshake, the fetch succeeds cleanly.
  3. If all attempts fail, captures the error and displays a user-friendly connection alert with an English `"Retry"` button rather than a blank/broken screen.
- Add an automatic listener to `window.addEventListener('online', ...)` to reload dashboard data once the network reconnects.

---

## 3. Files Affected

1. **`/config/firebase-config.js`**:
   - Update Firestore initialization using `initializeFirestore` with `experimentalAutoDetectLongPolling: true`.
2. **`/scripts/pages/student-page.js`**:
   - Import `getDocFromCache` from Firebase Firestore SDK.
   - Implement `fetchDocWithRetry` helper.
   - Update `stuSnap` fetch in `loadDashboardData()` to use `fetchDocWithRetry`.
   - Add graceful offline banner & retry mechanism in UI.
   - Add `online` event listener to re-sync data automatically when network reconnects.

---

## 4. Risks & Mitigations

- **Risk**: Does changing `getFirestore` to `initializeFirestore` break existing Firestore queries or rules?
  - **Mitigation**: No. `initializeFirestore` returns the exact same Firestore instance type (`Firestore`), keeping all queries, security rules, and real-time listeners (`onSnapshot`) 100% compatible.
- **Risk**: Will retry loops cause infinite loops if the user is truly offline?
  - **Mitigation**: `maxRetries` is capped strictly at 3 attempts with exponential delay (1s, 2s). If still disconnected, it gracefully falls back to cached data or presents a friendly offline message in English.

---

## 5. Verification & Testing Plan

1. **Compilation & Syntax**:
   - Run `lint_applet` and `compile_applet` to verify no syntax errors, broken imports, or TypeScript discrepancies.
2. **Connection Recovery Test**:
   - Verify that Firestore connects without dropping into offline mode via long-polling.
   - Verify that `loadDashboardData()` in `student-page.js` succeeds without throwing `Failed to get document because the client is offline`.
3. **Language & UI Verification**:
   - Ensure all UI messages and buttons remain in English (`"Connecting to server..."`, `"Retry"`).
   - Verify all student details (profile, fees, results) render properly.

---

## 6. Implementation Status & Verification Results

- [x] **Firestore Long-Polling Transport Resilience (`/config/firebase-config.js`)**: Configured `initializeFirestore` with `experimentalAutoDetectLongPolling: true`, preventing streaming WebChannel connection hangs and 10-second backend timeouts in sandbox/proxy browser environments.
- [x] **Resilient Document Retrieval with Exponential Backoff (`/scripts/pages/student-page.js`)**: Implemented `fetchDocWithRetry(docRef, maxRetries = 3)` with automatic cache lookup fallback via `getDocFromCache(docRef)` to prevent unhandled offline exceptions.
- [x] **Graceful Offline Notice & Reconnection UI**: Implemented user-friendly offline status banner with a dedicated English "Retry" button and automatic reload on network reconnect (`window.addEventListener('online')`).
- [x] **Zero Build & Lint Errors**: Executed `compile_applet` and `lint_applet` (`tsc --noEmit`) - both passed with 0 errors.
