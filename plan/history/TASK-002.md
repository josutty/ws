start-sha: 515fb09c0265ec2236f62c91c48118db55d657f4

## test-engineer — RED tests

Wrote `src/features/auth-login/ui/LoginForm.test.tsx` (connected component, `renderWithProviders`
+ `@mocks/node` MSW server). Covers the six UI states from TASK-002.md: default, submit-success
(clears loading), invalid-unknown-identifier, invalid-wrong-password, submitting
(button disabled/loading while in flight, ad-hoc `delay('infinite')` handler — no named scenario
exists for a slow login), error (`scenarios.loginServerError` → `role="alert"` banner), and
cycle-banner-error (`scenarios.cyclePreviewServerError` → banner text absent, form still usable).

### RED proof (`npx vitest run src/features/auth-login`)

Whole suite fails to collect for one reason, all 7 tests:
- RED because: `Failed to resolve import "./LoginForm" from "src/features/auth-login/ui/LoginForm.test.tsx". Does the file exist?`

`LoginForm.tsx` is listed `new` in TASK-002's Files table and does not exist yet — valid RED per
AGENTS.md §5, no setup crash.

### Lint

`npm run lint` — 0 ESLint errors/warnings on `LoginForm.test.tsx`. The run's overall exit is 1 from
pre-existing Steiger FSD findings on other agents' files (`src/app/providers`, various
entities/features slices with no references yet) — out of scope here.
