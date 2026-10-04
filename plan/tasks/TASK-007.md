---
id: TASK-007
status: open
revision: 1
depends-on: [TASK-001]
created: 2026-10-01
---

# TASK-007: features/submit-indent — CommitBar, ReviewSubmitModal

> **Open item (G4, flagged not blocking):** `reopenDealerIndent` (CommitBar's "Reopen" button) has
> an endpoint (mock/proposed) and a slice placement, but `plan/generated/backend-gaps.md` notes
> `inputs/fsd-spec.md` §5 never documents a dealer-initiated "reopen before cutoff" step (only tier
> lock/auto-submit/escalation). Per architect dispatch this TASK **includes** the Reopen button and
> its mocked endpoint call (do not drop the UI), but the business rule — whether reopening is
> self-service or requires ASM approval — is **unconfirmed**. Reviewer/human: confirm this before
> treating the current contract (client-initiated, no approval step, per
> `reopenDealerIndent`'s spec) as final. If the rule changes, this TASK needs a new revision.

## Files
| Path | Kind | Owner | new/edit |
|---|---|---|---|
| src/features/submit-indent/ui/CommitBar.tsx | connected | coder | new |
| src/features/submit-indent/ui/ReviewSubmitModal.tsx | connected | coder | new |
| src/features/submit-indent/index.ts | public api | coder | new |
| src/features/submit-indent/ui/CommitBar.test.tsx | test | test-engineer | new |
| src/features/submit-indent/ui/ReviewSubmitModal.test.tsx | test | test-engineer | new |

Agents may touch ONLY these files. Reviewer diffs against this table.

## Integration contract
- Server data: `useGetDealerCycleSummaryQuery` (`entities/cycle`, status refresh), own
  `useGetDealerSubmitPreviewQuery` (`api/submitApi.ts`, ReviewSubmitModal only, disabled until the
  modal opens).
- Client state: none — `filtered`/lock state passed down from `widgets/indent-grid`'s D23-adjacent
  local state, not a slice.
- Mutations: own `useSubmitDealerIndentMutation`, `useReopenDealerIndentMutation` — both invalidate
  `DealerCycleSummary` + `IndentLine` LIST (per api-integration-points.md).
- Errors: `getDealerSubmitPreview` failure → banner + retry inside `ReviewSubmitModal`, Submit
  button disabled until loaded; `submitDealerIndent` failure → inline modal error banner, modal
  stays open; 409 (`submitConflict` scenario, already submitted/past cutoff) → banner + "Cancel"
  only (Submit button removed, not just disabled); `reopenDealerIndent` failure → toast; 409
  (`reopenConflict` scenario, past cutoff) → toast + Reopen button disabled.
- No fetch/axios/useState for server data (AGENTS.md §3).

## Components

### CommitBar (connected)
- Props: `submitted: boolean; lineCount: number; julyUnits: number; totalUnits: number;
  missingCount: number; filtered: boolean; onReviewSubmit: () => void; onViewSent: () => void`
  (matches components.md; `onReopen` handled internally via `useReopenDealerIndentMutation`, not a
  prop, since this component owns that mutation directly per the integration contract above)
- Shows (not-submitted): line count, July units, Jul–Sep total, count of lines with no entry (if
  any), filtered-view note when active, "Review & submit" button. Shows (submitted): check mark,
  "`{vertical}` submitted · `{n}` lines · `{n}` units for July", "View what was sent" and "Reopen"
  buttons.
- Interactions: "Reopen" calls `useReopenDealerIndentMutation` directly (see G4 note above) —
  disabled + loading while pending, disabled permanently on a 409.

### ReviewSubmitModal (connected)
- Props: `open: boolean; onClose: () => void` (all summary figures sourced from
  `useGetDealerSubmitPreviewQuery`, not passed in as props, so the preview is always fresh when the
  modal opens)
- Shows: subheading (cycle · period · dealer · vertical); conditional notices — filtered-view
  warning, missing-entries warning (with "Review them first" link that closes the modal and jumps
  to the `noentry` filter — implemented as an `onReviewMissing?: () => void` prop, wired by the
  composing page), added-lines notice; summary (Lines, July W1–W4, August, September, delta/% vs
  last cycle); closing note (next-tier recipient, resubmit deadline); Cancel / "Submit indent".
- Interactions: "Submit indent" calls `useSubmitDealerIndentMutation`; on success closes the modal
  and shows a confirmation toast; button shows `loading` and is disabled while submitting (AGENTS.md
  §12 Forms — source UX draws this synchronously with no loading state, a gap `ui-states.md`
  itself flags; this TASK implements the standard, not the source gap).

## UI states (from analysis/ui-states.md)
| State | Rendered by | Expected UI |
|---|---|---|
| not-submitted-no-missing | CommitBar | stat row + "Review & submit" |
| not-submitted-some-missing | CommitBar | additional amber "`{n}` with no entry" stat |
| filtered-view | CommitBar | amber filtered-total note |
| submitted | CommitBar | check mark, submitted summary, "View what was sent" + "Reopen" |
| reopen-pending | CommitBar | Reopen disabled + loading |
| reopen-conflict (409) | CommitBar | toast, Reopen disabled (`reopenConflict` scenario) |
| clean | ReviewSubmitModal | only summary block + closing note |
| filtered-warning | ReviewSubmitModal | amber notice re: full-indent submission |
| missing-entries-warning | ReviewSubmitModal | amber notice + "Review them first" link |
| added-lines-notice | ReviewSubmitModal | info notice re: ASM flag |
| submitting | ReviewSubmitModal | Submit button loading + disabled (AGENTS.md §12, not in source) |
| preview-error | ReviewSubmitModal | banner + retry, Submit disabled until loaded |
| submit-conflict (409) | ReviewSubmitModal | banner + Cancel-only (`submitConflict` scenario) |

## Tests (test-engineer writes RED first)
- CommitBar.test.tsx: not-submitted renders stat row and "Review & submit"; `missingCount > 0`
  shows the amber stat; `filtered=true` shows the filtered note; `submitted=true` renders check
  mark + "View what was sent"/"Reopen"; clicking Reopen calls `reopenDealerIndent`;
  `reopenConflict` scenario → toast + Reopen disabled.
- ReviewSubmitModal.test.tsx: renders summary from `getDealerSubmitPreview`; `submitPreviewServerError`
  scenario → banner + retry, Submit disabled; filtered/missing/added flags each render their notice
  independently; "Submit indent" calls `submitDealerIndent`, shows loading+disabled while pending,
  closes on success with a confirmation toast; `submitConflict` scenario → banner, only Cancel
  shown (Submit button absent).
- Coverage target: 80% (features).

## Styling
Semantic tokens only. No `dark:`.

## Teardown
- None beyond `Modal`'s own (TASK-001) inside ReviewSubmitModal.

## Dry-run (DoD)
```bash
npm run typecheck && npm run lint && npx vitest run src/features/submit-indent
```

## Reflections applied
- none (no entries under notes/memory/reflections/)
