# 43-spark — EVA Spark Springboard

> **Purpose**: Reusable launchpad for building and iterating on EVA UI/UX frontends using [GitHub Spark](https://github.com/spark).  
> **Current target**: EVA-JP v1.2 Frontend Rebuild (see `44-eva-jp-spark/`).  
> **Pattern**: Draft prompt here → refine → paste into Spark → iterate in Codespace with agent mode.

---

## What Is This Folder?

`43-spark` is a **shared design system and prompt engineering workspace** for all EVA UI/UX projects that use GitHub Spark as the scaffolding vehicle. It contains:

| Path | Purpose |
|------|---------|
| `README.md` | This file — orientation and quick-start |
| `PLAN.md` | Phased delivery plan and Spark iteration strategy |
| `STATUS.md` | Live tracking board — what is done, in progress, blocked |
| `ACCEPTANCE.md` | Master acceptance criteria used across all EVA Spark apps |
| `.github/copilot-instructions.md` | Repository-level Copilot instructions (agent mode context) |
| `.copilot/skills/eva-ui.md` | Skill: `@eva/ui` + `@eva/gc-design-system` component patterns and GC token rules |
| `.copilot/skills/a11y-wcag.md` | Skill: WCAG 2.1 AA rules enforced in every EVA app |
| `.copilot/skills/i18n-enfr.md` | Skill: EN/FR internationalisation with i18next |
| `.copilot/skills/spark-workflow.md` | Skill: How to structure Spark prompts and iterate |
| `workflows/spark-prompt-workflow.md` | Step-by-step workflow from requirement → final Spark prompt |
| `prompts/FINAL-SPARK-PROMPT.md` | **The consolidated, ready-to-paste Spark prompt for EVA-JP** |

---

## Quick Start

### 1 — Paste the initial Spark prompt
Copy **everything** inside the first fenced code block in `prompts/FINAL-SPARK-PROMPT.md` and paste it into [github.com/spark](https://github.com/spark).

Spark will generate a React + TypeScript app with a live preview. This consumes **4 premium Copilot requests**.

### 2 — Review the preview
Check the live preview for:
- Header renders with logo, nav, language toggle
- Chat screen is the default route with a streaming input area
- No visible v8 Fluent component artefacts

### 3 — Open in Codespace
Click **"Open in Codespace"** in Spark. Once the Codespace starts:
```bash
npm run dev
```
The app will be served at `localhost:5173`.

### 4 — Attach context and iterate
In VS Code inside the Codespace, open Copilot Chat in **agent mode** and attach:
- `eva-jp-rebuild-spec.json` (from `44-eva-jp-spark/`)
- `PRD-REBUILD-FLUENT-V9.md` (from `44-eva-jp-spark/`)

Then paste each **ITERATION PROMPT** block from `prompts/FINAL-SPARK-PROMPT.md` in sequence.

### 5 — Validate
Run the acceptance checks in `ACCEPTANCE.md`. All AC-01 through AC-15 must pass before merging.

---

## How Spark Works (Key Facts for Prompting)

| Fact | Impact on prompting |
|------|-------------------|
| Stack is **React + TypeScript** (opinionated, cannot be changed) | Aligns perfectly with EVA; no adaptation needed |
| UI layer must use **`@eva/ui` + `@eva/gc-design-system`** | Import `EvaXxx` wrappers first; raw `@fluentui/react-components` only for un-wrapped primitives |
| External libraries are **allowed but not guaranteed** compatible | List exact versions; test after each library add |
| Each natural-language prompt costs **4 premium Copilot requests** | Use iteration prompts sparingly; batch related work |
| **Organisation sharing** available | Publish spark to your GitHub org for team review |
| **Two-way sync** with a GitHub repo | Create a repo immediately after first generation; keep Codespace changes in sync |
| Agent mode in Codespace ≠ Spark prompts | Agent mode is for surgical edits; Spark prompts scaffold structure |
| Spark does **not** know your internal APIs | All API paths must be explicit in the prompt (e.g. `/api/userGroupInfo`) |

---

## Folder Conventions

```
43-spark/
├── .github/
│   └── copilot-instructions.md   ← Agent mode context for this repo
├── .copilot/
│   └── skills/
│       ├── fluent-v9.md
│       ├── a11y-wcag.md
│       ├── i18n-enfr.md
│       └── spark-workflow.md
├── workflows/
│   └── spark-prompt-workflow.md  ← Human process guide
├── prompts/
│   └── FINAL-SPARK-PROMPT.md     ← Ready-to-paste prompt (single source of truth)
├── README.md
├── PLAN.md
├── STATUS.md
└── ACCEPTANCE.md
```

---

## Related Projects

| Folder | Description |
|--------|-------------|
| `44-eva-jp-spark/` | EVA-JP v1.2 Spark output + spec files → **[github.com/eva-foundry/44-eva-jp-spark](https://github.com/eva-foundry/44-eva-jp-spark)** |
| `31-eva-faces/` | Design specs, ADO work items, PRD documents |
| `40-eva-devbench/` | Local dev server for EVA frontend testing |

---

## ⚡ What Is Next (February 2026)

Phase 0 is **complete**. The standalone repo https://github.com/eva-foundry/eva-jp-spark has been created.

**Immediate next steps:**
1. `cd 44-eva-jp-spark && npm install` — install all runtime + dev dependencies
2. Open [github.com/spark](https://github.com/spark), paste **ITERATION PROMPT 1** from `prompts/FINAL-SPARK-PROMPT.md`
3. Review the live preview (Layout shell, language toggle, skip link)
4. Open Codespace → attach `eva-jp-rebuild-spec.json` → paste Iteration 2, 3, 4, 5 in sequence
5. After each iteration, run the AC checks in `ACCEPTANCE.md`
6. Add `.github/workflows/ci.yml` to the `eva-foundry/44-eva-jp-spark` repo for build + typecheck gate

---

## Key References

- [GitHub Spark docs](https://docs.github.com/en/copilot/concepts/spark)
- [`@eva/ui` — EVA shared component library](../../31-eva-faces/shared/eva-ui/README.md)
- [`@eva/gc-design-system` — GC theme + tokens](../../31-eva-faces/shared/gc-design-system/README.md)
- [Fluent UI React v9](https://react.fluentui.dev/) (underlying primitive library; use @eva wrappers first)
- [PubSec-Info-Assistant frontend](https://github.com/microsoft/PubSec-Info-Assistant/tree/main/app/frontend) — API contract source
- [WCAG 2.1 AA](https://www.w3.org/TR/WCAG21/) — accessibility standard
- [GC Standard on Web Accessibility](https://www.tbs-sct.canada.ca/pol/doc-eng.aspx?id=23601)
