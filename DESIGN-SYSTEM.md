# Design System — Where is my Salary

Documento di riferimento del design system dell'app. È un'estrazione dei token CSS, dei componenti e dei pattern presenti in `index.html`, con annotato il **perché** delle scelte — così le iterazioni future restano coerenti senza dover ricostruire l'intento ogni volta.

L'estetica si rifà a **Quiet Ledger** (vedi [`quiet-ledger-philosophy.md`](./quiet-ledger-philosophy.md)): carta calda come superficie primaria, italico serif come voce, un singolo accento sienna usato come sigillo di ceralacca. La traduzione di quella filosofia in regole pratiche è quello che segue.

---

## 1. Principi guida

Tre regole che vincono in caso di dubbio:

1. **Sottrarre prima di aggiungere.** Ogni icona, divider, ombra in più deve guadagnarsi il posto. Il primo istinto è togliere.
2. **L'accento è uno solo.** Sienna `#C96442` è il rosso ceralacca. Compare nei punti di significato (numero che conta, voce dell'app, CTA primaria, "oggi"). Non si distribuisce — si concentra.
3. **La cifra è poesia.** I numeri vivono in serif italic, tabular-nums, con aria intorno. Quando un numero deve essere letto, è grande e ha respiro; quando è solo un dato di servizio, sta zitto.

---

## 2. Colore

Tutti i token sono dichiarati in `:root` (vedi `index.html:26–66`). Ogni colore è in un ruolo, non in un "ruolo generico" — significa che cambiarne uno cambia un comportamento, non un'apparenza.

### Superfici (carta)

| Token            | Hex        | Uso                                                    |
|------------------|------------|--------------------------------------------------------|
| `--paper`        | `#FAF7F2`  | Sfondo applicazione, theme color iOS/Android           |
| `--paper-tint`   | `#F5F0E6`  | Sfondi secondari (icone componente, weekend nel cal)   |
| `--card`         | `#FFFEFB`  | Cards principali (mese, ore-card, week-row)            |
| `--card-elev`    | `#FFFFFF`  | Cards elevate (hero, sheets, confirm card)             |
| `--line`         | `#E8DFCF`  | Border standard                                        |
| `--line-soft`    | `#F0E8D8`  | Divider interni (riga + riga dentro a una card)        |

### Inchiostro (testo)

| Token          | Hex       | Uso                                                    |
|----------------|-----------|--------------------------------------------------------|
| `--ink`        | `#2A2622` | Testo primario (titoli, nomi voce, valori cifra)       |
| `--ink-soft`   | `#5C534A` | Testo secondario (hero-sub, hint, descrizioni)         |
| `--ink-muted`  | `#978C7E` | Label uppercase, stato neutro, copy di servizio        |
| `--ink-faint`  | `#BFB5A4` | Placeholder, "in arrivo", date weekend in calendario   |

### Accento sienna (l'unico)

| Token             | Hex                          | Uso                                                |
|-------------------|------------------------------|----------------------------------------------------|
| `--sienna`        | `#C96442`                    | "Salary" nel logo, CTA primaria, "oggi", giorno con eventi |
| `--sienna-deep`   | `#A4502F`                    | Hover/active dell'accento                          |
| `--sienna-soft`   | `rgba(201,100,66,.10)`       | Background di chip attiva, "current month" outline |
| `--sienna-tint`   | `#FAEFE8`                    | Hero radial-gradient, sezione attiva               |

### Companions (parchi, mai protagonisti)

| Ruolo              | Token                | Hex       | Uso                                                |
|--------------------|----------------------|-----------|----------------------------------------------------|
| Verde "ok"         | `--sage`             | `#6B8E5A` | Stato "ricevuto", hero-stat ok                     |
| Rosso "manca"      | `--terracotta`       | `#B85449` | Stato "mancante", weekend con eventi nel cal       |
| Oro "in attesa"    | `--gold`             | `#C29B3D` | Stato "warn" (qualcosa è arrivato, qualcosa no)    |
| Blu "info"         | `--ocean`            | `#4F7CAC` | Riservato; non usato attivamente, lasciato per estensioni |

Ognuno ha un `-soft` (alpha 10–12%) e un `-tint` (pastel) per backgrounds.

### Regole sull'uso del colore

- **Mai usare due accenti diversi nella stessa schermata** se non per significare due stati diversi (ricevuto/mancante).
- **Il sienna non si usa per "decorare".** Si usa per indicare *significato* (questa è l'app; questa cifra è la cifra; questo giorno è oggi).
- **Sage/terracotta/gold sono semantici**, mai estetici. Se rimuovi una semantica, rimuovi il colore.

---

## 3. Tipografia

Tre famiglie, ognuna con una voce. Caricate via Google Fonts ma con fallback a system stack (vedi `index.html:14–16` e i tokens in `:root`).

### Famiglie

| Token       | Stack                                       | Voce                                                  |
|-------------|---------------------------------------------|-------------------------------------------------------|
| `--serif`   | Source Serif 4 · Charter · Georgia          | La voce intima. Cifre, titoli mese, totali, hero amount |
| `--sans`    | Inter · -apple-system · SF Pro Text         | Il sussurro. Label, hint, body, bottoni, form         |
| `--hand`    | Caveat · Bradley Hand · Marker Felt         | Il respiro. Saluti ("ciao,"), "oggi", firma           |

### Scala

Non è una scala "modulare" rigida — segue piuttosto la **funzione**. Riferimento alle dimensioni che effettivamente esistono nel CSS:

| Ruolo                       | Family    | Size                       | Weight | Style    |
|-----------------------------|-----------|----------------------------|--------|----------|
| Logo (brand-name)           | serif     | `clamp(24px, 5vw, 30px)`   | 500    | regular  |
| Logo accento (em "Salary")  | serif     | uguale                     | 500    | *italic* |
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
| Salute ("ciao,")            | hand      | 22px                       | 600    | regular  |
| Empty-greet                 | hand      | 22px                       | 600    | regular  |
| "oggi" badge (current month)| hand      | 16px                       | 600    | regular  |
| Signature                   | hand      | 16px                       | 500    | regular  |

### Regole tipografiche

- **`font-variant-numeric: tabular-nums`** ovunque ci siano cifre da allineare verticalmente (hero-amount, comp-amount, hero-stat .v, oms-value, owr/oyr .v, day numbers nel calendario). L'allineamento dei numeri è una questione morale (cit. *Quiet Ledger*).
- **Italic serif per le voci intime** (saluti, sub-titoli, totali, mesi, nomi delle sezioni). Italic non è enfasi: è prossimità.
- **Sans serif quando la funzione è strumentale** (etichette, bottoni, valori in field di form). Non grida mai.
- **Letter-spacing negativo (-.012em ÷ -.025em)** sui titoli grandi serif per non farli sentire "spaziati" — il serif di default ha già la giusta crenatura.
- **Letter-spacing positivo (.04em ÷ .08em) + uppercase** sulle label di servizio (`.k`, `.field label`, `.cal-h`). Sono "etichette di archivio", non vogliono distinguersi.
- **Hand font usato solo in 4 punti** (saluti hero, "oggi", "empty-greet", signature). È il sale: se compare ovunque, perde il sapore.

---

## 4. Spaziatura, raggi, ombre

### Border radius

| Token              | Valore  | Uso                                                    |
|--------------------|---------|--------------------------------------------------------|
| `--radius-card`    | 18px    | Cards (month, hero, ore-card, sheet desktop, summary)  |
| `--radius-soft`    | 12px    | Bottoni, input pillola, day-strip                      |
| `--radius-pill`    | 999px   | Pill (year-picker, status-pill, tri-state, tabbar)     |

Inoltre, valori `9–11px` ricorrono per `icon-btn` e `brand-mark` (storico). Le card mese e l'hero usano i radii grandi (18px) per dare la sensazione di "blocco di carta", non di "scheda software".

### Ombre

Tre livelli, tutti caldi (tinta terra di Siena, mai grigi neutri):

| Token             | Valore                                                                                | Uso                                              |
|-------------------|---------------------------------------------------------------------------------------|--------------------------------------------------|
| `--shadow-rest`   | `0 1px 0 rgba(125,105,75,.04), 0 1px 2px rgba(125,105,75,.06)`                        | Cards a riposo (mese, week-row, day-row, btn-sec)|
| `--shadow-soft`   | `0 6px 18px rgba(125,105,75,.08), 0 1px 2px rgba(125,105,75,.05)`                     | Hover delle card, mese aperto                    |
| `--shadow-warm`   | `0 14px 40px rgba(155,110,75,.12), 0 1px 2px rgba(125,105,75,.04)`                    | Hero, sheet, confirm-card, empty-card            |

Le ombre **non sono nere abbassate** — sono ocra (RGB `125,105,75` e `155,110,75`). Differenza minuscola, percezione enorme: l'oggetto sta su carta, non galleggia sul vuoto.

### Spaziatura

Non c'è una scala di spacing rigida (8, 16, 24…). I valori reali sono pragmatici: `4 / 6 / 8 / 10 / 12 / 14 / 18 / 22 / 26` px ricorrono spesso. Pattern principali:

- **Card padding**: `14px 16px` (compatte) ÷ `26px 24px 22px` (hero).
- **Sheet body padding**: `18px 22px`. Sheet foot: `12px 22px max(16px, env(safe-area-inset-bottom))`.
- **Gap tra card mese**: `12px`. Gap tra ore-card consecutive: `10px`. Gap tra giorno e giorno nel calendario: `4px`.
- **Margin-top di una nuova sezione**: `22px ÷ 30px` (`.section-head` ha `margin: 30px 4px 14px`).

Le **safe area inset** (`env(safe-area-inset-*)`) sono rispettate in body padding e sheet-foot (l'app è installabile come PWA su iOS).

---

## 5. Componenti

I componenti riusati nel codice, descritti come unità con una loro grammatica.

### 5.1 Hero

Card "leggio" in cima a ogni pagina (`Salary` e `Overtime`).

Struttura:
```
.hero
├── .hero-greeting    (saluto handwriting + nome)
├── .hero-amount      (la cifra protagonista, serif italic enorme)
├── .hero-sub         (frase di contesto, serif italic 14.5px)
├── .hero-tagline     (battuta ironica tra virgolette, ink-faint)
└── .hero-stats       (3 KPI in griglia 3 col, flex column con valore in basso)
```

Particolarità:
- Pseudo-element `::before` con radial-gradient sienna-soft in alto a destra (la "macchia di luce").
- Pseudo-element `::after` con underline mosso disegnato come SVG inline (la traccia umana).
- `.hero-amount` ha lo stato `.ok` (sage) e `.danger` (terracotta). Senza modifier è ink.
- Le cents (`.cents`) sono il 55% della size dell'integer — il valore principale si legge, gli spiccioli no.
- `.hero-stat` flex column con `margin-top: auto` sul valore → le tre KPI si allineano sulla stessa baseline anche quando le label vanno a capo.

### 5.2 Card mese (Salary)

```
.month  (.current se è il mese in corso, .open se accordion aperto)
├── .month-head
│   ├── .month-name      (serif italic lowercase)
│   ├── .month-status    (chip semantica: ok / warn / danger / future / "non spettava")
│   └── .month-chevron   (ruota 180° quando aperto)
└── .month-body          (display: none di default; .open .month-body { display: block })
    └── .comp × N        (una riga per componente visibile)
```

`.month.current` ha bordo sienna + outer-ring `sienna-soft` (3px) + chip "oggi" handwriting accanto al nome. È l'unica card che parla.

### 5.3 Componente row

L'unità di base nel mese. Layout grid con aree nominate (vedi `index.html:373–451`):

```
desktop:                          mobile (<480px):
"icon info amount tri"            "icon info  tri"
                                  "icon amount tri"
```

Quattro pezzi:
1. **`.comp-icon`** — quadratino 38×38 con emoji + bordo `--line-soft` su `--paper-tint`. L'emoji è l'unica concessione visiva colorata, dentro un contenitore neutro.
2. **`.comp-info`** — nome (`.comp-name`, sans 14.5px bold) + dettaglio (`.comp-detail`, sans 12.5px muted). Il dettaglio può contenere input inline (vedi 5.4).
3. **`.comp-amount`** — la cifra del mese, serif tabular-nums. Modifier `.dim / .ok / .danger`. Empty → `display: none` (no trattino orfano).
4. **`.tri`** — tri-state pill (vedi 5.5).

### 5.4 Input inline nel componente

Gli input che vivono nella riga (override stipendio, giorni ticket, importo bonus, ore straordinari, importo 13a/14a/15a) hanno una regola specifica:

```css
.comp-detail input{
  background: transparent;
  border: none;
  border-bottom: 1px dashed var(--line);
  field-sizing: content;       /* auto-grow su browser moderni */
  width: 5ch;                  /* fallback */
  min-width: 3ch;
  max-width: 14ch;
  padding: 1px 2px;
  font-variant-numeric: tabular-nums;
}
```

Inoltre la helper JS `inputWidthStyle(value, placeholder, min)` (in `index.html:1834`) inietta `style="width: Nch"` inline su ogni render, in modo che la width "iniziale" sia già giusta prima che il browser applichi `field-sizing`. Risultato: `0,00` e `1000,00` occupano lo spazio che serve loro, niente troncamenti né buchi.

### 5.5 Tri-state pill

Tre bottoni in una pill (ricevuto / mancante / non spettato):

```
.tri
├── button[data-state="received"]      (✓)  → on: background sage, color white
├── button[data-state="missing"]       (!)  → on: background terracotta, color white
└── button[data-state="not_expected"]  (—)  → on: background ink-faint, color white
```

Non c'è un "sì/no boolean" perché la semantica reale è ternaria. La differenza tra "mancante" e "non spettato" è il cuore del modello dati.

### 5.6 Tabbar flottante

iOS-style pill in basso, sempre visibile (`position: fixed`), con due tab: **Salary** e **Overtime**.

- `z-index: 25`. Sopra ci sono: scrim sheet (20+), sheet (21+), confirm-overlay (30).
- Quando un sheet o un confirm è aperto, la tabbar viene nascosta (`body.modal-open .tabbar` / `body.overlay-open .tabbar` `→ display: none`). È stata una richiesta esplicita per non avere doppia "barra galleggiante".
- Il tab attivo ha background `--sienna-soft` e text `--sienna-deep`. Non c'è altro indicatore (niente underline, niente badge).

### 5.7 Sheet (modal a comparsa)

Pattern usato per:
- **Settings** (`Le tue cifre`) — sezioni *Salary*, *Overtime*, *Sistema*.
- **New profile** (`Iniziamo da te`) — stesso shape del settings, senza sezione Sistema.
- **Sync configure** (`Configura sync cloud`).
- **Overtime event** (`Nuovo / Modifica evento`).

Struttura:

```
.scrim (backdrop)
.sheet
├── .sheet-handle    (la pillola di trascinamento)
├── .sheet-head      (titolo + chiusura)
├── .sheet-body      (scroll area — flex: 1 1 auto, min-height: 0)
└── .sheet-foot      (CTA primaria + secondarie, pinned in basso)
```

Comportamento responsive:
- **Mobile** (< 640px): sale dal basso a tutta larghezza, copre fino a `92dvh`.
- **Desktop**: centrato, max-width 560px, popup-style con scale-in.

Le sezioni interne usano `.settings-section-title` (serif italic 16px sienna, dashed underline) come heading.

### 5.8 Calendario (Overtime)

Single grid `repeat(7, minmax(0, 1fr))` con `aspect-ratio: 1` sulle celle giorno. Headers (`.cal-h`) sulla riga 1 con altezza auto. La prima cella numerica usa `grid-column-start` per allinearsi al giorno della settimana corretto (niente celle "empty" — quel pattern rompeva l'altezza della prima riga su alcuni browser).

Stati cella:
- **Default**: numero ink, bordo invisibile.
- **`.weekend`**: paper-tint background, ink-muted text.
- **`.has-events`**: sienna full, white text, bold.
- **`.has-events.weekend`**: terracotta (lavorare nel weekend è già un'altra cosa).
- **`.today`**: outline 2px sienna.

### 5.9 Bottoni

Una sola famiglia, tre varianti:

| Classe              | Background          | Testo  | Quando                                       |
|---------------------|---------------------|--------|----------------------------------------------|
| `.btn`              | `--sienna`          | white  | CTA primaria (Salva, Crea profilo, Aggiungi) |
| `.btn.secondary`    | `--card`            | ink    | Azione neutrale (Scarica, Carica)            |
| `.btn.danger`       | `--terracotta`      | white  | Azione distruttiva (Azzera dati, Elimina)    |

Modifier: `.full` (width 100%), `.large` (padding 14px 22px, font 15px).

Tutti hanno la stessa ombra interna `inset 0 1px 0 rgba(255,255,255,.16)` per simulare la "pressione di un timbro".

### 5.10 Status pill (month-status)

Chip piccola con quattro varianti semantiche:

| Modifier   | Background          | Quando                                     |
|------------|---------------------|--------------------------------------------|
| (none)     | paper-tint          | "non spettava" (mese senza voci attese)    |
| `.ok`      | sage-tint           | "tutto incassato"                          |
| `.warn`    | gold-tint           | "manca X" (parziale)                       |
| `.danger`  | terracotta-tint     | "da ricevere X" (niente ancora arrivato)   |
| `.future`  | transparent + line  | "in arrivo"                                |

Il testo varia ma i colori sono fissi e mappati 1:1 a stato → semantica.

### 5.11 Form

Pattern unico:

```
.field
├── label                  (sans 11.5px uppercase muted)
├── input / select         (paper bg, line border, radius 10px, focus → sienna ring)
└── .hint                  (sans 12.5px italic muted, opzionale)
```

`.field-row` mette due `.field` affiancati in due colonne 1fr/1fr.

`.workday-row` è il selettore lun-dom: una pillola per giorno, sienna soft + sienna bordo quando selezionato.

---

## 6. Iconografia

### Emoji come icone componente

Le voci nel mese usano emoji come icona dentro un quadratino neutro (`.comp-icon`):

| Voce             | Emoji  |
|------------------|--------|
| Stipendio        | 💼     |
| Tredicesima      | 🎄     |
| Quattordicesima  | 🌞     |
| Quindicesima     | ✨     |
| Welfare          | 🌿     |
| Buoni pasto      | 🍱     |
| Fringe benefit   | 🎁     |
| Bonus            | ⭐     |
| Rimborso spese   | 🧾     |
| Straordinari     | 🕐     |

**Regola**: l'emoji vive *solo* dentro `.comp-icon`. Non comparire in label, header, bottoni, tab. La tabbar è stata esplicitamente ripulita (niente `🕐` accanto a "Overtime") per non gridare.

### SVG inline

I controlli (settings gear, chevron giù, year-picker freccia) sono SVG inline 16–18px con `stroke: currentColor` — così ereditano l'ink-soft del bottone contenitore e lo `--sienna` al hover.

### Pseudo-decorazioni

L'underline mosso dell'hero (`hero::after`) e i pip della legend (`.dot.ok / .miss / .na`) sono fatti senza asset esterni — SVG inline come data-URI o piccoli `<span>` colorati. Niente immagini caricate da rete.

---

## 7. Voice & tone

L'app parla in **italiano** ovunque tranne due punti, ambedue intenzionali:

- **Logo "Where is my *Salary*"** — il nome originale, con accento italic sienna sull'ultima parola.
- **Tab "Salary" / "Overtime"** — coerenti col logo, e ironici (cit. *"Overtime is the work you do that the paycheck doesn't see"*).

### Caratteristiche del copy

1. **Frasi brevi, virgolette, pausa.** Le tagline sono tra virgolette tipografiche (`\201C`/`\201D` via pseudo-element) per dare l'idea di "detto a bassa voce".
2. **Ironia sull'attesa.** Tutte le tagline (`TAGLINES_STIPENDIO`, `TAGLINES_ORE`) sono pensate per essere il commento dolceamaro del libro mastro che ha visto troppi 27 del mese:
   - *"Il 27 non è una data. È una promessa."*
   - *"Le ore non si recuperano. I soldi sì."*
3. **Saluti contestuali.** Buongiorno / buon pomeriggio / buonasera / buona notte cambiano in base all'ora — il libro mastro è sveglio quando lo sei tu.
4. **Confirm/alert** sostituiscono i nativi (`showConfirm`, `showAlert`, `showChoice`) — il copy può quindi mantenere lo stesso tono ovunque.

### Glossario interno

| Termine UI           | Significato                                                  |
|----------------------|--------------------------------------------------------------|
| "ricevuto"           | importo arrivato in busta — verde sage                       |
| "mancante"           | dovuto, non ancora arrivato — terracotta                     |
| "non spettato"       | non era previsto per quel mese — ink-faint                   |
| "manca X"            | parzialmente arrivato                                        |
| "da ricevere X"      | tutto ancora da incassare                                    |
| "tutto incassato"    | mese chiuso, conti tornano                                   |
| "in arrivo"          | mese futuro                                                  |
| "non spettava"       | mese passato senza voci attese (vuoto legittimo)             |
| "le tue cifre"       | titolo del sheet impostazioni                                |
| "Azzera dati"        | non "Elimina profilo" — il termine "profilo" non esiste più  |

---

## 8. Hierarchy & rhythm

Tre regole di gerarchia visiva, in ordine di forza:

1. **Scale** — l'hero amount è 11vw, le altre cifre sono 15–18px. La differenza è di ~4×, non di ~30%. Quando una cifra deve parlare, parla forte.
2. **Italic vs roman** — l'italic serif segnala intimità (mese, totale, saluto). Il roman serif segnala "valore di servizio" (importo nelle righe componente). Sans = strumento.
3. **Accento sienna** — l'unico vero "shouting". Compare al massimo 2–3 volte per schermata: brand, current month, CTA. Mai per decorare.

### Ritmo

Le card mese sono dodici, identiche, in colonna — la **ripetizione paziente** che dà alla griglia il carattere di registro. La stessa logica vale per `ore-month-grid` (12 quadrati) e per le sette colonne del calendario. Nessuna delle dodici card "spicca" se non per il contenuto: l'unica differenza visiva è `.current` (sienna outline) — e quella distingue solo *un* mese in *un* anno.

---

## 9. Token reference (sintesi)

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

## 10. Regole di estensione

Quando aggiungi un componente o uno schermo nuovo:

1. **Parte dai token, non da valori magici.** Se serve un colore nuovo, è un segnale: probabilmente devi rinominare il ruolo, non aggiungere un hex.
2. **Niente nuove font.** Le tre famiglie bastano. Una quarta voce dilurebbe le esistenti.
3. **Niente nuovi accenti.** Se hai bisogno di "spiccare", controlla prima che non sia un problema di hierarchy (scale + italic).
4. **Ogni nuovo componente nasce con un'ombra di livello esistente.** Non inventare `box-shadow` ad hoc — usa `--shadow-rest / soft / warm`.
5. **Mai testo intero in uppercase tranne le label di servizio.** L'uppercase è una decisione di "etichetta", non di enfasi.
6. **Sempre `tabular-nums` su qualsiasi cifra.** L'allineamento delle cifre è una questione morale.
7. **Niente animation > 300ms.** Le transizioni sono brevi (`.12–.28s`) con curve gentili (`cubic-bezier(.2,.8,.2,1)` o `ease`). L'app non "intrattiene": registra.
8. **Mai chiamare `alert()`/`confirm()` nativi.** Usa `showAlert`/`showConfirm` — coerenza visiva e niente sorprese su iOS.
9. **`escapeHtml` su qualunque stringa utente** interpolata in template + `innerHTML`. È convenzione anti self-XSS.
10. **Se aggiungi un'emoji, vive solo dentro `.comp-icon`.** Niente emoji altrove.

---

## 11. Quando uno di questi vincoli ti sembra di troppo

Il design system non è un confessionale. Se hai un'idea che lo viola e funziona meglio, **mettila nel CHANGELOG con il `Why:`** e aggiorna questo file di conseguenza. La filosofia *Quiet Ledger* esiste per evitare decisioni casuali, non per impedire decisioni informate.

---

<p align="center"><i>contato a mano</i></p>
