<h1 align="center">Where is my <em>Salary</em></h1>

<p align="center">
  <i>Month by month, counted by hand.</i>
</p>

<p align="center">
  A small app to keep track of an Italian salary and of unpaid overtime.<br/>
  No account, no cloud (unless you want it), no dependencies. A single <code>.html</code> file that works offline and installs as a PWA on iPhone and Android.
</p>

<p align="center">
  <b><a href="https://massimodascola.github.io/where-is-my-salary/">Open the app</a></b> · <a href="README.it.md">Leggi in italiano</a>
</p>

> **Language:** the app's interface is in Italian, because it follows Italian payslips (13th/14th month pay, meal vouchers, national holidays, CCNL overtime allowance). This README is in English; an Italian translation is in [README.it.md](README.it.md).

---

## What it does

Have you ever reached payday wondering whether **every** part of your payslip actually arrived, and how many overtime hours you are piling up that never show up on it? This app answers both questions: it works out what you were owed, what you actually received and how much extra work went unpaid.

The app has two main pages, reachable from the tab bar at the bottom of the screen.

### **Salary** page: the payslip, month by month

* **What they still owe you**: a single figure at the top, recalculated over the months whose payday has already passed.
* **Tracked pay items**: salary, welfare, fringe benefits, meal vouchers, bonuses, expense refunds, plus the 13th/14th/15th month pay as separate items in the right months (12 to 15 monthly payments per year). Items you don't have (welfare, meal vouchers or fringe benefits set to 0) don't appear in the months or in the yearly summary.
* **Status per month**: ✓ received · ! missing · — not due. Plus an "Overtime" column that links to the Overtime page.
* **Configurable payday**: your pay day (1 to 28) decides when the app expects the month's salary. A switch covers people paid the following month (for example, May paid on 15 June).
* **Per-month override of any amount**: if a salary, a 13th month pay or a bonus arrives different from the default, you set it on that single cell.
* **Automatic meal vouchers**: counts Italian working days, excluding weekends and national holidays (Easter Monday included, computed correctly for every year). You can override it for holidays, sick leave or different remote-work days. The weekdays that count are configurable (Monday to Friday by default).

### **Overtime** page: the unpaid extra hours

* **Calendar for daily logging**: open a day and log the hours you worked (weekend hours always count as overtime). In the day detail, the `‹ Week N ›` arrows move to the previous or next week. With the keyboard, type the hour and press Tab: the minutes become `00`.
* **Day types: Work, Holiday, Public holiday**: holidays and public holidays are logged without times and count as an 8-hour day. In the calendar holidays are blue and public holidays gold; holiday days are counted in the Months, Weeks and Years views.
* **Usual schedule and break**: every new entry starts with your usual hours (for example 9:00 to 18:00) and your break (30 minutes by default), set in the setup wizard and in the settings. On days that differ, you change them in the entry.
* **Automatic threshold**: contract hours (40 h/week by default) plus any CCNL flat allowance. Everything above becomes unpaid overtime.
* **Hero box with two figures**: the amount in euros (if you set an hourly rate) next to the total unpaid hours.
* **Yearly KPI**: overtime hours for the current year.
* **Can be turned off**: if you don't want to track hours, switch off "Overtime" in the settings and the page disappears, tab bar included.

### General

* **Installable as a PWA**: add it to the home screen of an iPhone or Android phone; it updates itself at the next launch with a network connection (network-first service worker).
* **CSV backup**: export or import to move your data between devices (semicolon-separated, readable in Italian Excel and Numbers).
* **Optional cloud sync**: a private GitHub gist as shared storage between Mac, iPhone and office. No new services, no extra accounts.

---

## Getting started (30 seconds)

1. [Open the app](https://massimodascola.github.io/where-is-my-salary/), or open `index.html` with a double click: it opens in the browser.
2. On first launch the app opens the **"Iniziamo da te"** ("Let's start with you") wizard, one step at a time: name; salary, number of monthly payments (12 to 15) and payday; welfare and meal vouchers; fringe benefits; whether you want to track hours (working days and contract hours); overtime (allowance and rate); your usual schedule (start, end, break); and a summary to check before the profile is created. Welfare, meal vouchers and fringe benefits are yes/no questions: if you don't have them, those items don't appear in the months. If you don't want to track hours, the hour steps are skipped. If you close the wizard halfway, it resumes where you left off.
3. For every past month, open its card on the **Salary** page and set the status of each item. For extra hours, go to the **Overtime** page and log the days.
4. To change the settings at any time: ⚙️ icon at the top right → Impostazioni (Settings). To wipe everything and start over: "Azzera dati" (Reset data) at the bottom of the settings.

---

## Using it on your phone

The quickest way is the published version: **[massimodascola.github.io/where-is-my-salary](https://massimodascola.github.io/where-is-my-salary/)**. Open it on your phone, then:

* **iPhone (Safari)**: Share → **Add to Home Screen**.
* **Android (Chrome)**: menu ⋮ → **Add to Home screen** or **Install app**.

It becomes an icon like a real app, works offline and updates itself.

Your data stays on your device even when you use the published version: every browser starts from zero, and nothing is sent to the site (see [Privacy and security](#privacy-and-security)).

### Hosting your own copy

If you prefer your own copy, three options, from the simplest to the most "pro":

1. **Local file via iCloud Drive or Google Drive (no setup)**. Save `index.html`, `sw.js` and `manifest.webmanifest` together in the same folder (the service worker needs them side by side). On the phone, open `index.html` from the Files app, then add it to the Home Screen.
2. **Netlify Drop (5 minutes, no account)**. Go to [app.netlify.com/drop](https://app.netlify.com/drop), drag the folder with the three files into the page and you get a URL like `https://random-name.netlify.app/`. Open it on the phone and add it to the Home Screen.
3. **GitHub Pages (1 minute with a repo)**. Fork this repository, then **Settings → Pages → Branch: `main`, folder: `/ (root)` → Save**. After a few minutes it is live at `https://<username>.github.io/<repo>/`.

> ⚠️ Data lives in each device's browser separately. To keep the same data on Mac and iPhone, use the **cloud sync** or export the CSV from one and import it on the other.

> ⚠️ If you installed the app on the Home Screen before the service worker existed (versions before 3.4.1) you may still see an old build. One-time fix: remove the icon from the Home Screen → iOS Settings → Safari → Clear History and Website Data → reopen the site → Add to Home Screen. From then on, updates are automatic.

---

## Privacy and security

* **All data stays in the browser** (`localStorage`). No server, no cloud (unless you turn on sync), no telemetry.
* If the app is published online (Netlify or Pages), the URL is public but **your data is not**: everyone who opens it starts from zero, because storage is per browser.
* **Cloud sync (optional)**: the GitHub Personal Access Token is stored in plain text in the device's `localStorage`. It is a deliberate choice: the app is a single file with no backend and cannot do better. If the browser is compromised (malicious extension, shared session), **revoke the token on GitHub** (Settings → Developer settings → Personal access tokens) and sync stops on every device. The token only needs the `gist` scope and cannot do anything else on your account.
* **Fonts**: Source Serif 4, Inter and Caveat are loaded from Google Fonts when you are online; without a connection the app falls back to system fonts.

---

## How "what they owe you" is calculated

### Salary

For every item and every month:

| Status | Counted as |
|---|---|
| ✓ received | received, and expected |
| ! missing | expected; counted as "still to receive" only if **that month's payday has already passed** |
| — not due | not counted |

What is "past" versus "upcoming" doesn't depend on the end of the month but on your configured payday. If you are paid on the 27th of the same month, May becomes "due" from 27 May. If you are paid on the 15th of the following month, May becomes "due" on 15 June. The logic lives in `paymentDateFor` and `isPaymentDue`.

The expected amount comes from the profile settings (salary, welfare, fringe benefits, meal vouchers × working days, extra monthly payments) or from the amount you type in (bonuses, refunds). The 13th/14th/15th month pay rows appear only in their payment month, and only if the configured number of monthly payments reaches them.

Working days exclude the weekdays you marked as non-working (Saturday and Sunday by default) and these **Italian national holidays**: New Year's Day, Epiphany, Easter Monday, Liberation Day, Labour Day, Republic Day, Ferragosto, All Saints' Day, Immaculate Conception, Christmas, St Stephen's Day. Easter is computed with the Gauss/Meeus algorithm.

> ⚠️ Local patron saint holidays (Sant'Ambrogio in Milan, San Petronio in Bologna and so on) are **not** included: the app only counts national holidays. If you need them, override that month's number of days by hand.

### Overtime

For every day logged in the calendar:

* **Weekend hours** (days you marked as non-working) count entirely as overtime.
* **Weekday hours** count as overtime only for the part above the weekly threshold: `paid = max(0, weekdayHours - (contractHours + allowance))`.
* **Holidays and public holidays** count as 8 hours towards the weekly threshold, like a worked day, because your salary pays them anyway. In a week with one day off, 32 worked hours reach 40; anything above is overtime. Holidays can only be logged on working days.
* A **week spanning two months** (or two years) counts as a whole. The threshold fills up day by day, in date order, and overtime goes to the month of the days where the week goes over it. Example: Monday 28/9 to Friday 2/10, 10 hours a day. September 0 h (30 h, still under the threshold), October 10 h, as in the Weeks view. A finished month no longer changes when you log hours in the next one.

The amount in euros is `hours × overtime rate`. If the rate is 0, the hero box only shows hours.

---

## Tech stack

> Three files in total, no build step, no required CDN.

* `index.html`: HTML + CSS + vanilla JS in a single file (markup, styles, logic and inline data-URI icons).
* `sw.js`: service worker for automatic updates of the Home Screen version (network-first on the HTML, cache-first on assets).
* `manifest.webmanifest`: PWA manifest with `display: standalone`, theme color and inline SVG icons.
* `localStorage` for storing user data locally.
* GitHub Gist API for the optional cloud sync (private gist, `gist` scope).
* Optional Google Fonts (Source Serif 4, Inter, Caveat), with fallback to system serif and sans.
* Easter and Italian holidays computed in the app (see `easterSunday`, `italianHolidays`).

No frameworks, no bundler, no package manager. Any text editor will do.

---

## How the code is organized

All the logic lives in `index.html`. The best way to find a block is to search for the function names, because line numbers change from release to release. Logical sections, in order:

| Block | What it does |
|---|---|
| Constants (`VERSION`, `STORAGE_KEY`, `SYNC_KEY`, `MONTHS_IT`, `COMPONENTS`, `EXTRA_SALARY_DEFAULT_MONTHS`, `TAGLINES_*`) | Base definitions. `COMPONENTS` lists the pay items with their flags (`expectedDefault`, `hasAmount`, `hasDays`): change it to add or remove items. |
| `easterSunday`, `italianHolidays`, `workingDaysInMonth` | Italian calendar and working days (respecting the `workdays` in the settings). |
| `paymentDateFor`, `isPaymentDue` | When month N's salary becomes "due", based on `payDay` and `payDayNextMonth`. |
| `loadStore`, `saveStore`, `defaultSettings`, `defaultMonth`, `ensureProfile`, `ensureYear`, `activeProfile`, `runMigrations` | Persistence and data shape in `localStorage`: `{ profiles: { id: { name, settings, years: { 2026: { 1: {...} } } } }, activeProfile, currentYear, _migrations }`. `runMigrations` runs at startup and applies idempotent migrations tagged in `_migrations`. |
| `expectedAmountFor`, `computeYearTotals`, `monthStatus`, `extraSalaryApplies` | "Expected vs received" logic for Salary, 13th/14th/15th month pay included. |
| `paidHoursForMonth` and related | Weekly overtime with the `contractHours + allowance` threshold. For weeks spanning two months it also needs the days of the same week in the previous month: always pass the full list of entries, never one already filtered by month. |
| `render`, `renderTopbar`, `renderMain`, `renderMonthCard`, `renderPageOre`, `renderOvertimeVisibility`, `switchTab` | Rendering of the two pages. **The app rebuilds the whole `#main` with `innerHTML` on every interaction**: simple and fine at this scale, but careful with inputs that must keep focus. |
| `openSheet`, `closeSheets`, `openNewProfile` + `$("#btn-...")` handlers | Bottom sheets and buttons. |
| CSV backup (`buildCsv`, `parseCsv`, `parseCsvLine`, `fmtNum`, `parseNum`, `csvEscape`, `applyImportedData`) | Export and import. Sectioned format, `;` separator, `,` decimal, UTF-8 with BOM. States mapped between English and Italian via `STATE_TO_IT` / `STATE_FROM_IT`. Tolerant parser for older versions. |
| `toast`, `escapeHtml`, `inputWidthStyle`, `interpolateTagline`, `ticketPhrase` | Utilities. **Use `escapeHtml` on any user string placed in a template string + `innerHTML`**, to avoid self-XSS. |
| `showConfirm` / `showAlert` / `showChoice` | Promise-based custom dialogs that replace native `confirm()` / `alert()`. |
| Cloud sync (`syncState`, `ghFetch`, `cloudConfigure`, `cloudPull`, `cloudPush`, `schedulePush`, `cloudDisconnect`, `refreshSyncSection`) | Sync through a GitHub gist. `schedulePush` is debounced (about 1.8 s). `cloudPull` compares local and remote `_lastModified` to decide which one wins. |
| `bootstrap()` + service worker registration | Startup: version tag, current month, optional cloud pull, new profile form if empty, then registers `sw.js`. |

### Storage keys

| Key | Content |
|---|---|
| `stipendio.v1` | Main store (profiles, months, settings, migration flags). Versioned in its name to ease future migrations. |
| `stipendio.sync.v1` | Cloud sync state (token, gist ID, user, last sync, last error). |

### Defensive conventions

* **Always `escapeHtml(s)`** for profile names, GitHub users and error messages placed in HTML.
* **Always clamp numeric input** from the user, like `Math.min(max, Math.max(0, Number(v) || 0))` (see `clamp` in the settings).
* **`saveStore()` updates `_lastModified`** and starts the debounced sync: don't write to `localStorage` directly, except in cases like `cloudPull` that must avoid the loop.
* **Never call native `alert` / `confirm`**: use `showAlert` / `showConfirm` / `showChoice`.
* **Store migrations live in `runMigrations()`**, each guarded by a `store._migrations.<name>` flag: to add one, add a new `if (!store._migrations.<name>)` block and set the flag to `true`.
* **Bump the service worker cache key at every release** (`wims-vX.Y.Z` in `sw.js`): the `activate` event deletes caches with a different key.

---

## Settings

Everything is configurable from the ⚙️ icon at the top, in three sections: **Salary**, **Overtime**, **Sistema** (System).

### Salary

* Monthly salary (up to €100,000)
* Monthly payments per year (12 to 15): turns on the 13th/14th/15th month pay in the `EXTRA_SALARY_DEFAULT_MONTHS` (December, June, July)
* Payday (1 to 28, default 27), capped at 28 so February is never skipped
* Paid the following month (switch), for cycles like "work in May, paid on 15 June"
* Monthly welfare (0 if you don't have it: the item disappears from the months)
* Yearly fringe benefits (0 if you don't have them) and the month they arrive (December by default)
* Meal voucher per working day (0 if you don't have meal vouchers)
* Standard bonus, used when a bonus is marked as expected without an amount

### Overtime

* Overtime on or off: when off, the page and its tab disappear and overtime doesn't count in the totals
* Weekly contract hours (default 40 h)
* Weekly CCNL flat allowance (default 0 h): extra hours within it are not overtime
* Hourly overtime rate (default €0/h)
* Usual start and end time (empty means no pre-filled times)
* Default break in minutes (default 30)
* Working weekdays (default Monday to Friday)

### System

* CSV backup (export and import)
* GitHub Gist cloud sync (token and gist ID)
* Reset data

If a value changes (for example a pay rise), the expected amounts are recalculated, also in months already marked. Per-month overrides always win over the defaults.

---

## Backup and portability

From the settings → **Backup CSV** you can:

* **Download a file**: a CSV with every profile and every year, using Italian conventions (`;` separator, `,` decimal, UTF-8 with BOM), so Italian Excel and Numbers open it in clean columns.
* **Load a file**: imports a previously exported CSV. It replaces all current data (after asking). Strict validation: any malformed row rejects the whole import, never half a state.

File format (sections `# SETTINGS`, `# MESI` and `# STRAORDINARI`; column names are in Italian, as in the app):

```
# SETTINGS
profilo;stipendio;welfare;fringe;mese_fringe;ticket_giorno;bonus_default;forfait_settimana;tariffa_ora;giorni_lavorativi;contratto_settimana;mensilita;pay_giorno;pay_mese_dopo;overtime_abilitato;pausa_predefinita;orario_inizio;orario_fine
Anna;2000,00;150,00;500,00;12;8,00;0,00;0,00;0,00;1,2,3,4,5;40,00;14;27;0;1;30;09:00;18:00

# MESI
profilo;anno;mese;stipendio_stato;stipendio_importo;welfare;ticket_stato;ticket_giorni;fringe;bonus_stato;bonus_importo;rimborso_stato;rimborso_importo;straord_stato;straord_ore_pagare;salary13_stato;salary14_stato;salary15_stato;salary13_importo;salary14_importo;salary15_importo
Anna;2026;1;ricevuto;2100,00;ricevuto;ricevuto;22;non_atteso;non_atteso;0,00;non_atteso;0,00;mancante;;non_atteso;non_atteso;non_atteso;;;

# STRAORDINARI
profilo;data;inizio;fine;pausa;nota;non_straordinario;festivo;ferie
Anna;2026-05-13;09:00;19:30;00:45;riunione clienti;0;0;0
Anna;2026-08-10;;;;;0;0;1
```

In `# STRAORDINARI` (overtime) the `non_straordinario`, `festivo` and `ferie` columns are `0` or `1`; public holiday and holiday rows have no times. `pausa_predefinita` is in minutes; `orario_inizio` and `orario_fine` are `HH:MM` or empty.

Allowed states: `ricevuto` (received), `mancante` (missing), `non_atteso` (not due). Months left at their defaults are omitted from the export.

The `*_importo` columns are optional **per-month overrides**: empty means the default from the settings, a value wins over it.

The parser is **tolerant**: a CSV exported by an older version (without some newer columns) imports without errors, and missing values fall back to the current defaults. Booleans accept `0/1`, `sì/no` and `true/false`.

### Worked hours export

"Download file" asks whether you want the **full salary CSV** or **worked hours only**. The second one is an entry-by-entry export of the Overtime calendar, with the same columns as the `# STRAORDINARI` section.

Keeping an export now and then is a good idea, both as a backup and to move your data to another phone or computer. With cloud sync on, the CSV is just an extra safety net (sync uses JSON in the gist).

---

## Roadmap

Ideas under consideration, not promises:

* A payday notification for items still missing
* PDF export of the yearly summary
* A "forecast" mode to simulate pay rises, contract changes or a new overtime rate
* Optional local holidays for the meal voucher count
* Accrued severance pay (TFR)

---

## Design philosophy

The look follows a small visual philosophy called **Quiet Ledger** ([`quiet-ledger-philosophy.md`](quiet-ledger-philosophy.md), in Italian): warm paper as the main surface, serif italics as the voice, a single sienna accent used like a wax seal. A quiet nod to Luca Pacioli's *Summa de Arithmetica* (Venice, 1494), the treatise that codified double-entry bookkeeping. [`DESIGN-SYSTEM.md`](DESIGN-SYSTEM.md) and [`CHANGELOG.md`](CHANGELOG.md) are in Italian too.

---

## License

MIT, see [LICENSE](LICENSE). Do what you like, but a mention is always appreciated.

Made by [Massimo D'Ascola](https://github.com/massimodascola).

---

<p align="center">
  <i>counted by hand, month by month</i>
</p>
