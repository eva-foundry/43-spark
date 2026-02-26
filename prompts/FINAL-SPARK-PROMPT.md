# FINAL SPARK PROMPT — EVA-JP v1.2 Frontend

> **Self-contained** — this file is the single authoritative, paste-ready source for all GitHub Spark sessions  
> and Copilot agent mode work on the EVA-JP v1.2 frontend rebuild.  
>
> Acceptance Criteria (AC-01 to AC-15), all five iteration prompts, and the full product backlog  
> are consolidated here. No other files are required to run a Spark session.
>
> **Integration target**: The Spark output is copied into `C:\AICOE\eva-foundation\31-eva-faces\eva-jp\`  
> and registered as a workspace in the **eva-faces monorepo**. It consumes `@eva/gc-design-system`  
> and `@eva/ui` directly via the monorepo's npm workspace symlinks — no publish step required.
>
> **How to use:**  
> 1. Open [github.com/spark](https://github.com/spark)  
> 2. Paste **ITERATION PROMPT 1** verbatim to scaffold the foundation  
> 3. In the **same Spark session**, paste each subsequent iteration prompt in order  
> 4. Copy the generated code into `31-eva-faces/eva-jp/`  
> 5. Run `npm install` from the monorepo root — workspace symlinks wire the shared packages  
> 6. Use Copilot agent mode with `eva-jp-rebuild-spec.json` attached for surgical edits  
>
> **Cost**: 5 iterations × 4 premium requests = **20 premium Copilot requests** for a full build.

---

## ITERATION PROMPT 1 — Foundation, Shell & Layout

```
Build a bilingual (English / French) AI assistant web app called EVA-JP for the Canadian federal government. Use React 18 with TypeScript strict mode, Vite as the build tool, and Fluent UI React v9 (@fluentui/react-components) as the ONLY UI component library — no Material UI, no Ant Design, no Bootstrap, and ZERO imports from @fluentui/react (v8).

Reference this open-source app as the source of truth for API shapes, business logic, and existing component patterns:
https://github.com/microsoft/PubSec-Info-Assistant/tree/main/app/frontend

---

## App Overview

EVA-JP is a chat-first productivity suite with these screens (all behind a persistent shell):

1. Chat (default /) — send questions, receive AI-streamed answers with citations, analysis panel, chat history drawer
2. Manage Content (/content) — upload files (PDF, DOCX, XLSX, CSV, EML/MSG), view upload status DataGrid, embedded URL scraper tab
3. Translator (/translator) — folder → file picker → target language → translate → download
4. URL Scraper (/urlscrapper) — paste URLs, preview discovered links in a paginated dialog, submit selected
5. Tabular Data Assistant (/tda) — upload CSV, ask questions, stream answers + chart images
6. Math Tutor (/tutor) — type a math problem, get clues / step-by-step / direct answer via streamed SSE
7. 404 — friendly not-found with return-to-chat link

Navigation uses react-router-dom v6 HashRouter. A <Layout> shell wraps all routes via <Outlet />.

---

## Tech Stack (strict — do not deviate)

- UI: @fluentui/react-components ^9.72 + @fluentui/react-icons ^2.0
- ZERO imports from @fluentui/react (v8) — this is a hard build-breaking constraint
- Theme: @eva/gc-design-system ^1.0.0 — use GCThemeProvider variant="light" as the root provider (replaces FluentProvider + webLightTheme). The gcTheme is a GC-branded Fluent v9 theme — all tokens.* values are already available inside it.
- Shared components: @eva/ui ^1.0.0 — use Eva* wrappers (EvaButton, EvaInput, EvaTextArea, EvaDialog, EvaDrawer, EvaTabs, EvaDataGrid, EvaSpinner, EvaToast) instead of raw Fluent primitives wherever available. Custom components still use makeStyles + Fluent v9 primitives directly.
- Styling: makeStyles + mergeClasses from Fluent v9 only. All colours → tokens.*, all spacing → tokens.spacingHorizontal* / tokens.spacingVertical*, all typography → tokens.typographyStyles.*. Zero hardcoded hex or pixel values.
- i18n: i18next ^25 + react-i18next ^16. Every user-visible string uses t("key") — no string literals in JSX ever.
- Routing: react-router-dom ^6.8, HashRouter
- Build: Vite 5, TypeScript strict. Path alias @ → src/. Follows the same vite.config.ts pattern as the sibling apps in the monorepo.
- Other allowed: react-markdown, rehype-sanitize, rehype-raw, papaparse, mammoth, xlsx, nanoid, dompurify

---

## Accessibility — WCAG 2.1 AA (Government of Canada standard)

Every component and screen must satisfy ALL of the following:

- Focus ring: outline: 3px solid tokens.colorStrokeFocus2; outlineOffset: 2px on every interactive element — visible, never suppressed.
- Skip link: First focusable DOM element — <a href="#mainContent">Skip to main content</a> — visually hidden until focused, visible on :focus.
- Landmarks: <header role="banner">, <main id="mainContent" role="main" tabIndex={-1}>, <nav aria-label="Primary navigation">, <footer role="contentinfo">.
- Heading hierarchy: One <h1> in the header. Each page screen owns an <h2> as its first heading inside #mainContent.
- Live regions: aria-live="polite" for loading and success announcements; aria-live="assertive" for blocking errors only. The chat message list carries role="log" aria-live="polite".
- Announcer: AnnouncerProvider renders a visually-hidden role="status" aria-live="polite" aria-atomic="true" div. A useAnnouncer() hook returns (message: string) => void. Use it on every async state change.
- Dialogs / Drawers: All modals trap focus. Escape always closes. Focus returns to the trigger element on close — store triggerRef = useRef<HTMLButtonElement>() and call triggerRef.current?.focus() in onOpenChange when open becomes false.
- Icon-only buttons: Every icon-only button has an explicit aria-label.
- Form labels: Every input, textarea, Combobox, Dropdown has a visible <label> or aria-label. No placeholder-only labelling.
- Route change: On every navigation — (1) update document.title with the localized page title, (2) push the page name to useAnnouncer(), (3) after 150 ms focus the first <h2> inside #mainContent.
- High-contrast mode: Every makeStyles call that uses colour or background includes a @media (forced-colors: active) block using only Canvas, CanvasText, Highlight, HighlightText, ButtonFace, ButtonText. This is mandatory, not optional.
- Colour contrast: 4.5:1 for normal text, 3:1 for large text and UI components.
- Toggle button groups (response length, temperature, chat mode): wrap in <div role="radiogroup">, each ToggleButton gets role="radio" and aria-checked.
- document.documentElement.lang must reflect the active locale (en or fr) at all times via i18n.on("languageChanged").

---

## Internationalisation

- Locales: English (en) and French (fr). Detect from navigator.language on first visit via i18next-browser-languagedetector.
- Locale files: src/i18n/locales/en/resources_en.json and src/i18n/locales/fr/resources_fr.json.
- Keys use the human-readable English string: t("Send question"), t("Clear chat"). Not camelCase.
- Language toggle button in the header switches EN ↔ FR without a page reload.
- aria-label on toggle: "Switch interface language to French" / "Switch interface language to English".
- Use Intl.DateTimeFormat and Intl.NumberFormat with the active locale for all dates and numbers.
- French locale file MUST contain real French translations (not copies of English strings).

---

## RBAC

Users belong to a named group and have a role (reader, contributor, admin, owner). Fetch from /api/userGroupInfo on app load.

- readers: see Chat, TDA, Math Tutor only.
- contributors / admins / owners: also see Manage Content, Translator, URL Scraper nav items.
- owners / admins only: see the Examples editing tab in the Chat settings drawer.
- Feature flags ENABLE_TABULAR_DATA_ASSISTANT and ENABLE_MATH_ASSISTANT gate TDA and Math Tutor nav items. Fetch from /api/featureFlags on load.

---

## Key Shared Components

Build these in src/components/ (Phase 2 prompt covers each in detail).  
Where an `@eva/ui` wrapper exists, **import it** instead of building from raw Fluent primitives.  
All Eva* wrappers already use makeStyles + tokens.* internally and are v9-only.

| ID | Name | Source |
|----|------|--------|
| SC-01 | AnnouncerProvider / useAnnouncer | custom — hidden `role=status` div + hook |
| SC-02 | WarningBanner | `@eva/ui` → `EvaToast` or raw `MessageBar` intent="warning" |
| SC-03 | QuestionInput | custom — raw `Textarea` + `EvaButton`; `:focus-within` ring on container |
| SC-04 | UserChatMessage | custom — raw `Text`; right-aligned bubble, `role=article` |
| SC-05 | Answer | custom — `EvaButton`, `EvaDialog` (feedback), `MessageBar` intent=error |
| SC-06 | CharacterStreamer | custom — div with `aria-live="polite"` for streamed tokens |
| SC-07 | AnalysisPanel | `@eva/ui` → `EvaTabs` wrapping Thought Process / Supporting Content / Citation |
| SC-08 | ChatHistory | `@eva/ui` → `EvaDrawer` (left) + `EvaDialog` delete confirm + pagination |
| SC-09 | ExampleList | custom — `EvaButton` appearance=subtle chips |
| SC-10 | FolderPicker | custom — `@fluentui/react-components` `Combobox` + `EvaDialog` create folder |
| SC-11 | TagPicker | custom — `Combobox` + Tag chips |
| SC-12 | FilePicker | custom — drag-drop zone + visually-hidden `input[type=file]` + `ProgressBar` |
| SC-13 | FileStatus | `@eva/ui` → `EvaDataGrid` — sortable, filterable, keyboard navigable |
| SC-15 | ResponseLengthButtonGroup | custom — 3 × `ToggleButton` in `role=radiogroup` |
| SC-16 | ChatModeButtonGroup | custom — 3 × `ToggleButton` (Work / Web / Hybrid) in `role=radiogroup` |
| SC-17 | RAIPanel | custom — `Accordion` from `@fluentui/react-components` |
| SC-18 | SupportingContent | custom — `EvaButton` appearance=subtle per retrieved chunk |
| SC-19 | SettingsIconButton | `@eva/ui` → `EvaDrawer` (right) with `EvaTabs` inside; trigger uses `EvaButton` icon |
| SC-20 | Title / LoadTitle | `@eva/ui` → `EvaSpinner` while fetching app title |

---

## Phase 1 Bootstrap — Build These Files First

0. package.json: name="eva-jp", type="module". Dependencies: @eva/gc-design-system ^1.0.0, @eva/ui ^1.0.0, @fluentui/react-components ^9.72, @fluentui/react-icons ^2.0, react ^18.2, react-dom ^18.2, react-router-dom ^6.8, i18next ^25, react-i18next ^16, i18next-browser-languagedetector, react-markdown, rehype-sanitize, dompurify, nanoid. DevDependencies mirror portal-face: vite, typescript, @vitejs/plugin-react, vitest, @testing-library/react.
0b. vite.config.ts: plugins=[react()], resolve.alias { '@': './src', '@api': './src/api', '@components': './src/components', '@hooks': './src/hooks', '@pages': './src/pages' }. test.environment='jsdom'.
0c. tsconfig.json: extends "../tsconfig.base.json". Include ["src"].
0d. index.html: lang="en" (will be updated by i18n), id="root" div.
1. src/main.tsx: <React.StrictMode><GCThemeProvider variant="light"><AnnouncerProvider><HashRouter><App /></HashRouter></AnnouncerProvider></GCThemeProvider></React.StrictMode>. Import GCThemeProvider from @eva/gc-design-system. No FluentProvider or webLightTheme — GCThemeProvider wraps FluentProvider internally.
2. src/i18n/i18n.tsx: init i18next with navigator.language detection, both locale files, fallbackLng: "en". Call i18n.on("languageChanged", lng => document.documentElement.lang = lng).
3. src/i18n/locales/en/resources_en.json: seed with all keys listed below.
4. src/i18n/locales/fr/resources_fr.json: seed with real French translations for all keys.
5. src/components/Annoucement/AnnouncerProvider.tsx: AnnouncerProvider + useAnnouncer hook.
6. src/App.tsx: HashRouter routes with Layout wrapping all screens via Outlet.
7. src/pages/layout/Layout.tsx: full header (skip link → brand cluster → nav → utility cluster), <main id="mainContent">, footer. Show full-page EvaSpinner while /api/userGroupInfo loads. Wire RBAC-gated nav, language toggle, group selector Dropdown, route-change focus management (document.title + useAnnouncer + h2 focus after 150 ms).

---

## i18n Seed Keys

Seed the English and French locale files with at minimum these keys. French values must be real translations.

Layout / Shell: "Chat", "Manage Content", "Translator", "URL Scrapper", "Math Assistant", "Tabular Data Assistant", "Welcome", "Loading...If more than 15 seconds, please refresh your browser", "Switch interface language to English", "Switch interface language to French", "Language", "Primary navigation", "Open menu", "Close menu", "Select user group", "Change Group", "Preview", "Skip to main content"

Chat: "Send question", "Clear chat", "Adjust settings", "Chat messages", "Work documents only", "Web search", "Work documents and web search", "New chat", "Loading answer", "Regenerate answer", "Show analysis", "Concise", "Balanced", "Detailed", "Precise", "Creative", "Give feedback"

Manage Content / Upload: "Upload Files", "Upload Status", "URL Scraper", "Supported file types"

Translator: "Select folder", "Select file", "Target language", "English", "French", "Translate", "Loading the folders, Please wait", "Folders loaded successfully!", "Translation successful", "Translation failed", "Download translated file"

URL Scraper: "URLs (one per line)", "Blacklisted URLs (one per line)", "Max links", "Preview links", "Check all", "Uncheck all", "Cancel", "Selected: {{count}} links"

TDA: "Tabular Data Assistant", "Upload a CSV file", "Drop CSV here or click to select", "Enter your question", "Analyse", "Analysis result", "Generated chart", "An error occurred."

Math Tutor: "Math Tutor", "Math problem", "Give Me Clues", "Show Me How to Solve It", "Show Me the Answer", "Math answer", "Max retries reached. Last error:"

404: "Page not found", "The page you are looking for does not exist.", "Return to Chat"

---

## Start Here

Scaffold Phase 1 first. Begin with package.json and vite.config.ts (they define the workspace identity), then src/main.tsx using GCThemeProvider from @eva/gc-design-system, then i18n setup, then AnnouncerProvider, then App.tsx routes, then Layout.tsx with the full header/main/footer structure. Do not create src/theme.ts — the GC theme is provided by @eva/gc-design-system.
```

---

## ITERATION PROMPT 2 — Shared Components

```
Build all shared components in this order, one file at a time. Apply all WCAG AA rules, Fluent v9 only, makeStyles only, t() for every string, @media (forced-colors: active) on every colour-bearing style — from the ground rules in Iteration 1.

Build order:
AnnouncerProvider → WarningBanner → CharacterStreamer → UserChatMessage → Answer → QuestionInput → AnalysisPanel → ChatHistory → ExampleList → FolderPicker → TagPicker → FilePicker → FileStatus → ResponseLengthButtonGroup → ChatModeButtonGroup → RAIPanel → SupportingContent → SettingsIconButton → Title/LoadTitle

For the Answer component:
- While the ReadableStream is open, render CharacterStreamer.
- After stream closes, render ReactMarkdown with rehype-sanitize.
- Citation inline links are Button appearance="subtle" and keyboard reachable.
- Follow-up suggestion chips are Button components below the answer text.
- Thumbs-up / thumbs-down are ToggleButtons. Thumbs-down opens a FeedbackModal Dialog.
- Action row buttons (regenerate, open analysis panel, etc.): all icon+label with aria-label.

For QuestionInput:
- The outer div uses makeStyles with :focus-within that renders the focus ring (tokens.colorStrokeFocus2, 3px solid, offset 2px). The inner Textarea does NOT show a duplicate ring.
- Enter key calls onSend. Shift+Enter inserts a newline.
- Send button is disabled when input is empty or the disabled prop is true.

For all Drawers (AnalysisPanel, ChatHistory, SettingsIconButton):
- Store triggerRef = useRef<HTMLButtonElement>().
- On Drawer close (onOpenChange: open becomes false): call triggerRef.current?.focus().
- Escape closes automatically via Fluent Drawer — do not override.

For toggle button groups (ResponseLengthButtonGroup, ChatModeButtonGroup, ResponseTempButtonGroup):
- Wrapper: <div role="radiogroup" aria-label={t("...")}>.
- Each ToggleButton: role="radio" aria-checked={isSelected} checked={isSelected}.
```

---

## ITERATION PROMPT 3 — Chat Screen

```
Build src/pages/chat/Chat.tsx following all ground rules from Iteration 1.

Structure:
- <section aria-label={t("Chat")}> containing a visually-hidden <h2>{t("Chat")}</h2>.
- TOP BAR: ChatHistoryDrawerTrigger (left), Clear Chat button (centre-left), SettingsIconButton (right) — all always visible.
- CHAT AREA: scrollable div with role="log" aria-live="polite" aria-label={t("Chat messages")}. When answers is empty, show ExampleList. When there are messages, show UserChatMessage + Answer pairs. A non-focusable div ref at the bottom for auto-scroll.
- BOTTOM BAR: FolderPicker + TagPicker row, QuestionInput below, ChatModeButtonGroup below that.

Settings Drawer (right):
- Triggered by SettingsIconButton with a stored triggerRef; on close, focus returns to trigger.
- TabList with tabs: Info, Adjust, Examples (owner/admin only).
- Adjust tab: ResponseLengthButtonGroup (Concise 1024 / Balanced 2048 / Detailed 3072), ResponseTempButtonGroup (Precise 0.2 / Balanced 0.6 / Creative 0.9), strict-mode Switch, suggest-follow-ups Switch, retrieve-count SpinButton (1–50), past-turns SpinButton (0–10).
- Examples tab: up to 6 Textarea inputs, Save button, Reset button.

Session management:
- Call createNewSession() on mount and on clear.
- Store session ID in both useState and useRef to avoid stale closures in async API callbacks.

Error handling:
- MessageBar intent="error" between message list and QuestionInput on API failure.
- Error text includes a friendly message with contact email links as <ul><li><a>.
- Wrap in <div aria-live="assertive">.

Analysis panel:
- Opens to the right of the chat area (not overlaid). Chat area flex-shrinks.
- Resizable AnalysisPanel (SC-07) with tabs: Thought Process, Supporting Content, Citation.

Streaming:
- Pass ReadableStream to Answer. Answer delegates to CharacterStreamer while open.
- After stream closes, Answer switches to ReactMarkdown + rehype-sanitize.
```

---

## ITERATION PROMPT 4 — Remaining Screens

```
Build the remaining screens one at a time in this order. Apply ALL WCAG AA rules, Fluent v9 only, t() for every string, @media (forced-colors: active) on every colour-bearing makeStyles call.

--- Manage Content (/content) ---
<section> with <h2>{t("Manage Content")}</h2>. TabList: Upload Files | Upload Status | URL Scraper (admin/contributor only).
Upload Files tab: file-types info block (icons aria-hidden), characters-notice with visually-hidden symbol names, then FolderPicker + TagPicker + FilePicker stacked vertically.
Upload Status tab: <FileStatus /> DataGrid.
URL Scraper tab: <Urlscrapper /> (same component as /urlscrapper route).

--- Translator (/translator) ---
<section> with <h2>{t("Translator")}</h2> + decorative TranslateFilled icon (aria-hidden).
On mount: fetch folders → populate Combobox → announce t("Folders loaded successfully!") via useAnnouncer().
Folder selection → fetch files → populate file Combobox.
Target language Dropdown: English | French.
Translate Button: shows Spinner inside while loading.
Success: MessageBar intent="success" + accessible download link. Error: MessageBar intent="error" + announce.
Both Comboboxes need @media (forced-colors: active) styles.

--- URL Scraper (/urlscrapper) ---
<section> with <h2>{t("URL Scraper")}</h2>.
Form: Textarea for URLs (one per line), Textarea for blacklisted URLs, scraper-layer Dropdown (1/2/3) with InfoButton Tooltip, optional max-links Input, include-header Checkbox, include-footer Checkbox.
Preview Button (primary) → Dialog with paginated checkboxes (5 per page), Check all / Uncheck all, pagination prev/next, aria-live="polite" t("Selected: {{count}} links"), Cancel + Submit footer.
If checking all would exceed effectiveMaxLinks → second confirm Dialog.
On Dialog close for any reason → focus returns to Preview Button triggerRef.
Submit Button disabled until preview confirmed.

--- Tabular Data Assistant (/tda) ---
Render only when ENABLE_TABULAR_DATA_ASSISTANT feature flag is true.
<section> with <h2>{t("Tabular Data Assistant")}</h2>.
Drop zone: visually-hidden input[type=file accept=".csv"], its <label> is the keyboard-accessible drop target, aria-describedby points to max-file-size hint text (fetched from API).
ExampleList with 4 preset CSV queries.
Textarea aria-label={t("Enter your question")} + Analyse Button.
On Analyse: close existing eventSourceRef.current if open, open new EventSource stream.
Answer area: role="region" aria-label={t("Analysis result")}, CharacterStreamer, img with alt={t("Generated chart")}.

--- Math Tutor (/tutor) ---
Render only when ENABLE_MATH_ASSISTANT feature flag is true.
<section> with <h2>{t("Math Tutor")}</h2> + decorative MathFormatProfessionalFilled icon (aria-hidden).
ExampleList with 4 preset math problems.
Form: <label> + Input aria-label={t("Math problem")}.
Three Buttons: Give Me Clues (getHint), Show Me How to Solve It (streamData SSE), Show Me the Answer (processAgentResponse).
All three calls wrapped in retryAsyncFn(fn, retries=3, delayMs=1000).
On final retry failure: MessageBar intent="error" + useAnnouncer(t("Max retries reached. Last error:") + " " + error.message).
Answer area: role="region" aria-label={t("Math answer")}, CharacterStreamer.

--- 404 No Page ---
<main id="mainContent" role="main" tabIndex={-1}>
  <h2 ref={h2Ref}>{t("Page not found")}</h2>
  <p>{t("The page you are looking for does not exist.")}</p>
  <Link to="/">{t("Return to Chat")}</Link>
</main>
On mount: document.title = t("Page not found") + " — EVA"; h2Ref.current?.focus();
```

---

## ITERATION PROMPT 5 — Quality Pass

```
Perform a final quality pass across ALL files in the codebase:

1. ZERO imports from @fluentui/react (v8). Grep every file. If found, replace with the v9 equivalent from @fluentui/react-components.

2. Every makeStyles call that sets color, backgroundColor, borderColor, fill, or stroke MUST have a @media (forced-colors: active) companion block. Add any that are missing. Use only: Canvas, CanvasText, Highlight, HighlightText, ButtonFace, ButtonText.

3. Every icon used decoratively has aria-hidden="true". Every icon-only interactive element has an explicit aria-label. No icon-only Button exists without one.

4. Route change audit: on each HashRouter location change, confirm (a) document.title updates, (b) useAnnouncer() fires with the page name, (c) #mainContent h2 receives focus after 150 ms.

5. Confirm document.documentElement.lang is updated via i18n.on("languageChanged").

6. Confirm the skip link is the first focusable element in the DOM and becomes visible on :focus using makeStyles (not inline style).

7. Confirm ALL Drawers and Dialogs: (a) trap focus while open, (b) close on Escape, (c) return focus to their triggerRef on close.

8. Confirm ChatModeButtonGroup, ResponseLengthButtonGroup, ResponseTempButtonGroup all have role="radiogroup" on the wrapper and role="radio" + aria-checked on each ToggleButton.

9. Confirm every form control has either a wrapping <label> (via Fluent Field component) or aria-label. Zero placeholder-only labelling.

10. Confirm the chat message list has role="log" aria-live="polite".

11. Verify French locale file has a real translation (not a copy of the English key) for every string used in JSX. If any value matches its key exactly, it must be translated.
```

---

## Acceptance Criteria

All 15 ACs must pass before any branch is merged. Verification commands are in `ACCEPTANCE.md`.

| ID | Criterion | Verification |
|----|-----------|-------------|
| AC-01 | Zero imports from `@fluentui/react` (v8) across all rebuilt files | `grep -r "from ['\"]@fluentui/react['\"]" src/` → 0 results |
| AC-02 | axe-core scan: 0 critical, 0 serious violations per screen | Playwright + axe-playwright, one test per route |
| AC-03 | All user-visible strings use `t()` — no hardcoded literals in JSX | `grep -rn ">[A-Z]" src/ \| grep -v "t("` → 0 results |
| AC-04 | Language toggle switches all text with no page reload | Click toggle → verify all visible strings change locale |
| AC-05 | `document.documentElement.lang` reflects active locale | `document.documentElement.lang` equals `'en'` or `'fr'` |
| AC-06 | `document.title` updates on every route change | Navigate each route → verify `document.title` |
| AC-07 | Keyboard focus lands on first `<h2>` after navigation | `document.activeElement === document.querySelector('#mainContent h2')` after 200ms |
| AC-08 | All dialogs trap focus; close on Escape; restore focus to trigger | Tab cycles inside dialog; Escape closes; trigger has focus |
| AC-09 | All icon-only buttons have `aria-label` | `querySelectorAll('button:not([aria-label])')` → 0 icon-only results |
| AC-10 | `aria-live` regions announce async status changes | Screen reader hears loading/success/error without focus move |
| AC-11 | High-contrast mode: all UI visible and operable | Edge → Windows HC mode → visual inspection of every screen |
| AC-12 | Chat history, FileStatus table, preview dialog paginate correctly | 11 items → 2 pages; page 2 loads; prev/next keyboard operable |
| AC-13 | Upload flow completes and status reflected in FileStatus table | Upload file → DataGrid row appears with correct status |
| AC-14 | Translation flow completes and download link is accessible | Select folder/file/lang → translate → download link focusable |
| AC-15 | URL scraper preview dialog returns focus to trigger on close | Open dialog → close via Escape or Cancel → triggerRef.current has focus |

---

## Product Backlog

Work item hierarchy: **Epic → Feature → User Story**  
Priority: **M** = Must Have · **S** = Should Have · **C** = Could Have

---

### EPIC E-01 — EVA-JP v1.2 Frontend Rebuild

**Goal:** Replace the legacy Fluent UI v8 frontend with a production-grade React 18 / Fluent UI v9 application that meets WCAG 2.1 AA, bilingual EN/FR requirements, and RBAC-gated navigation for the Government of Canada.

---

#### Feature F-01 — Project Foundation & Infrastructure

| # | User Story | Priority | Linked AC |
|---|-----------|----------|-----------|
| US-F01-01 | As a **developer**, I want Vite + React 18 + TypeScript strict so builds are fast and type errors caught at compile time. | M | — |
| US-F01-02 | As a **developer**, I want `src/theme.ts` exporting `webLightTheme` so every component inherits Fluent v9 tokens automatically. | M | AC-01 |
| US-F01-03 | As a **developer**, I want all `@fluentui/react` v8 imports removed to reduce bundle size and eliminate conflicting style systems. | M | AC-01 |
| US-F01-04 | As a **developer**, I want `makeStyles`/`mergeClasses` as the sole styling method with `tokens.*` for all values for consistent theming. | M | AC-11 |
| US-F01-05 | As a **developer**, I want `.vscode/launch.json` and `tasks.json` configured so the dev server starts on F5. | S | — |

**DoD:** `vite build` exits 0 · zero `@fluentui/react` imports · all colours via `tokens.*`

---

#### Feature F-02 — Bilingual Support (EN / FR)

| # | User Story | Priority | Linked AC |
|---|-----------|----------|-----------|
| US-F02-01 | As a **user**, I want the app to detect my browser language on first visit so I see content in my preferred language automatically. | M | AC-05 |
| US-F02-02 | As a **user**, I want a language toggle in the header to switch EN ↔ FR without losing my current page state. | M | AC-04, AC-05 |
| US-F02-03 | As a **screen reader user**, I want the language toggle to have a descriptive `aria-label` so I know what language I will switch to before activating. | M | AC-09 |
| US-F02-04 | As a **developer**, I want every visible string from locale files via `t()` so adding a third language requires only a new locale file. | M | AC-03 |
| US-F02-05 | As a **French-speaking user**, I want dates and numbers formatted in French locale conventions. | S | AC-03 |
| US-F02-06 | As a **developer**, I want the French locale file to contain real translations (not English fallbacks) for all keys. | M | AC-03, AC-04 |

**DoD:** `t()` on every visible string · FR values ≠ EN values · `document.documentElement.lang` reflects active locale

---

#### Feature F-03 — Accessibility (WCAG 2.1 AA)

| # | User Story | Priority | Linked AC |
|---|-----------|----------|-----------|
| US-F03-01 | As a **keyboard user**, I want a skip-to-main-content link as the first focusable DOM element. | M | AC-07 |
| US-F03-02 | As a **keyboard user**, I want a visible 3 px focus ring on every interactive element using `tokens.colorStrokeFocus2`. | M | AC-02 |
| US-F03-03 | As a **screen reader user**, I want live regions announcing loading, success, and error states. | M | AC-10 |
| US-F03-04 | As a **screen reader user**, I want route changes to update `document.title`, announce the page name, and focus `<h2>` after 150 ms. | M | AC-06, AC-07 |
| US-F03-05 | As a **screen reader user**, I want all dialogs/drawers to trap focus, close on Escape, and return focus to trigger. | M | AC-08 |
| US-F03-06 | As a **HC mode user**, I want all UI to remain visible and operable in Windows High Contrast mode. | M | AC-11 |
| US-F03-07 | As a **user**, I want all form controls to have visible labels. | M | AC-02 |
| US-F03-08 | As a **screen reader user**, I want toggle button groups announced as radio groups. | M | AC-02 |
| US-F03-09 | As a **developer**, I want axe-core scans of every screen to return 0 critical and 0 serious violations. | M | AC-02 |
| US-F03-10 | As a **screen reader user**, I want the chat message list to carry `role="log"` so new answers are announced automatically. | M | AC-10 |
| US-F03-11 | As a **screen reader user**, I want `useAnnouncer()` on every async flow for consistent state change communication. | M | AC-10 |

**DoD:** axe-core 0 critical/serious · HC media queries on all coloured components · `role="log"` on chat list

---

#### Feature F-04 — Shell, Navigation & RBAC

| # | User Story | Priority | Linked AC |
|---|-----------|----------|-----------|
| US-F04-01 | As a **user**, I want a persistent header with logo, title, nav links, language toggle, and group selector. | M | — |
| US-F04-02 | As a **reader-role user**, I want Manage Content, Translator, and URL Scraper links hidden so I am not confused. | M | — |
| US-F04-03 | As a **user on narrow viewport (< 960 px)**, I want a hamburger button to toggle navigation. | S | AC-08 |
| US-F04-04 | As a **user**, I want a full-page Spinner while group information loads. | M | AC-10 |
| US-F04-05 | As a **user without an assigned group**, I want an informative unauthorised message with access-request links. | M | — |
| US-F04-06 | As a **user**, I want `<footer role="contentinfo">` containing the WarningBanner. | M | — |
| US-F04-07 | As a **user with TDA or Math Tutor enabled**, I want corresponding nav links to appear automatically. | S | — |

**DoD:** landmarks present · RBAC gates confirmed · `aria-current="page"` on active link

---

#### Feature F-05 — Chat Screen

| # | User Story | Priority | Linked AC |
|---|-----------|----------|-----------|
| US-F05-01 | As a **user**, I want to send a question and receive a streamed AI answer character-by-character. | M | — |
| US-F05-02 | As a **user**, I want inline citation buttons in answer text to trace claims to source documents. | M | AC-02 |
| US-F05-03 | As a **user**, I want a resizable Analysis Panel (Thought Process / Supporting Content / Citation tabs). | M | AC-08, AC-12 |
| US-F05-04 | As a **user**, I want past chat sessions in a left drawer with 10-per-page pagination. | M | AC-12 |
| US-F05-05 | As a **user**, I want to clear the chat and start a fresh session with one click. | M | — |
| US-F05-06 | As a **user**, I want to switch between Work-only, Web, and Work+Web chat modes. | M | — |
| US-F05-07 | As a **user**, I want to adjust response length, temperature, retrieve count, and past-turns. | S | AC-08 |
| US-F05-08 | As a **user**, I want thumbs-up / thumbs-down feedback buttons on every answer. | S | AC-09 |
| US-F05-09 | As a **user**, I want example prompt chips when the chat is empty. | S | AC-02 |
| US-F05-10 | As a **user**, I want a clear error message with contact details when the API fails. | M | AC-10 |
| US-F05-11 | As an **owner/admin**, I want an Examples tab in the settings drawer to edit, save, and reset prompt chips. | C | — |

**DoD:** streaming via CharacterStreamer → ReactMarkdown · session ID in state + ref · error in assertive live region

---

#### Feature F-06 — Content Management Screen

| # | User Story | Priority | Linked AC |
|---|-----------|----------|-----------|
| US-F06-01 | As a **contributor**, I want to drag-drop or browse for files (PDF, DOCX, XLSX, CSV, EML, MSG) into a tagged folder. | M | AC-13 |
| US-F06-02 | As a **contributor**, I want to create a new folder from the folder picker without navigating away. | S | — |
| US-F06-03 | As a **contributor**, I want to add custom tags to uploaded files. | S | — |
| US-F06-04 | As a **contributor**, I want a sortable/filterable DataGrid of uploaded files with Status, Tags, Folder, Upload Date columns. | M | AC-13 |
| US-F06-05 | As an **admin**, I want the URL Scraper tab in Manage Content. | M | — |
| US-F06-06 | As a **screen reader user**, I want file status icons to carry `aria-label` text. | M | AC-02, AC-09 |

**DoD:** upload completes → DataGrid row appears · folder/tag/date filter works · URL Scraper tab hidden for reader role

---

#### Feature F-07 — Translator Screen

| # | User Story | Priority | Linked AC |
|---|-----------|----------|-----------|
| US-F07-01 | As a **contributor**, I want to select folder → file → target language → Translate to produce a translated document. | M | AC-14 |
| US-F07-02 | As a **user**, I want a Spinner inside the folder combobox while loading with an announced message. | S | AC-10 |
| US-F07-03 | As a **user**, I want a success message with a keyboard-accessible download link after translation completes. | M | AC-14 |
| US-F07-04 | As a **HC mode user**, I want dropdown borders and placeholder text to remain visible. | M | AC-11 |
| US-F07-05 | As a **screen reader user**, I want translation errors announced immediately via live region. | M | AC-10 |

**DoD:** full translate workflow end-to-end · forced-colors on Comboboxes · announcements on load/success/error

---

#### Feature F-08 — URL Scraper Screen

| # | User Story | Priority | Linked AC |
|---|-----------|----------|-----------|
| US-F08-01 | As an **admin**, I want to paste URLs and preview discovered links in a paginated dialog before submitting. | M | AC-08, AC-12 |
| US-F08-02 | As a **user**, I want Check All / Uncheck All with a guard when selection exceeds max-links. | S | — |
| US-F08-03 | As a **user**, I want an `aria-live="polite"` "Selected: N links" count in the dialog. | S | AC-10 |
| US-F08-04 | As a **keyboard user**, I want the dialog to return focus to the Preview button on close. | M | AC-15 |
| US-F08-05 | As a **user**, I want the Submit button disabled until a preview is confirmed. | M | — |

**DoD:** AC-15 confirmed · second confirm dialog for over-limit check-all · Submit re-disabled when URL list changes

---

#### Feature F-09 — Tabular Data Assistant

| # | User Story | Priority | Linked AC |
|---|-----------|----------|-----------|
| US-F09-01 | As a **data analyst**, I want to upload a CSV by drag-drop or browse to ask natural-language questions. | M | — |
| US-F09-02 | As a **user**, I want to type or select a preset query and receive a streamed text answer. | M | AC-10 |
| US-F09-03 | As a **user**, I want auto-generated chart images with meaningful alt text. | S | AC-02 |
| US-F09-04 | As a **keyboard user**, I want the CSV drop zone activatable by keyboard. | M | AC-02 |
| US-F09-05 | As a **screen reader user**, I want the answer region announced progressively as it streams. | M | AC-10 |
| US-F09-06 | As a **user**, I want this screen hidden unless `ENABLE_TABULAR_DATA_ASSISTANT` is true. | M | — |

**DoD:** EventSource opens/closes cleanly · alt on all charts · feature-flag gate confirmed

---

#### Feature F-10 — Math Tutor Screen

| # | User Story | Priority | Linked AC |
|---|-----------|----------|-----------|
| US-F10-01 | As a **student**, I want to enter a math problem and choose clues, method, or direct answer. | M | — |
| US-F10-02 | As a **user**, I want failed math API calls retried up to 3 times before showing an error. | M | AC-10 |
| US-F10-03 | As a **user**, I want four preset example math problems to auto-populate the input. | S | — |
| US-F10-04 | As a **screen reader user**, I want a clear error announcement with the last error message when all retries fail. | M | AC-10 |
| US-F10-05 | As a **screen reader user**, I want the streamed math answer announced progressively. | M | AC-10 |
| US-F10-06 | As a **user**, I want this screen hidden unless `ENABLE_MATH_ASSISTANT` is true. | M | — |

**DoD:** retryAsyncFn wraps all 3 API calls · answer streams via CharacterStreamer · feature-flag gate confirmed

---

## API Endpoints Reference

| Method | Endpoint | Used by |
|--------|----------|---------|
| GET | `/api/userGroupInfo` | Layout — user RBAC info |
| GET | `/api/featureFlags` | Layout — TDA and Math Tutor flags |
| GET | `/api/applicationtitle` | LoadTitle component |
| POST | `/api/conversation` | Chat — main streaming endpoint |
| POST | `/api/feedback` | Answer — thumbs feedback |
| GET | `/api/chathistory/sessions` | ChatHistory — list sessions |
| POST | `/api/chathistory/session` | Chat — create new session |
| DELETE | `/api/chathistory/session/{id}` | ChatHistory — delete session |
| GET | `/api/uploadstatus` | FileStatus — list files |
| POST | `/api/upload` | FilePicker — upload file |
| GET | `/api/getfolders` | FolderPicker / Translator |
| POST | `/api/createfolder` | FolderPicker |
| GET | `/api/gettags` | TagPicker |
| POST | `/api/translatefile` | Translator |
| POST | `/api/urlscrape/preview` | URL Scraper — preview links |
| POST | `/api/urlscrape/submit` | URL Scraper — submit selected |
| POST | `/api/tda/analyse` | TDA — EventSource stream |
| GET | `/api/tda/maxcsvfilesize` | TDA |
| GET | `/api/tda/images` | TDA — chart images |
| GET | `/api/math/hint` | Math Tutor — getHint |
| POST | `/api/math/stream` | Math Tutor — streamData SSE |
| POST | `/api/math/answer` | Math Tutor — processAgentResponse |

---

## Source Reference

- GitHub source of truth: https://github.com/microsoft/PubSec-Info-Assistant/tree/main/app/frontend
- Spec JSON: `c:\AICOE\eva-foundation\44-eva-jp-spark\eva-jp-rebuild-spec.json`
- PRD: `c:\AICOE\eva-foundation\44-eva-jp-spark\PRD-REBUILD-FLUENT-V9.md`
- Acceptance detail: `c:\AICOE\eva-foundation\43-spark\ACCEPTANCE.md`
- Copilot instructions: `c:\AICOE\eva-foundation\43-spark\.github\copilot-instructions.md`

## Monorepo Integration

The output of this Spark build is placed at `C:\AICOE\eva-foundation\31-eva-faces\eva-jp\`.

- The monorepo root `package.json` lists `eva-jp` in `workspaces` — `npm install` from the root symlinks `@eva/gc-design-system` and `@eva/ui` automatically.
- `tsconfig.json` in `eva-jp/` extends `../tsconfig.base.json` which already maps `@eva/*` to their `src/` paths for TypeScript resolution.
- Sibling apps for reference: `chat-face/` (uses `@eva/gc-design-system` + `@eva/ui`) and `portal-face/` (uses `@eva/gc-design-system` + `react-router-dom`).
- Run from monorepo root: `npm run dev:eva-jp` — starts the Vite dev server for EVA-JP only.
- Run from monorepo root: `npm run build` — builds shared libs first, then all apps including EVA-JP.
