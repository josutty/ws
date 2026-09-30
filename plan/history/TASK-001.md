start-sha: 4eb5342f682dbc0c050d094b5aff54018d2cface

## test-engineer — RED tests

Harness: created `test/setup.ts` and `test/test-utils.tsx` (did not exist — first TASK).

Tests written (all pure presentational primitives, plain RTL `render`, no `renderWithProviders`):
- `src/shared/ui/Button.test.tsx`
- `src/shared/ui/Input.test.tsx`
- `src/shared/ui/Select.test.tsx`
- `src/shared/ui/Badge.test.tsx`
- `src/shared/ui/Modal.test.tsx`
- `src/shared/ui/Toast.test.tsx`

### RED proof (`npx vitest run src/shared/ui`)

All 6 suites fail identically, each for the same valid reason — the component file listed as
`new` in TASK-001's Files table does not exist yet:

- Button.test.tsx — RED because: `Failed to resolve import "./Button" from "src/shared/ui/Button.test.tsx". Does the file exist?`
- Input.test.tsx — RED because: `Failed to resolve import "./Input" from "src/shared/ui/Input.test.tsx". Does the file exist?`
- Select.test.tsx — RED because: `Failed to resolve import "./Select" from "src/shared/ui/Select.test.tsx". Does the file exist?`
- Badge.test.tsx — RED because: `Failed to resolve import "./Badge" from "src/shared/ui/Badge.test.tsx". Does the file exist?`
- Modal.test.tsx — RED because: `Failed to resolve import "./Modal" from "src/shared/ui/Modal.test.tsx". Does the file exist?`
- Toast.test.tsx — RED because: `Failed to resolve import "./Toast" from "src/shared/ui/Toast.test.tsx". Does the file exist?`

No setup crash, no syntax error — every failure is an unresolved import of a not-yet-created
`new` file, which is a valid RED per AGENTS.md §5.

### Lint

`npm run lint` — no ESLint errors/warnings on any file listed above; Steiger findings are on
other agents' files (`src/app/providers`, `src/entities/*`, `src/features/*`), out of scope here.

## component-generator — GREEN implementation

Implemented all six presentational primitives + `index.ts` public API + one `.stories.tsx` per
component, exactly per TASK-001's Files table (no other files touched):
`Button.tsx`, `Input.tsx`, `Select.tsx`, `Badge.tsx`, `Modal.tsx`, `Toast.tsx`, `index.ts`, and the
six `.stories.tsx`.

- Styling: semantic tokens only, using the exact literal recipes from `plan/design-system.md`
  (primary/ghost/quiet/add button fills, input default/invalid/disabled, badge tone map). Button
  sizes use the pixel-exact `h-8` (32px) / `h-[27px]` (27px, arbitrary value per design-system.md's
  pixel-perfect exception, since TASK-001 explicitly calls out 32px/27px control heights).
- Modal: focus trap (Tab/Shift+Tab cycle within dialog's focusable descendants, falls back to the
  dialog container itself when none — `tabIndex={-1}`), Escape closes via a `document` keydown
  listener added on open and removed on close/unmount, and focus returns to the previously focused
  element (the trigger) in the same effect cleanup. `role="dialog"` `aria-modal="true"`
  `aria-labelledby` → title `<h3>`.
- Toast: `message === null` renders nothing; otherwise `role="status" aria-live="polite"`.

### Dry-run (DoD) results

- `npx vitest run src/shared/ui` — 6 files / 22 tests, all GREEN, no tests left waiting on
  coder-owned files (none of TASK-001's components need any).
- `npm run typecheck` — clean (0 errors). Fixed one `noUncheckedIndexedAccess` issue in
  `Modal.tsx`'s Tab-trap handler (`first`/`last` from `querySelectorAll` indexing) during
  implementation, before the final clean run.
- `npm run lint` — 0 errors/warnings on any file this TASK owns. The run's overall exit is 1
  (1 error, 8 warnings), but every finding is on pre-existing files outside TASK-001's Files
  table: `fsd/insignificant-slice` warnings on `src/entities/{accuracy,cycle,dealer,indent-line}`
  and `src/features/{add-fert-line,auth-login,edit-indent-line,submit-indent}`, and one
  `fsd/segments-by-purpose` error on `src/app/providers` (app-bootstrap-owned). Confirmed via
  `git status` that none of those paths were touched in this step.
- `npm run build-storybook` — builds clean (only the expected "chunks larger than 500 kB" Vite
  warning, which AGENTS.md §12 flags as a reviewer P2 finding, not an error).

