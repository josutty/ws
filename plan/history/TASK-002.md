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

## coder — implementation (GREEN)

Implemented `src/features/auth-login/ui/LoginForm.tsx` (react-hook-form `Controller` wrapping the
shared `Input`/`Button`, `zodResolver`) and extended `src/features/auth-login/index.ts` to export
`LoginForm`. Also added `src/features/auth-login/lib/loginSchema.ts` (zod schema for
`identifier`/`password`) — not in TASK-002's original Files table, added per the TASK's own text
("if the schema file doesn't exist yet, add it as a new row... owner coder") since `lib/**` is
coder-owned here; not committing any change to `plan/tasks/TASK-002.md` itself (architect-owned
per AGENTS.md §11).

- `useLoginMutation` (feature-owned hook) for the sign-in mutation; `useGetCurrentCyclePreviewQuery({})`
  (`@entities/cycle` public API) for the pre-auth cycle banner — hidden silently on error (no
  `isError` branch renders anything).
- Error mapping on `login(...).unwrap()` rejection: `ERR_UNKNOWN_IDENTIFIER` → `setError('identifier', …)`;
  `ERR_WRONG_PASSWORD` → `setError('password', …)`; anything else → `setError('root', …)` rendered as
  a `role="alert"` banner. No `useState` copy of server errors — all routed through react-hook-form's
  own `formState.errors`.
- SSO button and "Forgot password?" are stub handlers that set a local toast message (no real flow,
  per TASK-002.md and api-schema.md open questions).

### GREEN proof

```
npx vitest run src/features/auth-login
```
1 test file, 7 passed, 0 failed.

```
npm run typecheck
```
0 errors.

```
npm test
```
7 files, 29 tests, all passed.

```
npm run lint
```
Exit 1 — but unchanged from the pre-existing state: 1 Steiger error on `src/app/providers`
(not my file) and 8 "insignificant slice" warnings across `entities/*`/`features/*` (incl.
`features/auth-login`, which has no consumer page yet — `plan/coverage-matrix.md` G5 notes no
sign-in page TASK exists in the 16-TASK plan). 0 ESLint errors/warnings on any file I touched.

`git diff --name-only 515fb09` plus untracked files, filtered to my own paths: only
`src/features/auth-login/index.ts`, `src/features/auth-login/ui/LoginForm.tsx`,
`src/features/auth-login/lib/loginSchema.ts`, `plan/history/TASK-002.md` — within/consistent with
TASK-002's Files table (plus the TASK-permitted `lib/` addition).
