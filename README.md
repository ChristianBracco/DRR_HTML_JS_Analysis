# 🛰 SIEMENS FLC QC Viewer — Console di Missione CQ

```
      .        .        *        .          .       .      *      .
   .      ___________________          .        *        .     .
        /                     \    .          .      .
   *   |   FLC-QC-SESSION      |        ▲            .     *
       |   [ ==== MISSION ==== ]|      ╱ ╲     .          .
   .    \_____________________/       ╱███╲        *      .
            |  |      |  |          ╱██▓▓▓██╲   .      .
   .     ~~~~~~~~~~~~~~~~~~~~      ╱─────────╲        .    *
```

> **Rotta attuale:** v19 — Analisi CQ + Dati Report + Import Excel completo
> **Ambiente operativo:** singolo file HTML, tutto a bordo, nessun collegamento a terra.

Benvenuto a bordo. Questo strumento è la **console di controllo qualità** per i rivelatori digitali **Siemens FLC**. Immagini, ROI, analisi, report e confronti viaggiano tutti dentro un unico file HTML autocontenuto: nessun server, nessun database, nessun upload. I dati radiologici restano nel tuo browser, come in una capsula sigillata.

---

## 🌌 Missione

Rendere il controllo di qualità dei sistemi DR **semplice, riproducibile e tracciabile**, dal primo lancio (caricamento immagini) fino al rientro (report firmato).

Rotta di volo:

```
   ACQUISIZIONE ─▶ VERIFICA/RINOMINA ─▶ ANALISI ─▶ DATI STRUTTURATI ─▶ REPORT
        Step 1            Step 1            Step 2         (FLC-QC-SESSION)     Step 3
```

Interfaccia pensata anche per operatori non esperti:

1. carichi una singola immagine o una cartella;
2. verifichi ed eventualmente rinomini le acquisizioni;
3. disegni o richiami le ROI;
4. esegui le analisi CQ;
5. **precompili tutto da un vecchio Excel o JSON**;
6. copi i risultati in Excel oppure esporti la sessione in JSON;
7. generi il report finale.

---

## 🚀 Ponte di comando — le tre postazioni

L'app è organizzata in **3 step** (wizard) con una **dashboard di missione** che mostra, a colpo d'occhio, quanto sei vicino al report pronto.

- **Step 1 — Scheda acquisizione:** dati identificativi, condizioni di lavoro, tabelle STP / EI / ripetibilità / TOR CDR / ghost.
- **Step 2 — Analisi immagini:** ROI sul range, non uniformità DR, bad pixel, ghost.
- **Step 3 — Report:** foglio compilabile con calcoli automatici, grafici, esiti e firme.

La **🛰 Controllo missione** calcola una percentuale di completamento e ti dice cosa manca prima del report.

---

## 🛰 Sensori di bordo — visualizzazione

### Siemens FLC

Gestisce le acquisizioni composte da:

```text
file.hdr
file           (raster senza estensione)
file.pp
```

Preset per le principali matrici Siemens:

- MAX mini — 1920 × 1520
- MAX wi-D — 2350 × 2866
- MAX static — 2868 × 2874
- MAX dynamic RAD — 2840 × 2874
- MAX dynamic FLU / DFR
- FLC storage — 3072 × 2657

Dall'header proprietario `FLC_V7`, quando disponibili, ricava automaticamente: matrice, pixel spacing, data/ora, kVp, mAs, corrente tubo, tempo di esposizione (con controlli di plausibilità).

### DICOM

Riconosce anche file DICOM, compresi quelli senza estensione `.dcm` se presente il preambolo. Transfer syntax **non compresse**:

- Implicit VR Little Endian
- Explicit VR Little Endian
- Explicit VR Big Endian

Legge, quando presenti: Acquisition Date/Time, matrice, pixel spacing, Window Center/Width, kVp, Exposure Time, Tube Current, Exposure/mAs, SID e info su apparecchiatura/serie/istanza.

Per i mAs, in ordine: **Exposure** `(0018,1152)` → **Exposure in µAs** `(0018,1153)` → calcolo `mA × ms / 1000`.

> **Limite:** i DICOM compressi (JPEG, JPEG-LS, JPEG 2000, RLE) non sono ancora decodificati.

---

## 🔧 Manovre di allineamento — rinomina guidata

Il pulsante `✎ Rinomina immagini` apre una cartella e costruisce la lista ordinata delle esposizioni.

- **Ordinamento:** per `Acquisition Date + Acquisition Time`, per ricostruire la sequenza reale.
- **Proposta nomi:** `1, 2, 3, …` (abitudine operativa), tutti modificabili.
- **Info per esposizione:** progressivo, anteprima, data/ora, kVp, mAs, nome originale, componenti (HDR/RAW/PP o DICOM), nuovo nome.

Una esposizione Siemens è un **unico oggetto logico**: `1.hdr`, `1`, `1.pp` mantengono sempre lo stesso nome base.

**Sicurezza:** prima di scrivere vengono verificati duplicati, caratteri non validi, nomi riservati Windows e collisioni. La rinomina avviene in due fasi (nome temporaneo `.__flcqc_*` → nome definitivo) per gestire scambi e collisioni.

> Richiede un browser Chromium recente (Edge/Chrome) con File System Access API.

---

## 🎯 Strumenti di puntamento — ROI

Tre strumenti: **rettangolo**, **cerchio/ellisse**, **misura lineare**.

Per ogni ROI: numero pixel, area (se noto il pixel spacing), media, deviazione standard, minimo, massimo, coefficiente di variazione.

Le ROI si possono selezionare, trascinare, duplicare, eliminare e muovere da tastiera:

```text
Frecce          spostamento di 1 pixel
Shift + Frecce  spostamento di 10 pixel
Ctrl/Cmd + D    duplica ROI
Canc/Backspace  elimina ROI
```

**Template ROI:** salvabili nel browser, richiamabili, esportabili/importabili in JSON. Conservano matrice di riferimento, pixel spacing e coordinate normalizzate, così le ROI si riadattano se la matrice cambia.

---

## 🔬 Laboratorio di bordo — Analisi CQ

Pannello unico: **Analisi CQ — scegli cosa devi fare**.

### 1. ROI sul range
Le ROI correnti vengono applicate a tutte le immagini del range. Per ogni coppia `immagine × ROI`: area, n. pixel, media, SD, min, max, CV. Copia per Excel disponibile (`Copia 1–6`, `Copia 7–8`, `Copia tutto`, formato TSV).

### 2. Non uniformità DR
Calcolo guidato su margine, lato ROI e livelli di esposizione (di norma **2,5** e **10 mGy**). Indici: **NULS, NUGS, NULSNR, NUGSNR**.
Sulla prima immagine di uniformità viene eseguita anche la **ricerca bad pixel** (immagine, media, SD, soglia, min, max, conteggio, coordinate).

### 3. Ghost / immagini latenti
Servono 2 ROI sull'immagine irradiata (piombo / fuori piombo); le stesse coordinate valgono sull'immagine buia. Vengono misurate le quattro combinazioni:

```text
ROI1immIRR   ROI2immBuio   ROI3immIRR   ROI4immBuio
```

---

## 🧭 Registro di volo — FLC-QC-SESSION

Struttura dati unica che raccoglie progressivamente il lavoro della sessione (schema `FLC-QC-SESSION`):

```text
sessione
├── sorgente        (tipo, range, matrice, pixel spacing)
├── acquisizioni    (n. immagine, file, HDR, data/ora, tecnica: kVp/mAs/mA/ms)
├── ROI             (nome, tipo, geometria)
├── misure          (ROI sul range · uniformità · bad pixel · ghost)
└── esportazioni
```

Caricando una nuova serie parte una nuova sessione, senza mescolare controlli diversi.

---

## 📡 Aggancio orbitale — Import da Excel / JSON

> **Novità della rotta attuale.** Nella scheda acquisizione trovi la zona **"Precompila da controllo precedente"**: trascina un vecchio **`.xlsx`** o un **`.json`** salvato in precedenza.

### Import Excel (.xlsx)

Il lettore ZIP + DEFLATE è **nativo e offline** (usa `DecompressionStream('deflate-raw')`), quindi **nessuna libreria esterna**. Legge i fogli **`Report`** e **`Inserimento dati`** e **popola tutti i campi**, comprese le misure che servono per il calcolo del CQ:

- **Identificativi:** azienda, sala, unità, ditta/modello, serie tubo, operatore, software, note, data controllo.
- **Setup:** tipo DR, protocollo, griglia, filtrazione, DFR, kV, distanze (fuori potter / lettino / stativo), tipo curva PV ed EI (LIN/LOG).
- **STP (6 punti):** mAs, Karia, `<PV>`, SD, EI (dal foglio Report; fallback su Inserimento dati).
- **Ripetibilità EI:** `<PV>`, EI, Karia, mAs.
- **Uniformità:** NULS/NUGS a 2,5 e 10 µGy (in %).
- **Bad pixel** e **artefatti**.
- **Basso contrasto (TOR CDR):** Karia, mAs, oggetti visibili.
- **Ghost:** medie ROI2/ROI3/ROI4, più mAs/kV di ghost e minimo carico.

Al termine i valori condivisi (kV/Karia/mAs allo stesso µGy) vengono riconciliati e il report ricalcola tutte le grandezze derivate.

> Per l'import `.xlsx` serve un browser Chromium recente (Edge/Chrome). Con altri browser resta disponibile l'import **JSON**.

---

## 🛸 Confronto di rotta — CQ costanza vs accettazione

Poiché l'import popola **tutti** i campi, ogni Excel importato diventa un **CQ completo e confrontabile**.

Flusso consigliato:

```
   Import Excel  ─▶  campi popolati + CQ calcolato  ─▶  Esporta dati JSON  ─▶  Confronto controlli
```

1. Importa l'Excel del controllo precedente → tutti i campi si riempiono.
2. **Esporta dati JSON** per ottenere il file di quel controllo.
3. Apri **Confronto controlli** (accettazione vs costanza), oppure usa **"il controllo corrente come B"**.

Il confronto mostra le **variazioni %** delle grandezze chiave (R² PV, coefficienti a/b e c/d, NULS/NUGS, lag, `<K>` medio) ed evidenzia gli scostamenti oltre **±10%**.

---

## 📈 Rotta di conversione — PV teorico e scelta della curva

La funzione di risposta STP viene modellata con una regressione, la cui **ascissa dipende dal tipo di curva**:

```text
LIN :  <PV> = a · Karia      + b
LOG :  <PV> = a · ln(Karia)  + b        (definito solo per Karia > 0)
```

Il **PV teorico** di ogni riga usa esattamente la stessa trasformazione del fit, quindi tabella, coefficienti e grafico restano coerenti. Se un Karia non è valido per la curva scelta (es. `≤ 0` in LOG), la cella mostra `—` invece di un numero anomalo. Analogo trattamento robusto per la curva EI (Karia calcolato da EI).

### 🧑‍🚀 Copilota della curva

Sotto la nota di linearità compare, quando serve, un **avviso**: se l'altra curva (LIN ↔ LOG) darebbe un fit migliore, lo strumento te lo segnala.

- Appare solo se l'alternativa ha **R² maggiore** e la curva attuale è sotto soglia (**R² < 0,98**) **oppure** il guadagno è rilevante (**≥ 0,5 punti** di R²).
- Se l'alternativa supera 0,98 dice "sarebbe conforme"; altrimenti dice solo "con R² migliore", senza promettere conformità.
- Nessun avviso se la curva scelta è già ottima: niente rumore inutile.

---

## 🗄 Diario di bordo — storico e andamento

- **🗄 Archivia:** salva il controllo corrente nello storico locale (browser).
- **Andamento nel tempo:** sparkline per R² PV/EI, NULS/NUGS, lag ghost, `<K>` medio, bad pixel.

---

## 📤 Esportazioni

Nel pannello **Dati sessione / report**:

- **📄 Genera report** — foglio compilato con calcoli, grafici ed esiti.
- **Copia tutte le misure per Excel** — blocco TSV unico (ROI sul range, non uniformità, bad pixel, ghost), incollabile in Excel.
- **Esporta dati JSON** — `FLC_QC_...json` con l'intera struttura: interfaccia fra analisi immagini → dati strutturati → report.
- **Salva/Carica dati inseriti** — bundle JSON di tutte e tre le pagine, ricaricabile su qualsiasi PC.
- **Salva report come HTML** — report autonomo e stampabile.
- **CLEAN UP** — azzera scheda, misure, ROI, storico e report corrente (le preferenze di tema/zoom restano). Operazione non reversibile.

---

## 🧯 Sicurezza della capsula — privacy e funzionamento locale

Un singolo file HTML. Niente server, database, installazione, upload o servizi esterni per l'analisi. Ideale per dati tecnici o sanitari che non devono lasciare la postazione.

---

## 🕹 Avvio

1. scarica il file HTML;
2. aprilo con un browser moderno;
3. carica una singola immagine o una cartella.

Per **rinomina file** e **import Excel** usa preferibilmente **Microsoft Edge** o **Google Chrome**.

---

## 🛰 Plancia di stato

### A bordo (implementato)

- [x] Visualizzazione Siemens FLC e DICOM non compresso
- [x] Lettura automatica matrice, pixel spacing, data/ora, kVp, mAs
- [x] Anteprime e ordinamento temporale
- [x] Rinomina guidata `1, 2, 3…` con gestione HDR + RAW + PP
- [x] ROI rettangolari/circolari + template ROI
- [x] Analisi ROI sul range · Non uniformità DR · Bad pixel · Ghost
- [x] Struttura dati `FLC-QC-SESSION` + export JSON
- [x] Copia dati per Excel (TSV)
- [x] **Import Excel completo (tutti i campi, incluse le misure)**
- [x] **CQ confrontabile via JSON (accettazione vs costanza)**
- [x] **PV teorico coerente LIN/LOG + robustezza sui valori non validi**
- [x] **Avviso di scelta curva (suggerisce LIN/LOG in base a R²)**
- [x] Report compilabile con calcoli, grafici, esiti e firme
- [x] Storico controlli e andamento nel tempo

### Prossime rotte (roadmap)

- [ ] Riepilogo automatico delle acquisizioni utilizzate nel report
- [ ] Esportazione report in ulteriori formati stampabili
- [ ] Eventuale supporto ai DICOM compressi
- [ ] Ulteriore semplificazione del flusso guidato

---

## 📎 Note sul formato Siemens FLC

Alcuni parametri tecnici derivano dall'header proprietario `FLC_V7`, con controlli di plausibilità su kVp, tempo di esposizione, mAs e corrente ricostruita. Essendo un formato proprietario, nuovi layout dell'header potrebbero richiedere verifiche: **confronta sempre i parametri mostrati con il contesto dell'acquisizione**.

---

## ⚠️ Uso previsto

Strumento di **supporto** al controllo di qualità e all'organizzazione delle misure. Non sostituisce:

- le procedure aziendali;
- i protocolli di controllo qualità applicabili;
- la valutazione professionale del fisico medico;
- la verifica dei risultati prima della registrazione ufficiale.

---

## 🧩 Struttura del progetto

Single-file application:

```text
SIEMENS_TLC_CalculatorHDR_v19_DatiReport_Excel.html
```

HTML, CSS e JavaScript sono incorporati nello stesso file. Vantaggi: distribuzione semplice, nessuna dipendenza, uso offline, archiviazione facile insieme alla documentazione CQ.

```
        ·  ·   ✦   ·        ·      ✦        ·   ·      ·
   ·        Buona missione, e che i tuoi R² siano sempre > 0,98.  ✦
        ·        ·     ·        ✦      ·        ·         ·   ·
```
