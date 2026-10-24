# interface-eng-shared-components

Shared UI components for the eight TeSA Cohort 6 capstone products. This repo is **not deployed** and has no app of its own. Copy the components you need into `src/components/ui/` in your product. Then copy `src/styles/tokens.css` and change its values to match your team's look.

Stack: React + TypeScript + Tailwind CSS + Radix UI (see [docs/decisions.md](docs/decisions.md)).

## Working rules

- Work on a branch, open a pull request, get one other interface engineer to review it.
- CI runs `npm run lint` and `npm run typecheck` on every pull request. Both must pass.
- No `any`. No hard-coded colours; use the variables in `tokens.css`.

## Setup

```
npm install
npm run lint
npm run typecheck
```

There is no `build` script: the repo is not deployed, so lint and typecheck are the full CI check ([.github/workflows/ci.yml](.github/workflows/ci.yml), Node 20, `npm ci`).

## Component inventory

Owners come from the file headers and [docs/decisions.md](docs/decisions.md). All paths are under `src/components/ui/`.

| File | Exports | Purpose | Owner | Needs (besides React) |
| --- | --- | --- | --- | --- |
| `button.tsx` | `Button`, `ButtonProps` | Actions and form submits | Muizz Jamiu | `cn` |
| `card.tsx` | `Card`, `CardHeader`, `CardBody`, `CardFooter`, `CardImage` + prop types | One item in a list or grid, optionally a link | Ufoma Onowho | none |
| `checkbox.tsx` | `Checkbox`, `CheckboxProps` | A single yes/no choice | Ehimare J. Okosun | `cn`, `@radix-ui/react-checkbox`, `lucide-react` |
| `dialog.tsx` | `Dialog`, `ConfirmDialog` + prop types | Modal content and confirmations | Victor Oyeleke | `cn`, `button.tsx`, `@radix-ui/react-dialog` |
| `empty-state.tsx` | `EmptyState` + types | First use, or no results for a filter | Ufoma Onowho | `cn`, `button.tsx` |
| `error-state.tsx` | `ErrorState` + types | Something failed to load or save | Ufoma Onowho | `cn`, `button.tsx` |
| `filter-bar.tsx` | `FilterBar`, `FilterGroup` + prop types | Search and filters above a list | Ehimare J. Okosun | `cn`, `button.tsx`, `form-field.tsx`, `input.tsx` |
| `form-field.tsx` | `FormField`, `FormFieldProps`, `FieldControlProps` | Label, hint, error and required wiring for any control | Muizz Jamiu | `cn` |
| `input.tsx` | `Input`, `InputProps` | Single-line text input | Muizz Jamiu | `cn` |
| `select.tsx` | `Select`, `SelectOption`, `SelectProps` | Pick one option from a list | Ehimare J. Okosun | `cn`, `@radix-ui/react-select`, `lucide-react` |
| `skeleton.tsx` | `Skeleton`, `SkeletonProps` | Loading placeholder | Ufoma Onowho | `cn` |
| `status-badge.tsx` | `StatusBadge` + types | Shows an item's current status | Ufoma Onowho | `cn` |
| `textarea.tsx` | `Textarea`, `TextareaProps` | Multi-line text input, with an optional character count | Muizz Jamiu | `cn` |
| `toast.tsx` | `ToastProvider`, `useToast` + types | Short notifications after an action | Victor Oyeleke | `cn`, `@radix-ui/react-toast` |

`cn` is [src/lib/cn.ts](src/lib/cn.ts) (`clsx` + `tailwind-merge`).

### Button

Props: `variant` (`"primary"` default, `"secondary"`, `"danger"`, `"ghost"`), `size` (`"sm"`, `"md"` default, `"lg"`), `loading`, `iconOnly`, plus all native `<button>` attributes. Forwards its ref.

- It defaults to `type="button"`, so pass `type="submit"` on form submit buttons.
- `loading` shows a spinner, keeps the button's width and sets `aria-busy`. Further clicks are ignored.
- `iconOnly` requires `aria-label`. TypeScript enforces this.

```tsx
<Button type="submit" loading={isSaving}>Save changes</Button>
<Button iconOnly aria-label="Delete item" variant="ghost"><TrashIcon /></Button>
```

### Card

`Card` props: `children` (required), `href`, `linkLabel`, `image`, `imageAlt` (default `""`), plus native `<div>` attributes. `CardHeader`, `CardBody` and `CardFooter` take `children` (required) and `<div>` attributes. `CardImage` takes a required `alt`, an optional `src` and `<img>` attributes. Without `src`, it shows the same "Image unavailable" placeholder as Card.

- `Card` always renders a 16:9 image area at the top. Without `image`, it shows an "Image unavailable" placeholder that is hidden from screen readers.
- With `href`, the whole card becomes one link that covers the card. Pass `linkLabel` as its accessible name. Don't put other buttons or links inside a linked card, because the link sits on top of them.

```tsx
<Card href={`/events/${event.id}`} linkLabel={`View ${event.name}`} image={event.photoUrl} imageAlt="">
  <CardHeader><h3 className="text-lg font-semibold">{event.name}</h3></CardHeader>
  <CardBody><p>{event.summary}</p></CardBody>
  <CardFooter><StatusBadge tone="success">Open</StatusBadge></CardFooter>
</Card>
```

### Checkbox

Props: `label` (required), `checked` (required), `onCheckedChange(checked: boolean)` (required), `description`, `disabled`, `name`, `value`, `required`, `id`, `className`. It is controlled only.

```tsx
<Checkbox label="Send me reminders" checked={reminders} onCheckedChange={setReminders} />
```

### Dialog and ConfirmDialog

`Dialog` props: `title` and `description` (both required, so screen readers always announce them), `trigger`, `children`, `footer`, `open`, `defaultOpen`, `onOpenChange`, `onOpenAutoFocus`, `className`.

`ConfirmDialog` takes the same props except `footer` and `onOpenAutoFocus`. It also takes `confirmLabel` (required: name the action, never "OK"), `onConfirm` (required, may return a promise), `cancelLabel` (default `"Cancel"`), `tone` (`"default"` default, or `"danger"`) and `loading`. Its `children` are shown between the description and the buttons.

- If `onConfirm` returns a promise, the confirm button shows its loading state until the promise settles.
- The dialog closes when the promise resolves. If it throws, the dialog stays open and you report the error yourself, for example with a Toast.
- While the confirm is running, Escape, an overlay click and the close button do not close the dialog.

```tsx
<ConfirmDialog
  trigger={<Button variant="danger">Delete item</Button>}
  title="Delete this item?"
  description="This removes it from your stock list. You cannot undo this."
  confirmLabel="Delete item"
  tone="danger"
  onConfirm={async () => { await deleteItem(id); }}
/>
```

### EmptyState

Props: `title` and `description` (required), `icon`, `action` (`{ label, onClick }`, rendered as the shared Button), `headingLevel` (`"h2"` default to `"h6"`), plus `<div>` attributes.

Use one component for both cases: first use ("No events yet" / "Add your first event") and no filter results ("No events match these filters" / "Clear filters"). Write copy that invites an action instead of apologising.

```tsx
<EmptyState
  title="No events match these filters"
  description="Try a different search or clear the filters."
  action={{ label: "Clear filters", onClick: clearFilters }}
/>
```

### ErrorState

Props: `title` and `description` (required, **strings only**), `icon`, `headingLevel`, `onRetry`, `retryLabel` (default `"Try again"`), `retrying`, plus `<div>` attributes (`role` is fixed to `"alert"`).

Never pass an `Error` or `error.message`. Map the failure to plain copy first, so users never see raw codes. Set `retrying` while the retry is in flight.

```tsx
<ErrorState
  title="We couldn't load your events"
  description="Check your connection and try again."
  onRetry={() => refetch()}
  retrying={isFetching}
/>
```

### FilterBar and FilterGroup

`FilterBar` props: `query`, `onQueryChange`, `searchLabel` (visible label), `activeCount` and `onClearAll` (all required), `searchHint`, `searchPlaceholder`, `children` (your Select, Checkbox or FilterGroup filters), `resultMessage`, `label` (default `"Filters"`), `className`.

- It holds no state. Your page owns the filter values and passes them in.
- `activeCount` counts the filters in `children`. A non-empty search is added for you.
- Set `resultMessage` when results arrive, not on every keystroke. Leave it `undefined` while loading.

`FilterGroup` props: `legend` and `children` (both required), `className`. It wraps related checkboxes in a `<fieldset>`.

```tsx
<FilterBar
  query={query}
  onQueryChange={setQuery}
  searchLabel="Search by patient name or reference"
  activeCount={(status ? 1 : 0) + (urgentOnly ? 1 : 0)}
  onClearAll={clearAll}
  resultMessage={data ? `${data.length} appointments found` : undefined}
>
  <FormField label="Status">
    {(field) => <Select {...field} options={statusOptions} value={status} onValueChange={setStatus} allowAny />}
  </FormField>
  <FilterGroup legend="Priority">
    <Checkbox label="Urgent only" checked={urgentOnly} onCheckedChange={setUrgentOnly} />
  </FilterGroup>
</FilterBar>
```

### FormField

Props: `label` (required), `children` (required, a function that receives the props to spread on the control), `hint`, `error`, `required`, `id`, `className`.

It ties the label to the control and links the hint and error with `aria-describedby`. It sets `aria-invalid` when there is an error and `aria-required` when the field is required. It works with Input, Textarea and Select.

```tsx
<FormField label="Food name" hint="As donors would say it" error={errors.title?.message} required>
  {(field) => <Input {...field} {...register("title")} />}
</FormField>
```

### Input

Props: every native `<input>` attribute (`type` defaults to `"text"`). Forwards its ref, so it works with React Hook Form's `register()`. Error styling switches on from `aria-invalid`, which FormField sets. Always use it inside FormField.

```tsx
<FormField label="Quantity" required>
  {(field) => <Input type="number" {...field} {...register("quantity")} />}
</FormField>
```

### Select

Props: `options` (`{ value, label, disabled? }[]`, every `value` must be a non-empty string), `value` (`string | undefined`), `onValueChange` (all required), `placeholder` (default `"Select…"`), `allowAny`, `anyLabel` (default `"Any"`), `disabled`, `name`, `required`, `id`, ARIA props, `className`, `contentClassName`.

- `allowAny` adds an "Any" option that maps to `undefined`. Use it in filters.
- Always wrap Select in FormField, because `placeholder` is not a label.
- Use `aria-label` only when there is no visible label, which should be rare.

```tsx
<FormField label="Region">
  {(field) => <Select {...field} options={regions} value={region} onValueChange={setRegion} />}
</FormField>
```

### Skeleton

Props: `shape` (`"text"` default, `"image"`, `"card"`), `lines` (for `"text"`, default 1, clamped to 1–20), plus `<div>` attributes.

- Skeleton is always hidden from screen readers. Put `aria-busy="true"` on the region being filled instead.
- `"card"` matches the Card layout.

```tsx
<ul aria-busy="true" className="grid gap-4">
  {[1, 2, 3].map((n) => <li key={n}><Skeleton shape="card" /></li>)}
</ul>
```

### StatusBadge

Props: `children` (required, the status written as text), `tone` (`"neutral"` default, `"info"`, `"success"`, `"warning"`, `"danger"`), `icon` (decorative), plus `<span>` attributes.

It shows the current state. It does not announce changes; use Toast for those.

Tokens used by each tone:

| Tone | Background | Text | Border |
| --- | --- | --- | --- |
| `neutral` | `--color-surface` | `--color-text` | `--color-border-input` |
| `info` | `--color-primary` | `--color-on-primary` | none (transparent) |
| `success` | `--color-success` | `--color-on-success` | none (transparent) |
| `warning` | `--color-warning` | `--color-on-warning` | none (transparent) |
| `danger` | `--color-danger` | `--color-on-danger` | none (transparent) |

`tokens.css` has no info colour, so `info` borrows the primary role. Changing `--color-primary` also changes info badges. `neutral` sits on the same surface as Card, so it gets a visible border to keep its edge. The default text contrast is 16.1:1 for neutral, and the role figures in [Contrast](#contrast) for the others.

```tsx
<StatusBadge tone="warning">Pending review</StatusBadge>
```

### Textarea

Props: every native `<textarea>` attribute (`rows` defaults to 4), plus `showCount`. Forwards its ref, so it works with React Hook Form's `register()`. Formik's `getFieldProps()` works too. Error styling switches on from `aria-invalid`, which FormField sets. Users can resize it vertically only. Always use it inside FormField.

- `showCount` together with `maxLength` shows a count such as "120 / 500" under the field. Without `maxLength`, `showCount` does nothing.
- The count is linked to the field with `aria-describedby`, so screen readers hear "120 of 500 characters used" when the field gets focus.
- It is announced politely while typing only in the last 20 characters, so screen readers aren't interrupted on every keystroke.

```tsx
<FormField label="Delivery notes" hint="Gate code, landmarks, anything the driver needs" error={errors.notes?.message}>
  {(field) => <Textarea {...field} {...register("notes")} maxLength={500} showCount />}
</FormField>

// Formik
<FormField label="Delivery notes" error={formik.touched.notes ? formik.errors.notes : undefined}>
  {(field) => <Textarea {...field} {...formik.getFieldProps("notes")} maxLength={500} showCount />}
</FormField>
```

### Toast

Wrap the app root in `<ToastProvider>` once (`viewportClassName` repositions the stack). Then call `useToast()` anywhere below it. `useToast()` throws if the provider is missing.

`useToast()` returns `{ toast, dismiss }`. `toast()` takes `title` (required), `description`, `variant` (`"info"` default, `"success"`, `"error"`), `action` (`{ label, onClick }`) and `duration` (ms, default 5000), and returns an id you can pass to `dismiss(id)`.

- Success and info toasts dismiss themselves.
- Errors and toasts with an action stay until the user dismisses them.
- At most four toasts are visible. Adding a fifth removes the oldest, even if it is an error or has an action.

```tsx
// main.tsx
<ToastProvider><App /></ToastProvider>

// anywhere inside
const { toast } = useToast();
toast({ variant: "success", title: "Request approved" });
toast({
  variant: "error",
  title: "Could not save your change",
  description: "Your edit was undone. Check your connection and try again.",
  action: { label: "Retry", onClick: () => save() },
});
```

## Using the components

1. Copy each component file you need into your product's `src/components/ui/`. Also copy every file in its "Needs" column (see the table). For example, FilterBar also needs `button.tsx`, `form-field.tsx` and `input.tsx`.
2. Copy [src/lib/cn.ts](src/lib/cn.ts) to `src/lib/cn.ts`. Every component except Card imports it as `../../lib/cn`, so keep the same folder layout or fix the imports.
3. Install the npm packages your chosen components use:
   ```
   npm install clsx tailwind-merge                 # always (cn)
   npm install @radix-ui/react-dialog              # Dialog
   npm install @radix-ui/react-toast               # Toast
   npm install @radix-ui/react-select lucide-react # Select
   npm install @radix-ui/react-checkbox lucide-react # Checkbox
   ```
4. Set up Tailwind CSS in your product. The components are styled only with Tailwind classes, and this repo does not include Tailwind (it is not in `package.json`).
5. Copy and load `tokens.css` (next section).
6. Copy [.claude/skills/shared-components/](.claude/skills/shared-components/) into your product's `.claude/skills/`, so that Claude Code in your product uses the shared set instead of writing its own components.

## Design tokens

Copy [src/styles/tokens.css](src/styles/tokens.css) to `src/styles/tokens.css` in your product. Import it once in your global stylesheet, after Tailwind:

```css
@import "tailwindcss";
@import "./styles/tokens.css";
```

**Change the values, never the names.** The components read these variables by name, so a renamed variable silently stops working. To give your product its own look, edit values such as `--color-primary`, `--radius-md` or `--font-sans`. Don't hard-code colours in components.

The tokens are surfaces (`--color-bg`, `--color-surface`, `--color-border`, `--color-border-input`), text (`--color-text`, `--color-text-muted`), and the roles primary, danger, success and warning. Each filled role has an `--color-on-*` colour for text on top of it, and primary and danger also have `-hover`. The rest are the focus ring (`--color-focus`, `--focus-ring-width`, `--focus-ring-offset`), shape (`--radius-sm`, `--radius-md`) and `--font-sans`.

`--radius` is an alias of `--radius-md`. Dialog and Toast read it, so changing `--radius-md` changes their corners too.

### Contrast

The default values all pass WCAG 2.2 AA. Ratios are computed from the values in `tokens.css` and rounded to one decimal place:

| Foreground | Background | Ratio | Needs |
| --- | --- | --- | --- |
| `--color-text` | `--color-bg` | 17.7:1 | 4.5:1 |
| `--color-text-muted` | `--color-bg` | 7.7:1 | 4.5:1 |
| `--color-primary` | `--color-bg` | 6.7:1 | 4.5:1 |
| `--color-danger` | `--color-bg` | 6.5:1 | 4.5:1 |
| `--color-success` | `--color-bg` | 5.0:1 | 4.5:1 |
| `--color-warning` | `--color-bg` | 4.9:1 | 4.5:1 |
| `--color-focus` | `--color-bg` | 5.2:1 | 3:1 |
| `--color-border-input` | `--color-bg` | 4.8:1 | 3:1 |
| `--color-on-primary` | `--color-primary` | 6.7:1 | 4.5:1 |
| `--color-on-primary` | `--color-primary-hover` | 8.7:1 | 4.5:1 |
| `--color-on-danger` | `--color-danger` | 6.5:1 | 4.5:1 |
| `--color-on-danger` | `--color-danger-hover` | 8.3:1 | 4.5:1 |
| `--color-on-success` | `--color-success` | 5.0:1 | 4.5:1 |
| `--color-on-warning` | `--color-warning` | 4.9:1 | 4.5:1 |

The default `--color-on-*` colours and `--color-bg` are all `#ffffff`, so each role scores the same against white either way. If you change `--color-bg` or an `--color-on-*` colour, check both pairs separately.

If you change a colour, re-check it:

- Text needs 4.5:1 or more against its background.
- Borders and the focus ring need 3:1 or more.
- `--color-border-input` must stay at 3:1 or more, because it is the visible edge of form controls.
- Each `--color-on-*` must still reach 4.5:1 on its role colour, because StatusBadge, Button and Checkbox put text on filled role colours.

## Accessibility

What the components already do. Don't undo it when you use them.

- **Keyboard**
  - Every control is a native element or a Radix primitive, so it works with Tab, Enter and Space.
  - Select: arrows move through options, typing a letter jumps to an option, Escape closes, and focus returns to the trigger.
  - Checkbox: Space toggles it. Clicking its label also toggles it.
  - Dialog: focus moves into the dialog and is trapped there, Escape closes it, and focus returns to the trigger. A `danger` ConfirmDialog starts focus on Cancel.
  - FilterBar: after "Clear all", focus moves to the search box.
  - Toast: Radix's default F8 hotkey moves focus to the notifications region.
- **Screen readers**
  - FormField links the label, hint and error to the control and sets `aria-invalid` and `aria-required`. The visual `*` is hidden from screen readers; they hear "required" instead.
  - Dialog requires `title` and `description`.
  - ErrorState has `role="alert"`. It is only announced when added after page load, so render it when the failure happens.
  - EmptyState and StatusBadge are not live regions.
  - FilterBar has `role="search"` and announces `resultMessage` politely.
  - Toast: errors are announced assertively, success and info politely. Each toast includes a visually hidden word ("Success", "Error", "Information").
  - Skeleton is `aria-hidden`.
  - Icons in EmptyState, ErrorState and StatusBadge are decorative and hidden from screen readers.
  - A loading Button keeps focus and its accessible name, and sets `aria-busy`.
- **Not colour alone.**
  - Errors are written as text with a `!` prefix.
  - Badges always show text.
  - Checked checkboxes show a check mark.
  - Toasts have an icon and a variant word.
  - Error states say in words what went wrong.
- **Focus.** Visible `focus-visible` outlines use `--color-focus`. A linked Card draws the outline around the whole card.
- **Target size.** Button `sm` is 32px tall. Checkbox's hit area is 24×24px. Select options are at least 40px tall. All meet the WCAG 2.2 minimum of 24px.
- **Responsive.**
  - Long text wraps instead of overflowing in Card, EmptyState, ErrorState, StatusBadge and Select options.
  - Dialog fits the screen width minus 2rem and scrolls past 85% of screen height. Its buttons stack on phones.
  - FilterBar stacks on phones and becomes a row from `sm` up.
  - Select's list stays inside the viewport.
  - Skeleton fills its container's width.
- **Reduced motion.** The Skeleton pulse and the Button spinner stop when `prefers-reduced-motion` is on.

Before you merge: do a keyboard-only test, and check once with a screen reader. Card, Skeleton, StatusBadge, EmptyState and ErrorState have passed both. Button and Textarea have passed the keyboard checks that can be verified from their code; their screen reader check is pending (Muizz Jamiu). The other components have no checklist yet.

## Known gaps (need confirmation)

- **Tailwind theme classes.** Checkbox, Select and FilterBar use classes such as `bg-surface`, `text-text`, `text-muted`, `border-border`, `bg-primary`, `text-on-primary`, `outline-focus`, `border-danger`, `bg-surface-hover` and `rounded-control`. These need a Tailwind theme mapping that this repo does not define. The other components use `var(--…)` directly. Confirm the intended mapping with Ehimare J. Okosun before you rely on those styles.
- **Checkbox and Select edges.** These two draw their edge with `border-border`, not `--color-border-input`. If `border-border` is mapped to `--color-border` (`#d4d4d8`, 1.5:1 on `--color-bg`), the edge of the unchecked checkbox and the Select trigger falls below the 3:1 minimum. Map it to `--color-border-input`, or change the class (owner: Ehimare J. Okosun).
- **Toast drops sticky toasts.** The proposal says error toasts stay until the user closes them, and products use an error toast with a Retry action to report a rolled-back update. But Toast keeps at most four and removes the oldest when a fifth arrives, even if that oldest is an error or has an action. A burst of toasts can therefore remove a rollback notice before the user sees it. Until this is fixed, don't rely on an error toast being the only record of a failed save (owner: Victor Oyeleke).
- **Typecheck locally.** `npm run typecheck` fails with "Cannot find module 'lucide-react'" if your `node_modules` is older than `package.json`. Run `npm install` (or `npm ci`) to fix it.
- **`@radix-ui/react-slot`** is listed in `dependencies` but no component imports it.
