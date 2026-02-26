# Skill: GitHub Spark Workflow

> Load this skill when structuring, refining, or troubleshooting a GitHub Spark prompt.  
> Also load when planning how many Spark iterations to use for a UI/UX task.

---

## How Spark Works

GitHub Spark is a natural-language app builder at [github.com/spark](https://github.com/spark).

| Fact | Implication |
|------|------------|
| **Stack is React + TypeScript** (fixed) | No configuration needed; aligns with EVA |
| **4 premium Copilot requests per prompt** | Batch related work; split large tasks into phases |
| **External libraries allowed** (with caveats) | Specify exact package names and versions; test after adding |
| **No knowledge of your APIs** | Every `/api/*` path must be explicit in the prompt |
| **Organisation sharing** | Publish to your GitHub org for stakeholder review |
| **Two-way GitHub repo sync** | Create a repo after first generation; keep Codespace in sync |
| **Spark = scaffold; Codespace = edit** | Use Spark for structure; use Copilot agent mode in Codespace for surgical edits |

---

## Prompt Structure Template

A well-formed EVA Spark prompt has these sections in order:

```
1. APP IDENTITY
   One paragraph: what is the app, what does it do, who uses it.

2. REFERENCE SOURCE
   URL to the source app or API contract.

3. SCREENS (numbered list)
   Each screen: route, purpose, key interactions.

4. TECH STACK (strict list)
   Packages, versions, zero-deviation language.

5. ACCESSIBILITY (WCAG AA rules)
   Exhaustive list: focus ring, skip link, landmarks, live regions,
   dialog behaviour, route change, HC mode, toggle group patterns.

6. INTERNATIONALISATION
   Languages, detection, locale files, key convention, date/number format.

7. RBAC
   Role list, what each role can see/do, feature flags.

8. SHARED COMPONENTS TABLE
   ID | Name | Key Fluent v9 primitives

9. PHASE 1 BOOTSTRAP (what to build first)
   Ordered list of files to create in Phase 1.

10. START HERE (imperative sentence)
    Tell Spark exactly which file to produce first.
```

---

## Iteration Strategy

Split large apps into 5 iterations. Each costs 4 requests and covers a cohesive phase.

| Iteration | Content | Size guide |
|-----------|---------|-----------|
| 1 — Foundation | FluentProvider, theme, i18n, AnnouncerProvider, Layout shell | ~1 500 tokens |
| 2 — Components | All shared components in dependency order | ~1 800 tokens |
| 3 — Primary screen | Most complex screen (usually Chat) | ~1 200 tokens |
| 4 — Remaining screens | All other screens | ~1 800 tokens |
| 5 — Quality pass | Constraint audit, HC media queries, v8 leak check | ~1 000 tokens |

**Total cost**: 20 premium Copilot requests for a full 8-screen app.

---

## Writing Effective Constraints

Rules that Spark tends to violate without explicit instruction:

```
❌ Spark may drift to:            ✅ Make explicit:
- Inline style objects            - "Use makeStyles and mergeClasses only"
- Hex colours                     - "All colours via tokens.* — zero hex"
- v8 Fluent imports               - "Zero imports from @fluentui/react (v8)"
- Missing HC media queries        - "Every makeStyles with colour includes @media (forced-colors: active)"
- Placeholder-only labels         - "No placeholder-only labelling — every control has <label> or aria-label"
- Missing aria-label on icons     - "Every icon-only button has explicit aria-label"
- String literals in JSX          - "Every string uses t('key') — no literals"
- Focus not returning after close - "On Drawer/Dialog close, call triggerRef.current?.focus()"
```

State the most critical constraints **twice** (once in the initial prompt's ground rules, once within the affected iteration prompt).

---

## Validating a Spark Output

After each Spark generation, run these checks before proceeding to the next iteration:

```bash
# 1. v8 import leak
grep -r "from \"@fluentui/react\"" src/
grep -r "from '@fluentui/react'" src/

# 2. Hardcoded colours
grep -rn "#[0-9a-fA-F]\{3,6\}" src/

# 3. String literals in JSX (rough check)
grep -rn ">[A-Z][a-zA-Z ]" src/pages/ src/components/ | grep -v "t(" | grep -v "//"

# 4. TypeScript errors
npx tsc --noEmit

# 5. Build
npm run build
```

If any check fails, provide a targeted correction prompt:
```
"In the current codebase, fix: [specific issue].
Do not change anything else."
```

---

## Codespace Iteration vs Spark Prompt

Use Spark prompts for:
- Generating new files or entire new components
- Structural changes (layout reorganisation)
- Adding a new screen

Use Copilot agent mode in Codespace for:
- Fixing specific bugs or accessibility gaps
- Wiring real API calls (replacing mock data)
- Adding tests
- Edge case handling (e.g. retry logic, EventSource lifecycle)

Codespace agent mode prompt pattern:
```
"In src/pages/tutor/Tutor.tsx:
- Implement retryAsyncFn(fn, retries=3, delayMs=1000)
- Wrap getHint, streamData, and processAgentResponse calls with it
- On final failure, show MessageBar intent=error and call useAnnouncer()
- Do not change any other files"
```

---

## Common Spark Failures and Fixes

| Failure | Cause | Fix prompt |
|---------|-------|------------|
| v8 imports appear | Spark defaults to v8 patterns | "Replace all imports from `@fluentui/react` with equivalents from `@fluentui/react-components`" |
| `style={{color: '#...'}}` appears | Spark uses inline styles | "Move all styles to makeStyles. Replace hex values with the equivalent tokens.* value" |
| Dialog does not return focus | Spark omits focus management | "Add a triggerRef, call triggerRef.current?.focus() in onOpenChange when open becomes false" |
| Missing HC media queries | Spark does not add them by default | "Add @media (forced-colors: active) blocks to every makeStyles call that sets color or background" |
| French locale = English copy | Spark copies keys | "Replace all French locale values with real French translations" |
| Missing h2 on page screen | Spark uses div instead | "Add <h2> as the first element inside #mainContent on this screen" |
| No role=radiogroup on toggles | Spark renders plain buttons | "Wrap the three ToggleButtons in <div role='radiogroup'>; add role='radio' and aria-checked to each" |

---

## Sharing a Spark for Stakeholder Review

1. In Spark UI → **Publish** → **Organisation** → select your GitHub org
2. Share the Spark URL with stakeholders
3. They can view and interact with the live app without a GitHub Codespace
4. For read-only showcasing: enable **Read-only** in publish settings

---

## Repository and CI Setup (after initial Spark)

```bash
# In Codespace terminal after connecting to repo:
git clone <repo-url>
cd <app>
npm install
npm run dev      # verify localhost:5173 works

# Set up CI (create .github/workflows/ci.yml):
# - npm run build
# - npx tsc --noEmit
# - playwright test (axe-core per screen)
```

GitHub Actions workflow for EVA Spark projects:
```yaml
name: CI
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm ci
      - run: npx tsc --noEmit
      - run: npm run build
      - run: npx playwright install --with-deps
      - run: npx playwright test
```
