# Implementation Plan: Modular Backup & Selective Recovery

Security & Settings പേജിലെ **Backup & Recovery** സെക്ഷൻ പരിഷ്കരിച്ച്, ഡാറ്റാ പർജ്ജ് (Data Purge) സംവിധാനത്തിന് സമാനമായി ഓരോ കാറ്റഗറികളും തരംതിരിച്ച് (Classified Categories) ആവശ്യമുള്ളവ മാത്രം തിരഞ്ഞെടുത്ത് ഇംപോർട്ട് ചെയ്യാനോ അല്ലെങ്കിൽ മുഴുവൻ ഡാറ്റയും ഒറ്റയടിക്ക് അപ്‌ലോഡ്/റിക്കവർ ചെയ്യാനോ ഉള്ള മോഡുലാർ സംവിധാനം വിജയകരമായി നടപ്പിലാക്കി.

---

### User Requirement Summary (ഉപയോക്തൃ ആവശ്യങ്ങൾ)
- **സെക്യൂരിറ്റി ആൻഡ് സെറ്റിംഗ്സ് -> Backup & Recovery**:
  - നിലവിൽ എല്ലാ ഡാറ്റയും ഒരുമിച്ച് മൊത്തമായി മാത്രം എക്സ്പോർട്ട്/ഇംപോർട്ട് ചെയ്യാനേ കഴിയുമായിരുന്നുള്ളൂ.
  - ഡാറ്റ പർജ്ജ് (Institutional Data Purge) ചെയ്യുന്നതുപോലെ കാറ്റഗറികൾ തിരിച്ച് (Classified):
    1. ഫുൾ ഡാറ്റ ഒന്നിച്ച് റിക്കവർ/അപ്‌ലോഡ് ചെയ്യാനുള്ള ഓപ്ഷൻ.
    2. ഓരോ കാറ്റഗറിയും (Students, Fees, Staff, Groups, Results, Years, Settings മുതലായവ) വ്യക്തിഗതമായി ടിക്ക് ചെയ്ത് തിരഞ്ഞെടുത്ത് സെലക്ടീവായി ഇംപോർട്ട്/റിക്കവർ ചെയ്യാനുള്ള ഓപ്ഷൻ.

---

### Implementation Details

#### 1. Classified Data Categories (വിഭാഗങ്ങളുടെ വർഗ്ഗീകരണം)
പർജ്ജ് സെക്ഷനിലുള്ളതുപോലെ തന്നെ താഴെ പറയുന്ന സ്റ്റാൻഡേർഡ് കാറ്റഗറികളായി ബാക്കപ്പ് ഡാറ്റ തരംതിരിച്ചു:
1. **Students & Admissions** (`students`)
2. **Fee Records & Payments** (`payments`)
3. **Exam Results & Schemas** (`examResults`, `resultSchemas`, `resultCenter`)
4. **Staff Directory & Accounts** (`staff` - സൂപ്പർ അഡ്മിൻ സംരക്ഷിക്കപ്പെടുന്നു)
5. **Student Groups & Batches** (`studentGroups`)
6. **Website & Directory** (`content`, `publicDirectory`)
7. **Academic Years & Classes** (`academicYears`)
8. **System Settings & App Config** (`config`)
9. **System Audit Trail Logs** (`auditLogs`)

#### 2. UI/UX in `admin.html`
- **Backup & Recovery Card**:
  - `xl:col-span-5` ആയി കാർഡ് വിപുലീകരിച്ചു.
  - "Export Full Backup" ബട്ടൺ വഴി എല്ലാ കളക്ഷനുകളും അടങ്ങിയ സമഗ്ര ബാക്കപ്പ് JSON ലഭിക്കുന്നു.
  - "Upload Backup File" വഴി JSON ഫയൽ തിരഞ്ഞെടുക്കുമ്പോൾ:
    - ഫയൽ തീയതി, സ്ഥാപനത്തിന്റെ വിവരങ്ങൾ, സ്റ്റാറ്റസ് ബാഡ്ജ് എന്നിവ കാണിക്കുന്നു.
    - **"Select All Categories (Full Restore)"** മാസ്റ്റർ ടോഗിൾ.
    - ഓരോ കാറ്റഗറിയുടെയും ഐക്കൺ, പേര്, വിവരണം, ഫയലിലുള്ള റെക്കോർഡുകളുടെ എണ്ണം (ഉദാ: `142 records`, `Available`, `Not in snapshot`) എന്നിവയടങ്ങുന്ന ഇന്ററാക്ടീവ് ലിസ്റ്റ്.
    - **"Restore Selected Categories"** ബട്ടൺ (സെലക്ട് ചെയ്ത വിഭാഗങ്ങൾ മാത്രം നിലവിലെ കളക്ഷനുകളിൽ പുനഃസ്ഥാപിക്കുന്നു; അൺസെലക്റ്റ് ചെയ്തവ സുരക്ഷിതമായി നിലനിൽക്കും).
    - **"Restore All (Full Snapshot)"** ബട്ടൺ (പൂർണ്ണമായ റീസ്റ്റോർ).
    - റീസ്റ്റോർ പ്രോഗ്രസ് ബാറും സ്റ്റാറ്റസ് ഇൻഡിക്കേറ്ററും (`#restore-progress-container`).

#### 3. JavaScript Logic in `scripts/pages/admin-page.js`
- **`BACKUP_CATEGORIES_CONFIG`**: 9 കാറ്റഗറികളുടെയും ഡിറ്റക്ഷൻ, കൗണ്ടിംഗ്, പ്രത്യേക റീസ്റ്റോർ ഫംഗ്ഷനുകൾ അടങ്ങിയ ഘടന.
- **`buildBackupPayload()`**: എല്ലാ പ്രധാന കളക്ഷനുകളും (`examResults`, `publicDirectory` അടക്കം) അടങ്ങിയ പൂർണ്ണ ബാക്കപ്പ് കയറ്റുമതി.
- **`validateBackupPayload(payload)`**: പഴയതും പുതിയതുമായ എല്ലാത്തരം ബാക്കപ്പ് ഫോർമാറ്റുകളും പരിശോധിക്കുന്ന കരുത്തുറ്റ വാലിഡേഷൻ.
- **`renderModularRestorePreview(payload)`**: കാറ്റഗറി ചെക്ക്ബോക്സുകളും കൗണ്ടുകളും റെൻഡർ ചെയ്യുന്നു.
- **`executeCategorizedRestore(payload, targetCategories)`**:
  - തിരഞ്ഞെടുത്ത കാറ്റഗറികൾ മാത്രം റീസ്റ്റോർ ചെയ്യുന്നു.
  - പുരോഗതി ശതമാനം ലൈവായി കാണിക്കുന്നു.
  - ഓഡിറ്റ് ട്രയലിൽ (`auditLogs`) രേഖപ്പെടുത്തുന്നു.
  - സൂപ്പർ അഡ്മിൻ സുരക്ഷിതത്വം ഉറപ്പാക്കുന്നു.

---

### Verification
- **Compilation Check**: `compile_applet` ബിൽഡ് പരിശോധന വിജയകരമായി പൂർത്തിയായി (Build succeeded).
- **TypeScript Check**: `npx tsc --noEmit` യാതൊരു പിഴവുകളുമില്ലാതെ പൂർത്തിയായി.
- **Linter Check**: `lint_applet` വിജയിച്ചു.
