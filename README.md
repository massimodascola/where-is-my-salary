<h1 align="center">Where is my <em>Salary</em></h1>

<p align="center">
  <i>Mese per mese, contato a mano.</i>
</p>

<p align="center">
  Una piccola app per tenere traccia dello stipendio italiano: salario, welfare, fringe, buoni pasto, bonus e rimborsi.<br/>
  Niente account, niente cloud, niente dipendenze. Un solo file <code>.html</code> che funziona offline e gira anche dal telefono.
</p>

---

## Cosa fa

Sei mai uscito di casa il 27 del mese chiedendoti se ti sono arrivati davvero **tutti** i pezzi della busta paga? Questa app risponde a quella domanda, calcolando ciò che ti spettava e ciò che ti è effettivamente arrivato.

- **Quanto ti devono ancora** — un'unica cifra in cima alla pagina, sempre aggiornata sui mesi già passati.
- **Sei voci di paga** — stipendio, welfare, fringe benefit, buoni pasto, bonus, rimborso spese.
- **Stato per mese** ✓ ricevuto · ! mancante · — non spettato.
- **Buoni pasto in automatico** — calcola i giorni lavorativi italiani escludendo weekend e festività nazionali (Pasquetta inclusa, calcolata correttamente per ogni anno). Sovrascrivibile a mano per ferie, malattia, smart working diverso.
- **Fringe nel mese giusto** — di default Dicembre, configurabile.
- **Bonus & rimborsi a importo libero** — inseriti mese per mese.
- **Multi-profilo** — Tu, il/la collega, chiunque altro. Ogni profilo ha le sue cifre e il suo storico, con uno switch in alto.
- **Backup JSON** — esporta o importa per spostare i dati tra dispositivi.

---

## Come avviarla

Hai tre opzioni, dalla più semplice alla più "pro".

### 1. File locale via iCloud / Google Drive (zero lavoro)

1. Scarica `where-is-my-salary.html`.
2. Salvalo su iCloud Drive (o Google Drive, OneDrive…).
3. Sul telefono apri il file dall'app File — si apre nel browser.
4. Aggiungilo alla schermata Home: in Safari → Condividi → **Aggiungi a Home**. Diventerà un'icona come una vera app.

> ⚠️ I dati vivono nel browser di ogni dispositivo separatamente. Per averli identici su Mac e iPhone, esporta JSON da uno e importalo sull'altro.

### 2. Netlify Drop (5 minuti, zero account)

1. Vai su [app.netlify.com/drop](https://app.netlify.com/drop).
2. Trascina `where-is-my-salary.html` nella zona bianca.
3. Netlify ti dà un URL del tipo `https://nome-casuale.netlify.app/where-is-my-salary.html`.
4. Apri l'URL dal telefono → "Aggiungi a Home". Fatto.

### 3. GitHub Pages (1 minuto se hai già una repo)

1. Crea una repo **pubblica** (Pages free non funziona con repo private).
2. Carica `where-is-my-salary.html` rinominandolo `index.html`.
3. **Settings** → **Pages** → branch: `main`, folder: `/ (root)` → Save.
4. Dopo qualche minuto: `https://<username>.github.io/<repo>/`.

---

## Privacy

Tutto vive nel `localStorage` del browser. Nessun server, nessun cloud, nessuna telemetria.

Se pubblichi l'app online (Netlify o Pages), l'URL è tecnicamente pubblico ma **i tuoi dati no**: ognuno che apre l'URL parte da zero perché il `localStorage` è per-browser. È la stessa logica per cui un blocco note in tasca è privato anche se la fabbrica del blocco note ha mille copie identiche.

---

## Come si calcola "quanto ti devono"

Per ogni voce e ogni mese:

| Stato | Conteggio |
|---|---|
| ✓ ricevuto | conta come incassato, e come atteso |
| ! mancante | conta come atteso; conta come "da ricevere" solo se il mese è già passato o è quello corrente |
| — non spettato | non conta |

L'importo atteso è ricavato dalle impostazioni del profilo (stipendio, welfare, fringe, ticket × giorni lavorativi) o dall'importo che inserisci direttamente (bonus, rimborsi).

I giorni lavorativi sono calcolati con weekend italiani esclusi e queste festività nazionali: Capodanno, Epifania, Pasquetta, Liberazione, Festa del Lavoro, Festa della Repubblica, Ferragosto, Ognissanti, Immacolata, Natale, Santo Stefano. La data di Pasqua si ricava con l'algoritmo di Gauss/Meeus.

---

## Tech stack

> Un solo file, niente build, niente CDN obbligatorie.

- HTML + CSS + JS vanilla (~1100 righe, in un unico `.html`)
- `localStorage` per la persistenza
- Google Fonts opzionali (Source Serif 4, Inter, Caveat) — caricate via CDN, ma con fallback a system serif/sans, quindi l'app è leggibile anche senza connessione
- Calcolo Pasqua e festività italiane fatto in-app (vedi `easterSunday`, `italianHolidays`)

Non ci sono framework, bundler, package manager. Il file si modifica con un editor di testo qualunque.

---

## Personalizzazione

Tutto è configurabile dall'icona ⚙️ in alto. Per ogni profilo puoi impostare:

- Stipendio mensile
- Welfare mensile (default 400 €)
- Fringe benefit annuale (default 1000 €)
- Mese in cui arriva il fringe (default Dicembre)
- Ticket per giorno lavorativo (default 10 €)
- Bonus standard (per quando segni un bonus come atteso senza specificarne l'importo)

Se i valori cambiano (es. aumento di stipendio), modificarli ricalcola automaticamente anche gli importi attesi nei mesi già marcati.

---

## Backup & portabilità

Dalle impostazioni → **Backup e portabilità** puoi:

- **Esporta JSON** — un file di backup con tutti i profili, le impostazioni e lo storico mensile.
- **Importa** — un file precedentemente esportato. Sovrascrive i dati attuali (chiede conferma).

Conservare un export ogni tanto è una buona idea: serve sia come backup, sia per portare i dati su un altro telefono o computer.

---

## Roadmap

Idee in attesa di valutazione, non promesse:

- Sincronizzazione cross-device opzionale (un backend leggero — Cloudflare Workers + KV, o simile)
- Notifica a fine mese per "voci ancora mancanti"
- Esportazione PDF del riepilogo annuale
- Gestione TFR e tredicesima/quattordicesima
- Modalità "previsione" per simulare aumenti o cambi contratto

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
