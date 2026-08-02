# Ledger rules

Single source of truth for how this Beancount ledger is organized and
written. Every `.bean` file points here instead of repeating these
rules, so a new year file (e.g. `2027.bean`) starts clean and this
document only needs to be updated in one place.

## 1. File organization

- `main.bean` — entry point. Includes every other file, in load
  order. Contains no accounts or transactions of its own.
- `accounts.bean` — chart of accounts. Included once, by `main.bean`,
  and nowhere else.
- `prices.bean` — foreign currency / investment prices.
- `YYYY.bean` (e.g. `2026.bean`) — one file per calendar year of
  transactions. Add a new `include "YYYY.bean"` line in `main.bean`
  when a new year starts.

## 2. Comment separators

Applies to every `.bean` file.

- A three-line `====` block (blank line before and after) marks a
  top-level structural section: in `main.bean`, a distinct included
  file; in `accounts.bean`, one of the four account types (Assets,
  Liabilities, Income, Expenses) or, within Expenses, a budget-category
  tier (Future, Essential, Nonessential, Unknown).
- A single-line `; --- Label ---` comment marks something one level
  down: a specific entry within a section (e.g. one year of
  transactions in `main.bean`) or a topic subgroup within a tier
  (e.g. Cash, Bank accounts, Food, Transport in `accounts.bean`).

## 3. Transaction rules (year files)

- **Beneficiary rules**
  - Do not write the ledger owner's name for beneficiaries.
  - Leave your own expenses without a beneficiary tag (implicitly
    yours).
  - Tag other people using the `beneficiary` metadata, indented
    under their specific expense line.
  - For split expenses, create separate posting lines and tag only
    the other person on their respective line.

- **Metadata and formatting**
  - Location-specific metadata (such as `beneficiary` or `property`)
    goes under the specific posting line, never under the main
    transaction header.
  - Start every transaction date at the absolute left margin, with
    no indentation.
  - Keep narrations short and concise. Use the `note` metadata on
    the following line for long explanations or context.
  - Omit the `time` and `location` lines entirely if specific
    transaction details are unknown.
  - Do not repeat account names or line amounts in the main
    narration if the data is already in the posting lines.
  - State every posting's amount explicitly, including the
    balancing line (write it as a negative number rather than
    leaving it for Beancount to infer). This keeps every outflow and
    inflow visible at a glance.

- **Chronological sorting**
  - Keep all transactions strictly organized in chronological order
    by date.

- **Tag standardization**
  - Write all standard custom metadata values (such as
    `transport_mode`) entirely in lowercase.
  - Maintain correct capitalization and spelling for proper nouns
    inside tags (such as beneficiary names or specific property
    locations).

- **Transport mode tracking**
  - Use the `transport_mode` tag, indented under transport posting
    lines, to track the specific vehicle type.

## 4. Chart of accounts rules (`accounts.bean`)

- **Naming**
  - Disambiguation rule: only use a budget-category suffix in the
    account name to disambiguate a broad category that is split
    (e.g. `Expenses:Food:Essential` vs. `Expenses:Food:Nonessential`).
  - Absolute categories: if an account strictly belongs to one
    budget category, omit the suffix (e.g. `Expenses:Health`,
    `Expenses:Entertainment`).
  - Specificity: asset and liability accounts must explicitly state
    the institution name.
  - Person suffix: only `Assets:Receivables`, `Liabilities:Payables`,
    and `Income:Gifts` use a person's full name as a suffix, since
    these track an actual balance owed to or by that person. Expense
    accounts never get a person suffix, even for gifts — use the
    `beneficiary` metadata on the posting line instead (see section 5
    below).

- **Metadata**
  - Every expense account must have a `budget_category` metadata
    tag.
  - Tags must be strictly lowercase: `"future"`, `"essential"`, or
    `"nonessential"`.

- **Person ordering**
  - Wherever more than one person appears within the same section
    (Receivables, Payables, or, once more than one person's gift
    income exists, Income:Gifts), list them in this fixed order:
    Rocío Vanegas Misas, Luna Marysol Rodríguez Vanegas, Felipe
    Arbeláez Rodríguez, Luisa Fernanda Lozano Eljach.

- **Account lifecycle**
  - `Assets:Receivables` and `Liabilities:Payables` are opened for
    all four people up front, since a loan can happen with any of
    them at any time and you'd rather have the account ready than
    discover it's missing mid-transaction.
  - `Income:Gifts` and all topic/category accounts (Food, Transport,
    Housing, etc.) are opened lazily, only once a transaction
    actually needs them, to keep the chart lean. Remove an
    account here if it stops being used — it costs nothing to
    reopen it later with a fresh date.

## 5. Tracking people: metadata vs. a dedicated account

Use this test: **does the amount need to net to a real, standalone
balance that must eventually reach zero?**

- **Yes → dedicated account.** A loan you gave or received has to be
  tracked as an actual balance until it's repaid — that's what
  `Assets:Receivables:X` and `Liabilities:Payables:X` are for. The
  account *is* the balance; there's no other way to know how much
  Luisa currently owes you.
- **No → metadata tag.** A gift, a shared meal, a shared ride is
  spent and gone — there's nothing to settle later, you just want to
  remember who it was for. Use the `beneficiary` tag on a normal
  expense posting (`Expenses:Gifts`, `Expenses:Food:Dining`, etc.)
  instead of creating `Expenses:Gifts:PersonName`. This also means
  you never need to create a new account just because you gave a
  gift to someone for the first time — the tag handles it, and you
  can still query/report by beneficiary at any time.

`Income:Gifts:X` is the one exception worth calling out: it's a
dedicated account, but that's fine because it represents actual money
received into your accounts (an inflow you might want to track
separately per person), not an expense you're merely tagging.