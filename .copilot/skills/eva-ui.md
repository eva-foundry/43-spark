# Skill: @eva/ui + @eva/gc-design-system

> Load this skill when implementing any React component in EVA-JP.
> It encodes the constraints, component catalogue, and common patterns for the EVA shared UI library,
> which wraps Fluent UI React v9 with Government of Canada design tokens.

---

## Core Import Rules

```ts
// ✅ Correct — use @eva/ui wrappers for all covered components
import { EvaButton, EvaInput, EvaDialog, EvaDrawer, EvaDataGrid, EvaBadge, EvaSpinner } from '@eva/ui';
import { GCThemeProvider } from '@eva/gc-design-system';
import { makeStyles, tokens, mergeClasses } from '@fluentui/react-components'; // ok for styling + unwrapped primitives
import { Send28Filled, Broom28Filled } from '@fluentui/react-icons';

// ❌ Never — raw Fluent v9 where an EvaXxx wrapper exists
import { Button, Input, Dialog } from '@fluentui/react-components'; // use EvaButton, EvaInput, EvaDialog

// ❌ Never
import { DefaultButton, Spinner } from '@fluentui/react'; // v8 — forbidden
```

**`@eva/ui` wrapper catalogue**:
| EvaXxx component | Replaces raw Fluent primitive |
|---|---|
| `EvaButton` | `Button`, `ToggleButton` |
| `EvaInput` | `Input` |
| `EvaTextArea` | `Textarea` |
| `EvaSelect` | `Select` |
| `EvaDialog` | `Dialog`, `DialogBody`, `DialogActions` |
| `EvaDrawer` | `Drawer`, `DrawerHeader`, `DrawerBody` |
| `EvaBadge` | `Badge`, `CounterBadge` |
| `EvaDataGrid` | `DataGrid`, `DataGridHeader`, `DataGridBody` |
| `EvaSpinner` | `Spinner` |
| `EvaIcon` | `Icon` wrapper |
| `EvaTabs` | `TabList`, `Tab` |
| `EvaToast` | `Toaster`, `Toast` |
| `EvaSelect` | `Select` |
| `EvaDateRangePicker` | date range selection |
| `EvaJsonViewer` | JSON display |

---

## Styling Pattern

Every component uses `makeStyles`. No exceptions.

```ts
import { makeStyles, mergeClasses, tokens } from '@fluentui/react-components';

const useStyles = makeStyles({
  root: {
    display: 'flex',
    gap: tokens.spacingHorizontalM,
    padding: tokens.spacingVerticalL,
    backgroundColor: tokens.colorNeutralBackground1,
    color: tokens.colorNeutralForeground1,
    // High contrast is ALWAYS required when colour is set:
    '@media (forced-colors: active)': {
      backgroundColor: 'Canvas',
      color: 'CanvasText',
      borderColor: 'CanvasText',
    },
  },
  focused: {
    outline: `3px solid ${tokens.colorStrokeFocus2}`,
    outlineOffset: '2px',
  },
});

const MyComponent = () => {
  const styles = useStyles();
  return <div className={styles.root} />;
};
```

Allowed `tokens.*` namespaces:
- `tokens.colorNeutral*` · `tokens.colorBrand*` · `tokens.colorStatus*` · `tokens.colorPalette*`
- `tokens.spacingHorizontal*` (XS S M L XL XXL) · `tokens.spacingVertical*`
- `tokens.typographyStyles.*` (Body1, Body1Strong, Subtitle1, Subtitle2, Title1, Title2, Title3, LargeTitle, Display)
- `tokens.fontWeight*` · `tokens.fontSize*` · `tokens.lineHeight*`
- `tokens.borderRadius*` · `tokens.strokeWidth*`
- `tokens.colorStrokeFocus2` — for focus rings

---

## Component Catalogue

### Buttons

```tsx
// Standard button
<Button appearance="primary" icon={<Send28Filled />} onClick={handleSend}>
  {t("Send question")}
</Button>

// Icon-only — MUST have aria-label
<Button appearance="subtle" icon={<Broom28Filled />} aria-label={t("Clear chat")} />

// Toggle button (in a radio group)
<div role="radiogroup" aria-label={t("Chat mode")}>
  <ToggleButton
    role="radio"
    aria-checked={mode === 'work'}
    checked={mode === 'work'}
    onClick={() => setMode('work')}
  >
    {t("Work documents only")}
  </ToggleButton>
</div>
```

### Inputs and Forms

```tsx
// Labeled Textarea
<Field label={t("Enter your question")} required>
  <Textarea
    value={value}
    onChange={(_, d) => setValue(d.value)}
    onKeyDown={(e) => {
      if (e.key === 'Enter' && !e.shiftKey) { e.preventDefault(); onSend(); }
    }}
    resize="vertical"
  />
</Field>

// Combobox (type-to-filter)
<Field label={t("Select folder")}>
  <Combobox
    placeholder={t("Select folder")}
    value={selectedFolder}
    onOptionSelect={(_, d) => setSelectedFolder(d.optionValue ?? '')}
  >
    {folders.map(f => <Option key={f} value={f}>{f}</Option>)}
  </Combobox>
</Field>
```

### Dialog (focus trap built-in)

```tsx
const triggerRef = useRef<HTMLButtonElement>(null);

<Dialog
  open={open}
  onOpenChange={(_, d) => {
    setOpen(d.open);
    if (!d.open) triggerRef.current?.focus(); // AC-08, AC-15
  }}
>
  <DialogTrigger disableButtonEnhancement>
    <Button ref={triggerRef}>{t("Preview")}</Button>
  </DialogTrigger>
  <DialogSurface>
    <DialogTitle>{t("Preview links")}</DialogTitle>
    <DialogBody>
      {/* content */}
    </DialogBody>
    <DialogActions>
      <DialogTrigger disableButtonEnhancement>
        <Button appearance="secondary">{t("Cancel")}</Button>
      </DialogTrigger>
      <Button appearance="primary" onClick={handleSubmit}>{t("Submit selected")}</Button>
    </DialogActions>
  </DialogSurface>
</Dialog>
```

### Drawer (with focus return)

```tsx
const triggerRef = useRef<HTMLButtonElement>(null);

<Drawer
  type="overlay"
  position="left"
  open={open}
  onOpenChange={(_, d) => {
    setOpen(d.open);
    if (!d.open) triggerRef.current?.focus();
  }}
>
  <DrawerHeader><DrawerHeaderTitle>{t("Chat history")}</DrawerHeaderTitle></DrawerHeader>
  <DrawerBody>{/* content */}</DrawerBody>
</Drawer>
<Button ref={triggerRef} icon={<History24Regular />} aria-label={t("Open chat history")} onClick={() => setOpen(true)} />
```

### DataGrid (for FileStatus)

```tsx
<DataGrid
  items={files}
  columns={columns}
  sortable
  getRowId={(item) => item.id}
  focusMode="composite" // keyboard cell navigation
>
  <DataGridHeader>
    <DataGridRow>
      {({ renderHeaderCell }) => <DataGridHeaderCell>{renderHeaderCell()}</DataGridHeaderCell>}
    </DataGridRow>
  </DataGridHeader>
  <DataGridBody<FileItem>>
    {({ item, rowId }) => (
      <DataGridRow<FileItem> key={rowId}>
        {({ renderCell }) => <DataGridCell>{renderCell(item)}</DataGridCell>}
      </DataGridRow>
    )}
  </DataGridBody>
</DataGrid>
```

### TabList

```tsx
<TabList selectedValue={tab} onTabSelect={(_, d) => setTab(d.value as string)}>
  <Tab value="upload">{t("Upload Files")}</Tab>
  <Tab value="status">{t("Upload Status")}</Tab>
  {isAdminOrContributor && <Tab value="scraper">{t("URL Scraper")}</Tab>}
</TabList>
<div role="tabpanel" aria-labelledby={`tab-${tab}`}>
  {tab === 'upload' && <UploadFilesPanel />}
  {tab === 'status' && <FileStatus />}
  {tab === 'scraper' && <Urlscrapper />}
</div>
```

### MessageBar

```tsx
// Success
<MessageBar intent="success">
  <MessageBarBody>{t("Translation successful")}</MessageBarBody>
</MessageBar>

// Warning (WarningBanner)
<MessageBar intent="warning" aria-live="polite">
  <MessageBarBody>{warningText}</MessageBarBody>
</MessageBar>

// Error (assertive for blocking)
<div aria-live="assertive">
  <MessageBar intent="error">
    <MessageBarBody>
      {t("An error occurred.")}
      <ul>{contacts.map(c => <li key={c}><a href={`mailto:${c}`}>{c}</a></li>)}</ul>
    </MessageBarBody>
  </MessageBar>
</div>
```

### Spinner (loading state)

```tsx
// Full page
<div style={{ display: 'flex', justifyContent: 'center', alignItems: 'center', height: '100vh' }}>
  <Spinner label={t("Loading...")} />
</div>

// Inside a button
<Button disabled={loading} icon={loading ? <Spinner size="tiny" /> : <TranslateFilled />}>
  {loading ? t("Translating...") : t("Translate")}
</Button>
```

### ProgressBar (file upload)

```tsx
<ProgressBar
  value={uploadProgress}
  max={100}
  aria-label={t("Upload progress")}
  aria-valuenow={uploadProgress}
/>
```

---

## Focus Ring Pattern (`:focus-within` on containers)

```ts
const useStyles = makeStyles({
  inputContainer: {
    border: `1px solid ${tokens.colorNeutralStroke1}`,
    borderRadius: tokens.borderRadiusMedium,
    ':focus-within': {
      outline: `3px solid ${tokens.colorStrokeFocus2}`,
      outlineOffset: '2px',
    },
    '@media (forced-colors: active)': {
      ':focus-within': {
        outline: '3px solid Highlight',
      },
    },
  },
});
```

---

## Theme Setup

```ts
// src/index.tsx — use GCThemeProvider, never bare FluentProvider at root
import { GCThemeProvider } from '@eva/gc-design-system';
import { AnnouncerProvider } from './components/Annoucement/AnnouncerProvider';

root.render(
  <GCThemeProvider>
    <AnnouncerProvider>
      <App />
    </AnnouncerProvider>
  </GCThemeProvider>
);
```

`GCThemeProvider` internally wraps `FluentProvider` with the GC design token overrides. Do **not** add another `FluentProvider` inside it.

---

## Common Pitfalls

| Pitfall | Correct approach |
|---------|-----------------|
| `<DefaultButton>` from v8 | `<EvaButton>` from `@eva/ui` |
| `<Spinner>` from v8 | `<EvaSpinner>` from `@eva/ui` |
| `<Panel>` from v8 | `<EvaDrawer>` from `@eva/ui` |
| `<Dialog>` from v8 | `<EvaDialog>` from `@eva/ui` |
| `<DetailsList>` from v8 | `<EvaDataGrid>` from `@eva/ui` |
| `<PrimaryButton>` v8 | `<EvaButton appearance="primary">` |
| `<IconButton>` v8 | `<EvaButton icon={<.../>} aria-label="...">` |
| Raw `<Button>` from v9 | `<EvaButton>` from `@eva/ui` |
| Raw `<Input>` from v9 | `<EvaInput>` from `@eva/ui` |
| Raw `<Dialog>` from v9 | `<EvaDialog>` from `@eva/ui` |
| Bare `<FluentProvider>` at root | `<GCThemeProvider>` from `@eva/gc-design-system` |
| Inline `style={{ color: '#0078d4' }}` | `makeStyles` with `tokens.colorBrandForeground1` |
| `mergeStyles` from `@fluentui/merge-styles` | `makeStyles` + `mergeClasses` |
