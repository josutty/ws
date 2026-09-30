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
