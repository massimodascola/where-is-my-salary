# Design System: Where is my Salary

Reference document for the app's design system. It is an extraction of the CSS tokens, components and patterns found in `index.html`, annotated with the **why** behind the choices, so that future iterations stay consistent without having to reconstruct the intent every time.

The aesthetic draws on **Quiet Ledger** (see [`quiet-ledger-philosophy.md`](./quiet-ledger-philosophy.md)): warm paper as the primary surface, italic serif as the voice, a single sienna accent used like a wax seal. What follows translates that philosophy into practical rules.

---

## 1. Guiding principles

Three rules that win when in doubt:

1. **Subtract before adding.** Every extra icon, divider or shadow has to earn its place. The first instinct is to remove.
2. **There is only one accent.** Sienna `#C96442` is the sealing-wax red. It appears at the points of meaning (the number that counts, the app's voice, the primary CTA, "today"). It isn't spread around: it's concentrated.
3. **The figure is poetry.** Numbers live in serif, tabular-nums, with air around them. When a number has to be read, it is large and has room to breathe; when it's just service data, it keeps quiet.

---

## 2. Color

All tokens are declared in `:root` (see `index.html:27-78`). Every color has a role, not a "generic role": changing one changes a behavior, not an appearance.

### Surfaces (paper)

| Token            | Hex        | Use                                                    |
|------------------|------------|--------------------------------------------------------|
| `--paper`        | `#FAF7F2`  | App background, iOS/Android theme color                |
| `--paper-tint`   | `#F5F0E6`  | Secondary backgrounds (item icons, weekends in the calendar) |
| `--card`         | `#FFFEFB`  | Main cards (month, ore-card, week-row)                 |
| `--card-elev`    | `#FFFFFF`  | Elevated cards (hero, sheets, confirm card)            |
| `--line`         | `#E8DFCF`  | Standard border                                        |
| `--line-soft`    | `#F0E8D8`  | Inner dividers (row after row inside a card)           |

### Ink (text)

| Token          | Hex       | Use                                                    |
|----------------|-----------|--------------------------------------------------------|
| `--ink`        | `#2A2622` | Primary text (titles, item names, figures)             |
| `--ink-soft`   | `#5C534A` | Secondary text (hero-sub, hints, descriptions)         |
| `--ink-muted`  | `#978C7E` | Uppercase labels, neutral state, service copy          |
| `--ink-faint`  | `#BFB5A4` | Placeholders, "upcoming", weekend labels in the calendar |

### Sienna accent (the only one)

| Token             | Hex                          | Use                                                |
|-------------------|------------------------------|----------------------------------------------------|
| `--sienna`        | `#C96442`                    | "Salary" in the logo, primary CTA, "today", days with entries |
| `--sienna-deep`   | `#A4502F`                    | Hover/active state of the accent                   |
| `--sienna-soft`   | `rgba(201,100,66,.10)`       | Active chip background, "current month" outline, hero glow |
| `--sienna-tint`   | `#FAEFE8`                    | Warm radial glow in the page background (`body`)   |

### Companions (used sparingly, never the lead)

| Role               | Token                | Hex       | Use                                                |
|--------------------|----------------------|-----------|----------------------------------------------------|
| Green "ok"         | `--sage`             | `#6B8E5A` | "received" state, hero-stat ok                     |
| Red "missing"      | `--terracotta`       | `#B85449` | "missing" state, weekend days with entries in the calendar |
| Gold "pending"     | `--gold`             | `#C29B3D` | "warn" state (something arrived, something didn't); public holidays in Overtime |
| Blue "holiday"     | `--ocean`            | `#4F7CAC` | Holiday days in Overtime: calendar, week strip, entry card, counts, "Day type" selector |

Each one has a `-soft` (alpha 10 to 12%) and a `-tint` (pastel) for backgrounds.

### Rules for using color

* **Never use two different accents on the same screen** unless they signal two different states (received/missing).
* **Sienna is not for "decorating".** It signals *meaning* (this is the app; this figure is the figure; this day is today).
* **Sage/terracotta/gold/ocean are semantic**, never aesthetic. If you remove a meaning, remove the color.

---

## 3. Typography

Three families, each with its own voice. Loaded from Google Fonts with a fallback to the system stack (see `index.html:15-17` and the tokens in `:root`).

### Families

| Token       | Stack                                       | Voice                                                 |
|-------------|---------------------------------------------|-------------------------------------------------------|
| `--serif`   | Source Serif 4 · Charter · Georgia          | The intimate voice. Figures, month titles, totals, hero amount |
| `--sans`    | Inter · -apple-system · SF Pro Text         | The whisper. Labels, hints, body, buttons, forms      |
| `--hand`    | Caveat · Bradley Hand · Marker Felt         | The breath. Greetings ("good morning,", "Hi,"), "today", signature |

### Scale

It isn't a rigid "modular" scale: it follows **function** instead. Reference for the sizes that actually exist in the CSS:

| Role                        | Family    | Size                       | Weight | Style    |
|-----------------------------|-----------|----------------------------|--------|----------|
| Logo (brand-name)           | serif     | `clamp(24px, 5vw, 30px)`   | 500    | regular  |
| Logo accent (em "Salary")   | serif     | same                       | 500    | *italic* |
| Hero amount                 | serif     | `clamp(40px, 11vw, 56px)`  | 600    | regular  |
| Hero amount cents           | serif     | `.55em`                    | 500    | regular  |
| Section-title               | serif     | 22px                       | 500    | *italic* |
| Settings-section-title      | serif     | 16px                       | 500    | *italic* |
| Month-name                  | serif     | 18px                       | 500    | *italic* |
| Comp-amount                 | serif     | 15px                       | 500    | regular  |
| Hero-stat value             | serif     | 18px                       | 500    | regular  |
| Hero-greeting               | serif     | 16px                       | 400    | *italic* |
| Hero-sub                    | serif     | 14.5px                     | 400    | *italic* |
| Hero-tagline                | serif     | 13px                       | 400    | *italic* |
| Comp-name                   | sans      | 14.5px                     | 600    | regular  |
| Tab label                   | sans      | 14px                       | 600    | regular  |
| Comp-detail                 | sans      | 12.5px                     | 400    | regular  |
| Field label (uppercase)     | sans      | 11.5px                     | 600    | regular  |
| Hint                        | sans      | 12.5px                     | 400    | *italic* |
| Hero-stat key (uppercase)   | sans      | 11px                       | 600    | regular  |
| Salute ("good morning,")    | hand      | 22px                       | not set | regular |
| Empty-greet                 | hand      | 22px                       | 600    | regular  |
| "today" badge (current month)| hand     | 16px                       | 600    | regular  |
| Signature                   | hand      | 16px                       | 500    | regular  |

### Typographic rules

* **`font-variant-numeric: tabular-nums`** wherever figures need to line up vertically (hero-amount, comp-amount, hero-stat .v, oms-value, owr/oyr .v, the day numbers in the week strip). The alignment of numbers is a moral question (quoting *Quiet Ledger*). The month calendar cells (`.cal-c`) don't set it yet.
* **Italic serif for the intimate voices** (greetings, subtitles, months, section names). Italic isn't emphasis: it's closeness.
* **Sans serif when the function is instrumental** (labels, buttons, values in form fields). It never shouts.
* **Negative letter-spacing (-.012em to -.025em)** on large serif titles, so they don't feel "spaced out": the default serif already has the right kerning.
* **Positive letter-spacing (.04em to .08em) + uppercase** on service labels (`.k`, `.field label`, `.cal-h`). They are "archive labels" and don't want to stand out.
* **The hand font is used in only 4 places** (hero greetings, "today", "empty-greet", signature). It's the salt: if it shows up everywhere, it loses its flavor.

---

## 4. Spacing, radii, shadows

### Border radius

| Token              | Value   | Use                                                    |
|--------------------|---------|--------------------------------------------------------|
| `--radius-card`    | 18px    | Cards (month, hero, ore-card, desktop sheet, summary)  |
| `--radius-soft`    | 12px    | Buttons, pill inputs, day strip                        |
| `--radius-pill`    | 999px   | Pills (year-picker, status-pill, tri-state, tabbar)    |

Values of 9 to 11px also recur on small elements: `.icon-btn` and `.comp-icon` (11px), form fields (10px), the `.kind-switch` pills (9px). The month cards and the hero use the large radius (18px) to feel like a "block of paper", not a "software card".

### Shadows

Three levels, all warm (an earthy sienna tint, never neutral grays):

| Token             | Value                                                                                 | Use                                              |
|-------------------|---------------------------------------------------------------------------------------|--------------------------------------------------|
| `--shadow-rest`   | `0 1px 0 rgba(125,105,75,.04), 0 1px 2px rgba(125,105,75,.06)`                        | Cards at rest (month, week-row, day-row, btn-sec)|
| `--shadow-soft`   | `0 6px 18px rgba(125,105,75,.08), 0 1px 2px rgba(125,105,75,.05)`                     | Card hover, current month                        |
| `--shadow-warm`   | `0 14px 40px rgba(155,110,75,.12), 0 1px 2px rgba(125,105,75,.04)`                    | Hero, tab bar, confirm-card, empty-card, toast   |

Sheets use their own upward shadow (`0 -10px 40px rgba(125,105,75,.18)`) in the same tint.

Shadows **are not dimmed black**: they are ochre (RGB `125,105,75` and `155,110,75`). A tiny difference, a huge change in perception: the object sits on paper, it doesn't float in a void.

### Spacing

There is no rigid spacing scale (8, 16, 24…). The real values are pragmatic: `4 / 6 / 8 / 10 / 12 / 14 / 18 / 22 / 26` px recur often. Main patterns:

* **Card padding**: `14px 16px` (compact) to `26px 24px 22px` (hero).
* **Sheet body padding**: `18px 22px`. Sheet foot: `12px 22px max(16px, env(safe-area-inset-bottom))`.
* **Gap between month cards**: `12px`. Gap between consecutive ore-cards: `10px`. Gap between days in the calendar: `4px`.
* **Top margin of a new section**: `22px` to `30px` (`.section-head` has `margin: 30px 4px 14px`).

The **safe area insets** (`env(safe-area-inset-*)`) are respected in the body padding and in the sheet-foot (the app can be installed as a PWA on iOS).

---

## 5. Components

The components reused in the code, described as units with their own grammar.

### 5.1 Hero

The "lectern" card at the top of each page (`Salary` and `Overtime`).

Structure:
```
.hero
├── .hero-greeting    (handwritten greeting + name)
├── .hero-amount      (the leading figure, large serif)
├── .hero-sub         (context sentence, serif italic 14.5px)
├── .hero-tagline     (wry line in quotation marks, ink-faint)
└── .hero-stats       (3 KPIs in a 3-column grid, flex column with the value at the bottom)
```

Details:
* `::before` pseudo-element with a sienna-soft radial gradient in the top right corner (the "patch of light").
* `::after` pseudo-element with a wavy underline drawn as inline SVG (the human trace).
* `.hero-amount` has the `.ok` (sage) and `.danger` (terracotta) states. Without a modifier it is ink.
* The cents (`.cents`) are 55% of the integer's size: the main value gets read, the small change doesn't.
* `.hero-stat` is a flex column with `margin-top: auto` on the value → the three KPIs line up on the same baseline even when the labels wrap.
* On the Overtime page `.hero-amount` sits inside a `.hero-amounts` row that can hold two figures side by side (euros and hours, same size and weight), wrapping to two lines only on very narrow screens.

### 5.2 Month card (Salary)

```
.month  (.current for the current month, .open when the accordion is open)
├── .month-head
│   ├── .month-name      (serif italic; lowercase in the Italian interface only)
│   ├── .month-status    (semantic chip: ok / warn / danger / future / "not due")
│   └── .month-chevron   (rotates 180° when open)
└── .month-body          (display: none by default; .open .month-body { display: block })
    └── .comp × N        (one row per visible item)
```

Month names come from the `months` list of the `I18N` dictionary. Lowercase is applied to the Italian interface only, via `html[lang="it"] .month-name`; English keeps the capitals.

`.month.current` has a sienna border + a 3px `sienna-soft` outer ring + a handwritten "today" chip next to the name (its text comes from the `I18N` dictionary through the `data-today` attribute). It is the only card that speaks.

### 5.3 Component row

The basic unit inside a month. A grid layout with named areas (see `index.html:373-450`):

```
desktop:                          mobile (<480px):
"icon info amount tri"            "icon info  tri"
                                  "icon amount tri"
```

Four pieces:
1. **`.comp-icon`**: a small 38×38 square with the emoji, `--line-soft` border on `--paper-tint`. The emoji is the only colorful visual concession, inside a neutral container.
2. **`.comp-info`**: name (`.comp-name`, sans 14.5px bold) + detail (`.comp-detail`, sans 12.5px muted). The detail can contain inline inputs (see 5.4).
3. **`.comp-amount`**: the month's figure, serif tabular-nums. Modifiers `.dim / .ok / .danger`. Empty → `display: none` (no orphan dash).
4. **`.tri`**: tri-state pill (see 5.5).

### 5.4 Inline inputs in the component row

The inputs that live in the row (salary override, meal voucher days, bonus and expense refund amounts, overtime hours, 13th/14th/15th month pay amounts) follow a specific rule:

```css
.comp-detail input{
  background: transparent;
  border: none;
  border-bottom: 1px dashed var(--line);
  field-sizing: content;       /* auto-grow in modern browsers */
  width: 5ch;                  /* fallback */
  min-width: 3ch;
  max-width: 14ch;
  padding: 1px 2px;
  font-variant-numeric: tabular-nums;
}
```

On top of that, the JS helper `inputWidthStyle(value, placeholder, min)` (in `index.html:3721`) injects an inline `style="width: Nch"` on every render, so the "initial" width is already right before the browser applies `field-sizing`. Result: `0.00` and `1000.00` take exactly the space they need, with no truncation and no gaps.

### 5.5 Tri-state pill

Three buttons in one pill (received / missing / not due):

```
.tri
├── button[data-state="received"]      (✓)  → on: background sage, color white
├── button[data-state="missing"]       (!)  → on: background terracotta, color white
└── button[data-state="not_expected"]  (—)  → on: background ink-faint, color white
```

There is no yes/no boolean because the real semantics are ternary. The difference between "missing" and "not due" is the heart of the data model.

### 5.6 Floating tab bar

An iOS-style pill at the bottom, fixed in place (`position: fixed`), with two tabs: **Salary** and **Overtime**. It disappears when there is no profile yet, or when Overtime is switched off in the settings (only Salary is left, so there is nothing to switch to).

* `z-index: 25`: above the sheet scrim (20) and the sheet (21), below the confirm-overlay (30).
* When a sheet or a confirm is open, the tab bar is hidden (`body.modal-open .tabbar` / `body.overlay-open .tabbar` `→ display: none`). This was an explicit request, to avoid having a second "floating bar".
* The active tab has a `--sienna-soft` background and `--sienna-deep` text. There is no other indicator (no underline, no badge).

### 5.7 Sheet (slide-up modal)

Pattern used for:
* **Settings** (`Your numbers`): sections *Salary*, *Overtime*, *System*.
* **First-run wizard** (`Let's start with you`): one step at a time, see below.
* **Sync setup** (`Set up cloud sync`).
* **Overtime entry** (`New entry` / `Edit entry`).

Structure:

```
.scrim (backdrop)
.sheet
├── .sheet-handle    (the drag handle pill)
├── .sheet-head      (title + close)
├── .sheet-body      (scroll area: flex: 1 1 auto, min-height: 0)
└── .sheet-foot      (primary + secondary CTAs, pinned to the bottom)
```

Responsive behavior:
* **Mobile** (< 640px): slides up from the bottom at full width, covering up to `92dvh`.
* **Desktop**: centered, max-width 560px, popup-style with a scale-in.

The inner sections use `.settings-section-title` (serif italic 16px sienna, dashed underline) as headings.

**First-run wizard** (`#new-profile-sheet`): the same sheet, with a `.wz-progress` (a 3px sienna bar) under the head, "Step N of M" (`.wz-count`, 11px uppercase faint) and a single visible `.wz-step`, which opens with a `.wz-intro` sentence (serif italic). Yes/no questions use `.wz-q` (15px, ink) plus the `.kind-switch` selector; the fields that a "Yes" reveals carry `data-show-if`. The sheet has a fixed height (`min(92dvh, 680px)`, desktop `min(86vh, 680px)`) so that the Back/Next buttons in the foot don't move from one step to the next. The last step is a `.wz-summary`: `.wz-row` rows with a dashed separator, clickable to go back to that step.

### 5.8 Calendar (Overtime)

A single grid `repeat(7, minmax(0, 1fr))` with `aspect-ratio: 1` on the day cells. Headers (`.cal-h`) on row 1 with auto height. The first numbered cell uses `grid-column-start` to line up with the right weekday (no "empty" cells: that pattern broke the height of the first row in some browsers).

Cell states:
* **Default**: ink number, invisible border.
* **`.weekend`**: paper-tint background, ink-muted text.
* **`.has-events`**: full sienna, white text, bold.
* **`.has-events.weekend`**: terracotta (working at the weekend is another matter altogether).
* **`.festivo`** (public holiday): full gold, white text. Wins over `.has-events`.
* **`.pto`** (holiday): full ocean, white text. Wins over `.has-events`.
* **`.today`**: 2px sienna outline.

In the week strip of the day detail the same states use the `-tint` (gold-tint, ocean-tint) as background; `.selected` always wins.

Header of the sub-views (month calendar, day detail): a single `.ore-subhead` row with the back button (`‹ Months`, `‹ Days`) on the left and the period's `.year-picker` on the right (`‹ September 2026 ›`, `‹ Week 39 ›`). No big title: the period lives in the picker, so the content starts right below. The label has `min-width: 9em` so that the left arrow doesn't shift while scrolling through months with names of different length.

### 5.9 Buttons

One family, three variants:

| Class               | Background          | Text   | When                                         |
|---------------------|---------------------|--------|----------------------------------------------|
| `.btn`              | `--sienna`          | white  | Primary CTA (Save, Create my profile, Add entry) |
| `.btn.secondary`    | `--card`            | ink    | Neutral action (Export, Import)              |
| `.btn.danger`       | `--terracotta`      | white  | Destructive action (Reset data, Delete)      |

Modifiers: `.full` (width 100%), `.large` (padding 14px 22px, font 15px).

`.btn` and `.btn.danger` share the inner highlight `inset 0 1px 0 rgba(255,255,255,.16)` (under a soft sienna drop shadow) to suggest the "press of a stamp"; `.btn.secondary` replaces it with `--shadow-rest`.

### 5.10 Status pill (month-status)

A small chip with four semantic variants:

| Modifier   | Background          | When                                       |
|------------|---------------------|--------------------------------------------|
| (none)     | paper-tint          | "not due" (month with no expected items)   |
| `.ok`      | sage-tint           | "all received"                             |
| `.warn`    | gold-tint           | "missing X" (partial); also the fallback "pending" |
| `.danger`  | terracotta-tint     | "owed X" (nothing has arrived yet)         |
| `.future`  | transparent + line  | "upcoming"                                 |

The text varies but the colors are fixed and map 1:1 from state to meaning.

### 5.11 Form

A single pattern:

```
.field
├── label                  (sans 11.5px uppercase muted)
├── input / select         (paper bg, line border, radius 10px, focus → sienna ring)
└── .hint                  (sans 12.5px italic muted, optional)
```

`.field-row` places two `.field` side by side in two equal columns (a flex row with `flex: 1 1 0` children, so native time inputs can shrink on narrow screens).

`.workday-row` is the Monday to Sunday selector: one pill per day, sienna soft + sienna border when selected.

`.kind-switch` is the pill selector for exclusive choices (day type in the entry form, yes/no in the wizard): real `radio` inputs; the active pill is card-elev with sienna-deep text, or takes the color of its type (ocean for Holiday, gold for Public holiday). The `.kind-switch.compact` variant (lower pills, 12.5px text, aligned right) is the language choice at the top of the wizard's first step; in the settings the same choice uses the regular `.kind-switch`. In English the day-type pills take the width of their text ("Public holiday" is long); in Italian they stay three equal thirds.

On touch screens (`pointer: coarse`) the inputs and selects of `.field` are 16px instead of 15px: below 16px, iOS Safari zooms in on the page when the field gets the caret, and doesn't zoom back out on its own.

---

## 6. Iconography

### Emoji as item icons

The items in a month use emoji as icons inside a neutral small square (`.comp-icon`):

| Item             | Emoji  |
|------------------|--------|
| Salary           | 💼     |
| 13th month pay   | 🎄     |
| 14th month pay   | 🌞     |
| 15th month pay   | ✨     |
| Welfare          | 🌿     |
| Meal vouchers    | 🍱     |
| Fringe benefits  | 🎁     |
| Bonus            | ⭐     |
| Expense refund   | 🧾     |
| Overtime         | 🕐     |

**Rule**: the emoji lives *only* inside `.comp-icon`. It doesn't appear in labels, headers, buttons or tabs. The tab bar was explicitly cleaned up (no `🕐` next to "Overtime") so that it doesn't shout.

### Inline SVG

The controls (settings gear, month chevron) are inline SVGs of 16 to 18px with `stroke: currentColor`, so they inherit the color of their container: `--ink-soft` in the `.icon-btn`, `--sienna` on hover. The year-picker arrows are plain `‹` `›` characters that follow the same logic (`--ink-muted`, `--sienna` on hover).

### Pseudo-decorations

The hero's wavy underline (`hero::after`) and the legend pips (`.dot.ok / .miss / .na`) are made without external assets: inline SVG as a data URI, or small colored `<span>`s. No images loaded from the network.

---

## 7. Voice & tone

The interface is available in **English** and **Italian**: English by default, Italian when the device language is Italian, and the choice can be changed in the settings (*System* section) and on the first step of the wizard. In both languages two things stay in English, both intentional:

* **Logo "Where is my *Salary*"**: the original name, with the italic sienna accent on the last word.
* **Tabs "Salary" / "Overtime"** (and the settings sections with the same names): consistent with the logo, and wry (quoting *"Overtime is the work you do that the paycheck doesn't see"*).

### Traits of the copy

1. **Short sentences, quotation marks, pause.** The taglines sit between typographic quotes (`\201C`/`\201D` via pseudo-element) to give the sense of something "said quietly".
2. **Irony about waiting.** All the taglines (`taglines_salary` and `taglines_ore` in the `I18N` dictionary, in Italian and English) are meant as the bittersweet remarks of a ledger that has seen too many 27ths of the month:
   * *"The 27th isn't a date. It's a promise."* (the day is the payday set in the profile, 27 by default)
   * *"You can't get the hours back. The money, you can."*
3. **Contextual greetings.** "good morning" / "good afternoon" / "good evening" / "still up" change with the time of day: the ledger is awake when you are.
4. **Confirm/alert dialogs** replace the native ones (`showConfirm`, `showAlert`, `showChoice`), so the copy can keep the same tone everywhere.

### Internal glossary

| UI term              | Meaning                                                      |
|----------------------|--------------------------------------------------------------|
| "received"           | amount that arrived in the payslip, sage green               |
| "missing"            | due, not arrived yet, terracotta                             |
| "not due"            | not expected for that month, ink-faint                       |
| "missing X"          | partially arrived                                            |
| "owed X"             | everything still to be collected                             |
| "all received"       | month closed, the numbers add up                             |
| "upcoming"           | month whose payday hasn't come yet                           |
| "not due"            | month with no expected items (a legitimate blank)            |
| "your numbers"       | title of the settings sheet                                  |
| "Reset data"         | not "Delete profile": it wipes all the data (months, settings and overtime) at once |

---

## 8. Hierarchy & rhythm

Three rules of visual hierarchy, in order of strength:

1. **Scale**: the hero amount is `clamp(40px, 11vw, 56px)`, the other figures are 15 to 18px. The difference is up to ~4×, not ~30%. When a figure has to speak, it speaks loudly.
2. **Italic vs roman**: italic serif signals intimacy (month, greeting). Roman serif signals a "service value" (the amount in the component rows), and the hero amount uses it too, standing out through scale instead (point 1). Sans = tool.
3. **Sienna accent**: the only real "shouting". It appears at most 2 or 3 times per screen: brand, current month, CTA. Never to decorate.

### Rhythm

The month cards are twelve, identical, in a column: the **patient repetition** that gives the grid the character of a register. The same logic applies to `ore-month-grid` (12 squares) and to the seven columns of the calendar. None of the twelve cards "stands out" except through its content: the only visual difference is `.current` (sienna outline), and it sets apart only *one* month in *one* year.

---

## 9. Token reference (summary)

```css
:root {
  /* Paper & surfaces */
  --paper:           #FAF7F2;
  --paper-tint:      #F5F0E6;
  --card:            #FFFEFB;
  --card-elev:       #FFFFFF;
  --line:            #E8DFCF;
  --line-soft:       #F0E8D8;

  /* Text */
  --ink:             #2A2622;
  --ink-soft:        #5C534A;
  --ink-muted:       #978C7E;
  --ink-faint:       #BFB5A4;

  /* Sienna accent */
  --sienna:          #C96442;
  --sienna-deep:     #A4502F;
  --sienna-soft:     rgba(201, 100, 66, .10);
  --sienna-tint:     #FAEFE8;

  /* Companions */
  --sage:            #6B8E5A;
  --sage-soft:       rgba(107, 142, 90, .12);
  --sage-tint:       #ECF1E6;
  --terracotta:      #B85449;
  --terracotta-soft: rgba(184, 84, 73, .10);
  --terracotta-tint: #F8E9E5;
  --gold:            #C29B3D;
  --gold-soft:       rgba(194, 155, 61, .12);
  --gold-tint:       #F6EFD9;
  --ocean:           #4F7CAC;
  --ocean-soft:      rgba(79, 124, 172, .10);
  --ocean-tint:      #E6EDF5;

  /* Type */
  --serif: "Source Serif 4", "Source Serif Pro", Charter, Georgia, "Times New Roman", serif;
  --sans:  Inter, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", system-ui, sans-serif;
  --hand:  Caveat, "Bradley Hand", "Marker Felt", cursive;

  /* Geometry */
  --radius-card:    18px;
  --radius-soft:    12px;
  --radius-pill:    999px;

  /* Shadow */
  --shadow-rest:  0 1px 0 rgba(125,105,75,.04), 0 1px 2px rgba(125,105,75,.06);
  --shadow-soft:  0 6px 18px rgba(125,105,75,.08), 0 1px 2px rgba(125,105,75,.05);
  --shadow-warm:  0 14px 40px rgba(155,110,75,.12), 0 1px 2px rgba(125,105,75,.04);
}
```

---

## 10. Extension rules

When you add a new component or screen:

1. **Start from the tokens, not from magic values.** If you need a new color, that's a signal: you probably need to rename the role, not add a hex.
2. **No new fonts.** The three families are enough. A fourth voice would dilute the existing ones.
3. **No new accents.** If you need something to "stand out", first check that it isn't a hierarchy problem (scale + italic).
4. **Every new component is born with an existing shadow level.** Don't invent ad hoc `box-shadow`s: use `--shadow-rest / soft / warm`.
5. **Never set whole text in uppercase, except service labels.** Uppercase is a "label" decision, not emphasis.
6. **Always `tabular-nums` on any figure.** The alignment of figures is a moral question.
7. **No animation longer than 300ms.** Transitions are short (`.08s` to `.28s`) with gentle curves (`cubic-bezier(.2,.8,.2,1)` or `ease`). The app doesn't "entertain": it records. The one exception is the looping `pulse` (1.2s) of the sync indicator while it syncs.
8. **Never call the native `alert()`/`confirm()`.** Use `showAlert`/`showConfirm`: visual consistency and no surprises on iOS.
9. **`escapeHtml` on any user string** interpolated into a template + `innerHTML`. It's an anti self-XSS convention.
10. **If you add an emoji, it lives only inside `.comp-icon`.** No emoji anywhere else.

---

## 11. When one of these constraints feels like too much

The design system is not dogma. If you have an idea that breaks it and works better, **record it in the CHANGELOG with its *why*** and update this file accordingly. The *Quiet Ledger* philosophy exists to prevent random decisions, not to prevent informed ones.

---

<p align="center"><i>counted by hand</i></p>
