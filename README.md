<h1 align="center">Where is my <em>Salary</em></h1>

<p align="center">
  <i>Mese per mese, contato a mano.</i>
</p>

<p align="center">
  Una piccola app per tenere traccia dello stipendio italiano e degli straordinari non pagati.<br/>
  Niente account, niente cloud (opzionale), niente dipendenze. Un solo file <code>.html</code> che funziona offline, installabile come PWA su iPhone e Android.
</p>

---

## Cosa fa

Sei mai arrivato al giorno di paga chiedendoti se ti sono arrivati davvero **tutti** i pezzi della busta paga, e quante ore di straordinario stai accumulando senza che la busta le veda? Questa app risponde a entrambe le domande, calcolando ciò che ti spettava, ciò che ti è effettivamente arrivato e quanto lavoro extra hai fatto senza compenso.

L'app ha due pagine principali, accessibili dalla tabbar in fondo allo schermo:

### Pagina **Salary** — la busta paga mese per mese

- **Quanto ti devono ancora** — un'unica cifra in cima alla pagina, ricalcolata sui mesi il cui giorno di pagamento è già passato.
- **Voci di paga tracciate** — stipendio, welfare, fringe benefit, buoni pasto, bonus, rimborso spese, più 13ª/14ª/15ª come voci separate nei mesi giusti (configurabili da 12 a 15 mensilità annue).
- **Stato per mese** — ✓ ricevuto · ! mancante · — non spettato. Più una colonna "Straordinari" che rimanda alla pagina Overtime.
- **Giorno di pagamento configurabile** — il tuo `payDay` (1–28) determina quando l'app considera "atteso" lo stipendio del mese. Toggle per chi viene pagato il mese successivo (es. maggio incassato il 15 giugno).
- **Override per-mese di ogni importo** — se uno stipendio mensile, una 13ª o un bonus arriva diverso dal default, lo metti a mano sulla singola cella.
- **Buoni pasto in automatico** — calcola i giorni lavorativi italiani escludendo weekend e festività nazionali (Pasquetta inclusa, calcolata correttamente per ogni anno). Sovrascrivibile a mano per ferie, malattia, smart working diverso. Giorni della settimana "che contano" configurabili (default lun-ven).

### Pagina **Overtime** — le ore extra non pagate

- **Calendario per logging giornaliero** — apri il giorno, segni le ore lavorate (e quelle di weekend, contate sempre come straordinario).
- **Soglia automatica** — ore contratto (default 40h/sett.) + eventuale forfait CCNL. Tutto quello che eccede diventa straordinario non pagato.
- **Hero box con metrica doppia** — importo in euro (se hai impostato una tariffa) accanto al totale ore non pagate. Mostra entrambe le risposte con un'occhiata.
- **KPI annuo dedicato** — ore di straordinario complessive dell'anno corrente.
- **Disattivabile** — se non ti interessa tracciare le ore, dalle impostazioni spegni "Overtime" e la pagina sparisce, tabbar inclusa.

### Generale

- **Installabile come PWA** — aggiungila alla home screen di iPhone/Android e si aggiorna automaticamente al prossimo lancio con rete (service worker network-first).
- **Backup CSV** — esporta o importa per spostare i dati tra dispositivi (formato `;`-separato leggibile in Excel/Numbers italiani).
- **Sync cloud opzionale** — un gist GitHub privato come deposito comune tra Mac, iPhone, ufficio. Niente nuovi servizi, niente account aggiuntivi.

---

## Setup iniziale (30 secondi)

1. Apri `index.html` con doppio click — si apre nel browser.
2. Al primo avvio l'app apre da sola il modulo **"Iniziamo da te"**: scrivi il tuo nome, lo stipendio mensile, le mensilità annue (12–15), il giorno di pagamento, e i parametri Overtime (ore contratto, eventuale forfait, tariffa straordinario, giorni lavorativi settimanali). Welfare, fringe e ticket hanno già default sensati (400 / 1000 / 10€).
3. Per ogni mese passato, apri la card della pagina **Salary** e imposta lo stato di ogni voce. Per le ore extra, vai sulla pagina **Overtime** e logga i giorni.
4. Per modificare i parametri in qualunque momento: icona ⚙️ in alto a destra → Impostazioni. Per azzerare tutto e ripartire: "Azzera dati" in fondo alle impostazioni.

---

## Come avviarla / accedervi dal telefono

Hai tre opzioni, dalla più semplice alla più "pro". Per **iPhone/Android** in tutti i casi puoi poi aggiungere l'icona alla home screen: l'app è una PWA con service worker, quindi una volta installata si aggiorna da sola al prossimo lancio con rete (network-first sull'HTML).

### 1. File locale via iCloud / Google Drive (zero lavoro)

1. Salva `index.html`, `sw.js` e `manifest.webmanifest` su iCloud Drive (o Google Drive, OneDrive…) — devono stare insieme nella stessa cartella perché il service worker abbia scope corretto.
2. Sul telefono apri `index.html` dall'app File — si apre nel browser.
3. Aggiungilo alla schermata Home: in Safari → **Condividi → Aggiungi a Home**. Diventerà un'icona come una vera app.

> ⚠️ I dati vivono nel browser di ogni dispositivo separatamente. Per averli identici su Mac e iPhone, usa la **Sync cloud** (sotto) oppure esporta il CSV da uno e importalo sull'altro.

### 2. Netlify Drop (5 minuti, zero account)

1. Vai su [app.netlify.com/drop](https://app.netlify.com/drop).
2. Trascina la cartella contenente `index.html` + `sw.js` + `manifest.webmanifest` nella zona bianca.
3. Netlify ti dà un URL del tipo `https://nome-casuale.netlify.app/`.
4. Apri l'URL dal telefono → "Aggiungi a Home". Fatto.

Per cambiare il sottodominio o aggiornare il file in futuro, registrati gratis su Netlify (accesso con Google/GitHub).

### 3. GitHub Pages (1 minuto se hai già una repo)

1. Crea una repo **pubblica** (Pages free non funziona con repo private).
2. Carica `index.html`, `sw.js` e `manifest.webmanifest` nella root.
3. **Settings** → **Pages** → branch: `main`, folder: `/ (root)` → Save.
4. Dopo qualche minuto: `https://<username>.github.io/<repo>/`.

> ⚠️ Se hai installato l'app sulla home screen prima dell'introduzione del service worker (versioni < 3.4.1) potresti vedere ancora una build vecchia. Rimedio una tantum: rimuovi l'icona dalla home → Impostazioni iOS → Safari → Cancella dati siti web → riapri il sito → Aggiungi a Home. Da quel momento gli update sono automatici.

---

## Privacy & sicurezza

- **Tutti i dati restano nel browser** (`localStorage`). Nessun server, nessun cloud (a meno che tu non attivi la sync), nessuna telemetria.
- Se pubblichi l'app online (Netlify o Pages), l'URL è tecnicamente pubblico ma **i tuoi dati no**: ognuno che apre l'URL parte da zero perché lo storage è per-browser. È la stessa logica per cui un blocco note in tasca è privato anche se la fabbrica del blocco note ha mille copie identiche.
- **Sync cloud (opzionale)**: il Personal Access Token GitHub viene salvato in chiaro nel `localStorage` del dispositivo. È una scelta consapevole — l'app è single-file senza backend e non può fare di meglio. Se il browser viene compromesso (estensione malevola, sessione condivisa), **revoca il token su GitHub** (Settings → Developer settings → Personal access tokens) e la sync è disattivata su tutti i dispositivi. Lo scope del token è limitato a `gist` — non può fare nient'altro sul tuo account.

---

## Come si calcola "quanto ti devono"

### Salary

Per ogni voce e ogni mese:

| Stato | Conteggio |
|---|---|
| ✓ ricevuto | conta come incassato, e come atteso |
| ! mancante | conta come atteso; conta come "da ricevere" solo se il **giorno di pagamento del mese è già passato** |
| — non spettato | non conta |

La discriminante "passato vs in arrivo" non è più la fine del mese: dipende dal tuo `payDay` configurato. Se sei pagato il 27 dello stesso mese, maggio diventa "dovuto" a partire dal 27 maggio. Se sei pagato il 15 del mese successivo, maggio diventa "dovuto" il 15 giugno. La logica vive nelle funzioni `paymentDateFor` e `isPaymentDue`.

L'importo atteso è ricavato dalle impostazioni del profilo (stipendio, welfare, fringe, ticket × giorni lavorativi, mensilità extra) o dall'importo che inserisci direttamente (bonus, rimborsi). Per le mensilità extra (13ª/14ª/15ª) la riga compare solo nel mese di pagamento e solo se le mensilità annue configurate raggiungono la soglia.

I giorni lavorativi sono calcolati escludendo i giorni della settimana che hai marcato come non lavorativi (default sabato e domenica) e queste **festività nazionali**: Capodanno, Epifania, Pasquetta, Liberazione, Festa del Lavoro, Festa della Repubblica, Ferragosto, Ognissanti, Immacolata, Natale, Santo Stefano. La data di Pasqua si ricava con l'algoritmo di Gauss/Meeus.

> ⚠️ Le festività dei santi patroni locali (Sant'Ambrogio a Milano, San Petronio a Bologna, ecc.) **non** sono incluse: l'app conta solo le festività nazionali. Se ne hai bisogno, sovrascrivi il numero di giorni a mano per quel mese.

### Overtime

Per ogni giorno loggato nel calendario:

- Le **ore weekend** (giorni che hai marcato come non lavorativi) contano per intero come straordinario.
- Le **ore feriali** contano come straordinario solo per la parte che eccede la soglia settimanale: `paid = max(0, oreFeriali - (oreContratto + forfait))`.

L'importo in euro è `ore × tariffa straordinario`. Se la tariffa è 0, l'hero mostra solo le ore.

---

## Tech stack

> Tre file in totale, niente build, niente CDN obbligatorie.

- `index.html` — HTML + CSS + JS vanilla, single-file (markup, stili, logica e icone inline come data-URI).
- `sw.js` — service worker per gli update automatici della versione installata su home screen (network-first sull'HTML, cache-first sugli asset).
- `manifest.webmanifest` — manifest PWA con `display: standalone`, theme color e icone SVG inline.
- `localStorage` per la persistenza locale dei dati utente.
- GitHub Gist API per la sync cloud opzionale (gist privato, scope `gist`).
- Google Fonts opzionali (Source Serif 4, Inter, Caveat) — caricate via CDN ma con fallback a system serif/sans, quindi l'app è leggibile anche senza connessione.
- Calcolo Pasqua e festività italiane fatto in-app (vedi `easterSunday`, `italianHolidays`).

Non ci sono framework, bundler, package manager. I file si modificano con un editor di testo qualunque.

---

## Come è organizzato il codice

Tutta la logica vive in `index.html` (markup, stili, script in unico file). I file accessori sono `sw.js` (service worker) e `manifest.webmanifest`. Per una mappatura puntuale dei blocchi nello script, l'approccio migliore è grep diretto sui nomi delle funzioni — i numeri di riga sono volatili da release a release. Sezioni logiche, in ordine di apparizione:

| Blocco | Cosa fa |
|---|---|
| Costanti (`VERSION`, `STORAGE_KEY`, `SYNC_KEY`, `MONTHS_IT`, `COMPONENTS`, `EXTRA_SALARY_DEFAULT_MONTHS`, `TAGLINES_*`) | Definizioni base. `COMPONENTS` è l'elenco delle voci di paga con flag (`expectedDefault`, `hasAmount`, `hasDays`) — modifica qui per aggiungere/rimuovere voci. |
| `easterSunday`, `italianHolidays`, `workingDaysInMonth` | Calendario italiano, calcolo giorni lavorativi (rispetta i `workdays` configurati nelle settings). |
| `paymentDateFor`, `isPaymentDue` | Determinano quando lo stipendio del mese N è "dovuto", in base a `payDay` e `payDayNextMonth`. |
| `loadStore`, `saveStore`, `defaultSettings`, `defaultMonth`, `ensureProfile`, `ensureYear`, `activeProfile`, `runMigrations` | Persistenza e shape del dato in `localStorage`. Lo store ha forma `{ profiles: { id: { name, settings, years: { 2026: { 1: {...} } } } }, activeProfile, currentYear, _migrations }`. `runMigrations` gira a bootstrap e applica trasformazioni idempotenti taggate in `_migrations`. |
| `expectedAmountFor`, `computeYearTotals`, `monthStatus`, `extraSalaryApplies` | Logica di calcolo "atteso vs ricevuto" per Salary, comprese 13ª/14ª/15ª. |
| `paidHoursForMonth` e correlati | Calcolo ore straordinario su base settimanale, con soglia `oreContratto + forfait`. |
| `render`, `renderTopbar`, `renderMain`, `renderMonthCard`, `renderPageOre`, `renderOvertimeVisibility`, `switchTab` | Rendering delle due pagine. **L'app rifà l'intero `#main` con `innerHTML` ad ogni interazione** — semplice, funziona bene su questa scala, ma occhio se aggiungi input con focus persistente. |
| `openSheet`, `closeSheets`, `openNewProfile` + handler `$("#btn-...")` | UI dei sheet (modali a fondo schermo) e bottoni. |
| Backup CSV (`buildCsv`, `parseCsv`, `parseCsvLine`, `fmtNum`, `parseNum`, `csvEscape`, `applyImportedData`) | Export/import. Formato sezionato (`# SETTINGS` + `# MESI`), separatore `;`, decimale `,`, UTF-8 con BOM. Stati mappati da inglese a italiano via `STATE_TO_IT` / `STATE_FROM_IT`. Parser tollerante (header *prefisso*) per retro-compatibilità tra versioni. |
| `toast`, `escapeHtml`, `inputWidthStyle`, `interpolateTagline`, `ticketPhrase` | Utility. **`escapeHtml` va usata su qualunque stringa utente interpolata in template string + `innerHTML`** — convenzione importante per evitare self-XSS. |
| `showConfirm` / `showAlert` / `showChoice` | Modali custom che rimpiazzano `confirm()` / `alert()` nativi (alcuni browser/estensioni li mangiano, sui telefoni bloccano l'UI). Promise-based. |
| Cloud sync (`syncState`, `ghFetch`, `cloudConfigure`, `cloudPull`, `cloudPush`, `schedulePush`, `cloudDisconnect`, `refreshSyncSection`) | Sync via gist GitHub. `schedulePush` è debounced (~1.8s) per non sparare una PATCH per ogni click. `cloudPull` confronta `_lastModified` locale vs remoto per decidere chi vince. |
| `bootstrap()` + service worker registration | Setup all'avvio: aggiorna version tag, apre il mese corrente, avvia eventuale pull cloud, apre form nuovo profilo se vuoto; subito dopo registra `sw.js`. |

### Storage keys

| Chiave | Contenuto |
|---|---|
| `stipendio.v1` | Lo store principale (profili, mesi, settings, flag migrazioni). Versionata nel nome per facilitare migrazioni future. |
| `stipendio.sync.v1` | Stato della sync cloud (PAT, gistId, user, lastSyncAt, lastError). |

### Convenzioni difensive

- **Sempre `escapeHtml(s)`** per nomi profilo, user GitHub, messaggi d'errore interpolati in HTML.
- **Sempre `Math.min(max, Math.max(0, Number(v) || 0))`** per input numerici provenienti dall'utente (vedi `clamp` nelle settings).
- **`saveStore()` aggiorna `_lastModified`** e fa partire la sync debounced — non bypassare scrivendo direttamente su `localStorage` se non in casi tipo `cloudPull` (dove serve evitare il ciclo).
- **Mai chiamare `alert/confirm` nativi** — usa i wrapper `showAlert/showConfirm/showChoice`.
- **Le migrazioni dello store vivono in `runMigrations()` e sono taggate in `store._migrations.<nome>`** — pattern idempotente: ogni migration controlla il flag prima di girare. Per aggiungerne una nuova: nuovo `if(!store._migrations.<nome>)` dentro `runMigrations`, poi setta il flag a `true`.
- **Cache key del service worker bumpato a ogni release** (`wims-vX.Y.Z` in `sw.js`) — l'`activate` event pulisce le cache con chiave diversa, così non restano asset stantii.

---

## Personalizzazione

Tutto è configurabile dall'icona ⚙️ in alto. Le impostazioni sono organizzate in tre sezioni: **Salary**, **Overtime**, **Sistema**.

### Salary

- Stipendio mensile (max 100.000 €)
- Mensilità annue (12–15) — abilita 13ª/14ª/15ª come voci separate nei mesi `EXTRA_SALARY_DEFAULT_MONTHS` (Dicembre / Giugno / Luglio)
- Giorno di pagamento (1–28, default 27) — clampato a 28 per evitare di "saltare" febbraio
- Pagamento mese successivo (toggle) — per ciclo sfalsato tipo "lavoro maggio, incasso il 15 giugno"
- Welfare mensile (default 400 €, max 100.000)
- Fringe benefit annuale (default 1000 €, max 100.000)
- Mese in cui arriva il fringe (default Dicembre)
- Ticket per giorno lavorativo (default 10 €, max 1.000)
- Bonus standard (per quando segni un bonus come atteso senza specificarne l'importo)

### Overtime

- Toggle Overtime ON/OFF — quando OFF, la pagina e la tabbar floating spariscono e gli straordinari non contribuiscono ai totali
- Ore contratto settimanali (default 40h)
- Forfait CCNL settimanale (default 0h) — soglia aggiuntiva oltre il contratto entro la quale le ore extra non sono considerate straordinario
- Tariffa straordinario oraria (default 0 €/h, max 1.000)
- Giorni lavorativi della settimana (default lun-ven) — definisce quali giorni rientrano nel calcolo settimanale e quali contano sempre come straordinario

### Sistema

- Backup CSV (export/import)
- Sync cloud GitHub Gist (configurazione PAT + gistId)
- Azzera dati (reset completo)

Se i valori cambiano (es. aumento di stipendio), modificarli ricalcola automaticamente anche gli importi attesi nei mesi già marcati. Le override per-mese restano sempre prevalenti sui default.

---

## Backup & portabilità

Dalle impostazioni → **Backup CSV** puoi:

- **Scarica file** — esporta un CSV con tutti i profili e tutti gli anni. Convenzione italiana: separatore `;`, decimale `,`, UTF-8 con BOM (Excel/Numbers italiani lo aprono in colonne pulite).
- **Carica file** — importa un CSV precedentemente esportato. Sovrascrive interamente i dati attuali (chiede conferma). Validazione strict: qualsiasi riga malformata → l'intero import viene rifiutato, niente stati a metà.

Formato del file (due sezioni separate da `# SETTINGS` e `# MESI`):

```
# SETTINGS
profilo;stipendio;welfare;fringe;mese_fringe;ticket_giorno;bonus_default;contratto_settimana;mensilita;pay_giorno;pay_mese_dopo;overtime_abilitato
Massimo;2800,00;400,00;1000,00;12;10,00;0,00;40;14;27;0;1

# MESI
profilo;anno;mese;stipendio_stato;stipendio_importo;welfare;ticket_stato;ticket_giorni;fringe;bonus_stato;bonus_importo;rimborso_stato;rimborso_importo;salary13_stato;salary14_stato;salary15_stato;salary13_importo;salary14_importo;salary15_importo
Massimo;2026;1;ricevuto;3000,00;ricevuto;ricevuto;22;non_atteso;non_atteso;0,00;non_atteso;0,00;non_atteso;non_atteso;non_atteso;;;
```

Stati ammessi: `ricevuto`, `mancante`, `non_atteso`. Mesi con tutti i valori a default vengono omessi dall'export per ridurre rumore.

Le colonne `*_importo` sono **override per-mese** opzionali: se vuote, l'app usa il default dalle settings; se valorizzate, vincono sull'importo di default (utile per mesi con cifra diversa dal solito).

Il parser è **tollerante**: header *prefisso* è accettato, quindi un CSV esportato da una versione precedente (senza alcune colonne nuove) si importa senza errori, con i valori mancanti che cadono sui default correnti. Per i booleani sono accettati sia `0/1` che `sì/no` / `true/false`.

### Export ore lavorate (separato)

Al click di "Scarica file" l'app ti chiede se vuoi il CSV **Stipendio (completo)** o **Solo ore lavorate**. Il secondo è un export evento-per-evento del calendario Overtime, schema:

```
profilo;data;inizio;fine;pausa;nota
Massimo;2026-05-13;09:00;19:30;45;riunione clienti
```

Granularità giornaliera, separato dal CSV principale per non gonfiare l'export Stipendio nei casi d'uso "solo busta paga".

Conservare un export ogni tanto è una buona idea: serve sia come backup, sia per portare i dati su un altro telefono o computer.

Se usi la **sync cloud**, il backup CSV ti serve solo come misura di sicurezza extra (la sync cloud continua a usare JSON nel gist, indipendente dal CSV).

---

## Roadmap

Idee in attesa di valutazione, non promesse:

- Notifica al giorno di pagamento per "voci ancora mancanti"
- Esportazione PDF del riepilogo annuale
- Modalità "previsione" per simulare aumenti, cambi contratto o rinegoziazione tariffa straordinario
- Festività comunali opzionali per il calcolo buoni pasto
- TFR maturato (visualizzazione)

---

## Filosofia di design

L'aspetto dell'app segue una piccola filosofia visiva chiamata **Quiet Ledger** (file `quiet-ledger-philosophy.md` nella repo): carta calda come superficie primaria, italico serif come voce, un singolo accento sienna usato come sigillo di ceralacca. Riferimento sotterraneo a Luca Pacioli — *Summa de Arithmetica*, Venezia 1494, il trattato che cinque secoli fa codificò la partita doppia, antenata silenziosa di ogni libro mastro personale.

---

## Licenza

MIT. Fai quello che ti pare, ma una citazione fa sempre piacere.

---

<p align="center">
  <i>contato a mano, mese per mese</i>
</p>
