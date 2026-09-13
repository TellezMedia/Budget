# Budget app, release notes

## v3.0.0-beta, phase 1 shell redesign

This is a frontend-only redesign of the app shell. It runs entirely on sample data or on
an empty, freshly set-up account. It does not yet talk to Google Sheets, Google OAuth,
or the Plaid Cloudflare Worker. That reconnection is phase 2.

### What changed

Navigation is now Budget, Recurring Charges, Register, Debts, Settings. Budget is the
default screen instead of Dashboard.

**Budget** is the new home screen. It shows income, recurring charges, and surplus for
the current month, a spending by category pie chart with the legend on the left, an
inline category manager where you can add a category and a planned monthly amount
without leaving the screen, progress bars for each category against its plan, and a
short upcoming charges list.

**Recurring Charges** replaces Bills. Same fields (amount, due day, account,
frequency), plus an optional category so it rolls into the Budget breakdown. The old
Scheduled screen was folded in here conceptually, a recurring charge with any
frequency covers what Scheduled used to handle for repeating items.

**Register** replaces Transactions and folds in the old Expenses screen. Accounts are
listed at the top, color coded, same as the old Dashboard's account cards. Manual entry
supports expenses, deposits, and transfers in one form. Any expense description that
repeats gets a "Looks recurring" badge, with a button that jumps to Recurring Charges
with the details pre-filled.

**Debts** is new. Tracks credit cards and loans individually: balance, minimum due,
credit limit, utilization, due day, and a free text status field. Kept separate from
Recurring Charges since paying down a balance isn't a monthly expense in the same
sense.

**Settings** replaces Accounts. Same account management as before (add, edit, color,
low balance alert), plus a Bank sync section with Plaid stubbed in as disabled and
labeled "Coming soon," and a Google account section showing sample data or
not-connected status.

**Setup modal** appears on first login (simulated here by "Continue with Google," which
starts from a blank slate). Step one adds accounts with colors. Step two shows the
Plaid connect stub, disabled, skippable. Finishing the modal drops you into the app on
the Budget screen. "Try it with sample data" skips the modal entirely and loads a
representative data set instead, since it's meant as a demo shortcut rather than a
real first login.

### Known gaps, expected for phase 1

No Google Sheets read or write. No Google OAuth. No Plaid connection, sync, or
webhook handling. No calendar view (the old Dashboard calendar was dropped from this
pass, let me know if you want it back as a secondary Budget view). Data resets to the
sample set on reload since nothing persists yet.

### Next up, phase 2 candidates

Reconnect Google Sheets read and write for every screen. Reconnect Google OAuth.
Reconnect the Plaid Worker for PNC and Chime, using the same recurring detection path
that manual entries use in Register. Decide whether the day by day cash flow calendar
comes back as a secondary Budget view.

### Suggested commit message

`Redesign app shell as v3.0.0-beta: budget-first nav, recurring detection in
register, debts tracking, first-login setup modal (sample data, no backend yet)`
