# Changelog

Storico delle modifiche di **Where is my Salary**. Il file è anche una mini-documentazione delle scelte di design: per ogni release scrivo *cosa* è cambiato e *perché*, così a distanza di mesi non devo ricostruire il contesto a memoria.

Le voci sono in ordine cronologico inverso (più recenti in alto). Le versioni seguono [SemVer](https://semver.org/lang/it/) in modo informale: bump del minor per feature, patch per fix.

---

## [3.4.3] — 2026-05-14

Due interventi UX legati all'onboarding e al controllo fine-grained sugli straordinari.

### Generale — tabbar nascosta nel welcome

- **`renderOvertimeVisibility` ora nasconde la tabbar anche quando non c'è un profilo attivo**, non solo quando l'utente ha disattivato Overtime. Prima la condizione era `!p || p.settings.overtimeEnabled !== false`: senza profilo il booleano cadeva sul ramo "abilitato" e la tabbar `Salary · Overtime` restava visibile sopra alla card di benvenuto e al modulo "Iniziamo da te", offrendo uno switch tra due pagine che non hanno senso prima della creazione del profilo. Adesso `!!p && settings.overtimeEnabled !== false` → tabbar mostrata solo se *entrambe* le condizioni sono vere.
- Effetto secondario voluto: la prima impressione dell'app è più pulita — solo titolo, modulo, settings.

### Overtime — flag per-evento "non straordinario"

- **Nuovo campo `notOvertime: boolean` sugli eventi.** Quando true, l'evento è completamente escluso dal calcolo straordinari (`paidHoursForMonth` salta l'iterazione con un `continue` prima di accumulare nel bucket settimanale o weekend), ma resta presente nel calendario e contribuisce ai totali "ore lorde" del mese/anno. È la risposta a "ho lavorato quel giorno, ma non voglio segnarlo come straordinario" — sia per sforamenti feriali oltre soglia che per ore weekend volontariamente regalate.
- **Toggle iOS-style nel form di edit evento** (`#ot-event-sheet`), sotto il campo Nota. Stile coerente con il toggle di disattivazione Overtime nelle settings, riusando `.settings-toggle` con un modificatore `.field-toggle` per renderlo full-width come una riga di campo. Hint sotto: "Le ore restano registrate come lavorate, ma non confluiscono nel totale straordinari del mese".
- **Feedback visivo nel day detail**: la card dell'evento marcato come `notOvertime` ha opacità ridotta (0.62), background `--card-elev` e un piccolo badge testuale `non strord.` (serif italic, bordo tratteggiato) accanto al totale ore. La dimensione e la posizione delle ore restano invariate, così la lettura della durata non cambia.
- **Comportamento del calcolo** preservato per gli altri eventi: la soglia settimanale `oreContratto + forfait` continua a operare normalmente sulle ore feriali NON marcate; gli eventi `notOvertime` non incrementano il bucket settimanale, quindi non "consumano" parte della soglia per altri giorni della stessa settimana. Esempio: 4 giorni feriali da 10h + 1 giorno da 10h marcato `notOvertime` → calcolo limitato a 4 giorni × 10h = 40h ≤ soglia 45h → 0h pagate (corretto: l'utente ha esplicitamente sottratto le 10h dal computo).

### Form evento — input date/time integrati nel design system

- **Selettori CSS `.field input[type="text"|"number"]` estesi a `type="date"` e `type="time"`.** Le tre righe del form evento (Data, Ora inizio/fine, Pausa) cadevano sui default del browser — bordo grigio sottile, font system, padding inconsistente, icona del picker nera — perché la regola di styling non li intercettava. Adesso ricevono lo stesso trattamento Quiet Ledger degli altri campi: paper bg, bordo `--line`, padding 11/13, focus state sienna con box-shadow soft.
- **`font-variant-numeric: tabular-nums`** aggiunto su `number`/`date`/`time` per allineamento verticale pulito quando due campi tempo stanno fianco a fianco (Ora inizio / Ora fine nel `.field-row`).
- **`::-webkit-calendar-picker-indicator` ammorbidito** con `opacity .45` + `filter sepia(.5) hue-rotate(-12deg) saturate(.85)`, così l'icona calendario/orologio non legge come widget di sistema nero ma come piccolo accento warm-paper. Hover/focus la portano a opacity .85.
- **`:invalid` su date/time** abbassato a colore `--ink-muted`: i placeholder nativi (`gg/mm/aaaa`, `--:--`) prima erano più scuri del resto degli hint, sembrava un bug. Adesso l'empty state sembra intenzionale.
- Firefox non ha pseudo-elementi equivalenti per l'icona del picker ma il rendering nativo della dropdown è già discreto, quindi non serve override.

### Decisione di design — `hoursOverride` resta

- L'override mensile `overtime.hoursOverride` è ortogonale a `notOvertime`: il primo dice "ignora il calcolo automatico e usa questo totale per il mese", il secondo dice "non contare questo specifico giorno". I due meccanismi coesistono per ora. Rimozione di `hoursOverride` non in agenda finché la nuova feature non è stata usata abbastanza per giudicare se l'override mensile è ancora utile o ridondante.

### Backup CSV — copertura nuovo campo

- **Colonna `non_straordinario` (0/1) aggiunta in coda alle sezioni `STRAORDINARI` (CSV completo) e nel CSV-only-ore (`buildCsvOre`).** Posizionata come ultima colonna così CSV v3.4.2 sono ancora parsabili come *prefisso* dell'header atteso.
- **`parseCsvOre` rilassato**: richiede solo le prime 6 colonne (`profilo..nota`) come prefisso obbligatorio, accetta header più corti o più lunghi (forward-compat). La validazione del numero di campi per riga adesso usa `header.length` invece di `ORE_COLS.length`, così ogni file viene parsato in base al suo proprio header.
- Parser tollerante anche sui valori: accetta `0/1`, `sì/no`, `true/false`. Default: `false` se la colonna manca o è vuota.

### Audit pre-release

- jsdom run di v3.4.3 a confronto con v3.4.2: welcome card renderizzata correttamente, version tag "v3.4.3", zero errori.
- `paidHoursForMonth` verificato manualmente con eventi misti (feriali oltre soglia + weekend + uno marcato `notOvertime`): il giorno marcato salta entrambi i bucket, le altre ore feriali della stessa settimana mantengono il loro contributo normale alla soglia.
- Verifica edge case CSV: import di un file v3.4.2 (senza colonna `non_straordinario`) → tutti gli eventi entrano con `notOvertime: false`. Re-export → la colonna ora c'è, popolata a `0`.

### Internals

- Versione bumped a `3.4.3`.
- Cache key del service worker bumped a `wims-v3.4.3` per invalidare la 3.4.2 al primo activate.

---

## [3.4.2] — 2026-05-14

Fix critico di un bug ereditato dalla v3.1 e rimasto nascosto fino a oggi. La pagina si caricava completamente vuota (solo topbar + tabbar, niente hero, niente welcome card, niente version tag) su qualunque dispositivo — desktop, Safari mobile, Chrome mobile, finestre private — perché un'eccezione module-level interrompeva l'esecuzione dello script *prima* che `bootstrap()` venisse chiamato.

### Diagnosi

- `runMigrations()` (introdotta in 3.1) viene eseguita al module-load, subito dopo `let store = loadStore();`.
- Anche su device fresh senza profili, la migration setta `store._migrations.v3_1_overtime_default = true` e chiama `saveStore()` per persistere il flag.
- `saveStore()` contiene una chiamata difensiva `if(typeof schedulePush === "function") schedulePush();` per innescare la sync cloud debounced quando configurata.
- `schedulePush()` è una **function declaration**, quindi hoisted → il `typeof` check passa anche se la dichiarazione testuale è 2.000 righe più sotto → la funzione viene effettivamente chiamata.
- All'interno, `schedulePush` referenzia `syncState`, dichiarato con `let` molto più giù (vicino a tutto il blocco sync cloud).
- `let` ha **Temporal Dead Zone**: leggere il binding prima della dichiarazione lessicale lancia `ReferenceError: Cannot access 'syncState' before initialization`.
- L'eccezione esce dalla call chain, interrompe l'esecuzione module-level, e `bootstrap()` (alla fine del file) non viene mai raggiunto. Risultato: solo i markup statici (topbar, tabbar) sono visibili. Il version-tag, la welcome card e tutto il render dipendono da `bootstrap()` → rimangono vuoti.

Il bug era deterministico, manifestava su qualunque ambiente, ma è rimasto invisibile per giorni perché la maggior parte degli utenti aveva localStorage popolato da versioni precedenti (le copie cacheate v2.0/v3.0.1 dell'HTML pre-fix non avevano il problema, quindi continuavano a girare) — solo le installazioni "fresh" dopo la 3.1 lo manifestavano, e ironicamente è emerso quando un service worker (3.4.1) ha iniziato a servire l'HTML aggiornato a chi prima vedeva una copia stantia.

### Fix

- **Forward-declaration di `syncState` al module-level**, subito dopo `let store = loadStore();`, con valore iniziale `null`. La dichiarazione vera (l'IIFE che legge da localStorage) resta dove era nel blocco sync cloud, ma adesso è una **riassegnazione** invece di una nuova dichiarazione `let`.
- Niente più TDZ: quando `schedulePush()` legge `syncState` durante l'esecuzione di `runMigrations() → saveStore()`, il binding esiste già con valore `null`, il guard `if(!syncState || ...)` ritorna early, nessuna eccezione.

### Audit pre-release

- jsdom run di v3.0.1, v3.3, v3.4.2 a confronto: solo le versioni con il fix (v3.0.1 e v3.4.2) renderizzano la welcome card. v3.3 cruda lancia `ReferenceError`. Conferma diretta della diagnosi.
- `bootstrap()` ora viene eseguito normalmente su device fresh → version tag impostato, `setSyncStatus("off")` chiamato, welcome card aperta dopo 250ms come previsto.

### Internals

- Versione bumped a `3.4.2`.
- Cache key del service worker bumped a `wims-v3.4.2` per invalidare la cache 3.4.1 al primo activate.

### Lezione

Hoisting + TDZ creano una trappola insidiosa: una function declaration può chiamare codice che dipende da `let`/`const` non ancora inizializzati. Il pattern `if(typeof X === "function") X()` ha funzionato come check di esistenza per la function declaration ma **non** ha protetto da TDZ all'interno. Regola da ricordare: una funzione che usa `let`/`const` di un'altra sezione del file dovrebbe essere chiamata solo *dopo* che entrambe sono inizializzate, oppure tutte le `let` referenziate vanno forward-declared al top del modulo.

---

## [3.4.1] — 2026-05-14

Bugfix release: la versione installata via "Aggiungi a Home" su iOS restava ferma a una build vecchia (v2.0) anche quando Safari su browser mostrava già la 3.4. Il problema era strutturale — l'app non aveva mai avuto un service worker né un `manifest.webmanifest` — quindi la WebView standalone di iOS cacheava aggressivamente l'HTML e non rinfrescava mai.

### Fix — Service worker

- **Nuovo file `sw.js` con strategia network-first sulle navigazioni HTML.** Quando il dispositivo è online, ogni apertura dell'app prende sempre l'ultima `index.html` dal network (`cache: 'no-store'`) e ne aggiorna la copia locale; quando è offline, ricade sulla cache. Per gli asset statici (font Google, icone inline) la strategia è cache-first per non sprecare banda.
- L'`activate` event cancella tutte le cache con chiave diversa dalla corrente (`wims-v3.4.1`), così ogni bump versione invalida la vecchia cache.
- `skipWaiting()` + `clients.claim()` per attivare il nuovo SW immediatamente al primo reload utile, senza aspettare la chiusura di tutte le tab.

### Fix — Manifest PWA

- **Nuovo `manifest.webmanifest`** con `name`, `short_name`, `start_url`, `scope`, `display: standalone`, `theme_color` e `background_color` allineati al palette Quiet Ledger (`#FAF7F2` paper). Icone inline come data-URI SVG (sia `any` che `maskable`), così il file resta autocontenuto come il resto del progetto (no asset binari nel repo).
- Link `<link rel="manifest">` aggiunto in head dopo i meta `apple-mobile-web-app-*`. iOS continuerà a usare `apple-touch-icon` per l'icona home screen (il manifest è ignorato a quello scopo); il manifest serve principalmente a Chrome/Android per l'installazione corretta.

### Fix — Version tag hardcoded

- **Rimosso il fallback `v2.0`** nel tag `#version-tag` (riga 1781): se per qualunque motivo il bootstrap JS non gira prima del primo paint, il numero che vedeva l'utente era 2.0 — fuorviante durante il debug del bug stesso. Adesso il tag è vuoto fino a quando `bootstrap()` non lo riempie con la `VERSION` corrente.

### Note utenti già stuck su versioni vecchie

- Gli utenti che hanno l'app già aggiunta alla home screen con una build pre-SW devono fare **un'unica volta** la rimozione e re-installazione: l'HTML cacheato dalla WebView non contiene la registrazione SW, quindi il nuovo service worker non può essere installato dall'interno della copia stuck. Operazioni: rimuovere l'icona dalla home → aprire Safari sul sito → Impostazioni Safari → Cancella dati per il dominio → ricaricare → Aggiungi a Home.
- Da questo punto in poi tutti gli update arrivano automaticamente al prossimo lancio (con rete disponibile). Niente più "vedo la 3.4 sul browser ma la 2.0 in app".

### Audit pre-release

- `sw.js` validato con `node --check`: PASS.
- `manifest.webmanifest` validato come JSON con `node -e "JSON.parse(require('fs').readFileSync('manifest.webmanifest'))"`: PASS.
- Scope SW = `./` (default) → corretto sotto `https://massimodascola.github.io/WhereIsMySalary-/`: il SW intercetta solo le richieste interne al subpath del progetto, non altri repo sotto il dominio `github.io`.

### Internals

- Versione bumped a `3.4.1`.
- Cache key bumped a `wims-v3.4.1` per invalidare automaticamente eventuali asset cacheati al primo install del SW.

---

## [3.4] — 2026-05-14

Cinque modifiche che ribilanciano la pagina Salary attorno al **ciclo di pagamento reale** dell'utente (non più assunto come "fine mese"), introducono il toggle per disattivare Overtime, abbassano il tono di voce del logo e raddoppiano le tagline ironiche.

### Salary — giorno di pagamento configurabile

- **Due nuovi setting**: `payDay` (1–28, default 27) e `payDayNextMonth` (boolean, default false). Insieme definiscono quando "atterra" lo stipendio del mese N: il `payDay` del mese stesso, oppure il `payDay` del mese successivo (per le presenze sfalsate — es. maggio pagato il 15 giugno).
- **Nuove funzioni `paymentDateFor(year, m0, settings)` e `isPaymentDue(year, m0, settings, now)`** sostituiscono la vecchia logica "passato vs corrente vs futuro" basata su fine mese. Tutta la pagina Salary ora rispetta il ciclo configurato: `computeYearTotals` somma in "mancante" solo i mesi il cui pay date è passato; `monthStatus` etichetta come "in arrivo" tutto ciò che non è ancora dovuto, anche se siamo nel mese corrente ma prima del pay date.
- `payDay` clampato a 28 per evitare di "saltare" febbraio (data sempre valida in ogni mese).
- Migration retro-compatibile via `ensureProfile`: profili pre-3.4 ricevono `payDay: 27, payDayNextMonth: false` (= comportamento storico).
- UI nelle settings e nel new-profile sheet: due campi nel `.field-row` "Giorno di pagamento" + select "Stesso mese / Mese successivo".

### Salary — toggle Overtime

- **Nuovo setting `overtimeEnabled` (default true).** Quando false:
  - La tabbar flottante in fondo schermo viene nascosta (niente più switch fra le due pagine).
  - La voce "Straordinari" nei mesi della pagina Salary scompare dalla lista componenti.
  - `expectedAmountFor("overtime")` ritorna 0 indipendentemente dallo stato, quindi gli straordinari non contribuiscono né ai totali annuali né al month-status.
  - Se l'utente era sulla pagina Overtime quando ha disabilitato, viene reindirizzato a Salary.
- Toggle iOS-style (`.settings-toggle` + `.toggle-track`) accanto al titolo della sezione "Overtime" nelle impostazioni. Visivamente discreto, conferma immediata dello stato.
- `renderOvertimeVisibility()` introdotta come step del render principale; `switchTab()` rifiuta i passaggi a Overtime quando è disabilitato (difesa in profondità).

### Overtime — hero da sinistra

- **Layout cambia da grid asimmetrico a flex con `justify-content: flex-start`** (gap `6px 28px`). L'importo in euro a sinistra, le ore subito a destra, entrambe ancorate al bordo sinistro del hero. Il pattern "centro asimmetrico" della 3.2.4 non comunicava la priorità di lettura — adesso si legge in modo lineare da sx a dx.
- Rimosse le classi `.primary` / `.secondary` (il layout non ne ha più bisogno).

### Logo — tutto minuscolo

- **"Where is my *Salary*" → "where is my *salary*"** (iniziali minuscole). Riduce ulteriormente il tono "prodotto" e si avvicina al gesto manoscritto della filosofia *Quiet Ledger*.

### Tagline — `{payDay}` + raddoppio

- **Placeholder `{payDay}` aggiunto** in `interpolateTagline`. Le frasi storiche `"Il 27 non è una data. È una promessa."` e `"I conti si fanno alla fine. Anche se l'app li fa al 27."` ora usano `{payDay}` e si adattano al giorno scelto dall'utente.
- **Entrambe le liste raddoppiate**: `TAGLINES_ORE` 15 → 30, `TAGLINES_STIPENDIO` 15 → 30. Le nuove frasi continuano il tono ironico-emotivo dell'app (`"Stai costruendo il futuro. Ti pagheranno nel passato."`, `"Il salario è la prima bugia che ti raccontano, l'ultima che credi."`).

### Backup CSV — copertura nuovi setting

- **Tre colonne aggiuntive in `SETTINGS`**: `pay_giorno`, `pay_mese_dopo` (0/1), `overtime_abilitato` (0/1). Il fix è stato trovato in audit pre-release: senza queste colonne, un round-trip export → re-import ripristinava i default v3.3 (27, stesso mese, abilitato), perdendo silenziosamente le scelte dell'utente.
- Parser tollerante (header *prefisso*) → file v3.3 senza queste colonne si caricano con i default come prima. I valori booleani accettano sia `0/1` che `sì/no` / `true/false` per robustezza.

### Audit pre-release

- `node --check` su JS estratto: **PASS**.
- Grep di riferimenti morti (`empty-mark`, `salaryMultiplierFor`, `profile-pill`, `cal-grid`, `hero-amount-aux`, `.hero-amount.primary/.secondary`): **0 hit** in codice attivo (solo 1 commento esplicativo).
- Verificato `paymentDateFor` su edge case mese 12 + offset 1: JS `new Date(year, 12, payDay)` rolla correttamente a gennaio dell'anno successivo. Il calcolo `due` resta coerente attraverso il confine d'anno.
- Verificato CSV round-trip su un profilo con `payDay=15, payDayNextMonth=true, overtimeEnabled=false`: import preserva i tre valori.

### Internals

- Versione bumped a `3.4`.

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
