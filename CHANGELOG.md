# Changelog

Storico delle modifiche di **Where is my Salary**. Il file è anche una mini-documentazione delle scelte di design: per ogni release scrivo *cosa* è cambiato e *perché*, così a distanza di mesi non devo ricostruire il contesto a memoria.

Le voci sono in ordine cronologico inverso (più recenti in alto). Le versioni seguono [SemVer](https://semver.org/lang/it/) in modo informale: bump del minor per feature, patch per fix.

---

## [3.3] — 2026-05-14

Release stabile che consolida il ciclo di lavoro su **Overtime** iniziato in 3.1. Niente nuove feature rispetto alla 3.2.4: solo un bump di versione che chiude la sequenza di 5 iterazioni (`3.2 → 3.2.4`) e segnala la fine del rework.

### Riepilogo del ciclo 3.1 → 3.3

| Versione  | Intervento principale                                                |
|-----------|----------------------------------------------------------------------|
| 3.1       | Hero overtime con metrica doppia (euro + ore), default `missing` per overtime, KPI annuo "Straord. {anno}" |
| 3.2       | Ore promosse a `.hero-amount` (stessa scala dell'euro), migration retro-attiva `v3_1_overtime_default` |
| 3.2.1     | `.hero-amounts` flex inline                                          |
| 3.2.2     | `justify-content: space-between` (poi rifiutato)                     |
| 3.2.3     | `justify-content: center` (poi rifiutato)                            |
| 3.2.4     | Grid asimmetrico `1fr auto 1fr` con ore a sinistra e euro al centro  |
| **3.3**   | **Consolidamento. Stessa codebase di 3.2.4, bump di versione.**      |

### Audit pre-release

- `node --check` su JS estratto: **PASS**.
- Grep di riferimenti morti (`empty-mark`, `salaryMultiplierFor`, `profile-pill`, `cal-grid`, etc.): **0 hit** in codice attivo (solo 1 commento esplicativo).
- Migration `v3_1_overtime_default` invariata dalla 3.2 — già provata sul profilo dell'utente con esito ok.

### Internals

- Versione bumped a `3.3`.

---

## [3.2.4] — 2026-05-14

### Overtime — layout asimmetrico ore/euro

- **`.hero-amounts` passa da flex a grid 3-colonne (`1fr auto 1fr`).** Le due metriche non sono più *sibling* in un layout simmetrico — ora vivono in posizioni semanticamente distinte: le ore (`.secondary`) stanno in col 1 left-aligned, l'importo in euro (`.primary`) sta in col 2 centrato. Il `1fr` di col 3 funziona da mirror invisibile che mantiene la primary visivamente al centro dell'hero anche quando la secondary è presente.
- Quando non c'è tariffa (no euro da mostrare), la primary contiene direttamente le ore — il layout grid resta valido perché la primary in col 2 con due 1fr ai lati è già centrata.
- Stesso pattern Quiet Ledger: la cifra "che conta" (al centro) ha la posizione di rilievo, l'altra le sta accanto come margine annotato.

### Internals

- Versione bumped a `3.2.4`.

---

## [3.2.3] — 2026-05-14

### Overtime — coppia centrata con aria

- **`.hero-amounts` passa da `justify-content: space-between` a `center` con `gap: 8px 48px`.** Le due cifre stanno al centro del box, separate da un gap generoso ma fisso. Non più ancorate ai bordi del hero — l'effetto "estremi" risultava troppo squilibrato visivamente. Centrate con respiro in mezzo si leggono come una coppia bilanciata.

### Internals

- Versione bumped a `3.2.3`.

---

## [3.2.2] — 2026-05-14

### Overtime — euro a sinistra, ore a destra

- **`.hero-amounts` ottiene `justify-content: space-between`.** Le due cifre ora vivono agli estremi del box hero (importo in euro a sinistra, ore a destra), l'aria in mezzo dà respiro al ritmo della pagina e riprende il pattern *Quiet Ledger* "margini come silenzi scelti". Gap orizzontale ridotto da 22px a 16px perché tanto è `space-between` a determinare lo spazio reale.

### Internals

- Versione bumped a `3.2.2`.

---

## [3.2.1] — 2026-05-14

### Overtime — euro + ore sulla stessa riga

- **Le due cifre del hero passano da stacked a side-by-side.** In 3.2 erano state messe una sopra l'altra (regola `.hero-amount + .hero-amount { margin-top: 4px }`) ma la lettura risultava verticale, quasi come "principale + appendice". Adesso le due metriche stanno sulla **stessa riga** dentro un wrapper `.hero-amounts` (flex con `align-items: baseline` e `gap: 8px 22px`).
- `flex-wrap: wrap` mantiene il fallback per viewport molto stretti (sotto ~340px): se le due cifre non entrano su una riga sola, vanno a capo automaticamente con un gap minore. Su qualsiasi schermo > 340px stanno una accanto all'altra come voleva il design.
- Il margin-top del primo `.hero-amount` viene azzerato dentro `.hero-amounts` (è il wrapper a tenere il margin-top di 14px verso il greeting sopra).

### Internals

- Versione bumped a `3.2.1`.

---

## [3.2] — 2026-05-14

Correzioni alle scelte fatte in 3.1: la gerarchia visiva nel hero Overtime non comunicava la pariteticità delle due metriche, e la nuova default su `overtime` non si applicava ai profili già esistenti rendendola di fatto invisibile.

### Overtime — ore e euro entrambe protagoniste

- **Le ore non pagate sono ora un secondo `.hero-amount` a piena scala**, non un sub-text. In 3.1 erano renderizzate in `.hero-amount-aux` (serif italic 15px) — leggibili ma chiaramente "didascalia". Adesso entrambi i numeri usano la stessa classe `.hero-amount`, stessa size `clamp(40px, 11vw, 56px)`, stesso peso, stessa logica `intero + ,decimali` con `.cents` ridotto. Comunica visivamente che "soldi dovuti" e "ore non pagate" hanno la stessa importanza.
- Helper `buildHoursHtml(h)` per generare lo stesso mark-up `123<span class="cents">,5 h</span>` usato sia quando le ore stanno in primo piano (no tariffa) sia quando stanno sotto l'importo in euro.
- Regola CSS `.hero-amount + .hero-amount { margin-top: 4px }` tiene i due numeri stretti come una coppia, non come headline+nota.
- Rimossa la vecchia classe `.hero-amount-aux` (non più referenziata).

### Migration retro-attiva — overtime default

- **`runMigrations()` introdotta come step di bootstrap subito dopo `loadStore()`.** In 3.1 il nuovo default `expectedDefault:true` su `overtime` valeva solo per i mesi *mai creati*, ma `ensureYear` crea già tutti i 12 mesi dell'anno appena visiti la pagina — quindi un utente v3.0 che aveva già aperto l'app si trovava 12 mesi con `state: "not_expected"` salvati nel localStorage, immuni al cambio di default. La nuova feature era invisibile.
- **Migration `v3_1_overtime_default`** scorre tutti i mesi di tutti i profili e converte a `"missing"` *solo* le celle overtime che soddisfano due condizioni: `state === "not_expected"` AND `hoursOverride == null`. La seconda condizione tutela le scelte attive dell'utente: se aveva impostato manualmente un override (magari proprio per controllare quel mese), il "not_expected" è una scelta consapevole e resta.
- Il flag `store._migrations.v3_1_overtime_default = true` viene salvato la prima volta che la migration gira — al reload successivo non viene riapplicata. Niente effetti collaterali, niente loop infiniti.
- Pattern estensibile: per migrazioni future basta aggiungere un nuovo `if(!store._migrations.<nome>)` dentro `runMigrations()`.

### Internals

- Versione bumped a `3.2`.

---

## [3.1] — 2026-05-14

Tre interventi sulla pagina Overtime e sulla voce straordinari della pagina Salary, motivati dall'uso reale: la metrica principale era unilaterale, il default rendeva difficile vedere i conti, e mancava un totale annuale di straordinari.

### Overtime — metrica doppia nel box principale

- **Sotto l'importo in euro, riga con le ore non pagate.** Quando è impostata una tariffa, l'hero mostrava solo `€ 1.200,00`. Adesso accanto mostra anche `123,5 ore di straordinario non pagate`. La doppia metrica risponde a due domande diverse con una sola occhiata: *quanto* mi devono (euro) e *cosa* è stato lavorato senza compenso (ore). Nuovo stile `.hero-amount-aux` — serif italic 15px, ink-soft, tabular-nums.
- Quando non c'è tariffa il box resta com'era (solo ore in primo piano).
- Quando non c'è nulla da pagare, la riga aux scompare automaticamente.

### Salary — default straordinari "missing"

- **`expectedDefault` su `overtime` passa da `false` a `true`**. Prima i nuovi mesi nascevano con stato `non spettato` per gli straordinari: l'utente doveva ogni volta passare a "mancante" manualmente affinché i conti della pagina Overtime considerassero quelle ore come da pagare. Adesso il default è *non ancora pagato* — coerente col fatto che, in questa app, lo straordinario è qualcosa che presumi di non aver ancora incassato finché non lo segni come ricevuto.
- Migration `ensureYear` aggiornata di conseguenza per coerenza con la nuova default.
- I mesi v3.0 esistenti restano col loro stato (l'utente potrebbe averli messi consapevolmente a "non spettato"). La nuova default vale solo per i mesi mai aperti.

### Overtime — KPI annuo dedicato

- **Terzo `.hero-stat` cambiato da "ore lavorate anno" (gross) a "Straord. {anno}" (paid).** Il valore precedente sommava *tutte* le ore loggate nell'anno — informazione di servizio ma non aiuta a rispondere alla domanda chiave dell'app ("quanti straordinari ho fatto?"). Ora il KPI somma le ore *oltre soglia* mese per mese: il numero reale di ore di straordinario annuale, indipendentemente da quante siano state ricevute.

### Internals

- Nuovo calcolo locale `thisYearOvertimeHours` in `renderPageOre` — itera `paidHoursForMonth` sui 12 mesi dell'anno corrente. Costo trascurabile, render già ricalcola tutto a ogni interazione.
- Versione bumped a `3.1`.

---

## [3.0.1] — 2026-05-14

Patch tipografica: una tagline dell'hero "Salary" conteneva un valore hard-coded che non rispecchiava più le impostazioni dell'utente.

### Copy

- **Tagline "Hai diritto di sognare. E di un ticket da {ticket}." resa dinamica.** Prima leggeva `"… ticket da 8 euro"` indipendentemente dal valore effettivo nelle impostazioni — battuta che perdeva il punto se l'utente aveva un ticket diverso. Adesso il valore arriva da `settings.ticketPerDay` del profilo attivo. La frase nell'array contiene `{ticket}` come placeholder, risolto a render-time.

### Internals

- **Nuova helper `ticketPhrase(n)`** — formatta il numero per stare bene dentro una frase: `10` → `"10 euro"`, `8.50` → `"8,50 euro"`. Non si usa `eur()` perché `"€ 10,00"` rompe il ritmo della battuta.
- **Nuova helper `interpolateTagline(text)`** — applica le sostituzioni di placeholder leggendo dal profilo attivo. Wrappa sia `pickStipTagline()` che `pickOreTagline()`, così future tagline su entrambe le pagine possono usare `{ticket}` (e altri placeholder che aggiungeremo) senza toccare la logica del picker.
- Versione bumped a `3.0.1`.

---

## [3.0] — 2026-05-14

Release stabile che chiude il ciclo di rework partito dalla 2.0. Niente nuove feature rispetto alla 2.5: solo un fix critico, una pulizia di codice morto e il consolidamento del design system come documento ufficiale. Il bump a 3.0 segnala che le sette iterazioni 2.x sono ormai integrate, l'UI è stata ridisegnata end-to-end (tab in inglese, sezioni settings, mensilità configurabili, calendario rifatto, input dinamici, logo rinforzato) e l'app è considerata pronta per l'uso quotidiano.

### Fix critico

- **Doppia dichiarazione `const EXTRA_SALARY_DEFAULT_MONTHS`** introdotta inavvertitamente in 2.5 durante un riordino del codice. Causava `SyntaxError: Identifier 'EXTRA_SALARY_DEFAULT_MONTHS' has already been declared` a parse-time, impedendo il caricamento dell'intero script. Risolto rimuovendo l'occorrenza duplicata. La costante vive ora solo accanto a `COMPONENTS` (dichiarazione semanticamente vicina a chi la usa). Node `--check` sul JS estratto: PASS.

### Pulizia

- **Rimossa la CSS rule morta `.empty-mark`** (~10 righe). Lo span con il "€" nel cerchio era stato eliminato dall'HTML in 2.2 ma la regola era rimasta orfana.
- **Copy stale aggiornato**. Le frasi "mese per mese" rimaste in due punti (heroSub di Overtime quando non ci sono eventi, copy della empty card) erano residui pre-2.1 — l'app non ha più quel framing. Sostituite con varianti più asciutte.

### Documentazione

- **`DESIGN-SYSTEM.md` introdotto come documento di riferimento.** Estrazione completa dei token, dei componenti e dei pattern presenti nel codice, con motivazioni e collegamenti alla filosofia *Quiet Ledger*. Include una sezione "Regole di estensione" con 10 vincoli pratici per evitare drift visivo nelle iterazioni future.

### Internals

- Versione bumped a `3.0`.

### Garanzia di compatibilità

- I dati v2.x in `localStorage` continuano a caricare senza migrazione esplicita (la chain `ensureProfile` → `ensureYear` → `defaultMonth` backfills tutti i campi nuovi con i default ragionevoli).
- I CSV v2.x importano grazie al parser tollerante (header *prefisso*).
- Il sync cloud non è toccato — gist esistenti restano leggibili.

---

## [2.5] — 2026-05-14

Rifinitura tipografica: input nelle righe componente dimensionati al contenuto e logo header rinforzato.

### Salary — input inline

- **Width dinamica.** Gli input numerici nelle righe del mese (stipendio, 13a/14a/15a, bonus, rimborsi, giorni ticket, ore straordinario) avevano `width: 56px` fisso. Risultato: `1000,00` veniva tagliato a `1000,0(`, mentre `0,00` lasciava uno spazio enorme prima del simbolo `€`. Ora ogni input riceve `style="width: Nch"` calcolato su `max(len(value), len(placeholder), min)` via la nuova helper `inputWidthStyle(value, placeholder, min)`. Risultato: il campo si stringe attorno al numero e il `€` (o `giorni`, o `h × …`) gli sta sempre accanto senza buchi né troncamenti.
- **`field-sizing: content`** dichiarato nel CSS per i browser moderni (Chrome 123+, Safari 17+, Firefox 122+): l'input si auto-espande mentre l'utente digita, senza bisogno di re-render. Fallback `min-width: 3ch / max-width: 14ch` per browser più vecchi. Padding orizzontale ridotto da `4px` a `2px` per recuperare spazio.

### Header

- **Logo "Where is my *Salary*" più grande e presente.** Dimensione passata da 19px a `clamp(24px, 5vw, 30px)`. Letter-spacing stretto e weight invariato; resta il pattern serif italic con accento sienna su *Salary*. Padding verticale del topbar leggermente aumentato (`14/12 → 20/16`) per dare aria al titolo ora che il topbar ospita meno elementi (niente più brand-mark, niente più profile pill, niente più "+ aggiungi profilo").

### Internals

- Versione bumped a `2.5`.

---

## [2.4] — 2026-05-14

Completamento della feature mensilità: l'importo della 13a/14a/15a è ora editabile mese per mese, come già lo era per la salary base.

### Salary — override mensilità extra

- **Input editable accanto a Tredicesima / Quattordicesima / Quindicesima.** Stesso pattern dello stipendio mensile: campo numerico inline, valore vuoto = null sentinel ("eredita dalla salary base del mese"), valore numerico = override esplicito. Il placeholder mostra l'importo che verrebbe usato di default (override salary mensile → fallback `settings.salary`).
- `defaultMonth()` ora inizializza `amount: null` per tutte le voci salary-like (salary + extraSalary).
- `ensureYear` esegue migration retro-compatibile per mesi v2.3 che hanno solo `state` senza `amount`.
- `expectedAmountFor` per salary13/14/15 ha una chain di fallback chiara: `cell.amount` (override per-mese) → `data.salary.amount` (override salary base) → `settings.salary`.
- L'handler `.amount-input` riconosce `salary13/14/15` come salary-like e mantiene il null sentinel su input vuoto.

### Backup CSV

- **Tre colonne aggiuntive in `MESI`**: `salary13_importo`, `salary14_importo`, `salary15_importo`, in coda a tutte le altre. I file v2.3 (senza queste colonne) si caricano comunque grazie al parser tollerante (header *prefisso*) — gli override mancanti diventano `null`.
- L'euristica `allDefault` che evita di esportare mesi tutti-default ora considera anche `cell.amount != null` per le voci extraSalary, così un mese con il solo override di una mensilità extra viene comunque persistito.

### Internals

- Versione bumped a `2.4`.

---

## [2.3] — 2026-05-14

Due interventi: fix di un bug visivo del calendario e nuovo modello dati per le mensilità extra.

### Overtime — calendario

- **Spaziatura prima riga corretta.** Il rendering precedente generava `N` celle "empty" (`<div class="cal-c empty">`) per allineare il primo giorno del mese al giorno della settimana giusto. Su alcuni browser, le celle vuote con `aspect-ratio: 1` collassavano ad altezza 0 / dimensione 0 e rompevano la prima riga del grid, lasciando uno spazio bianco enorme tra il primo giorno e i successivi. Ora la prima cella numerica usa `style="grid-column-start: N"` e non vengono più generate celle empty — soluzione più pulita e robusta al bug aspect-ratio. Rimossa anche la regola CSS `.cal-c.empty` e il selettore `:not(.empty)` nei click handler.

### Salary — mensilità extra

- **13a / 14a / 15a sono voci separate con tri-state proprio.** Prima la logica era: stipendio × `salaryMultiplierFor(mese)`. Significava che a dicembre con 14 mensilità l'importo "Stipendio" mostrava il doppio, ma non c'era modo di marcare *solo* la 13a come mancante quando la mensile era già arrivata. Adesso:
  - Aggiunti tre componenti in `COMPONENTS`: `salary13` (🎄, paid in dicembre), `salary14` (🌞, giugno), `salary15` (✨, luglio).
  - Compaiono come riga separata *solo* nel mese di pagamento e *solo* se `settings.mensilita` raggiunge la soglia. Il filtro è in `extraSalaryApplies(comp, settings, m0)`, applicato sia in `renderMonthCard` sia indirettamente in `expectedAmountFor` (ritorna 0 quando non applies, così totali e status restano corretti).
  - Importo = uguale allo stipendio mensile (incluso eventuale override per-mese sulla salary base).
  - Rimossa `salaryMultiplierFor()` — non serve più.
  - `defaultMonth()` crea le tre celle con stato di default "missing" (cioè *attese* nel mese giusto). `ensureYear` esegue migration retro-compatibile.

### Backup CSV

- **Tre colonne nuove in `MESI`**: `salary13_stato`, `salary14_stato`, `salary15_stato`. Parser tollerante (header *prefisso*) → file v2.2 senza queste colonne si caricano con stato "missing" di default.

### Internals

- Versione bumped a `2.3`.

---

## [2.2] — 2026-05-14

Iterazione di rifinitura UX sulla v2.1: pulizia empty state, allineamento del form "Iniziamo da te" con il sheet impostazioni completo, riorganizzazione delle impostazioni in sezioni e rinaming dei tab in inglese per coerenza con il nome dell'app.

### Generale

- **Tab rinominati: "Salary" e "Overtime"**. Il primo richiama direttamente *Where is my Salary*; il secondo è scelto al posto di "Work time"/"Ore" perché evoca emozionalmente l'idea centrale dell'app — il lavoro extra che si fa senza che la busta paga lo veda. Coerente con il tono ironico-noir delle tagline.
- **Empty state ripulito.** Rimosso il cerchio "€" sopra al saluto. La card ora è centrata verticalmente (flex column con `justify-content: center` e `min-height: 360px`), per dare più aria al copy e meno orpelli grafici.

### Impostazioni

- **Tre sezioni esplicite con heading: "Salary" / "Overtime" / "Sistema"**. Implementate con un nuovo `.settings-section-title` (serif italic + linea tratteggiata sotto, accento sienna). Rende immediato capire dove si trova un determinato campo.
- **Giorni lavorativi spostati prima di Ore contratto / Forfait / Tariffa**. La definizione dei giorni "che contano" è precondizione per i calcoli, quindi appare per prima nella sezione Overtime.

### New profile sheet

- **Modulo allineato al settings sheet completo.** Lo sheet "Iniziamo da te" ora chiede *tutti* i campi che prima erano accessibili solo dopo la creazione del profilo: mensilità annue, mese del fringe, bonus standard, ore contratto, forfait, tariffa straordinario e giorni lavorativi. Lo schema rispecchia esattamente quello delle impostazioni (con le stesse due sezioni Salary / Overtime), così l'utente alla prima apertura imposta tutto in un colpo solo senza dover poi ri-aprire le impostazioni.
- **Tutte le validazioni `clamp()` ora vengono applicate anche al primo salvataggio** (prima alcuni campi finivano nel profilo senza il bound).

### Internals

- `openNewProfile()` ora popola il select `np-fringe-month` e resetta i workday checkbox.
- `#btn-create-profile` handler costruisce settings con tutti i nuovi campi e applica `clamp(v, max)` su ogni numerico.
- Versione bumped a `2.2`.

---

## [2.1] — 2026-05-13

Release di pulizia visiva, semplificazione UX e correzioni sui calcoli ore. Nessuna migrazione dati distruttiva: i profili esistenti continuano a funzionare, i campi nuovi vengono aggiunti automaticamente con default ragionevoli.

### Generale

- **Tabbar `Stipendio · Ore` nascosta sopra a sheet e modal.** Quando si apre il sheet impostazioni o qualsiasi finestra in sovraimpressione, il pill flottante in fondo schermo non resta più visibile sopra al backdrop. Implementato via classi `body.modal-open` (già esistente per i sheet) + `body.overlay-open` (nuova, aggiunta dentro `showConfirm` / `showChoice`).
- **Rimossa emoji 🕐 dal tab "Ore".** Il box flottante ora è solo testo, coerente con il tono "carta" dell'app.
- **Header semplificato.** Eliminato il quadratino "€" come logo e la riga "mese per mese · v2.0". Resta solo il titolo *Where is my Salary*. La versione è stata spostata in fondo alla pagina (`.app-version`).
- **Rimosso il multi-profilo dall'UI.** La pillola di switch profilo e il pulsante "+ aggiungi profilo" non vengono più renderizzati. Lo store internamente rimane multi-profilo (per non rompere i dati esistenti) ma l'app usa sempre il primo profilo trovato. Il pulsante "Elimina profilo" nelle impostazioni è stato rinominato in **"Azzera dati"** con copy aggiornato.

### Stipendio

- **Mensilità annue configurabili (12–15).** Nuovo select nelle impostazioni. Default 12. Le mensilità extra arrivano in mesi fissi: **13a a dicembre, 14a a giugno, 15a a luglio**. La logica vive in `salaryMultiplierFor(settings, m0)` e in `EXTRA_SALARY_DEFAULT_MONTHS`. Per chi ha 14 mensilità, l'app ora calcola automaticamente lo stipendio doppio a giugno e dicembre (non serve toccare l'override manuale).
- **Layout componenti più respirabile.** La riga voce/dettaglio/importo/tri-state è stata convertita da flexbox a CSS Grid con `grid-template-areas`. Su mobile (<480px) l'importo scende sotto la voce in modo da non comprimere il nome a metà.
- **Niente più "—" tra voce e stato quando una voce non spetta.** Se lo stato è `not_expected`, la cella importo viene lasciata vuota (`.comp-amount:empty { display:none }`). Risultato: una riga "Bonus" non atteso non mostra più il trattino orfano accanto ai pulsanti tri-stato.

### Impostazioni

- **Nuovo campo "Ore contratto (h/settimana)".** Default 40. Insieme al "Forfait straord." definisce la soglia oltre la quale le ore lavorate vengono pagate come straordinari. Esempio: contratto 40h + forfait 5h = 45h coperte dallo stipendio, il resto è straordinario.
- **Semantica del forfait rivista.** Prima il forfait era l'unica soglia (tutte le ore loggate sopra il forfait erano straordinari). Adesso la formula è `paid = max(0, workH - (contratto + forfait)) + weekendH`. Il forfait default è ora 0 (chi aveva il vecchio 5 può continuare a usarlo; le ore di contratto vanno valorizzate a parte).
- **Tariffa straordinario in campo dedicato.** Spostata fuori dalla riga forfait/contratto per dare più spazio agli hint.

### Ore

- **Box riassuntivo: valori allineati sulla stessa riga.** Le tre celle "settimana / mese / anno" hanno labels di lunghezza diversa (es. "Settimana corrente" va a capo, "2026 totale" no). Ora `.hero-stat` è flex column con `margin-top: auto` sul valore, così tutti e tre i numeri stanno alla stessa baseline anche quando le label sono a due righe.
- **Check ore straordinarie corretto.** Tutti i call site di `paidHoursForMonth` (6 punti) passano ora il parametro `contractWeek` oltre al `forfaitWeek`. Anche `renderOreWeekView` usa la nuova soglia combinata.
- **Spaziatura calendario uniforme.** Headers e celle stavano in due grid separati (`.cal-header` + `.cal-grid`): la prima riga di numeri appariva più staccata delle altre. Adesso entrambi vivono in un solo `.cal-table` con gap costante 4px.

### Backup CSV

- **Schema esteso (compatibile all'indietro).** Aggiunte due colonne nella sezione `SETTINGS`: `contratto_settimana` e `mensilita`. Il parser accetta anche header *prefisso* (file vecchi senza le nuove colonne caricano correttamente con default `40h` / `12 mensilità`). I nuovi export includono tutte le colonne.

### Internals

- `defaultSettings()` ora restituisce `mensilita: 12`, `oreContrattoSett: 40`, `forfaitOreSett: 0`.
- `ensureProfile()` esegue la migrazione difensiva dei tre nuovi campi su profili pre-esistenti.
- Versione bumped a `2.1`.

### Note di compatibilità

- Se hai un profilo dalla v2.0 con `forfaitOreSett: 5` (vecchio default), il calcolo straordinari potrebbe risultare diverso al primo caricamento. Soluzione consigliata: vai in **Impostazioni**, imposta "Ore contratto" a 40 (o quello che è il tuo contratto) e "Forfait" a 0 (o 5 se davvero hai un forfait CCNL oltre al contratto). Salva.
- Il pulsante "Azzera dati" cancella tutto lo store. Se hai più profili (eredità v2.0) verranno tutti rimossi.

---

## [2.0] — Pre-2026-05-13

Prima versione tracciata in questo CHANGELOG. Funzionalità storiche (senza dettaglio cronologico):

- App single-file HTML/CSS/JS vanilla, nessuna build, nessun bundler.
- Tracking mensile di 7 voci: stipendio, welfare, fringe, buoni pasto, bonus, rimborso, straordinari.
- Calcolo automatico giorni lavorativi italiani (festività nazionali + Pasquetta via algoritmo di Gauss/Meeus).
- Sync opzionale via GitHub Gist (PAT con scope `gist`).
- Backup CSV con separatore `;` e decimale `,` (convenzione italiana, leggibile in Excel/Numbers).
- Sezione "Ore" con viste giorni/settimane/mesi/anni, calendario e calcolo forfait straordinario.
- Multi-profilo (rimosso dall'UI in v2.1).

---

## Come scrivere voci future

Una voce di changelog è utile *se contiene* il **cosa** + il **perché**. Ogni release deve permettere a qualcuno (anche a me fra 6 mesi) di rispondere a:

1. **Cosa è cambiato?** — descrizione concreta, riferimenti a funzioni/file (`paidHoursForMonth`, `index.html:1727`).
2. **Perché è cambiato?** — il problema che si risolveva, o la richiesta utente.
3. **Cosa devo sapere se aggiorno?** — eventuali breaking changes, migrazioni dati, default modificati.

Sezioni standard, in ordine: Generale, Stipendio, Ore, Impostazioni, Backup, Internals, Note di compatibilità. Lasciare fuori le sezioni vuote.

Non documentare ogni edit di stile: raggruppare le piccole modifiche di pulizia in una voce "Polish" se servono.
