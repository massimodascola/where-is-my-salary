# Changelog

Release history of **Where is my Salary**. The file is also a small record of the design choices: for each release I write *what* changed and *why*, so that months later I don't have to rebuild the context from memory.

Entries are in reverse chronological order (newest first). Versions follow [SemVer](https://semver.org/) informally: minor bump for features, patch for fixes.

---

## [3.9.0] - 2026-09-24

Opening the Overtime tab on a working day with nothing logged for today now opens the new-entry form by itself, so the day's hours get entered before they're forgotten.

### Today's entry prompt

* **When it opens**: on a tap on the Overtime tab, when the day is one of the working days set in the settings (Overtime section, Monday to Friday by default) and no entry of any kind (work, holiday, public holiday) exists for today. The form is the usual new entry: today's date, default times and break from the settings.
* **When it doesn't**: on the Salary page, at weekends or on any day outside the working days, when today already has an entry, and on switches made by the app itself (for example when Overtime is turned off). Tapping the tab while already on Overtime does nothing.
* **At most once per day per launch**: closing the form without saving and coming back to Overtime doesn't reopen it. It shows again the next time the app is opened, or the next day. The flag lives in memory only (`_todayPromptShownFor`): nothing is stored, synced or exported.
* **Where**: `maybePromptTodayEntry()`, called by the tab bar handler only when the tap moves the page from Salary to Overtime.

---

## [3.8.0] - 2026-09-23

The app now speaks English as well as Italian, with a switch inside the app. English is the default for the public; the Italian interface stays exactly as it was. From this release the changelog is written in English; the older entries have been translated from Italian, and the originals remain in the git history.

### English and Italian interface

* **Every interface text lives in one dictionary**, `I18N = { en, it }`, read through `t(key, params)` with `{name}` placeholders, and `tp(key, n)` for singular and plural. It covers labels, buttons, headings, placeholders, hints, toasts, dialogs, the first-run wizard, the settings, tab names, empty states, the hero taglines, month and weekday names, status labels, `aria-label` and `title` attributes, error messages and the backup and sync texts. The static markup is translated through `data-i18n`, `data-i18n-html` and `data-i18n-attr` attributes (`applyStaticI18n`). Pay item names moved from `COMPONENTS` to the dictionary (`comp_<key>`), and so did month names and taglines: `MONTHS_IT` and `TAGLINES_*` are gone.
* **Italian is unchanged.** Checked by rendering the same data with the 3.7.2 script and the new one, with the same clock and the same random tagline: 476 outputs out of 476 identical (both pages, every Overtime view, month cards, wizard steps and summary, entry form, alerts, sync and backup texts, CSV error messages). The static markup is identical too, once the two new language switches are left out.
* **English copy** keeps the Italian payslip and names it plainly: 13th/14th/15th month pay, meal vouchers, fringe benefits, welfare, CCNL allowance, holiday (ferie) and public holiday (festivo). The taglines are adapted rather than translated word for word; `{payDay}` becomes an ordinal ("the 27th") and `{ticket}` an amount ("€8.50").
* **Dates and numbers** follow the active language: `it-IT` or `en-GB`, always in euros ("1.234,56 €" or "€1,234.56"), decimal comma or point in hours and placeholders, long dates like "Monday 5 May 2026".

### Language switch

* **Settings, System section**: "Language / Lingua" with English and Italiano. It applies at once: the page re-renders and `<html lang>` follows, while values typed in an open sheet stay where they are.
* **First-run wizard**: the same switch, smaller, at the top of the first step.
* **Default**: the stored choice; without one, Italian when `navigator.language` starts with "it", English otherwise.
* **Stored apart**, under `stipendio.lang.v1`, outside the main store: the language is not synced to the gist, not exported, and changing it doesn't touch `_lastModified`.

### Data formats: no change

* `stipendio.v1`, `stipendio.sync.v1`, the store shape, the migrations and the gist JSON are the same.
* **The CSV is the same in both languages**: headers, `# SETTINGS` / `# MESI` / `# STRAORDINARI`, `ricevuto` / `mancante` / `non_atteso`, `;` separator and decimal comma. Checked on a test store: the export is byte-identical to the 3.7.2 export in English and in Italian (full and hours only), export then import then export gives the same file, a backup made in one language imports unchanged in the other, and the example in the README still imports. File names don't change either. Only the importer's error messages are translated.
* **Number inputs keep accepting "8,5" and "8.5"**: they carry `lang="it"` in both languages. Firefox parses number fields with the element's language, so with the page in English it would have refused the decimal comma. Chrome and Safari use the device settings and are not affected.

### Other

* The "today" mark next to the current month comes from the dictionary (`data-today` attribute) instead of the CSS.
* In English the day-type pills (Work, Holiday, Public holiday) take the width of their text, so "Public holiday" doesn't wrap on a narrow phone; the Italian ones keep three equal thirds.
* The web app manifest description is now in English (`"lang": "en"`): it is static and can't follow the in-app choice.
* Removed `formatItalianDate`, which was unused; `formatItalianDateLong` is now `formatDateLong`.

### Internals

* Version bumped to `3.8.0`.
* Service worker cache key bumped to `wims-v3.8.0`.

---

## [3.7.2] - 2026-09-21

Bug found while working on 3.7.1: a week split across two months lost its overtime in the monthly totals. It affects money, because `paidHoursForMonth` feeds the "how much you're owed" hero, the month cards, the Years view and the Overtime item on the Salary page.

### Weeks spanning two months

* **Problem**: `paidHoursForMonth` only looked at the month's entries, grouped them by ISO week and compared each group with the full threshold (contract + allowance). A week split across two months became two half weeks, each under the threshold. Measured example: from Monday 28/09 to Friday 02/10/2026, 10 h a day (50 h, threshold 40). Weeks view: 10 h to be paid. September: 0 h. October: 0 h.
* **Rule chosen**: the threshold fills up day by day, in date order, and overtime is the hours beyond the threshold: they go to the month of the days on which the week goes over it. In the example September stays at 0 h (30 h, still under the threshold) and October gets 10. Discarded: the whole week to the month of its Thursday (the ISO rule); to the month of its Sunday, which would also have moved to November the week from 26 to 30/10, worked entirely in October; split in proportion to the hours, which would have changed the earlier month after it ended and produced hours with decimals.
* **How it works**: for each week that touches the month, the function also adds up the working-day hours of the days of the same week that fall in the previous month (at most 6), and keeps only the part above the threshold that belongs to the month: `max(0, prima + mese - soglia) - max(0, prima - soglia)` (prima: the earlier days, mese: the month, soglia: the threshold). Across the two months the sum is exactly the one in the Weeks view, and a finished month no longer changes when you enter the hours of the following month. Hours on non-working days are still paid in full in their own month; "don't count as overtime", holidays, public holidays and the month's manual override work as before. The callers don't change: they all already pass the full list of entries, and the function's comment now says it must stay that way.
* **Across the year end** the same rule applies: week 53 of 2026 (from 28/12 to 03/01) is split between December 2026 and January 2027. The Weeks view lists it whole in 2026 (ISO year), so in that case the 2026 total in the Years view is lower than the sum of the 2026 weeks by exactly what went to January. The overall total matches.
* **Verification**: test profile at €20/h with four split weeks: 28/09-02/10; 26-30/10; 28/12/2026-02/01/2027 with a 4 h Saturday; 31/08-04/09 with one holiday day and one "don't count" day. Before: August 0, September 0, October 10, December 0, January 2027 4 h, total 14 h against the 38 h of the Weeks view (€480 lost). After: 0, 4, 20, 0, 14 h, total 38 h. In the browser everything checks out: the month cards, the hero (€80 by September), the October calendar (€400, €140 with the override at 7 h), the Salary page (20.0 h proposed for October) and the Years view (24 h in 2026 plus 14 h in 2027). Plus 3000 random cases in four time zones: sum of the months always equal to the sum of the weeks, closed month stable, never negative.

### Internals

* Version bumped to `3.7.2`.
* Service worker cache key bumped to `wims-v3.7.2`.

---

## [3.7.1] - 2026-09-21

Two bugs already present before 3.6.0, found while rereading the code for holidays: the Weeks view also counted hours marked "don't count as overtime" as to be paid, and the "today" date was the UTC one, which in Italy between midnight and 1 am (2 am with daylight saving time) is still yesterday.

### Weeks view: "not overtime" entries don't go into "To be paid"

* **Problem**: `renderOreWeekView` also added the entries with `notOvertime: true` into `workH`/`weekendH`, while `paidHoursForMonth` skips them entirely. The same week therefore gave two answers: verified example, week 38 with four days of 9.5 h, and a Friday of 9.5 h and a Saturday of 4 h marked "don't count": Weeks view 11.5 h to be paid (€195.50), month calculation 0 h.
* **Fix**: `notOvertime` entries go into a separate counter (`notOvertimeH`). They count in the "Worked" hours, but they don't go into "To be paid" and don't use up the weekly threshold, exactly as in `paidHoursForMonth`. After the fix: week 38 "Worked 51.5 h, to be paid 0.0 h"; control week 39 (no excluded entries) stays at 7.5 h. The sum of the weeks now matches the hero total.

### "Today" in local time

* **Problem**: `new Date().toISOString().slice(0, 10)` returns the UTC date. Between midnight and 2 am (daylight saving time) a new entry proposed yesterday's date, the calendar's "today" circle sat on the previous day and the "This week" KPI, on Monday night, still showed the previous week.
* **Fix**: the new entry (`openOvertimeForm`), the calendar (`renderCalendar`) and the KPI (`renderPageOre`) use `localDateStr(new Date())`, which builds year, month and day in local time (a helper already introduced in 3.6.1; its comment now says to always use it for "today"). The CSV backup file names also use the local date: a backup made at half past midnight carries that day's date. Only the sync timestamps (`lastSyncAt`) stay in UTC, since they are full times, not dates.
* **Verification**: clock simulated in the browser at Monday 28/09/2026 00:30 Rome time (Sunday 27, 22:30 UTC). Before: new entry on the 27th, "today" on the 27th, "This week" 47.5 h (week 39). After: 28, 28, and "This week" 0.0 h (week 40, still empty). With the real clock a new entry proposes today's date.

### Internals

* Version bumped to `3.7.1`.
* Service worker cache key bumped to `wims-v3.7.1`.

---

## [3.7.0] - 2026-09-21

Two direct requests. First: on first launch, a wizard that guides you step by step (name, salary, monthly payments, welfare and everything else that's needed) instead of the single "Let's start with you" form. Second: a usual start and end time, configurable, which like the break is already filled in every time you add an entry.

### First-run wizard

* **The single "Let's start with you" form becomes a step-by-step wizard**, in the same sheet (`#new-profile-sheet`): name → salary (amount, monthly payments, payday and pay month) → welfare and meal vouchers → fringe benefits → your hours → overtime (allowance and rate) → your schedule → summary. Progress bar under the title, "Step N of M", Back/Next buttons; Enter in a field is the same as Next. The fields keep the same `np-*` ids as before: the wizard only decides which step is visible, and creating the profile (`createProfileFromWizard`) is the same logic as before with the new answers.
* **Welfare, meal vouchers and fringe benefits are yes/no questions** (the same pill selector as "Day type"), with "No" preselected. "Yes" reveals the amount field and puts the cursor in it; with "Yes" the amount is required. "No" saves 0. Before, the form suggested €400 / €1000 / €10 as if everyone had them.
* **"Do you want to track your hours?"**: with "No" the overtime and schedule steps disappear (6 steps instead of 8) and the profile starts with Overtime turned off.
* **Final summary**: one row per answer; tapping a row goes back to that step and the button becomes "Back to summary", so you don't go through all the steps again after a correction.
* **Closing halfway loses nothing**: reopening it (from the welcome card) resumes from the step you were on. The answers are cleared only after the profile is created.
* **Fixed sheet height** (`min(92dvh, 680px)`, on desktop `min(86vh, 680px)`): the steps have different heights (from about 380 px for the name to 670 px for the summary, measured) and with automatic height "Next" jumped up and down at every step.
* **Cursor on the first field** of each step only with mouse and keyboard; on a phone only on the name, so the keyboard doesn't open at every step. After an error on a time the cursor starts again from the hours, not the minutes.
* The "Standard bonus" is not in the wizard: it's an advanced setting, it stays in the settings at 0.

### Items you don't have: hidden

* **New `componentApplies(comp, settings)`**: welfare, meal vouchers and fringe benefits at €0, extra monthly payments beyond the ones chosen, and overtime with Overtime turned off don't appear in the month cards or in the "Summary by item". Before, they appeared anyway, at €0.00. Totals and month status don't change: the expected amount of those items was already 0.
* A note in the settings says so: "Set welfare, meal vouchers and fringe to 0 if you don't have them: those items won't appear in the months."

### Usual schedule

* **New settings `inizioDefault` and `fineDefault`** ("HH:MM" or empty), in the Overtime section next to the default break. Every new entry starts with that schedule and with the break; when you open a holiday/public holiday entry (which has no times) the form is preloaded with the usual values, so if you switch it back to "Work" you start from there.
* **The wizard suggests 9:00-18:00**; if start and end are left empty, nothing is prefilled. Existing profiles get an empty schedule (through `ensureProfile`), so for them nothing changes until they set one.
* If start and end are both set, the end must come after the start: in the wizard the step blocks it, in the settings the save is refused without changing anything.
* **"Type the hour + Tab = minutes at 00"** now works for every time field in the app (entry, settings, wizard), not just the entry form.

### iPhone: no more zoom on fields

* iOS Safari zooms the page when a field with text under 16 px gets the cursor, and doesn't zoom back by itself. The form fields were 15 px: measured in the simulator, zoom 1.07 after the wizard put the cursor on the amount. Now on touch screens (`pointer: coarse`) the fields are 16 px: zoom 1.00. On desktop they stay at 15 px. This also applies to the entry form and the settings, where the zoom already happened just by tapping a field.

### CSV backup

* Columns `orario_inizio` and `orario_fine` appended to `SETTINGS_COLS`. Earlier files still import (empty schedule); an invalid time in the file blocks the import with a message that gives the line and column.

### Checks

* Chromium browser with real clicks and keys: automatic opening on first launch, warning on a missing name, Enter to advance, "Yes" showing the field and putting the cursor in it, warning on a missing amount, "No" to hours (8 → 6 steps and back), "end before start" error with correction, summary row → step → "Back to summary", profile creation with all the right settings.
* After creation: meal vouchers (answered no) absent from the month and from the yearly summary, 15th month pay absent with 14 monthly payments; new entry with 9:00-18:00, break 30 and duration 8.50 h; settings with an invalid time refused, with 8:30-17:30 saved and used by the form.
* Edge cases: profile created by an earlier version (empty schedule, no prefill); "Reset data" → wizard from the start; closing halfway and resuming; new CSV exported and reimported, 3.6 CSV without the columns, CSV with an invalid time.
* Width 375 px and iPhone 17 Pro simulator (iOS 26.4, Safari): welfare, schedule and summary steps, time fields as tall as the others (48 px with 16 px text), no zoom.

### Internals

* Version bumped to `3.7.0`.
* Service worker cache key bumped to `wims-v3.7.0`.

---

## [3.6.4] - 2026-09-21

On iPhone the empty "Start time" and "End time" fields were shorter than all the other boxes in the form, and only grew after a time was picked. Direct request: the same fixed height as the other fields.

### Entry form: date and time fields as tall as the others

* **Cause (iOS Safari)**: with `appearance: none` (needed since 3.4.4 to keep fields from overflowing on iPhone) an empty `<input type="time">` has no inner line of text, so it shrinks to padding + border. Measured in the iOS 26.4 simulator: empty "End time" 24 px, filled "Start time" 46 px, "Note" 46.5 px.
* **Solution**: `min-height` on the date and time fields equal to the height of the other fields, i.e. one line (`line-height: 1.5`, inherited from the body through `font: inherit`) + 11 px of padding top and bottom + 1 px of border top and bottom. Written as `calc(1lh + 24px)`, with `calc(1.5em + 24px)` as a fallback for browsers that don't know the `lh` unit.
* **Desktop**: in Chromium the date and time fields were already 2 px taller than the text fields (48.5 vs 46.5), because of the vertical padding of the inner box (`::-webkit-datetime-edit`). That padding is now zero: all the form fields have the same height on desktop too.

### Checks

* **iPhone 17 Pro simulator, iOS 26.4, Safari**, with a test page that opens the form by itself: before the fix 24 px for the empty field (problem reproduced as in the screenshots), after it 46.5 px for Date, filled Start time, empty End time and Note; same result with both times empty. Also checked by eye on the screenshots.
* Chromium desktop browser: all fields at 46.5 px, times centred, "9" + Tab = 09:00 still working.

### Internals

* Version bumped to `3.6.4`.
* Service worker cache key bumped to `wims-v3.6.4`.

---

## [3.6.3] - 2026-09-21

Touch-up to the month summary under the calendar: above the "Amount" row there were two lines stuck together, the dashed one of the "Hours to pay" row and the solid one of the total. Direct request: keep only the solid one.

### Month summary: a single separator above the total

* **Cause**: each `.oms-row` drew its dashed separator *below* itself (`border-bottom`), and the `.total` row added its solid line *above*. Between the two there were only the total's 2 px of `margin-top`, so two lines were visible.
* **Solution**: the separator now sits *above* every row except the first (`.oms-row + .oms-row`), the same pattern as the items on the Salary page (`.comp + .comp`). The total row replaces its dashed line with the solid one, so only one remains. Removed the `:last-child` rule, no longer needed.
* Without a rate (no "Amount" row) nothing changes: no line at the bottom of the box.

### Checks

* Chromium browser with a rate set: above "Amount" only the solid line, dashed lines unchanged between the other rows (the computed styles of every row were checked too). Without a rate: 4 rows, no border at the bottom.

### Internals

* Version bumped to `3.6.3`.
* Service worker cache key bumped to `wims-v3.6.3`.

---

## [3.6.2] - 2026-09-21

The month calendar had a lot of space above it: "‹ Months" on one line, then the big "September 2026" title with its margin, then the picker. Direct request: make it as compact as the day detail just redone in 3.6.1.

### Month calendar: header on one line

* **"‹ Months" on the left and `‹ Settembre 2026 ›` ("‹ September 2026 ›") on the right, on the same line**, like "‹ Days" and `‹ Settimana 39 ›` ("‹ Week 39 ›") in the day detail. The big title is gone: month and year now sit in the picker (before, it showed only "Set", the short Italian month name, because the full name was in the title). The calendar starts right below.
* **One class for both headers**: `.dd-head` (3.6.1) became `.ore-subhead`, used by both the calendar and the day detail.
* **Label with a fixed minimum width (`min-width: 9em`)**: month names have different lengths (in Italian, "Aprile 2026" about 100 px and "Novembre 2026" about 130 px, measured in the browser), so when scrolling with "‹" the arrow would have moved under the finger at every month. With the fixed width it stays put (checked: same position for all 12 months). As a result the week picker has the same width too, and the two headers line up.

### Checks

* Chromium browser with real clicks: next and previous month, opening a day from the calendar with the detail header unchanged.
* Width 375 px with the longest Italian label ("Novembre 2026"): everything on one line, no horizontal scroll. No errors in the console.

### Internals

* Version bumped to `3.6.2`.
* Service worker cache key bumped to `wims-v3.6.2`.

---

## [3.6.1] - 2026-09-21

The day detail had no way to go to the previous or next week: the week strip showed only the 7 days of the current week, and to change week you had to go back to the list or the calendar. Direct request, "like scrolling through the months".

### Day detail: week picker

* **New `‹ Settimana N ›` ("‹ Week N ›") picker at the top right**, on the same line as "‹ Days". It's the same `.year-picker` used for years and months, so the same look and the same behaviour.
* **The arrows move by 7 days keeping the weekday** (Friday 25 → Friday 2 October), as in week calendars. The change of year follows ISO weeks (31/12/2026 is week 53, the next one is week 1 of 2027).
* **The "WEEK N" line under the strip was removed**: the number is now in the picker, and repeating it twice a few pixels apart was noise. The day title got a bit of top margin to compensate.
* **The calendar's year and month follow the day shown** (`selectOreDate`), both with the arrows and when tapping a day in the strip. Without this, entering from the September calendar and going forward to 2 October, "‹ Days" took you back to September instead of October.

### Checks

* Chromium browser with real clicks: forward one week (week 40, 28 September-4 October), back two (week 38, with the public holiday on the 14th visible), back to the calendar on "October 2026" after the month changed.
* Change of year: 31/12/2026 (week 53) → 7/1/2027 (week 1); 1/1/2027 (week 53) → 25/12/2026 (week 52).
* Width 375 px: "‹ Days" and the picker fit on the same line, no horizontal scroll. No errors in the console.

### Internals

* Version bumped to `3.6.1`.
* Service worker cache key bumped to `wims-v3.6.1`.
* Helper `localDateStr(dt)` for the local date "YYYY-MM-DD", also used to build the strip.

---

## [3.6.0] - 2026-09-21

Three direct requests, all about the Overtime entry form: being able to mark holiday days and see them at a glance, not having to re-enter the usual 30-minute break every time, and not having to complete the minutes by hand when typing only the hour.

### Holiday (PTO) as a day type

* **The "Public holiday" toggle becomes a "Day type" selector: Work · Holiday · Public holiday.** Holiday and public holiday exclude each other (a day is one or the other): two independent toggles would have allowed meaningless combinations. The selector uses real `radio` inputs (keyboard arrows and screen readers work on their own) and is coloured like the calendar: Holiday in blue, Public holiday in gold.
* **New `pto: bool` flag on the entry.** Same model as the public holiday: no times (start/end/break saved empty), it counts as a standard 8-hour day. `FESTIVO_HOURS` was renamed `STANDARD_DAY_HOURS`, because it now applies to both.
* **Calculation semantics: the 8 holiday hours count towards the weekly threshold**, like a worked day. Reason: the salary pays for the holiday, so that day takes up 8 hours of the threshold already covered by the salary. Verified example: Monday on holiday + Tuesday-Friday at 9.5 h = 38 h worked + 8 h of holiday = 46 h against a threshold of 40, so 6 h of overtime. Before, with nothing marked on the Monday, the same week gave 0 h: the holiday "absorbed" the overtime done on the other days.
* **Holidays only on working days.** On a non-working day (e.g. Saturday with a Mon-Fri setting) a holiday makes no sense and would add 8 h to the week, inventing overtime: saving is blocked with a message that says where to change the working days, and the preview already flags it when "Holiday" is chosen. Public holidays stay as they were (they count at weekends too, a decision from 3.5.0).
* **Visibility**:
  * monthly calendar: solid blue circle (`--ocean`);
  * day detail week strip: `--ocean-tint` background;
  * entry card: "Holiday", a "holiday" badge and a blue bar on the left;
  * Days list: "Holiday" in blue instead of the times (and, for consistency, "Public holiday" in gold);
  * count of holiday days in the Months cards, in the Weeks and Years rows and in the month summary under the calendar (only when it's greater than zero).
* **"Hours worked" includes the standard days** (holidays and public holidays at 8 h), as already happened for public holidays. In the month summary, when there are any, the label says so: "Hours worked (days off at 8 h)".
* **Colour**: `--ocean` was "reserved, kept for extensions" in the design system; now it has a meaning, holidays. Added `--ocean-tint: #E6EDF5` for backgrounds.

### Default break

* **New "Default break (minutes)" setting** in the Overtime section, default 30. Every new entry starts with that break already in the duration picker; on a day when it's different you change it there, as before. With 0 you start with no break.
* Saved as `settings.pausaDefaultMin` (minutes, 0-720). Existing profiles get it at 30 through `ensureProfile`, like the other settings added over time.
* When editing an existing work entry, its break stays. A holiday/public holiday entry has no break: when you open it, the picker is preloaded with the default, so if you switch it back to "Work" you start from the usual value.

### Times: minutes at "00" when typing only the hour

* **Problem**: in `<input type="time">` fields on desktop, typing only the hour (e.g. "9", which moves on to the minutes by itself) and pressing Tab left the minutes at "--". For the browser a half-filled time field has no value (`value === ""`): the preview didn't compute the duration and saving answered "Enter start and end times".
* **Why it can't be completed afterwards**: the browser doesn't expose the typed hour until the field is complete, so on `blur` there's no way to read "09" and add ":00".
* **Solution**: on the first key (digit or up/down arrow) in an *empty* field, the field is initialised to "00:00" before the browser processes the key. The typed hour replaces the hours, and the minutes stay "00" until they're typed. Result: "9" + Tab = 09:00, "9" "30" = 09:30, "18" + Tab = 18:00.
* **Guards**: no initialisation if the field already has a value or is half filled (`validity.badInput`), so an hour already typed is never overwritten (checked: deleting only the minutes of 09:00 and typing "45" gives 09:45). Tabbing through an empty field without typing leaves it empty. The native iOS/Android wheels don't generate `keydown`, so on the phone the behaviour doesn't change.
* The duration preview now also updates on `blur` of the time fields.

### CSV backup

* Column `ferie` (0/1) appended to both `STRAORD_COLS` (full backup) and `ORE_COLS` (hours-only export). Column `pausa_predefinita` (minutes) appended to `SETTINGS_COLS`. All at the end, so files from earlier versions still import as a prefix: holiday = no, break = 30.
* Holiday rows have no times: time validation is skipped, as for public holidays. If a file has both `festivo` and `ferie` at 1, public holiday wins.
* README: the CSV format examples had fallen a few versions behind (columns and the `# STRAORDINARI` section were missing); realigned with the real headers and checked by having `parseCsv` read them.

### Checks

* Chromium browser with real keys on the time fields: "9" + Tab = 09:00 with focus on End time; "18" + Tab = 18:00; "9" "3" "0" = 09:30; "1" + Tab + Tab = 01:00; Tab on an empty field = stays empty; minutes deleted and retyped = hour kept. **Not checked on desktop Safari and Firefox.**
* Form: holiday on a Saturday blocked, editing holiday and work entries, switching work → holiday (times cleared) and back, default break set to 45 = new entry at 0h 45m.
* Months, Weeks and Years views, calendar, day detail and Days list with holiday and public holiday entries; form at 375 px with no horizontal scroll.
* CSV: full and hours-only round trip, import of 3.5.0 files without the new columns, a row with both flags.

### Internals

* Version bumped to `3.6.0`.
* Service worker cache key bumped to `wims-v3.6.0` to invalidate 3.5.0 on the first activate.

---

## [3.5.0] - 2026-06-05

New **"Public holiday"** option in the Overtime entry form. It comes from a direct request: being able to mark a day as a public holiday without typing the times, because that day is paid anyway like a normal working day, and it must stay recognisable at a glance in the calendar.

### `holiday` flag on the entry

* **Added a "Public holiday" toggle** in the add/edit entry sheet, placed right below the Date (before the times, so the flow is: pick the day → is it a public holiday? → if not, enter the times). It's stored on the entry as `holiday: bool`.
* **No times to enter: it counts as a fixed 8 hours.** When the toggle is on, the Start time / End time / Break fields (and the "not overtime" toggle, which conflicts with it) are hidden: the entry has no times and counts as a standard day of `FESTIVO_HOURS = 8` hours. `eventHours()` returns 8 for `holiday` entries, ignoring the clock times, which are saved empty.
* **Calculation semantics: "like a normal working day".** In `paidHoursForMonth` (and in the Weeks view) a `holiday` entry is treated as a weekday: its 8 hours go into the weekly threshold (`contratto + forfait`) instead of getting the automatic 100%-paid treatment of non-working days. A midweek public holiday counts as a normal worked day; for a public holiday at the weekend the effect is to count it towards the threshold instead of as full overtime: a literal reading of "I'd mark them as if I'd done a working day".
* **Visual indicators.** Public holidays are gold in both calendars: in the monthly grid (`renderCalendar`) the day circle is solid gold instead of the sienna/terracotta of days with entries; in the week strip it has a `--gold-tint` background (the `selected` state still wins). In the entry card the times line becomes "Public holiday", with a solid gold "public holiday" badge and a gold bar on the left.

### CSV persistence

* **Added a `festivo` column** at the end of both `STRAORD_COLS` (full backup) and `ORE_COLS` (hours-only export). It's appended at the end, so files exported by earlier versions still import: the shorter header matches as a prefix and the missing column is read as `holiday=false`. For public holiday rows time validation is skipped (start/end empty), so the export→import round trip stays valid.

---

## [3.4.4] - 2026-05-27

A round of UX polish on the Overtime entry form and on the add FAB. Three independent themes, all prompted by direct feedback: "the full-width FAB covers the content below", "the break field looks like a clock, not a duration", "start time and end time overflow the viewport on mobile and overlap".

### FAB: from a full-width button to a bottom-right "+" circle

* **Replaced the full-width `+ Aggiungi evento` ("+ Add entry") button with a 56×56 circular FAB** positioned at the bottom right, aligned with the right edge of `.app` (max-width 760 centred, padding 18). The full-width button created too much visual mass at the bottom and covered the content beneath it intrusively: on narrow viewports it became a "bar" wedged between the last card and the tabbar. The small circle is the standard FAB pattern (Material/iOS): content scrolls around it, no need for full-width masking.
* **From `position: sticky` to `position: fixed`.** Sticky had a sneaky behaviour: in short views (e.g. "Months" on desktop, with 12 cards that fit in the viewport without scrolling) the wrap was never pinned and fell back to its natural position at the end of the document, leaving the following cards visible "under" the FAB. With fixed, the position doesn't depend on scroll or page height.
* **Centred on `.app` instead of the viewport**: the wrap uses `left: 50%; transform: translateX(-50%); max-width: 760px; padding: 0 18px; justify-content: flex-end`. On a wide desktop the FAB stays next to the content instead of being stuck to the right edge of the viewport.
* **Vertical position**: `bottom: calc(74px + max(16px, env(safe-area-inset-bottom)))` → about 24px of visual gap above the tabbar pill, with the safe-area inset respected on iPhone.
* **Per-page visibility**: the wrap is a child of `#page-ore`, so when the user is on the Salary tab `.page{display:none}` hides it along with the rest. No cross-page leak of the FAB.
* **`#main-ore { padding-bottom: 140px }`** to reserve space below the content so the circle doesn't squash the last row.
* Simplified HTML: removed `.btn.full .btn.large`, text label "+ Add entry" → "+" with `aria-label="Aggiungi evento"` ("Add entry") and `title` for accessibility and a desktop tooltip.

### Break field: duration picker instead of a clock-time input

* **Replaced `<input type="time" id="ot-pausa">` with a custom duration picker**: two groups `−  Nh  +` and `−  Nm  +` with steps of 1h and 5min. The native `type="time"` input was a "clock" (HH:MM clock style) and created semantic ambiguity: the user read it as an absolute time instead of a duration. The new widget starts at `0h 0m` and goes up and down with the `+`/`−` buttons, saying explicitly "you're picking a duration".
* **Minutes↔hours wrap implemented**: pressing `+` on the minutes at 55 → they become 0 and the hours go up by 1 (clamp 0-23). Same logic in reverse for `−` at 0 minutes.
* **Storage format unchanged**: the duration picker still serialises the break as an `"HH:MM"` string (e.g. "01:30"), or an empty string if 0h 0m. So `parseHHMM(e.pausa)`, CSV import/export (`STRAORD_COLS` and `ORE_COLS`) and the "break 01:30" display in the day drilldown stay compatible with no data migration.
* Helpers added: `pausaPickerValue()` to read the two spans as HH:MM, `setPausaPicker(hhmm)` to fill the two spans from an existing entry, `adjustPausa(target, delta)` for the button clicks.

### `<input type="date"|"time">`: final fix for the overflow on iOS Safari

* **Diagnosis**: two successive fixes were needed. The first attempt (`.field-row` from `display: grid 1fr 1fr` to `display: flex; gap: 12px` with `min-width: 0` on the `.field` children) was a necessary but insufficient step: iOS Safari ignores `width: 100%` and `min-width: 0` on native form controls (`type="date"|"time"`) and renders them at their intrinsic min-content, the one required by the internal native picker. On narrow viewports (~390px) this min-content is more than half the sheet → "End time" overflows the right edge and visually overlaps the "Start time" box.
* **Solution**: `-webkit-appearance: none` + `appearance: none` on the DATE/START TIME/END TIME fields. The browser stops treating them as native widgets and respects `width: 100%` + `max-width: 100%` + `min-width: 0`. The native picker still opens on tap (the interaction isn't altered, only the widget chrome is disabled).
* **`::-webkit-calendar-picker-indicator` explicitly hidden** with `display: none`. The previous rule styling it with opacity/filter survived and was visible *together with* the SVG I added as `background-image` → two icons side by side in the box.
* **Inline SVG replacement icons** as `background-image` (a calendar for date, a clock for time), positioned on the right with `padding-right: 36px`. Stroke `#A88B6E` (= desaturated warm-paper sienna) for consistency with the design system. URL-encoded directly in the CSS, no separate files.
* **`text-align: center`** on the date/time fields, so the value (e.g. "26 May 2026", "14:30") is centred in the box instead of being left-aligned, with the padding-right pushing it optically off centre.

### Internals

* Version bumped to `3.4.4`.
* Service worker cache key bumped to `wims-v3.4.4` to invalidate 3.4.3 on the first activate.

---

## [3.4.3] - 2026-05-14

Two UX changes tied to onboarding and to fine-grained control over overtime.

### General: tabbar hidden on the welcome screen

* **`renderOvertimeVisibility` now also hides the tabbar when there's no active profile**, not only when the user has turned Overtime off. Before, the condition was `!p || p.settings.overtimeEnabled !== false`: with no profile the boolean fell into the "enabled" branch and the `Salary · Overtime` tabbar stayed visible above the welcome card and the "Let's start with you" form, offering a switch between two pages that make no sense before the profile is created. Now `!!p && settings.overtimeEnabled !== false` → tabbar shown only if *both* conditions are true.
* Intended side effect: the app's first impression is cleaner, just title, form, settings.

### Overtime: per-entry "not overtime" flag

* **New `notOvertime: boolean` field on entries.** When true, the entry is completely excluded from the overtime calculation (`paidHoursForMonth` skips the iteration with a `continue` before adding to the weekly or weekend bucket), but it stays in the calendar and contributes to the month/year "gross hours" totals. It's the answer to "I worked that day, but I don't want to log it as overtime", both for weekday hours over the threshold and for weekend hours given away on purpose.
* **iOS-style toggle in the edit entry form** (`#ot-event-sheet`), below the Note field. Styled consistently with the toggle that turns Overtime off in the settings, reusing `.settings-toggle` with a `.field-toggle` modifier to make it full width like a field row. Hint below: "The hours stay logged as worked, but don't count towards the month's overtime".
* **Visual feedback in the day detail**: the card of an entry marked `notOvertime` has reduced opacity (0.62), a `--card-elev` background and a small text badge `non strord.` ("not overtime"; serif italic, dashed border) next to the hours total. The size and position of the hours stay the same, so reading the duration doesn't change.
* **Calculation behaviour** preserved for the other entries: the weekly threshold `oreContratto + forfait` keeps working normally on unmarked weekday hours; `notOvertime` entries don't add to the weekly bucket, so they don't "use up" part of the threshold for other days of the same week. Example: 4 weekdays of 10h + 1 day of 10h marked `notOvertime` → calculation limited to 4 days × 10h = 40h ≤ threshold 45h → 0h paid (correct: the user explicitly took those 10h out of the count).

### Entry form: date/time inputs brought into the design system

* **CSS selectors `.field input[type="text"|"number"]` extended to `type="date"` and `type="time"`.** The three rows of the entry form (Date, Start/End time, Break) fell back to the browser defaults (thin grey border, system font, inconsistent padding, black picker icon) because the styling rule didn't catch them. Now they get the same Quiet Ledger treatment as the other fields: paper bg, `--line` border, padding 11/13, sienna focus state with a soft box-shadow.
* **`font-variant-numeric: tabular-nums`** added on `number`/`date`/`time` for clean vertical alignment when two time fields sit side by side (Start time / End time in the `.field-row`).
* **`::-webkit-calendar-picker-indicator` softened** with `opacity .45` + `filter sepia(.5) hue-rotate(-12deg) saturate(.85)`, so the calendar/clock icon doesn't read as a black system widget but as a small warm-paper accent. Hover/focus bring it to opacity .85.
* **`:invalid` on date/time** lowered to the `--ink-muted` colour: the native placeholders (`gg/mm/aaaa`, the Italian dd/mm/yyyy, and `--:--`) were darker than the rest of the hints, and it looked like a bug. Now the empty state looks intentional.
* Firefox has no equivalent pseudo-elements for the picker icon, but its native dropdown rendering is already discreet, so no override is needed.

### Design decision: `hoursOverride` stays

* The monthly override `overtime.hoursOverride` is orthogonal to `notOvertime`: the first says "ignore the automatic calculation and use this total for the month", the second says "don't count this specific day". The two mechanisms coexist for now. Removing `hoursOverride` is not planned until the new feature has been used enough to judge whether the monthly override is still useful or redundant.

### CSV backup: coverage of the new field

* **`non_straordinario` column (0/1) appended to the `STRAORDINARI` sections (full CSV) and to the hours-only CSV (`buildCsvOre`).** Placed as the last column so v3.4.2 CSVs can still be parsed as a *prefix* of the expected header.
* **`parseCsvOre` relaxed**: it only requires the first 6 columns (`profilo..nota`) as a mandatory prefix, and accepts shorter or longer headers (forward compatibility). The per-row field count check now uses `header.length` instead of `ORE_COLS.length`, so every file is parsed according to its own header.
* The parser is tolerant on values too: it accepts `0/1`, `sì/no`, `true/false`. Default: `false` if the column is missing or empty.

### Pre-release audit

* jsdom run of v3.4.3 compared with v3.4.2: welcome card rendered correctly, version tag "v3.4.3", zero errors.
* `paidHoursForMonth` checked by hand with mixed entries (weekdays over the threshold + weekend + one marked `notOvertime`): the marked day skips both buckets, the other weekday hours of the same week keep their normal contribution to the threshold.
* CSV edge case check: import of a v3.4.2 file (without the `non_straordinario` column) → all entries come in with `notOvertime: false`. Re-export → the column is now there, filled with `0`.

### Internals

* Version bumped to `3.4.3`.
* Service worker cache key bumped to `wims-v3.4.3` to invalidate 3.4.2 on the first activate.

---

## [3.4.2] - 2026-05-14

Critical fix for a bug inherited from v3.1 and hidden until today. The page loaded completely empty (only topbar + tabbar, no hero, no welcome card, no version tag) on any device (desktop, mobile Safari, mobile Chrome, private windows) because a module-level exception stopped the script *before* `bootstrap()` was called.

### Diagnosis

* `runMigrations()` (introduced in 3.1) runs at module load, right after `let store = loadStore();`.
* Even on a fresh device with no profiles, the migration sets `store._migrations.v3_1_overtime_default = true` and calls `saveStore()` to persist the flag.
* `saveStore()` contains a defensive call `if(typeof schedulePush === "function") schedulePush();` to trigger the debounced cloud sync when it's configured.
* `schedulePush()` is a **function declaration**, so it's hoisted → the `typeof` check passes even though the textual declaration is 2,000 lines further down → the function actually gets called.
* Inside it, `schedulePush` references `syncState`, declared with `let` much further down (next to the whole cloud sync block).
* `let` has a **Temporal Dead Zone**: reading the binding before its lexical declaration throws `ReferenceError: Cannot access 'syncState' before initialization`.
* The exception propagates out of the call chain, stops module-level execution, and `bootstrap()` (at the end of the file) is never reached. Result: only the static markup (topbar, tabbar) is visible. The version tag, the welcome card and all the rendering depend on `bootstrap()` → they stay empty.

The bug was deterministic and showed up in every environment, but it stayed invisible for days because most users had a localStorage populated by earlier versions (the cached v2.0/v3.0.1 copies of the pre-fix HTML didn't have the problem, so they kept running). Only "fresh" installs after 3.1 showed it, and ironically it surfaced when a service worker (3.4.1) started serving the updated HTML to people who had been seeing a stale copy.

### Fix

* **Forward declaration of `syncState` at module level**, right after `let store = loadStore();`, with initial value `null`. The real declaration (the IIFE that reads from localStorage) stays where it was in the cloud sync block, but it's now a **reassignment** instead of a new `let` declaration.
* No more TDZ: when `schedulePush()` reads `syncState` while `runMigrations() → saveStore()` runs, the binding already exists with value `null`, the `if(!syncState || ...)` guard returns early, no exception.

### Pre-release audit

* jsdom run of v3.0.1, v3.3 and v3.4.2 compared: only the versions with the fix (v3.0.1 and v3.4.2) render the welcome card. Raw v3.3 throws `ReferenceError`. Direct confirmation of the diagnosis.
* `bootstrap()` now runs normally on a fresh device → version tag set, `setSyncStatus("off")` called, welcome card opened after 250ms as expected.

### Internals

* Version bumped to `3.4.2`.
* Service worker cache key bumped to `wims-v3.4.2` to invalidate the 3.4.1 cache on the first activate.

### Lesson

Hoisting + TDZ make a sneaky trap: a function declaration can call code that depends on `let`/`const` bindings not yet initialised. The `if(typeof X === "function") X()` pattern worked as an existence check for the function declaration but did **not** protect against the TDZ inside it. Rule to remember: a function that uses `let`/`const` from another section of the file should be called only *after* both are initialised, or all the referenced `let` bindings should be forward-declared at the top of the module.

---

## [3.4.1] - 2026-05-14

Bugfix release: the version installed via "Add to Home Screen" on iOS stayed stuck on an old build (v2.0) even when Safari in the browser already showed 3.4. The problem was structural (the app had never had a service worker or a `manifest.webmanifest`), so the standalone iOS WebView cached the HTML aggressively and never refreshed it.

### Fix: service worker

* **New `sw.js` file with a network-first strategy for HTML navigations.** When the device is online, every launch of the app always takes the latest `index.html` from the network (`cache: 'no-store'`) and updates its local copy; when offline, it falls back to the cache. For static assets (Google fonts, inline icons) the strategy is cache-first, so as not to waste bandwidth.
* The `activate` event deletes all caches with a key different from the current one (`wims-v3.4.1`), so every version bump invalidates the old cache.
* `skipWaiting()` + `clients.claim()` to activate the new SW immediately on the first useful reload, without waiting for all tabs to close.

### Fix: PWA manifest

* **New `manifest.webmanifest`** with `name`, `short_name`, `start_url`, `scope`, `display: standalone`, `theme_color` and `background_color` matching the Quiet Ledger palette (`#FAF7F2` paper). Inline icons as SVG data URIs (both `any` and `maskable`), so the file stays self-contained like the rest of the project (no binary assets in the repo).
* `<link rel="manifest">` link added in the head after the `apple-mobile-web-app-*` meta tags. iOS will keep using `apple-touch-icon` for the home screen icon (the manifest is ignored for that purpose); the manifest mainly serves Chrome/Android for a proper install.

### Fix: hard-coded version tag

* **Removed the `v2.0` fallback** in the `#version-tag` tag (line 1781): if for any reason the bootstrap JS didn't run before the first paint, the number the user saw was 2.0, which was misleading while debugging this very bug. Now the tag is empty until `bootstrap()` fills it with the current `VERSION`.

### Note for users already stuck on old versions

* Users who already added the app to the home screen with a pre-SW build need to remove it and reinstall it **once**: the HTML cached by the WebView doesn't contain the SW registration, so the new service worker can't be installed from inside the stuck copy. Steps: remove the icon from the home screen → open Safari on the site → Safari settings → clear the website data for the domain → reload → Add to Home Screen.
* From then on, all updates arrive automatically at the next launch (with a network connection). No more "I see 3.4 in the browser but 2.0 in the app".

### Pre-release audit

* `sw.js` validated with `node --check`: PASS.
* `manifest.webmanifest` validated as JSON with `node -e "JSON.parse(require('fs').readFileSync('manifest.webmanifest'))"`: PASS.
* SW scope = `./` (default) → correct under `https://massimodascola.github.io/WhereIsMySalary-/`: the SW only intercepts requests within the project's subpath, not other repos under the `github.io` domain.

### Internals

* Version bumped to `3.4.1`.
* Cache key bumped to `wims-v3.4.1` to automatically invalidate any assets cached at the first SW install.

---

## [3.4] - 2026-05-14

Five changes that rebalance the Salary page around the user's **actual pay cycle** (no longer assumed to be "end of month"), add the toggle to turn Overtime off, tone down the logo and double the ironic taglines.

### Salary: configurable payday

* **Two new settings**: `payDay` (1-28, default 27) and `payDayNextMonth` (boolean, default false). Together they define when the salary for month N "lands": on the `payDay` of the same month, or on the `payDay` of the following month (for pay in arrears, e.g. May paid on 15 June).
* **New functions `paymentDateFor(year, m0, settings)` and `isPaymentDue(year, m0, settings, now)`** replace the old "past vs current vs future" logic based on the end of the month. The whole Salary page now follows the configured cycle: `computeYearTotals` adds to "missing" only the months whose pay date has passed; `monthStatus` labels as "upcoming" everything that isn't due yet, even in the current month before the pay date.
* `payDay` clamped to 28 to avoid "skipping" February (a date that's always valid in every month).
* Backward-compatible migration via `ensureProfile`: pre-3.4 profiles get `payDay: 27, payDayNextMonth: false` (= the historical behaviour).
* UI in the settings and in the new-profile sheet: two fields in the `.field-row`, "Payday" + a "Same month / Following month" select.

### Salary: Overtime toggle

* **New `overtimeEnabled` setting (default true).** When false:
  * The floating tabbar at the bottom of the screen is hidden (no more switching between the two pages).
  * The "Overtime" item disappears from the component list in the months on the Salary page.
  * `expectedAmountFor("overtime")` returns 0 whatever the state, so overtime contributes neither to the yearly totals nor to the month status.
  * If the user was on the Overtime page when turning it off, they are redirected to Salary.
* iOS-style toggle (`.settings-toggle` + `.toggle-track`) next to the "Overtime" section title in the settings. Visually discreet, immediate confirmation of the state.
* `renderOvertimeVisibility()` introduced as a step of the main render; `switchTab()` refuses switches to Overtime when it's disabled (defence in depth).

### Overtime: hero from the left

* **The layout changes from an asymmetric grid to flex with `justify-content: flex-start`** (gap `6px 28px`). The euro amount on the left, the hours right after it, both anchored to the left edge of the hero. The "asymmetric centre" pattern of 3.2.4 didn't convey the reading priority; now it reads linearly from left to right.
* Removed the `.primary` / `.secondary` classes (the layout no longer needs them).

### Logo: all lowercase

* **"Where is my *Salary*" → "where is my *salary*"** (lowercase initials). It tones down the "product" feel even further and moves closer to the handwritten gesture of the *Quiet Ledger* philosophy.

### Taglines: `{payDay}` + doubled

* **`{payDay}` placeholder added** in `interpolateTagline`. The historical lines `"Il 27 non è una data. È una promessa."` ("The 27th isn't a date. It's a promise.") and `"I conti si fanno alla fine. Anche se l'app li fa al 27."` ("The sums get done at the end. Even if the app does them on the 27th.") now use `{payDay}` and adapt to the day chosen by the user.
* **Both lists doubled**: `TAGLINES_ORE` 15 → 30, `TAGLINES_STIPENDIO` 15 → 30. The new lines carry on the app's ironic, emotional tone (`"Stai costruendo il futuro. Ti pagheranno nel passato."`, "You're building the future. They'll pay you in the past."; `"Il salario è la prima bugia che ti raccontano, l'ultima che credi."`, "Your salary is the first lie they tell you and the last one you believe.").

### CSV backup: coverage of the new settings

* **Three extra columns in `SETTINGS`**: `pay_giorno`, `pay_mese_dopo` (0/1), `overtime_abilitato` (0/1). The gap was found in the pre-release audit: without these columns, an export → re-import round trip restored the v3.3 defaults (27, same month, enabled), silently losing the user's choices.
* Tolerant parser (*prefix* header) → v3.3 files without these columns load with the defaults, as before. Boolean values accept `0/1` as well as `sì/no` / `true/false`, for robustness.

### Pre-release audit

* `node --check` on the extracted JS: **PASS**.
* Grep for dead references (`empty-mark`, `salaryMultiplierFor`, `profile-pill`, `cal-grid`, `hero-amount-aux`, `.hero-amount.primary/.secondary`): **0 hits** in active code (just 1 explanatory comment).
* Checked `paymentDateFor` on the month 12 + offset 1 edge case: JS `new Date(year, 12, payDay)` correctly rolls over to January of the following year. The `due` calculation stays consistent across the year boundary.
* Checked the CSV round trip on a profile with `payDay=15, payDayNextMonth=true, overtimeEnabled=false`: the import preserves all three values.

### Internals

* Version bumped to `3.4`.

---

## [3.3] - 2026-05-14

Stable release that consolidates the work cycle on **Overtime** started in 3.1. No new features compared with 3.2.4: just a version bump that closes the sequence of 5 iterations (`3.2 → 3.2.4`) and marks the end of the rework.

### Summary of the 3.1 → 3.3 cycle

| Version   | Main change                                                          |
|-----------|----------------------------------------------------------------------|
| 3.1       | Overtime hero with a double metric (euros + hours), `missing` default for overtime, yearly "Overtime {year}" KPI |
| 3.2       | Hours promoted to `.hero-amount` (same scale as the euros), retroactive `v3_1_overtime_default` migration |
| 3.2.1     | `.hero-amounts` inline flex                                          |
| 3.2.2     | `justify-content: space-between` (later rejected)                    |
| 3.2.3     | `justify-content: center` (later rejected)                           |
| 3.2.4     | Asymmetric `1fr auto 1fr` grid with the hours on the left and the euros in the centre |
| **3.3**   | **Consolidation. Same codebase as 3.2.4, version bump.**             |

### Pre-release audit

* `node --check` on the extracted JS: **PASS**.
* Grep for dead references (`empty-mark`, `salaryMultiplierFor`, `profile-pill`, `cal-grid`, etc.): **0 hits** in active code (just 1 explanatory comment).
* `v3_1_overtime_default` migration unchanged since 3.2, already tested on the user's profile with a good result.

### Internals

* Version bumped to `3.3`.

---

## [3.2.4] - 2026-05-14

### Overtime: asymmetric hours/euros layout

* **`.hero-amounts` goes from flex to a 3-column grid (`1fr auto 1fr`).** The two metrics are no longer *siblings* in a symmetric layout; they now live in semantically distinct positions: the hours (`.secondary`) sit in col 1, left-aligned, and the euro amount (`.primary`) sits in col 2, centred. The `1fr` of col 3 works as an invisible mirror that keeps the primary visually in the centre of the hero even when the secondary is present.
* When there's no rate (no euros to show), the primary holds the hours directly: the grid layout stays valid because the primary in col 2 with a 1fr on each side is already centred.
* Same Quiet Ledger pattern: the figure "that matters" (in the centre) has the prominent position, the other sits next to it like a note in the margin.

### Internals

* Version bumped to `3.2.4`.

---

## [3.2.3] - 2026-05-14

### Overtime: centred pair with breathing room

* **`.hero-amounts` goes from `justify-content: space-between` to `center` with `gap: 8px 48px`.** The two figures sit in the centre of the box, separated by a generous but fixed gap. No longer anchored to the edges of the hero: the "far ends" effect looked too unbalanced. Centred, with room in between, they read as a balanced pair.

### Internals

* Version bumped to `3.2.3`.

---

## [3.2.2] - 2026-05-14

### Overtime: euros on the left, hours on the right

* **`.hero-amounts` gets `justify-content: space-between`.** The two figures now live at the two ends of the hero box (euro amount on the left, hours on the right); the space in between lets the page's rhythm breathe and echoes the *Quiet Ledger* pattern of "margins as chosen silences". Horizontal gap reduced from 22px to 16px, since `space-between` sets the real spacing anyway.

### Internals

* Version bumped to `3.2.2`.

---

## [3.2.1] - 2026-05-14

### Overtime: euros + hours on the same line

* **The two hero figures go from stacked to side by side.** In 3.2 they had been placed one above the other (rule `.hero-amount + .hero-amount { margin-top: 4px }`) but they read vertically, almost like "main + appendix". Now the two metrics sit on the **same line** inside a `.hero-amounts` wrapper (flex with `align-items: baseline` and `gap: 8px 22px`).
* `flex-wrap: wrap` keeps a fallback for very narrow viewports (below ~340px): if the two figures don't fit on one line, they wrap automatically with a smaller gap. On any screen > 340px they sit next to each other, as the design intended.
* The margin-top of the first `.hero-amount` is reset inside `.hero-amounts` (the wrapper holds the 14px margin-top towards the greeting above).

### Internals

* Version bumped to `3.2.1`.

---

## [3.2] - 2026-05-14

Corrections to the choices made in 3.1: the visual hierarchy in the Overtime hero didn't convey that the two metrics are on an equal footing, and the new default for `overtime` didn't apply to existing profiles, which made it effectively invisible.

### Overtime: hours and euros both in the lead

* **Unpaid hours are now a second full-scale `.hero-amount`**, not a sub-text. In 3.1 they were rendered in `.hero-amount-aux` (serif italic 15px): readable but clearly a "caption". Now both numbers use the same `.hero-amount` class, the same size `clamp(40px, 11vw, 56px)`, the same weight, the same `intero + ,decimali` logic (integer part + decimals) with a smaller `.cents`. It shows visually that "money owed" and "unpaid hours" matter equally.
* Helper `buildHoursHtml(h)` to generate the same `123<span class="cents">,5 h</span>` markup, used both when the hours are in the foreground (no rate) and when they sit under the euro amount.
* CSS rule `.hero-amount + .hero-amount { margin-top: 4px }` keeps the two numbers close together like a pair, not like headline + note.
* Removed the old `.hero-amount-aux` class (no longer referenced).

### Retroactive migration: overtime default

* **`runMigrations()` introduced as a bootstrap step right after `loadStore()`.** In 3.1 the new `expectedDefault:true` default on `overtime` only applied to months *never created*, but `ensureYear` already creates all 12 months of the year as soon as you visit the page, so a v3.0 user who had already opened the app had 12 months with `state: "not_expected"` saved in localStorage, immune to the change of default. The new feature was invisible.
* **`v3_1_overtime_default` migration**: it goes through all the months of all profiles and converts to `"missing"` *only* the overtime cells that meet two conditions: `state === "not_expected"` AND `hoursOverride == null`. The second condition protects the user's active choices: if they had set an override by hand (perhaps precisely to control that month), the "not_expected" is a deliberate choice and stays.
* The `store._migrations.v3_1_overtime_default = true` flag is saved the first time the migration runs, so it isn't reapplied at the next reload. No side effects, no infinite loops.
* Extensible pattern: for future migrations, just add a new `if(!store._migrations.<nome>)` inside `runMigrations()`.

### Internals

* Version bumped to `3.2`.

---

## [3.1] - 2026-05-14

Three changes to the Overtime page and to the overtime item on the Salary page, prompted by real use: the main metric was one-sided, the default made the numbers hard to see, and there was no yearly overtime total.

### Overtime: double metric in the main box

* **Under the euro amount, a line with the unpaid hours.** When a rate is set, the hero only showed `€ 1.200,00`. Now it also shows `123,5 ore di straordinario non pagate` ("123.5 unpaid overtime hours") next to it. The double metric answers two different questions at a glance: *how much* I'm owed (euros) and *what* was worked without pay (hours). New `.hero-amount-aux` style: serif italic 15px, ink-soft, tabular-nums.
* When there's no rate the box stays as it was (only the hours in the foreground).
* When there's nothing to be paid, the aux line disappears automatically.

### Salary: overtime default "missing"

* **`expectedDefault` on `overtime` goes from `false` to `true`**. Before, new months were created with the state `non spettato` ("not due") for overtime: every time, the user had to switch it to "Missing" by hand for the Overtime page's numbers to count those hours as to be paid. Now the default is *not paid yet*, consistent with the fact that, in this app, overtime is something you assume you haven't been paid for until you mark it as received.
* `ensureYear` migration updated accordingly, for consistency with the new default.
* Existing v3.0 months keep their state (the user might have set them to "Not due" on purpose). The new default only applies to months never opened.

### Overtime: dedicated yearly KPI

* **Third `.hero-stat` changed from "hours worked this year" (gross) to "Overtime {year}" (paid).** The previous value added up *all* the hours logged in the year: useful information, but it doesn't help answer the app's key question ("how much overtime did I do?"). Now the KPI adds up the hours *above the threshold*, month by month: the real number of overtime hours in the year, regardless of how many have been received.

### Internals

* New local calculation `thisYearOvertimeHours` in `renderPageOre`: it runs `paidHoursForMonth` over the 12 months of the current year. Negligible cost, the render already recalculates everything at every interaction.
* Version bumped to `3.1`.

---

## [3.0.1] - 2026-05-14

Copy patch: a tagline in the "Salary" hero contained a hard-coded value that no longer matched the user's settings.

### Copy

* **Tagline "You have the right to dream. And to a {ticket} meal voucher." made dynamic.** Before, it read `"… ticket da 8 euro"` ("… an 8 euro voucher") whatever the actual value in the settings: a joke that missed the point if the user had a voucher of a different value. Now the value comes from `settings.ticketPerDay` of the active profile. The sentence in the array contains `{ticket}` as a placeholder, resolved at render time.

### Internals

* **New helper `ticketPhrase(n)`**: formats the number so that it sits well inside a sentence: `10` → `"10 euro"`, `8.50` → `"8,50 euro"`. `eur()` isn't used because `"€ 10,00"` breaks the rhythm of the joke.
* **New helper `interpolateTagline(text)`**: applies the placeholder substitutions, reading from the active profile. It wraps both `pickStipTagline()` and `pickOreTagline()`, so future taglines on both pages can use `{ticket}` (and other placeholders we'll add) without touching the picker logic.
* Version bumped to `3.0.1`.

---

## [3.0] - 2026-05-14

Stable release that closes the rework cycle started with 2.0. No new features compared with 2.5: just a critical fix, a cleanup of dead code and the consolidation of the design system as an official document. The bump to 3.0 signals that the seven 2.x iterations are now integrated, the UI has been redesigned end to end (tabs in English, settings sections, configurable monthly payments, redone calendar, dynamic inputs, stronger logo) and the app is considered ready for daily use.

### Critical fix

* **Double declaration of `const EXTRA_SALARY_DEFAULT_MONTHS`**, introduced by accident in 2.5 during a code reorganisation. It caused `SyntaxError: Identifier 'EXTRA_SALARY_DEFAULT_MONTHS' has already been declared` at parse time, preventing the whole script from loading. Fixed by removing the duplicate. The constant now lives only next to `COMPONENTS` (a declaration semantically close to what uses it). Node `--check` on the extracted JS: PASS.

### Cleanup

* **Removed the dead CSS rule `.empty-mark`** (~10 lines). The span with the "€" in the circle had been removed from the HTML in 2.2, but the rule had been left orphaned.
* **Stale copy updated**. The "month by month" phrases left in two places (the Overtime heroSub when there are no entries, the empty card copy) were pre-2.1 leftovers: the app no longer has that framing. Replaced with leaner variants.

### Documentation

* **`DESIGN-SYSTEM.md` introduced as the reference document.** A complete extraction of the tokens, components and patterns in the code, with the reasons behind them and links to the *Quiet Ledger* philosophy. It includes an "Extension rules" section with 10 practical constraints to avoid visual drift in future iterations.

### Internals

* Version bumped to `3.0`.

### Compatibility guarantee

* v2.x data in `localStorage` keeps loading without an explicit migration (the `ensureProfile` → `ensureYear` → `defaultMonth` chain backfills all the new fields with sensible defaults).
* v2.x CSVs import thanks to the tolerant parser (*prefix* header).
* Cloud sync is untouched: existing gists stay readable.

---

## [2.5] - 2026-05-14

Typographic refinement: inputs in the item rows sized to their content, and a stronger header logo.

### Salary: inline inputs

* **Dynamic width.** The number inputs in the month rows (salary, 13th/14th/15th, bonus, refunds, voucher days, overtime hours) had a fixed `width: 56px`. Result: `1000,00` was cut to `1000,0(`, while `0,00` left a huge gap before the `€` symbol. Now each input gets `style="width: Nch"`, computed as `max(len(value), len(placeholder), min)` through the new helper `inputWidthStyle(value, placeholder, min)`. Result: the field shrinks around the number and the `€` (or `giorni`, "days", or `h × …`) always sits right next to it, with no gaps or truncation.
* **`field-sizing: content`** declared in the CSS for modern browsers (Chrome 123+, Safari 17+, Firefox 122+): the input grows by itself as the user types, with no re-render needed. Fallback `min-width: 3ch / max-width: 14ch` for older browsers. Horizontal padding reduced from `4px` to `2px` to win back space.

### Header

* **"Where is my *Salary*" logo bigger and with more presence.** Size moved from 19px to `clamp(24px, 5vw, 30px)`. Tight letter-spacing and unchanged weight; the serif italic pattern with the sienna accent on *Salary* stays. Vertical padding of the topbar slightly increased (`14/12 → 20/16`) to give the title room, now that the topbar holds fewer elements (no more brand mark, no more profile pill, no more "+ add profile").

### Internals

* Version bumped to `2.5`.

---

## [2.4] - 2026-05-14

Completing the monthly payments feature: the 13th/14th/15th month pay amount can now be edited month by month, as the base salary already could.

### Salary: extra monthly payment override

* **Editable input next to 13th month pay / 14th month pay / 15th month pay.** Same pattern as the monthly salary: inline number field, empty value = null sentinel ("inherit from the month's base salary"), numeric value = explicit override. The placeholder shows the amount that would be used by default (monthly salary override → fallback `settings.salary`).
* `defaultMonth()` now initialises `amount: null` for all the salary-like items (salary + extraSalary).
* `ensureYear` runs a backward-compatible migration for v2.3 months that have only `state` and no `amount`.
* `expectedAmountFor` for salary13/14/15 has a clear fallback chain: `cell.amount` (per-month override) → `data.salary.amount` (base salary override) → `settings.salary`.
* The `.amount-input` handler recognises `salary13/14/15` as salary-like and keeps the null sentinel on empty input.

### CSV backup

* **Three extra columns in `MESI`**: `salary13_importo`, `salary14_importo`, `salary15_importo`, after all the others. v2.3 files (without these columns) still load thanks to the tolerant parser (*prefix* header): the missing overrides become `null`.
* The `allDefault` heuristic, which avoids exporting months that are all defaults, now also considers `cell.amount != null` for the extraSalary items, so a month whose only change is an extra monthly payment override is still persisted.

### Internals

* Version bumped to `2.4`.

---

## [2.3] - 2026-05-14

Two changes: a fix for a visual bug in the calendar and a new data model for the extra monthly payments.

### Overtime: calendar

* **First row spacing fixed.** The previous rendering generated `N` "empty" cells (`<div class="cal-c empty">`) to align the first day of the month with the right weekday. On some browsers the empty cells with `aspect-ratio: 1` collapsed to height 0 / size 0 and broke the first row of the grid, leaving a huge blank space between the first day and the following ones. Now the first number cell uses `style="grid-column-start: N"` and no empty cells are generated any more: a cleaner solution, robust to the aspect-ratio bug. Also removed the `.cal-c.empty` CSS rule and the `:not(.empty)` selector in the click handlers.

### Salary: extra monthly payments

* **13th / 14th / 15th are separate items with their own tri-state.** Before, the logic was: salary × `salaryMultiplierFor(mese)`. That meant that in December, with 14 monthly payments, the "Salary" amount showed double, but there was no way to mark *only* the 13th as missing when the monthly salary had already arrived. Now:
  * Added three components to `COMPONENTS`: `salary13` (🎄, paid in December), `salary14` (🌞, June), `salary15` (✨, July).
  * They appear as a separate row *only* in the payment month and *only* if `settings.mensilita` reaches the threshold. The filter is in `extraSalaryApplies(comp, settings, m0)`, applied both in `renderMonthCard` and indirectly in `expectedAmountFor` (it returns 0 when the item doesn't apply, so totals and status stay correct).
  * Amount = the same as the monthly salary (including any per-month override on the base salary).
  * Removed `salaryMultiplierFor()`: no longer needed.
  * `defaultMonth()` creates the three cells with the default state "missing" (i.e. *expected* in the right month). `ensureYear` runs a backward-compatible migration.

### CSV backup

* **Three new columns in `MESI`**: `salary13_stato`, `salary14_stato`, `salary15_stato`. Tolerant parser (*prefix* header) → v2.2 files without these columns load with the default state "missing".

### Internals

* Version bumped to `2.3`.

---

## [2.2] - 2026-05-14

A UX refinement iteration on v2.1: empty state cleanup, the "Let's start with you" form aligned with the full settings sheet, settings reorganised into sections, and tabs renamed in English for consistency with the app's name.

### General

* **Tabs renamed: "Salary" and "Overtime"**. The first directly recalls *Where is my Salary*; the second was chosen over "Work time"/"Ore" ("Hours") because it emotionally evokes the central idea of the app: the extra work you do without the payslip seeing it. Consistent with the ironic, noir tone of the taglines.
* **Empty state cleaned up.** Removed the "€" circle above the greeting. The card is now vertically centred (flex column with `justify-content: center` and `min-height: 360px`), to give the copy more room and cut down on graphic frills.

### Settings

* **Three explicit sections with headings: "Salary" / "Overtime" / "System"**. Implemented with a new `.settings-section-title` (serif italic + dashed line below, sienna accent). It makes it immediately clear where a given field is.
* **Working days moved before Contract hours / CCNL allowance / Overtime rate**. Defining the days "that count" is a precondition for the calculations, so it comes first in the Overtime section.

### New profile sheet

* **Form aligned with the full settings sheet.** The "Let's start with you" sheet now asks for *all* the fields that used to be reachable only after creating the profile: payments per year, fringe month, standard bonus, contract hours, CCNL allowance, overtime rate and working days. The layout mirrors the settings exactly (with the same two sections, Salary / Overtime), so on first launch the user sets everything in one go, without having to reopen the settings afterwards.
* **All the `clamp()` validations are now also applied on the first save** (before, some fields ended up in the profile without the bound).

### Internals

* `openNewProfile()` now fills the `np-fringe-month` select and resets the workday checkboxes.
* The `#btn-create-profile` handler builds the settings with all the new fields and applies `clamp(v, max)` to every number.
* Version bumped to `2.2`.

---

## [2.1] - 2026-05-13

A release of visual cleanup, UX simplification and fixes to the hours calculations. No destructive data migration: existing profiles keep working, and new fields are added automatically with sensible defaults.

### General

* **`Stipendio · Ore` ("Salary · Hours") tabbar hidden above sheets and modals.** When the settings sheet or any overlay window opens, the floating pill at the bottom of the screen no longer stays visible above the backdrop. Implemented via the `body.modal-open` class (already there for sheets) + `body.overlay-open` (new, added inside `showConfirm` / `showChoice`).
* **Removed the 🕐 emoji from the "Ore" ("Hours") tab.** The floating box is now text only, consistent with the app's "paper" tone.
* **Header simplified.** Removed the small "€" square used as a logo and the "month by month · v2.0" line. Only the *Where is my Salary* title remains. The version moved to the bottom of the page (`.app-version`).
* **Multi-profile removed from the UI.** The profile switch pill and the "+ add profile" button are no longer rendered. The store stays multi-profile internally (so as not to break existing data), but the app always uses the first profile it finds. The "Delete profile" button in the settings was renamed **"Reset data"**, with updated copy.

### Salary

* **Configurable payments per year (12-15).** New select in the settings. Default 12. The extra monthly payments arrive in fixed months: **13th in December, 14th in June, 15th in July**. The logic lives in `salaryMultiplierFor(settings, m0)` and in `EXTRA_SALARY_DEFAULT_MONTHS`. For those with 14 monthly payments, the app now automatically calculates a double salary in June and December (no need to touch the manual override).
* **Roomier component layout.** The item/detail/amount/tri-state row was converted from flexbox to CSS Grid with `grid-template-areas`. On mobile (<480px) the amount drops below the item, so the name isn't squeezed in half.
* **No more dash between item and state when an item isn't due.** If the state is `not_expected`, the amount cell is left empty (`.comp-amount:empty { display:none }`). Result: a "Bonus" row that isn't expected no longer shows the orphan dash next to the tri-state buttons.

### Settings

* **New "Contract hours (h/week)" field.** Default 40. Together with the "CCNL allowance" it defines the threshold beyond which worked hours are paid as overtime. Example: contract 40h + allowance 5h = 45h covered by the salary, the rest is overtime.
* **Allowance semantics revised.** Before, the allowance was the only threshold (all logged hours above the allowance were overtime). Now the formula is `paid = max(0, workH - (contratto + forfait)) + weekendH`. The default allowance is now 0 (those who had the old 5 can keep using it; contract hours are set separately).
* **Overtime rate in its own field.** Moved out of the allowance/contract row to give the hints more room.

### Hours

* **Summary box: values aligned on the same line.** The three "week / month / year" cells have labels of different lengths (in Italian, "Settimana corrente" wraps and "2026 totale" doesn't). Now `.hero-stat` is a flex column with `margin-top: auto` on the value, so all three numbers sit on the same baseline even when the labels take two lines.
* **Overtime hours check fixed.** All the call sites of `paidHoursForMonth` (6 places) now pass the `contractWeek` parameter as well as `forfaitWeek`. `renderOreWeekView` also uses the new combined threshold.
* **Uniform calendar spacing.** Headers and cells were in two separate grids (`.cal-header` + `.cal-grid`): the first row of numbers looked further apart than the others. Now both live in a single `.cal-table` with a constant 4px gap.

### CSV backup

* **Extended schema (backward compatible).** Added two columns to the `SETTINGS` section: `contratto_settimana` and `mensilita`. The parser also accepts a *prefix* header (old files without the new columns load correctly with the defaults `40h` / `12 mensilità`, "12 monthly payments"). New exports include all the columns.

### Internals

* `defaultSettings()` now returns `mensilita: 12`, `oreContrattoSett: 40`, `forfaitOreSett: 0`.
* `ensureProfile()` runs the defensive migration of the three new fields on existing profiles.
* Version bumped to `2.1`.

### Compatibility notes

* If you have a profile from v2.0 with `forfaitOreSett: 5` (the old default), the overtime calculation may come out different on the first load. Suggested fix: go to **Settings**, set "Contract hours" to 40 (or whatever your contract says) and "CCNL allowance" to 0 (or 5 if you really have a CCNL allowance on top of your contract). Save.
* The "Reset data" button deletes the whole store. If you have several profiles (a v2.0 legacy) they will all be removed.

---

## [2.0] - Pre-2026-05-13

First version tracked in this CHANGELOG. Historical features (without chronological detail):

* Single-file app in HTML/CSS/vanilla JS, no build, no bundler.
* Monthly tracking of 7 items: salary, welfare, fringe benefits, meal vouchers, bonus, refund, overtime.
* Automatic calculation of Italian working days (national public holidays + Easter Monday via the Gauss/Meeus algorithm).
* Optional sync via GitHub Gist (PAT with the `gist` scope).
* CSV backup with `;` as separator and `,` as decimal mark (Italian convention, readable in Excel/Numbers).
* "Ore" ("Hours") section with days/weeks/months/years views, calendar and CCNL overtime allowance calculation.
* Multi-profile (removed from the UI in v2.1).

---

## How to write future entries

A changelog entry is useful *if it contains* the **what** + the **why**. Every release must let someone (me included, 6 months from now) answer:

1. **What changed?** A concrete description, with references to functions/files (`paidHoursForMonth`, `index.html:1727`).
2. **Why did it change?** The problem being solved, or the user's request.
3. **What do I need to know if I update?** Any breaking changes, data migrations, changed defaults.

Standard sections, in this order: General, Salary, Hours, Settings, Backup, Internals, Compatibility notes. Leave out empty sections.

Don't document every style edit: group small cleanup changes into a "Polish" entry if needed.
