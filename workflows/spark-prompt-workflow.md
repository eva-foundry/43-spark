# Spark Prompt Workflow — EVA UI/UX

> This document describes the end-to-end human process for taking a UI/UX requirement  
> from stakeholder input through to a production-ready React front-end, using GitHub Spark  
> as the generation engine and GitHub Codespaces as the editing environment.

---

## Overview

```
Requirement Input
      │
      ▼
┌─────────────────────┐
│  1. Gather Context  │  Spec JSON · PRD · API catalogue · AC list
└─────────────────────┘
      │
      ▼
┌─────────────────────┐
│  2. Write Prompt 0  │  Full constraint prompt for Spark
└─────────────────────┘
      │
      ▼
┌─────────────────────┐
│  3. Spark: Run      │  5 iterations × 4 requests = 20 premium requests
│  Iterations 1-5     │
└─────────────────────┘
      │
      ▼
┌─────────────────────┐
│  4. Validate Output │  v8 leak · axe-core · HC mode · i18n parity
└─────────────────────┘
      │
      ▼
┌─────────────────────┐
│  5. Connect Repo    │  Spark → GitHub repo → clone to Codespace
└─────────────────────┘
      │
      ▼
┌─────────────────────┐
│  6. Wire Real APIs  │  Replace mock data; hook up /api/* endpoints
└─────────────────────┘
      │
      ▼
┌─────────────────────┐
│  7. CI Green        │  tsc · build · axe Playwright · WCAG audit
└─────────────────────┘
      │
      ▼
┌─────────────────────┐
│  8. UAT Sign-off    │  Stakeholder review via Spark published link
└─────────────────────┘
      │
      ▼
┌─────────────────────┐
│  9. Deploy          │  Azure Static Web App · GitHub Actions CD
└─────────────────────┘
```

---

## Step 1 — Gather Context

**Who**: Product owner + tech lead  
**Output**: Three artefacts

| Artefact | Format | Location |
|----------|--------|---------|
| Spec JSON | `*.json` — component tree, API contract, RBAC roles | `44-eva-jp-spark/eva-jp-rebuild-spec.json` |
| PRD | Markdown — screens, user stories, MoSCoW, AC table | `44-eva-jp-spark/PRD-REBUILD-FLUENT-V9.md` |
| API catalogue | Table in prompt / spec | Embedded in prompt or spec |

**Checklist**:
- [ ] Every screen is named and has a route
- [ ] Every API endpoint is listed with method + path + payload shape
- [ ] All 15 (or n) Acceptance Criteria are written with pass/fail criteria
- [ ] RBAC roles are defined (reader / contributor / admin / owner)
- [ ] Feature flags are listed (`ENABLE_TABULAR_DATA_ASSISTANT`, `ENABLE_MATH_ASSISTANT`)
- [ ] i18n string list is complete (both EN and FR values)

---

## Step 2 — Write the Spark Prompt

**Who**: Tech lead (use Copilot agent mode + skills in `.copilot/skills/`)  
**Output**: `prompts/FINAL-SPARK-PROMPT.md`

### Structure (use `spark-workflow.md` skill for detail)

1. App identity paragraph
2. Reference source URL
3. Screens list (route + purpose + key interactions)
4. Tech stack (non-negotiable list)
5. Accessibility rules (exhaustive WCAG AA list)
6. i18n rules (detection, key convention, locale files)
7. RBAC rules
8. Shared component catalogue (table)
9. Phase 1 bootstrap file list
10. "Start here" instruction

**Prompt writing tips**:
- State the most critical constraints twice (in the preamble and in the affected iteration)
- Be explicit about what NOT to do (`zero imports from @fluentui/react`)
- End every iteration prompt with `"Start here:"` pointing to the specific first file
- Keep total prompt < 2 000 tokens per iteration (Spark context limit)

---

## Step 3 — Run Spark Iterations

**Who**: Tech lead or senior developer  
**Tool**: https://github.com/spark

### Iteration 1 — Foundation

Paste **ITERATION PROMPT 1** verbatim. Wait for Spark to generate:
- `src/index.tsx` (FluentProvider + AnnouncerProvider + HashRouter)
- `src/theme.ts`
- `src/App.tsx`
- `src/i18n/i18n.tsx`
- `src/i18n/locales/en/resources_en.json`
- `src/i18n/locales/fr/resources_fr.json`
- `src/pages/layout/Layout.tsx`

**After**: Run validation commands (Step 4 mini-check). Fix before proceeding.

### Iteration 2 — Shared Components

Paste **ITERATION PROMPT 2** in the **same Spark session**. Expect 20 component files.  
Build order matters — generate in dependency order:
1. `AnnouncerProvider` → 2. `WarningBanner` → 3. `QuestionInput` → ... → 20. `SettingsIconButton`

### Iteration 3 — Chat Screen

Paste **ITERATION PROMPT 3**. This is the most complex screen (streaming, chat history, settings panel).

### Iteration 4 — Remaining Screens

Paste **ITERATION PROMPT 4**. Covers:
- Content Management (with DataGrid, upload, URL Scraper)
- Translator
- Math Tutor
- TDA
- Welcome / 404

### Iteration 5 — Quality Pass

Paste **ITERATION PROMPT 5**. Spark audits its own output for:
- v8 import leaks
- Missing HC media queries
- String literal leaks
- Accessibility gaps

---

## Step 4 — Validate Output

**Who**: Tech lead  
**Run after each iteration**:

```bash
# 0. Install
npm install

# 1. TypeScript
npx tsc --noEmit

# 2. Build
npm run build

# 3. v8 import leak
grep -r "from ['\"]@fluentui/react['\"]" src/

# 4. Hardcoded hex
grep -rn "#[0-9a-fA-F]\{3,6\}" src/

# 5. String literals in JSX
grep -rn ">[A-Z][a-zA-Z ,!?]" src/pages/ src/components/ | grep -v "t(" | grep -v "//"

# 6. Accessibility (in browser dev tools or CI)
# Install axe DevTools or run: npx playwright test (see workflows/ci.yml)
```

**Fix strategy**: For each failure, write a targeted Spark correction prompt:
```
"Fix only: [specific issue] in [file path].
Do not modify any other file or behaviour."
```

---

## Step 5 — Integrate into the eva-faces Monorepo

1. Copy the Spark-generated files into `C:\AICOE\eva-foundation\31-eva-faces\eva-jp\`
2. `eva-jp` is already registered as a workspace in the monorepo `package.json` — no edit needed
3. From the **monorepo root** run: `npm install` — this symlinks `@eva/gc-design-system` and `@eva/ui` into `eva-jp/node_modules/@eva/`
4. Verify TypeScript sees the shared packages: `cd eva-jp && npx tsc --noEmit`
5. Start dev server: from monorepo root run `npm run dev:eva-jp` → verify on port 5173
6. Optionally connect to GitHub via Spark UI → sync Codespace with with `31-eva-faces` repo for agent mode edits

> **Why no standalone repo?** `@eva/gc-design-system` and `@eva/ui` are consumed via workspace symlinks.
> Publishing them to a registry is only needed for external consumers outside the monorepo.

---

## Step 6 — Wire Real APIs

**Who**: Developer in Codespace (Copilot agent mode)  
**Pattern**: Replace all mock/static data with real fetch calls

For each API endpoint in your catalogue:
1. Create `src/api/<domain>.ts` with typed response interfaces
2. Replace mock data in the component with the fetch call
3. Handle loading state (Spinner + `useAnnouncer`)
4. Handle error state (MessageBar + `useAnnouncer` assertive)
5. Handle retry logic (3 retries, 1s delay exponential backoff)

Codespace agent mode prompt (use for each screen):
```
"In src/pages/chat/Chat.tsx, replace the mock answer array with a real fetch 
to POST /api/conversation. Use EventSource for streaming. 
On stream start: show Spinner and call useAnnouncer(t('Loading answer')).
On stream end: update answers state and call useAnnouncer(t('Answer received')).
On error: show MessageBar intent=error and call useAnnouncer with assertive priority.
Do not change any other component."
```

---

## Step 7 — CI Green

**Target**: Zero failures on all CI checks.

| Check | Tool | Pass criterion |
|-------|------|---------------|
| TypeScript compile | `tsc --noEmit` | 0 errors |
| Build | `npm run build` | exits 0 |
| v8 import leak | `grep` | 0 matches |
| Accessibility | axe-playwright | 0 critical/serious per screen |
| WCAG colour contrast | manual/automated | All ≥ 4.5:1 |
| High contrast mode | Edge HC mode | Fully operable |
| Screen reader | NVDA + Edge | All ACs pass |

---

## Step 8 — UAT Sign-off

1. Publish Spark to org (read-only): Spark UI → Publish → Organisation
2. Share URL with stakeholders
3. Conduct structured UAT against `ACCEPTANCE.md` AC table
4. All 15 ACs must show ✅ in the sign-off table before deployment

---

## Step 9 — Deploy

- Target: Azure Static Web App
- Trigger: merge to `main` triggers GitHub Actions CD pipeline
- SWA config: `staticwebapp.config.json` with route rewrite for HashRouter
- Health check: `GET /` returns HTTP 200 after deploy

```json
// staticwebapp.config.json (minimum for HashRouter)
{
  "navigationFallback": {
    "rewrite": "/index.html",
    "exclude": ["/api/*", "/assets/*"]
  }
}
```

---

## Key Files Reference

| File | Purpose |
|------|---------|
| `prompts/FINAL-SPARK-PROMPT.md` | The single-file, ready-to-paste Spark prompt |
| `ACCEPTANCE.md` | All 15 ACs with verification commands |
| `PLAN.md` | 6-phase delivery plan |
| `STATUS.md` | Live tracking board |
| `.github/copilot-instructions.md` | Copilot agent mode context |
| `.copilot/skills/fluent-v9.md` | Fluent v9 component patterns |
| `.copilot/skills/a11y-wcag.md` | WCAG AA rules and code recipes |
| `.copilot/skills/i18n-enfr.md` | i18n setup, conventions, full key list |
| `.copilot/skills/spark-workflow.md` | Spark prompt structure and validation |
| `44-eva-jp-spark/SPARK-PROMPT-EVA-JP-FRONTEND.md` | Source prompt (5 iterations + backlog) |
| `44-eva-jp-spark/eva-jp-rebuild-spec.json` | Component tree and API contract |
| `44-eva-jp-spark/PRD-REBUILD-FLUENT-V9.md` | PRD with screens, stories, ACs |
