# Budget app, release notes

## v3.4.0-beta, category card grid replaces pie chart and progress bars

The pie chart and the progress bar list were both answering "how's my spending by
category" in overlapping ways, made worse once default categories seeded in at zero.
Replaced both with a single 4-wide grid of square cards, one per category.

Each card gets a full background wash in that category's color, a percent-to-budget
badge in the top-right corner, and the budgeted versus actual amounts. A category with
no budget shows -- in the badge instead of a percent, since there's nothing to measure
against yet.

Categories with no budget and no spending stay hidden from the grid entirely, so
seeding twelve default categories doesn't mean seeing twelve empty cards, only ones
you've actually engaged with show up.

Canvas-based pie drawing is gone along with the color legend it needed, categories are
distinguished by their card's own wash now instead of a separate key.

## v3.3.2-beta, default categories restored

v2.5.2 had a fixed set of budget categories (Food, Gas, Groceries, Entertainment,
Shopping, Health, Utilities, Subscriptions, Dining Out, Personal Care, Auto, Other).
That list didn't carry over when Budget was rebuilt for this shell, categories became
fully free-form with nothing pre-filled, which meant a fresh sign-in started with an
empty dropdown everywhere a category is picked. That was a regression, not an
intended change.

Fixed: the same default list now seeds automatically the first time you set up, and
also retroactively for any account that's already signed in with zero categories, so
existing users pick it up on their next load too. Since categories here are editable
entries rather than a locked list, you can still rename, delete, or add more on top,
"Other" is just one of the defaults now rather than a special fallback.

## v3.3.1-beta, nav styling tweak

Nav text is larger, "Bill Tracking" is back to one line instead of stacked, and thin
vertical dividers now separate each nav item. No functional changes.

## v3.3.0-beta, Bill Tracking, running debt balances, history, recurring dismiss

**Bill Tracking replaces the separate Recurring Charges and Debts tabs.** One screen,
two sections. Recurring charges keep their own form and table at the top, debts sit
below. The standalone Debts nav item is gone.

**Recurring charges now have a real paid status.** "Mark paid" opens a small modal
asking which account it came from, deducts that account's balance, logs it to
Register, and flips the badge to Paid for the current cycle. It resets to Due
automatically once the next cycle starts, based on the charge's own paid month rather
than a manual reset.

**Debts get a running balance.** "Log a payment" asks for an amount and an account,
then reduces the debt balance, deducts the account balance, and logs it to Register,
same mechanism as recurring charges. "Schedule a payment" does the same for a future
date. Scheduled payments sit in a visible list and apply themselves automatically once
their date arrives, no confirmation step, checked every time the app loads or you
navigate.

**Low balance alert field is gone.** Removed from the Settings form, the account
table, and the Sheet write. The Accounts tab header dropped from four columns to
three, existing sheets with a leftover Threshold column are unaffected, the app just
stops reading and writing it.

**History lives in the Sheet.** A new History tab captures each closed month's income,
spend, surplus, and category breakdown. This only starts from your setup date, nothing
before it gets backfilled, which is also the boundary Plaid will respect once that's
connected, so bank sync and history never disagree about where the timeline starts. A
compact history list now shows on the Budget screen under Upcoming charges.

**Recurring detection can be dismissed.** A "Not recurring" option next to the "Looks
recurring" badge in Register. Dismissing it adds the description to an ignore list
that persists in the Sheet, so the same false positive won't keep coming back.

### A few things worth knowing

History's income figure for a closed month falls back to your current direct deposit
schedule if no deposit transactions were logged that month, since payroll generally
isn't itemized as a transaction in this app. That's an approximation, worth watching
once you have a few real months to compare against.

Recurring charges and debt payments now both write directly to Register when marked
paid or logged, so Register is becoming the single source of truth for what actually
happened, separate from what's planned.

### Suggested commit message

`Merge recurring charges and debts into Bill Tracking as v3.3.0-beta: real paid
status and account deduction for both, auto-applying scheduled debt payments, sheet-
backed month history from setup date, dismissible recurring detection, low balance
field removed`

## v3.2.0-beta, account balances on Budget, Google Sheets sync

**Account balances on the Budget screen.** A new top row shows each account as a color
coded chip, same style as the Register chips, with a combined total balance shown
underneath. This is read only here, editing still happens in Settings or by logging a
transaction in Register.

**Google Sheets sync.** "Continue with Google" now does a real sign-in and connects to
your existing "Budget App Data" Sheet from v2.5.2, reusing the same Client ID. Your old
tabs (Expenses, Bills, Scheduled, Balances, Transactions) are never read or written to.
Six new tabs hold this app's data: Accounts, Categories, Recurring, Register, Debts,
DirectDeposits. As agreed, nothing from the old tabs is migrated automatically, a fresh
Google sign-in starts empty and opens the setup modal, same as before, except now what
you add actually persists.

Every add, edit, and delete across Budget, Recurring Charges, Register, Debts, and
Settings writes through to the matching tab. Failed writes (offline, expired session)
queue locally and retry automatically the next time the app successfully talks to your
Sheet. Editing an account's balance from a Register transaction updates that account's
row in the Accounts tab too, so balances stay consistent everywhere.

"Try it with sample data" is unchanged, still a local demo that never touches your
Sheet.

"Reset all data" now branches by mode. In sample mode it resets to the sample set like
before. In Google mode, it's a real destructive action against your actual Sheet, so it
requires typing RESET before it clears the six new tabs. Your old v2.5.2 tabs are never
touched by this either.

A new "Open my Sheet" item in the header menu opens your connected Sheet directly, only
visible once you're signed in with Google.

### Known gaps

Color conflict checking (two accounts sharing a color) from v2.5.2 wasn't carried over
yet. Token refresh and offline retry logic is ported from v2.5.2 but hasn't been tested
against a real expired session in this rebuild, worth verifying once you're using it
for real. No calendar view, as noted in the previous release.

### Suggested commit messages

`Redesign app shell as v3.0.0-beta: budget-first nav, recurring detection in
register, debts tracking, first-login setup modal (sample data, no backend yet)`

`Add direct deposit section to settings as v3.1.0-beta: recurring payroll now drives
the budget income calculation instead of manual deposits`

`Add account balances to budget home and reconnect Google Sheets sync as v3.2.0-beta:
reuses the v2.5.2 sheet and client ID with six new tabs for the redesigned data model`

## v3.1.0-beta, direct deposit

Added a Direct deposit section to Settings, under Accounts. This is where regular
payroll lives now: name, amount, paying account, frequency (Weekly, Biweekly, Twice a
month, Monthly), and a next pay date field for cycles that aren't tied to a calendar
day, like every fourteen days.

Budget's Income this month now calculates from these entries instead of summing manual
deposit transactions. A biweekly $2,350 paycheck normalizes to roughly $5,096 a month,
the same normalization approach Recurring Charges already used for weekly and annual
items.

Manual deposits in Register are unchanged, they still log to transaction history and
adjust account balances, they just no longer drive the Income number. That's meant for
one-off money in, like a bonus or a reimbursement, without it inflating your monthly
surplus.

No changes to the Register transaction type dropdown or its plus and minus sign
display, confirmed that's already working as expected.

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
