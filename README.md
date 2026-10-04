# Finance Tracker

A private finance tracker that runs in your browser. Your statements and database stay in a folder on your computer;
nothing is sent anywhere.

## Run

Open the GitHub Pages site in **Chrome or Edge**. The app reads and saves files directly in a folder you pick, using
the browser's File System Access API, which Firefox and Safari don't support. Brave has it but ships it switched off:
turn on `brave://flags/#file-system-access-api` and relaunch. If the API is missing or blocked, the app shows steps
for your browser.

- **First visit:** click **Choose folder…** and pick your data folder: the one with your `Statements` folder and
  `finance-tracker.json`, or an empty folder to start fresh.
- **Later visits:** click **Open <folder>** to give the page permission again. If you chose Chrome's "Allow on every
  visit", it opens straight away.
- **Change folder:** use **Settings → Data folder → Choose…**.

**What the browser keeps:** only a handle to that folder (in IndexedDB), so it can ask to reopen the same one. It
stores no transactions, settings or preferences: they're all in `finance-tracker.json`. The page's Content Security
Policy blocks every network request, so it can't send your data anywhere either.

There's nothing to install or run: the app is the single `index.html` file. To try changes before pushing, open it
from any static web server on `localhost`. Opening `index.html` straight from disk (`file://`) doesn't work, because
the folder picker needs `http://localhost` or `https://`.

## Deploy (GitHub Pages)

`.github/workflows/pages.yml` publishes the site on every push to `main`. Only `index.html` is published.

1. Push this folder to a GitHub repository with a `main` branch.
2. In the repository, go to **Settings → Pages → Build and deployment → Source** and choose **GitHub Actions**.
3. Push to `main`, or run the workflow by hand from the **Actions** tab. The site's URL appears in the workflow run.

`.gitignore` keeps CSVs, `finance-tracker.json` and backups out of the repository in case any end up in this folder.

## Files

| File | Purpose |
|---|---|
| `index.html` | The entire app (UI, folder access, import rule engine, charts). No external libraries or internet needed. |
| `.github/workflows/pages.yml` | Deploys `index.html` to GitHub Pages on push to `main`. |
| `<data folder>/finance-tracker.json` | Your database: templates, rules, manual entries, transactions, budget, goals, cash on hand, settings, and view preferences (last page, date range, chart and diagram options, sidebar, theme), so they follow the data folder to any browser. View preferences are saved without a new backup. |
| `<data folder>/.finance-tracker-backups/` | Automatic backups (last 20, at most one per minute). |

## Workflow

1. Put each bank/account's CSV exports in its own subfolder of the data folder (e.g. `Statements/chase`).
   If the data folder has a `Statements` folder (any capitalisation; checked when the app loads or the data folder
   changes), it's treated as the root for statement folders: the app only offers folders inside it, shows them
   without the `Statements/` prefix (`chase`), and **Add another folder** creates new ones inside it.
   Either copy them there yourself, or use **Imported Data → Import**: pick the folder (or **Add another folder** to
   create one), then pick one or more `.csv`/`.tsv`/`.txt` files and they're copied into it. Nothing is overwritten:
   a file that's already there is left alone, and a different file with the same name is saved as `name (2).csv`.
   If a template reads that folder they're imported straight away (a Refresh runs); if not, you're offered a new
   template for it.
2. **Imported Data → Import templates → New template**: pick the folder; the newest file is shown as a preview.
   - **Column mapping**: choose which CSV column is the Date (and its format), Amount (one column, or separate
     debit/credit columns, with an optional sign flip), Description, Vendor ID and Category.
   - **Vendor rules**: "If [column] contains / starts with / equals / ends with {text} then vendor is {name}",
     with an optional category.
     Rules are checked top to bottom and ignore case. Below the rules, unmatched rows show in their raw form and
     matched rows show formatted. Highlight text in an unmatched cell and click **+ Rule** to start a rule from it.
   - Choose whether rows with no matching rule are imported (vendor from a fallback column) or skipped.
   - **Skip rules**: "If [column, or the vendor name] contains / starts with / equals / ends with {text} → skip".
     Matching rows are never imported and are listed in a Skipped rows table. Use the **Skip** buttons on
     unmatched rows (highlight text first) or matched rows (skips that vendor) to start one.
   - **Parent group** (optional), e.g. "Credit Card": everything this template imports is grouped under that
     node in the Cash Flow diagram (Income → Credit Card → its categories). Changing it later also
     regroups rows that were already imported.
3. **Refresh** rechecks every template's folder and imports, without asking, every file that is new, has changed,
   or whose template was edited since it was imported (so mapping and rule changes apply to rows already imported).
   A message says what it did. Rows already imported from another file are skipped as duplicates, and categories
   set by hand are kept. Renaming a template or changing its parent group doesn't re-read its files.
   **Imported Data → Files** shows every file in the statement folders as a folder tree (click a folder to open or
   close it): which template imported each file, when, and how many rows it added and skipped. Files that were
   imported but are no longer on disk still appear, with their transactions kept. Folders show totals for
   everything in them.
4. **Categories** come from each import template's vendor rules (e.g. "contains COSTCO → vendor Costco, category
   Groceries"), otherwise from the file's Category column. Two things from earlier versions still apply but can't
   be edited any more: categories set by hand on a transaction (these win over everything), and Settings category
   rules (used when no vendor rule gives a category, before the file's column). **Settings → Categories** sets each category's color. Every category counts in reports.
5. **Manual Entry** adds one-time or recurring (daily, weekly, every 2 weeks, monthly, quarterly, yearly) items,
   each with an optional category and parent group (works the same as a template's parent group).

6. **Budget** plans your income and spending by category.
   - **Monthly income** is what you earn in a usual month.
   - **Expense budget**: one line per category (click **Add** on a category in **Spending by category** to start it
     at its average). Each line has an amount and **how often** it comes up: monthly, every 2 months, quarterly,
     every 4 months, twice a year or yearly, plus which months its periods start in (e.g. quarterly in
     Jan · Apr · Jul · Oct, or yearly in March). **Per month** is the line's yearly total ÷ 12. **Spent so far** is this
     month for monthly lines, or the current period so far (e.g. Oct – Dec) for longer ones.
   - **Year at a glance** shows every calendar month, January to December, and repeats every year. Blank cells use the
     usual amount; type an amount to budget more or less in that month (e.g. more groceries in December, a bonus in
     income), or 0 to skip it. A longer line has one cell per period. ↻ puts a row back to its usual amount.
     The bottom rows show each month's total expenses and what's left over.
   - A quarterly or yearly line's amount counts in the month its period starts: that's when the forecast spends it,
     and the month it's in on the Income vs. Expenses chart.

7. **Goals** tracks things you're saving up for. Each goal has a target amount, an optional target date and notes.
   - The list is in priority order: drag the ⋮⋮ handle or use the arrows (or pick a priority when editing).
   - **Add money** records a deposit or withdrawal with a date and note; the dialog also shows (and lets you delete)
     the goal's history. When creating a goal you can enter what's already saved as a starting balance.
   - Each goal shows how much is needed per month to hit its target date, and when you'd get there at your recent
     pace (last 3 months of deposits). Reached goals can be marked done and move to a Completed list.
   - Goals are separate from your transactions; they don't change Income, Expenses, the Overview or Cash Flow.

8. **Savings** shows your cash on hand over time. Enter what you have now (with an "as of" date, which defaults to
   today); the line is that amount on that day plus or minus the net cash flow (income minus expenses) of every day
   since or before it, so changing the amount only shifts the whole line up or down. Days after today come from the
   forecast (each month's budget, spread evenly over its days) and are drawn dashed.

9. **Recurring** lists every vendor in your imported data that charges (or pays) you on a regular schedule (weekly,
   every 2 weeks, monthly, quarterly or yearly, over all your history
   and including ones that have ended). Each shows a small price-history line, its price now, the change since the
   first charge and its last price change. **Recent price changes** lists every change in the past 12 months. Click a
   vendor for a chart of every charge, its price periods and its transactions.
   - A price is a run of charges at the same amount. A single odd charge between two at the same price (a pro-rated
     month, say) isn't counted as a change.
   - Vendors whose amount changes most times (utilities, groceries) are marked **Varies** and show their typical
     amount (the median of the last 3) instead of price changes.

**Reports:** **Overview** is budget vs. actual by category. It opens on the current month; step through months with
‹ ›, click a month on the Income vs. expenses chart, or switch to the date range picked at the top right. Months after this one are shown as a forecast.
  - **Budget vs. actual**: one row per budget line with a progress bar (the solid mark is the budget; while the month
    is under way the dotted mark is how much of that category you usually spend by this day, from the last 3 full
    months, so rent paid on the 1st doesn't look "ahead of pace"). Spending in categories with no budget line is listed
    below. Click a row to see its transactions.
  - For a single month, quarterly, yearly and other longer lines are listed under **Longer budgets**: they compare
    everything spent since their current period started (e.g. Jul – Dec) with the whole period's amount, since that
    spending can land in any month of it. Over a longer date range, each period counts in the month it starts.

**Cash Flow** is the Sankey diagram of where money comes from and goes. **Income** and **Expenses** break each side
down by category or vendor.

**Picking a month:** Overview, Cash Flow, Income and Expenses share one picked month, starting on the current one.
Click a month on any of their monthly charts (or step with ‹ › on the Overview) and all four pages show that month;
the date range dropdown at the top right shows it as the selected value. Months after this one are a forecast.

**Forecast:** comes entirely from the **Budget** page: each future month gets that month's expected income and every
budget line's amount for it, with any month adjustments, and quarterly or yearly lines in the month their period starts
(a partial month gets its share of days). Spending in categories with no budget line, and manual
entries, aren't part of it, so a complete budget gives the most realistic forecast. The Income vs. Expenses chart's
Forecast option shows this month plus the next 11.
Choose a range from the dropdown to go back to it on all four pages.

Amounts are positive for income and negative for expenses.
