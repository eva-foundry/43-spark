# 43-spark — Master Acceptance Criteria

> **Standard**: WCAG 2.1 Level AA · Government of Canada Standard on Web Accessibility  
> **Scope**: EVA-JP v1.2 Frontend Rebuild — all 8 screens + 20 shared components  
> **How to use**: After each Spark iteration or Codespace session, run through the relevant rows. Mark ✅ when verified. A pull request may not merge until all Must-Have rows in the affected phase are ✅.

---

## AC Index

| ID | Category | Criterion | Priority | Phase |
|----|----------|-----------|----------|-------|
| AC-01 | Code Quality | Zero imports from `@fluentui/react` (v8); zero raw Fluent primitive usage where an `EvaXxx` wrapper exists | M | 1 |
| AC-02 | Accessibility | axe-core scan: 0 critical, 0 serious violations per screen | M | 3–4 |
| AC-03 | i18n | All user-visible strings use `t()` — no hardcoded literals in JSX | M | 1 |
| AC-04 | i18n | Language toggle switches all text with no page reload | M | 1 |
| AC-05 | i18n | `document.documentElement.lang` reflects active locale at all times | M | 1 |
| AC-06 | Accessibility | `document.title` updates on every route change | M | 1 |
| AC-07 | Accessibility | Keyboard focus lands on first `<h2>` inside `#mainContent` after navigation | M | 1 |
| AC-08 | Accessibility | All dialogs trap focus; close on Escape; restore focus to trigger element | M | 2 |
| AC-09 | Accessibility | All icon-only buttons have explicit `aria-label` | M | 2 |
| AC-10 | Accessibility | `aria-live` regions announce async status changes (loading, success, error) | M | 2 |
| AC-11 | Accessibility | High-contrast mode: all UI components visible and operable | M | 3–4 |
| AC-12 | Functionality | Chat history drawer, FileStatus table, and preview dialog paginate correctly | M | 3 |
| AC-13 | Functionality | Upload flow completes and status is reflected in the FileStatus table | M | 3 |
| AC-14 | Functionality | Translation flow completes and the download link is keyboard-accessible | M | 3 |
| AC-15 | Accessibility | URL scraper preview dialog returns focus to the Preview button on close | M | 3 |
| AC-16 | Code Quality | All components covered by `@eva/ui` use the `EvaXxx` wrapper; `GCThemeProvider` wraps the app root | M | 1 |

---

## Detailed Verification Procedures

### AC-01 — No Fluent UI v8 imports + enforce @eva/ui wrappers

**Automated check — v8 guard**:
```bash
grep -r "from \"@fluentui/react\"" src/
grep -r "from '@fluentui/react'" src/
```
Both commands must return no output.

**Automated check — @eva/ui wrapper enforcement**:
```bash
# Each of these raw primitives must NOT appear when the EvaXxx wrapper covers it
grep -rn "from '@fluentui/react-components'" src/ | \
  grep -E "Button[^G]|Input[^P]|Select[^O]|Dialog[^T]|Drawer[^H]|Badge[^P]|DataGrid|Spinner[^P]|TabList|Textarea|Toast[^C]" | \
  grep -v "//"
```
Result must be empty. Allowed raw-Fluent imports: `FluentProvider` (only if not using `GCThemeProvider`), `makeStyles`, `mergeClasses`, `tokens`, `useId`, layout primitives, and any component not yet wrapped in `@eva/ui`.

**Pass condition**: both grep checks return empty.  
**Fail action**: replace raw Fluent primitives with their `EvaXxx` equivalents from `@eva/ui`; verify `GCThemeProvider` is the app-root provider.

---

### AC-02 — axe-core zero critical/serious

**Tool**: axe DevTools browser extension (Chrome or Edge) or `@axe-core/playwright`.

**Procedure**:
1. Navigate to each route: `/`, `/content`, `/translator`, `/urlscrapper`, `/tda`, `/tutor`
2. Open a modal / drawer on each screen that has one
3. Run axe scan on each state
4. Record any violations in the table below

**Pass condition**: 0 critical violations, 0 serious violations across all scanned states.  
**Fail action**: fix violation, re-scan before proceeding.

**Violation log** (fill in during testing):

| Screen | State | Violation | Rule ID | Fixed |
|--------|-------|-----------|---------|-------|
| | | | | |

---

### AC-03 — All strings via `t()`

**Automated check**:
```bash
# Find string literals in JSX (non-exhaustive — review false positives)
grep -rn ">[A-Z][a-z]" src/pages/ src/components/ | grep -v "t(" | grep -v "//"
```

**Manual check**: toggle language to FR; screenshot each screen; verify all UI text is French.

**Pass condition**: no English literals visible in FR mode.

---

### AC-04 — Language toggle works without reload

**Procedure**:
1. Open the app in EN
2. Type a question in the chat input (do not send)
3. Click the language toggle → app switches to FR
4. Verify: the typed text remains, no page reload (check browser navigation history — no new entry)
5. Verify: all nav labels, button labels, placeholders are now in French

**Pass condition**: all text French; input preserved; no reload.

---

### AC-05 — `document.documentElement.lang`

**Procedure**:
```javascript
// In DevTools console after switching to FR:
document.documentElement.lang  // must return "fr"
// After switching back to EN:
document.documentElement.lang  // must return "en"
```

**Pass condition**: lang attribute matches active locale immediately after toggle.

---

### AC-06 — `document.title` updates on route change

**Procedure**: Navigate to each route and read `document.title` in the console.

| Route | Expected `document.title` (EN) | Expected `document.title` (FR) | Pass |
|-------|-------------------------------|-------------------------------|------|
| `/` | `Chat — EVA` | `Clavardage — EVA` | |
| `/content` | `Manage Content — EVA` | `Gérer le contenu — EVA` | |
| `/translator` | `Translator — EVA` | `Traducteur — EVA` | |
| `/urlscrapper` | `URL Scraper — EVA` | `Extracteur d'URL — EVA` | |
| `/tda` | `Tabular Data Assistant — EVA` | `Assistant données tabulaires — EVA` | |
| `/tutor` | `Math Tutor — EVA` | `Tuteur de maths — EVA` | |
| `*` (404) | `Page not found — EVA` | `Page introuvable — EVA` | |

---

### AC-07 — Focus lands on `<h2>` after navigation

**Procedure** (keyboard only):
1. Tab to a nav link
2. Press Enter to navigate
3. Observe: after ~150 ms, focus moves to the first `<h2>` inside `#mainContent`
4. Screen reader should announce the heading text

**Pass condition**: `document.activeElement` is the `<h2>` approximately 150–200 ms after navigation.

```javascript
// Check in DevTools after navigation:
setTimeout(() => console.log(document.activeElement), 200)
// Must log the h2 element
```

---

### AC-08 — Dialog/Drawer focus trap + Escape + focus return

**Procedure** (repeat for each dialog and drawer):
1. Open the dialog/drawer using its trigger button
2. Press Tab repeatedly — focus must not leave the dialog/drawer
3. Press Shift+Tab — focus cycles backward inside the dialog/drawer
4. Press Escape — dialog/drawer closes
5. Verify: focus returns to the exact element that opened it

**Dialogs/Drawers to test**:
- Chat History Drawer (left)
- Analysis Panel Drawer (right)
- Settings Drawer (right)
- Delete Session Confirm Dialog
- Folder Create Dialog
- Feedback Modal Dialog
- URL Scraper Preview Dialog
- URL Scraper Confirm (over-limit) Dialog

---

### AC-09 — Icon-only buttons have `aria-label`

**Automated check**:
```javascript
// DevTools console:
[...document.querySelectorAll('button')]
  .filter(b => b.textContent.trim() === '' && !b.getAttribute('aria-label'))
  .forEach(b => console.log(b))
// Must log nothing
```

**Manual check**: inspect each icon button in the accessibility tree — must show label.

**Key buttons to verify**: Send, Clear Chat, Adjust Settings, Close Analysis Panel, Chat History trigger, Thumbs Up, Thumbs Down, Hamburger menu (mobile).

---

### AC-10 — `aria-live` announces async changes

**Procedure** (use screen reader NVDA or built-in narrator):
1. Open Translator — hear "Loading the folders, please wait" within 500 ms
2. Folders load — hear "Folders loaded successfully!"
3. Submit translation — hear result announcement
4. Trigger an API error in Chat — hear error text
5. Open Chat History — hear loading state
6. Type in Chat and submit — hear "Loading answer"

**Pass condition**: each async state produces an audible announcement without the user navigating to the live region.

---

### AC-11 — High Contrast Mode

**Procedure**:
1. Enable Windows High Contrast Black (Settings → Ease of Access → High Contrast Black)
2. Open each screen
3. Verify all buttons, inputs, labels, text, and borders are visible
4. Interact with at least one dialog and one form per screen

**Pass condition**: no invisible text, no invisible borders, no colour-only status indicators.

**Key components to check**:
- Translator Comboboxes (explicit HC styles required)
- FileStatus status-icon cells
- Answer citation buttons
- Focus rings on all interactive elements

---

### AC-12 — Pagination works correctly

**Chat History Drawer**:
- More than 10 sessions exist → previous/next page buttons appear
- Page 1 → Previous disabled; last page → Next disabled
- Delete a session → list re-paginates correctly

**FileStatus DataGrid**:
- More than 20 files exist → pagination controls appear
- Sort by column → pagination resets to page 1
- Filter by folder/tag/date → pagination resets to page 1

**URL Scraper Preview Dialog**:
- More than 5 links discovered → prev/next buttons appear
- Selections persist across page changes
- "Selected: N" count is accurate across all pages

---

### AC-13 — Upload flow end-to-end

**Procedure**:
1. Navigate to `/content` → Upload Files tab
2. Select a folder and add at least one tag
3. Drag a PDF file or click to browse and select one
4. Observe: ProgressBar fills; upload completes
5. Switch to Upload Status tab
6. Verify: the uploaded file appears in the DataGrid with correct folder, tag, and status

**Pass condition**: file appears in DataGrid within 10 seconds; status eventually shows "Completed" or equivalent.

---

### AC-14 — Translation flow end-to-end

**Procedure**:
1. Navigate to `/translator`
2. Wait for folder list to load (hear announcement)
3. Select a folder containing a document
4. Select a file from the file dropdown
5. Select target language (opposite of source)
6. Click Translate
7. Observe: button shows Spinner; success MessageBar appears; download link is rendered

**Pass condition**:
- Download link is a proper `<a href="...">` element
- Link is keyboard-reachable (Tab stops on it)
- Clicking the link triggers a file download

---

### AC-15 — URL Scraper dialog focus return

**Procedure**:
1. Navigate to `/urlscrapper`
2. Tab to the Preview button and note it has focus
3. Activate Preview
4. Inside the dialog, make any selections
5. Close the dialog via: (a) Cancel button, (b) Escape key, (c) Submit button
6. Verify in all three cases: focus returns to the Preview button

**Automatic check**:
```javascript
// After dialog closes:
document.activeElement  // must be the Preview button
```

---

### AC-16 — `@eva/ui` wrappers + `GCThemeProvider` at root

**Automated check — wrapper usage**:
```bash
# These raw Fluent primitives must not be used directly when an EvaXxx wrapper covers them
grep -rn "from '@fluentui/react-components'" src/ | \
  grep -E "Button[^G]|Input[^P]|Select[^O]|Dialog[^T]|Drawer[^H]|Badge[^P]|DataGrid|Spinner[^P]|TabList|Textarea|Toast[^C]" | \
  grep -v "//"
```
Result must be empty.

**Automated check — GCThemeProvider**:
```bash
grep -r "GCThemeProvider" src/index.tsx   # must return a match
grep -r "<FluentProvider" src/index.tsx   # must return no match (replaced by GCThemeProvider)
```

**Pass condition**: wrapper grep empty; `GCThemeProvider` present in `index.tsx`; no bare `<FluentProvider>` at root.  
**Fail action**: replace raw Fluent imports with their `EvaXxx` counterparts; wrap app root with `<GCThemeProvider>` from `@eva/gc-design-system`.

---

## Screen Reader Compatibility Matrix

| Browser + Screen Reader | Status | Tester | Date |
|------------------------|--------|--------|------|
| Chrome + NVDA | ⏳ | | |
| Edge + JAWS | ⏳ | | |
| Safari + VoiceOver (macOS) | C (Could Have) | | |
| Firefox + NVDA | C (Could Have) | | |

---

## Responsive Breakpoints

| Breakpoint | Width | Key behaviour to verify |
|------------|-------|------------------------|
| Desktop | 1440 px | Full nav visible; Analysis Panel side-by-side |
| Laptop | 1024 px | Full nav visible; panels may compress |
| Tablet | 768 px | Hamburger nav appears; drawers still usable |
| Mobile | 375 px | Hamburger nav; bottom bar stacks vertically |

---

## Sign-Off Table

All rows must be signed off before the rebuild is declared production-ready.

| AC | Verified by | Date | Method |
|----|------------|------|--------|
| AC-01 | | | grep |
| AC-02 | | | axe DevTools |
| AC-03 | | | grep + FR review |
| AC-04 | | | Manual |
| AC-05 | | | DevTools console |
| AC-06 | | | DevTools console |
| AC-07 | | | DevTools console |
| AC-08 | | | Keyboard + screen reader |
| AC-09 | | | DevTools console |
| AC-10 | | | NVDA + Chrome |
| AC-11 | | | HC Black + Windows |
| AC-12 | | | Manual |
| AC-13 | | | Manual + API |
| AC-14 | | | Manual + API |
| AC-15 | | | DevTools console |
| AC-16 | | | grep + code review |
