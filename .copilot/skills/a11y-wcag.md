# Skill: Accessibility — WCAG 2.1 AA

> Load this skill when implementing or reviewing any interactive element, form, dialog, navigation, or dynamic content in EVA-JP.  
> Standard: WCAG 2.1 Level AA · Government of Canada Standard on Web Accessibility.

---

## The EVA A11y Checklist (apply to every component and screen)

- [ ] Visible focus ring on every interactive element (3px, `tokens.colorStrokeFocus2`)
- [ ] `aria-label` on every icon-only button
- [ ] `<label>` or `aria-label` on every form control (no placeholder-only)
- [ ] `@media (forced-colors: active)` in every `makeStyles` with colour/background
- [ ] `aria-live="polite"` on all status/loading output containers
- [ ] `aria-live="assertive"` on blocking error containers only
- [ ] `role="log" aria-live="polite"` on the chat message list
- [ ] Dialog/Drawer: focus trap + Escape closes + focus returns to trigger
- [ ] Route change: `document.title` → `useAnnouncer()` → focus `<h2>` after 150 ms
- [ ] Skip link as first DOM element: `<a href="#mainContent">Skip to main content</a>`
- [ ] One `<h1>` per page; `<h2>` as first heading in `#mainContent`
- [ ] `document.documentElement.lang` = active locale

---

## ARIA Patterns

### Page Landmarks

```html
<header role="banner">
  <nav id="primary-navigation" aria-label="Primary navigation">
    <ul>
      <li><a href="/" aria-current="page">Chat</a></li>
      <li><a href="/content">Manage Content</a></li>
    </ul>
  </nav>
</header>

<main id="mainContent" role="main" tabindex="-1">
  <h2>Chat</h2>  <!-- first h2 on every page screen -->
  <!-- page content -->
</main>

<footer role="contentinfo">
  <!-- WarningBanner -->
</footer>
```

### AnnouncerProvider Pattern

```tsx
// src/components/Annoucement/AnnouncerProvider.tsx
const AnnouncerContext = React.createContext<(msg: string) => void>(() => {});

export const AnnouncerProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [message, setMessage] = React.useState('');
  return (
    <AnnouncerContext.Provider value={setMessage}>
      {children}
      <span
        role="status"
        aria-live="polite"
        aria-atomic="true"
        style={{
          position: 'absolute', width: '1px', height: '1px',
          padding: 0, margin: '-1px', overflow: 'hidden',
          clip: 'rect(0,0,0,0)', whiteSpace: 'nowrap', border: 0,
        }}
      >
        {message}
      </span>
    </AnnouncerContext.Provider>
  );
};

export const useAnnouncer = () => React.useContext(AnnouncerContext);

// Usage anywhere:
const announce = useAnnouncer();
announce(t("Folders loaded successfully!"));
```

### Skip Link Pattern

```tsx
// makeStyles
const useStyles = makeStyles({
  skipLink: {
    position: 'absolute',
    top: '-100px',
    left: tokens.spacingHorizontalM,
    zIndex: 9999,
    padding: `${tokens.spacingVerticalS} ${tokens.spacingHorizontalM}`,
    backgroundColor: tokens.colorNeutralBackground1,
    color: tokens.colorNeutralForeground1,
    textDecoration: 'none',
    ':focus': {
      top: tokens.spacingVerticalM,
      outline: `3px solid ${tokens.colorStrokeFocus2}`,
      outlineOffset: '2px',
    },
    '@media (forced-colors: active)': {
      ':focus': { backgroundColor: 'Canvas', color: 'CanvasText', outlineColor: 'Highlight' },
    },
  },
});

// JSX — must be the first focusable element in the DOM
<a className={styles.skipLink} href="#mainContent">{t("Skip to main content")}</a>
```

### Route Change Focus Management

```tsx
// In Layout.tsx
const location = useLocation();
const announce = useAnnouncer();
const pageTitles: Record<string, string> = {
  '/': t('Chat'), '/content': t('Manage Content'), /* ... */
};

React.useEffect(() => {
  const title = pageTitles[location.pathname] ?? t('Page not found');
  document.title = `${title} — EVA`;
  announce(title);
  setTimeout(() => {
    const h2 = document.querySelector<HTMLElement>('#mainContent h2');
    h2?.focus();
  }, 150);
}, [location.pathname]);
```

### Focus Ring (universal)

```ts
// Apply to every interactive element that doesn't inherit Fluent's ring
':focus-visible': {
  outline: `3px solid ${tokens.colorStrokeFocus2}`,
  outlineOffset: '2px',
},
'@media (forced-colors: active)': {
  ':focus-visible': { outline: '3px solid Highlight', outlineOffset: '2px' },
},
```

### Radio Group Pattern (toggle button sets)

```tsx
<div role="radiogroup" aria-label={t("Response length")}>
  {[
    { label: t("Concise"), value: 1024 },
    { label: t("Balanced"), value: 2048 },
    { label: t("Detailed"), value: 3072 },
  ].map(({ label, value }) => (
    <ToggleButton
      key={value}
      role="radio"
      aria-checked={responseLength === value}
      checked={responseLength === value}
      onClick={() => setResponseLength(value)}
    >
      {label}
    </ToggleButton>
  ))}
</div>
```

### Dialog Focus Trap + Escape + Return

Fluent v9 `<Dialog>` traps focus automatically. You must:
1. Store the trigger ref
2. Call `triggerRef.current?.focus()` in `onOpenChange` when `open` becomes `false`

```tsx
const triggerRef = useRef<HTMLButtonElement>(null);
const [open, setOpen] = useState(false);

const handleClose = () => {
  setOpen(false);
  triggerRef.current?.focus(); // AC-08
};

<button ref={triggerRef} onClick={() => setOpen(true)}>Open</button>

<Dialog open={open} onOpenChange={(_, d) => { if (!d.open) handleClose(); }}>
  <DialogSurface>
    {/* Escape key is handled automatically by Fluent's Dialog */}
    ...
  </DialogSurface>
</Dialog>
```

### Chat Log

```tsx
<div
  role="log"
  aria-live="polite"
  aria-label={t("Chat messages")}
  ref={chatContainerRef}
>
  {answers.map((a, i) => (
    <React.Fragment key={i}>
      <UserChatMessage message={a.user} />
      <Answer answer={a.bot} />
    </React.Fragment>
  ))}
  <div ref={bottomRef} aria-hidden="true" /> {/* auto-scroll anchor */}
</div>
```

---

## High Contrast Mode

Every `makeStyles` call that touches `color`, `backgroundColor`, `borderColor`, `fill`, or `stroke` MUST include a forced-colors block.

```ts
someElement: {
  color: tokens.colorNeutralForeground1,
  backgroundColor: tokens.colorNeutralBackground1,
  border: `1px solid ${tokens.colorNeutralStroke1}`,
  '@media (forced-colors: active)': {
    color: 'CanvasText',
    backgroundColor: 'Canvas',
    borderColor: 'CanvasText',
  },
},
selectedState: {
  backgroundColor: tokens.colorBrandBackground,
  color: tokens.colorNeutralForegroundOnBrand,
  '@media (forced-colors: active)': {
    backgroundColor: 'Highlight',
    color: 'HighlightText',
  },
},
```

**Allowed forced-colors keywords**: `Canvas`, `CanvasText`, `Highlight`, `HighlightText`, `ButtonFace`, `ButtonText`, `GrayText`. No others.

---

## Form Labelling

```tsx
// ✅ Visible label + control
<Field label={t("Select folder")} required>
  <Combobox ... />
</Field>

// ✅ aria-label when visual label is not needed
<Textarea aria-label={t("Enter your question")} ... />

// ❌ Placeholder only — never acceptable
<Input placeholder={t("Type here")} /> // no label = WCAG failure
```

---

## Colour Contrast Requirements

| Content type | Minimum ratio |
|-------------|--------------|
| Normal text (< 18pt) | 4.5:1 |
| Large text (≥ 18pt or ≥ 14pt bold) | 3:1 |
| UI components and icons | 3:1 |
| Placeholder text | 4.5:1 |
| Disabled text | No requirement (but still visible preferred) |

Using `tokens.*` from Fluent v9 webLightTheme satisfies all ratios for standard text and UI. Only custom colours need manual verification.

---

## axe-core Rules Mapping

| axe Rule | Our Mitigation |
|----------|---------------|
| `label` | Every control has `<label>` or `aria-label` |
| `color-contrast` | All colours via `tokens.*` |
| `aria-allowed-attr` | Use correct ARIA roles with correct attributes |
| `aria-required-children` | Follow the patterns above exactly |
| `focus-order-semantics` | DOM order = reading order; no `tabIndex > 0` |
| `landmark-one-main` | One `<main id="mainContent">` per page |
| `page-has-heading-one` | One `<h1>` in the header |
| `region` | All visible content inside a landmark |
| `bypass` | Skip link as first focusable element |
| `document-title` | `document.title` updated on every route change |
| `html-has-lang` | `document.documentElement.lang` = active locale |
