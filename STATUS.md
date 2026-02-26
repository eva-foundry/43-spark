# 43-spark — Status Board

> **Last updated**: 2026-02-24 22:33 ET  
> **Updated by**: agent:copilot  

Legend: ✅ Done · 🔄 In Progress · ⏳ Not Started · 🚫 Blocked · ❌ Failed

---

## Overall Progress

```
Phase 0 — Prompt Engineering      ████████████████████████ 100% ✅
Phase 1 — Spark Bootstrap          ░░░░░░░░░░░░░░░░░░░░░    0% ⏳  ← NEXT
Phase 2 — Shared Components        ░░░░░░░░░░░░░░░░░░░░░    0% ⏳
Phase 3 — Screens                  ░░░░░░░░░░░░░░░░░░░░░    0% ⏳
Phase 4 — Quality Pass             ░░░░░░░░░░░░░░░░░░░░░    0% ⏳
Phase 5 — Repository & CI          ████████████████░░░░░   65% 🔄
Phase 6 — Codespace Hardening      ░░░░░░░░░░░░░░░░░░░░░    0% ⏳
```

---

## Phase 0 — Prompt Engineering

| Deliverable | Status | Notes |
|-------------|--------|-------|
| `eva-jp-rebuild-spec.json` | ✅ Done | `44-eva-jp-spark/` |
| `PRD-REBUILD-FLUENT-V9.md` | ✅ Done | `44-eva-jp-spark/` |
| 5 Spark iteration prompts | ✅ Done | `prompts/FINAL-SPARK-PROMPT.md` |
| Epic + 10 Features + 57 User Stories | ✅ Done | Bottom of `FINAL-SPARK-PROMPT.md` |
| 43-spark springboard scaffold | ✅ Done | Skills, copilot-instructions, workflows |
| Copilot instructions + skills | ✅ Done | `.github/copilot-instructions.md`, `.copilot/skills/` |

---

## Phase 1 — Spark Bootstrap

| Deliverable | Status | AC Gate | Notes |
|-------------|--------|---------|-------|
| Spark initial prompt pasted | ⏳ | — | Awaiting Spark access |
| `src/index.tsx` FluentProvider | ✅ Done | AC-01 | `GCThemeProvider` from `@eva/gc-design-system` wired |
| `src/theme.ts` | ✅ Done | AC-01 | `webLightTheme` removed; re-exports `Theme` type only |
| `src/i18n/` setup | ⏳ | AC-03–05 | |
| AnnouncerProvider | ⏳ | AC-10 | |
| Layout shell | ⏳ | AC-06, AC-07 | |
| Preview visual review | ⏳ | — | |

---

## Phase 2 — Shared Components

| Component | Status | AC Gate |
|-----------|--------|---------|
| AnnouncerProvider / useAnnouncer | ⏳ | AC-10 |
| WarningBanner | ⏳ | AC-10 |
| CharacterStreamer | ⏳ | AC-10 |
| UserChatMessage | ⏳ | AC-02 |
| Answer | ⏳ | AC-02, AC-09 |
| QuestionInput | ⏳ | AC-02 |
| AnalysisPanel | ⏳ | AC-08, AC-12 |
| ChatHistory | ⏳ | AC-08, AC-12 |
| ExampleList | ⏳ | AC-02 |
| FolderPicker | ⏳ | AC-02 |
| TagPicker | ⏳ | AC-02 |
| FilePicker | ⏳ | AC-13 |
| FileStatus | ⏳ | AC-13 |
| ResponseLengthButtonGroup | ⏳ | AC-02 |
| ChatModeButtonGroup | ⏳ | AC-02 |
| RAIPanel | ⏳ | AC-02 |
| SupportingContent | ✅ Done | AC-02 | `EvaButton variant="subtle"` |
| Title / LoadTitle | ✅ Done | AC-10 | `EvaSpinner` |
| QuestionInput | 🔄 Partial | AC-02 | `EvaButton` done; `Textarea` blocked (CSS `> textarea` selector) |
| ChatHistory | 🔄 Partial | AC-08, AC-12 | `EvaButton`/`EvaSpinner` done; `Drawer`/`Dialog` TODO(AC-16) |
| SettingsIconButton | 🔄 Partial | AC-08, AC-09 | `EvaButton` done; `Drawer`/`Tabs` TODO(AC-16) |

---

## Phase 3 — Screens

| Screen | Status | axe-core | AC Gates |
|--------|--------|----------|---------|
| Layout Shell | ⏳ | ⏳ | AC-06, AC-07 |
| Chat (`/`) | ⏳ | ⏳ | AC-08, AC-10, AC-12 |
| Manage Content (`/content`) | ⏳ | ⏳ | AC-13 |
| Translator (`/translator`) | ⏳ | ⏳ | AC-11, AC-14 |
| URL Scraper (`/urlscrapper`) | ⏳ | ⏳ | AC-12, AC-15 |
| TDA (`/tda`) | ⏳ | ⏳ | AC-10 |
| Math Tutor (`/tutor`) | ⏳ | ⏳ | AC-10 |
| 404 | ⏳ | ⏳ | AC-06, AC-07 |

---

## Phase 4 — Quality Pass

| Check | Status | Notes |
|-------|--------|-------|
| Zero `@fluentui/react` imports | ⏳ | `grep -r "@fluentui/react\"" src/` |
| Zero hardcoded hex in styles | ⏳ | `grep -r "#[0-9a-fA-F]" src/` |
| HC `forced-colors` on all colour styles | ⏳ | |
| All 15 AC items verified | ⏳ | See `ACCEPTANCE.md` |
| FR locale has real translations | ⏳ | Manual review |
| NVDA + Chrome screenreader test | ⏳ | |
| JAWS + Edge screenreader test | ⏳ | |
| Windows HC Black + White | ⏳ | |

---

## Phase 5 — Repository & CI

| Task | Status | Notes |
|------|--------|-------|
| Create `eva-foundry/44-eva-jp-spark` GitHub repo | ✅ Done | https://github.com/eva-foundry/44-eva-jp-spark |
| Initialise standalone git repo in `44-eva-jp-spark/` | ✅ Done | Detached from `eva-foundation` monorepo |
| Initial commit + push to `main` | ✅ Done | All stubs committed |
| Two-way sync confirmed | 🔄 Verify | Run `npm run dev` after `npm install` |
| `npm run build` CI action | ⏳ | Add `.github/workflows/ci.yml` |
| `tsc --noEmit` CI check | ⏳ | Included in build action |
| axe-core Playwright tests | ⏳ | Phase 4+ |
| Branch protection rules | ⏳ | After CI green |

---

## Phase 6 — Codespace Hardening

| Issue | Assigned to | Status |
|-------|-------------|--------|
| Wire mock APIs → real `/api/*` endpoints | Copilot coding agent | ⏳ |
| Keyboard nav tests for ChatHistory | Copilot coding agent | ⏳ |
| retryAsyncFn utility | Copilot coding agent | ⏳ |
| EventSource lifecycle in TDA | Copilot coding agent | ⏳ |

---

## Blockers

_None currently._

---

## Decisions Log

| Date | Decision | Rationale |
|------|----------|-----------|
| 2026-02-22 | Use HashRouter (not BrowserRouter) | Matches existing PubSec-Info-Assistant deployment pattern; no server-side routing needed |
| 2026-02-22 | 5-iteration Spark prompt structure | Each Spark prompt costs 4 requests; splitting phases keeps each generation focused and reversible |
| 2026-02-22 | `makeStyles` + `tokens.*` only | Ensures high-contrast mode works; prevents design drift; aligns with GC design token convention |
| 2026-02-22 | French locale values = real translations | GoC Official Languages Act compliance; reviewer sign-off required before Phase 4 closes |
| 2026-02-22 | Host as standalone repo `eva-foundry/eva-jp-spark` | Enables independent CI/CD, branch protection, and Codespace devcontainer without monorepo coupling |
| 2026-02-23 | `@eva/ui` replaces raw Fluent imports | AC-16: all new components must use EvaXxx wrappers from `31-eva-faces/shared/eva-ui` |
| 2026-02-23 | tsconfig paths → eva-ui/gc-design-system src | Avoids dual-React node_modules issue; source compiled with `44-eva-jp-spark`'s own `@types/react` |
| 2026-02-23 | Upgrade `@types/react` 18→19 | Matches `31-eva-faces` React types; React 19 types backward-compatible with React 18 code |
| 2026-02-23 | eva-ui `dts: true` + fixed type errors | EvaButton ref, EvaDialog children, EvaDrawer type alias, EvaIcon Partial cast |
| 2026-02-24 22:33 ET | eva-veritas reconcile: 0 gaps | Project 43 has no user stories (springboard); reconciliation.json + trust.json written |

---

## How to Update This File

When you complete a task:
1. Change `⏳` to `✅` (or `❌` if failed with explanation in Notes)
2. Update the progress bars in **Overall Progress** (count done/total)
3. Add a row to **Decisions Log** if a design decision was made
4. Update **Last updated** date at the top
