# Siemens FLC QC Viewer

Viewer locale e strumento di supporto ai **controlli di qualità (CQ) dei rivelatori digitali Siemens FLC**, sviluppato come singolo file HTML autocontenuto.

La versione attuale integra visualizzazione delle immagini, gestione delle ROI, analisi su serie di acquisizioni, uniformità, ghost, rinomina guidata delle immagini e raccolta strutturata delle misure in vista della generazione automatica del report.

> **Versione:** v19 — Analisi CQ + Dati Report

---

## Obiettivo

Il progetto nasce per rendere più semplice e riproducibile il flusso di lavoro dei controlli di qualità su immagini Siemens FLC.

L'idea è mantenere un'interfaccia utilizzabile anche da operatori non esperti:

1. caricare una singola immagine o una cartella;
2. verificare e, se necessario, rinominare le acquisizioni;
3. disegnare o richiamare le ROI;
4. eseguire le analisi CQ;
5. copiare i risultati in Excel oppure esportare l'intera sessione in JSON;
6. in una versione successiva, generare direttamente il report finale.

Tutte le elaborazioni vengono eseguite **localmente nel browser**.

---

## Funzioni principali

### Visualizzazione Siemens FLC

Supporta acquisizioni costituite da:

```text
file.hdr
file
file.pp
```

Il programma gestisce sia il raster senza estensione sia il file `.pp`.

Sono disponibili preset per diverse matrici Siemens, tra cui:

- MAX mini — 1920 × 1520
- MAX wi-D — 2350 × 2866
- MAX static — 2868 × 2874
- MAX dynamic RAD — 2840 × 2874
- MAX dynamic FLU / DFR
- FLC storage 3072 × 2657

Quando possibile vengono ricavati automaticamente dall'header:

- matrice;
- pixel spacing;
- data e ora di acquisizione;
- kVp;
- mAs;
- corrente tubo;
- tempo di esposizione.

---

### Supporto DICOM

Il viewer riconosce anche file DICOM, compresi file privi dell'estensione `.dcm` quando è presente il preambolo DICOM.

Sono gestite le transfer syntax non compresse:

- Implicit VR Little Endian;
- Explicit VR Little Endian;
- Explicit VR Big Endian.

Il parser legge, quando disponibili:

- Acquisition Date;
- Acquisition Time;
- matrice;
- pixel spacing;
- Window Center / Window Width;
- kVp;
- Exposure Time;
- Tube Current;
- Exposure / mAs;
- SID;
- informazioni su apparecchiatura, serie e istanza.

Per i mAs vengono utilizzati, in ordine:

1. **Exposure** `(0018,1152)`;
2. **Exposure in µAs** `(0018,1153)`, convertito in mAs;
3. calcolo da `mA × ms / 1000`, se necessario.

### Limite attuale DICOM

Non sono ancora decodificate le immagini DICOM compresse:

- JPEG;
- JPEG-LS;
- JPEG 2000;
- RLE.

---

## Rinomina guidata delle acquisizioni

Il pulsante:

```text
✎ Rinomina immagini
```

apre una cartella e costruisce una lista ordinata delle esposizioni.

### Logica di ordinamento

Le acquisizioni vengono ordinate per:

```text
Acquisition Date + Acquisition Time
```

L'ordine temporale serve a ricostruire la sequenza reale di acquisizione.

La proposta di rinomina, invece, mantiene volutamente la normale abitudine operativa:

```text
1
2
3
4
...
```

L'operatore può modificare liberamente ogni nome prima di applicarlo.

### Informazioni mostrate

Per ogni esposizione vengono visualizzati:

- numero progressivo;
- anteprima dell'immagine;
- data e ora di acquisizione;
- kVp;
- mAs;
- nome originale;
- componenti disponibili: HDR, RAW, PP oppure DICOM;
- nuovo nome proposto.

kVp e mAs sono particolarmente utili per riconoscere:

- acquisizioni ripetute;
- esposizioni fuori sequenza;
- misure effettuate con tempi o carichi differenti.

### Gruppi Siemens

Una esposizione Siemens viene trattata come un unico oggetto logico.

La rinomina viene quindi applicata insieme a:

```text
1.hdr
1
1.pp
```

In questo modo i tre componenti della stessa acquisizione mantengono sempre lo stesso nome base.

### Sicurezza della rinomina

Prima di scrivere i nuovi nomi vengono verificati:

- nomi duplicati;
- caratteri non validi;
- nomi riservati da Windows;
- collisioni con file già presenti nella cartella.

La rinomina viene eseguita in due fasi:

1. ogni file viene spostato temporaneamente a un nome `.__flcqc_*`;
2. i file temporanei vengono rinominati con il nome definitivo.

Questo permette di gestire anche scambi di nome e collisioni intermedie senza sovrascrivere direttamente i file originali.

> La rinomina diretta della cartella richiede un browser Chromium recente, ad esempio **Microsoft Edge** o **Google Chrome**, con supporto alla File System Access API.

---

## ROI

Sono disponibili tre strumenti:

- ROI rettangolare;
- ROI circolare / ellittica;
- misura lineare.

Per ciascuna ROI di misura vengono calcolati:

- numero di pixel;
- area, quando è noto il pixel spacing;
- media;
- deviazione standard;
- minimo;
- massimo;
- coefficiente di variazione.

### Gestione delle ROI

Le ROI possono essere:

- selezionate;
- trascinate;
- duplicate;
- eliminate;
- spostate con la tastiera.

Scorciatoie:

```text
Frecce          spostamento di 1 pixel
Shift + Frecce  spostamento di 10 pixel
Ctrl/Cmd + D    duplica ROI
Canc/Backspace  elimina ROI
```

---

## Template ROI

Un set di ROI può essere:

- salvato nel browser;
- richiamato successivamente;
- esportato in JSON;
- importato da JSON.

Il template conserva anche:

- matrice di riferimento;
- pixel spacing;
- coordinate normalizzate.

Questo consente di riadattare le ROI quando la matrice dell'immagine cambia.

---

# Analisi CQ

Le funzioni di analisi sono raccolte in un unico pannello:

```text
Analisi CQ — scegli cosa devi fare
```

La versione attuale contiene tre gruppi principali.

---

## 1. ROI sul range

Le ROI correnti vengono applicate automaticamente a tutte le immagini comprese nel range selezionato.

Per ogni coppia:

```text
immagine × ROI
```

vengono registrati:

- area;
- numero di pixel;
- media;
- deviazione standard;
- minimo;
- massimo;
- coefficiente di variazione.

I risultati vengono mostrati in tabella.

### Copia per Excel

Per ciascuna ROI sono disponibili:

```text
Copia 1–6 per Excel
Copia 7–8 per Excel
Copia tutto
```

I dati vengono copiati in formato TSV, quindi possono essere incollati direttamente in Excel.

---

## 2. Non uniformità DR

Il modulo esegue il calcolo guidato della non uniformità sulle immagini selezionate.

Impostazioni disponibili:

- margine della zona analizzata;
- lato delle ROI;
- immagini da associare ai livelli di esposizione.

La procedura propone normalmente:

```text
2,5 mGy
10 mGy
```

e calcola gli indici:

- NULS;
- NUGS;
- NULSNR;
- NUGSNR.

### Bad pixel

Sulla prima immagine utilizzata per l'analisi di uniformità viene eseguita anche la ricerca dei bad pixel.

Vengono conservati:

- immagine analizzata;
- media;
- deviazione standard;
- soglia;
- minimo;
- massimo;
- numero di bad pixel;
- coordinate dei pixel individuati.

I risultati dell'uniformità possono essere copiati per Excel.

---

## 3. Ghost / immagini latenti

Per il calcolo del ghost sono necessarie due ROI definite sull'immagine irradiata:

```text
ROI 1 → zona con piombo
ROI 2 → zona fuori dal piombo
```

Le stesse coordinate vengono quindi utilizzate anche sull'immagine buia.

L'utente sceglie:

- immagine irradiata;
- immagine buia / minimo carico.

Il programma misura automaticamente le quattro combinazioni:

```text
ROI1immIRR
ROI2immBuio
ROI3immIRR
ROI4immBuio
```

Per ciascuna vengono riportati:

- numero immagine;
- area;
- media;
- deviazione standard;
- minimo;
- massimo.

È disponibile il comando:

```text
Copia per Excel
```

---

# Dati della sessione

La v19 introduce una struttura dati unica pensata come base del futuro generatore di report.

Lo schema interno è identificato come:

```text
FLC-QC-SESSION
```

e raccoglie progressivamente il lavoro eseguito durante la sessione.

Struttura concettuale:

```text
sessione
├── sorgente
│   ├── tipo
│   ├── range
│   ├── matrice
│   └── pixel spacing
│
├── acquisizioni
│   ├── numero immagine
│   ├── file
│   ├── HDR
│   ├── data acquisizione
│   ├── ora acquisizione
│   └── tecnica
│       ├── kVp
│       ├── mAs
│       ├── mA
│       └── ms
│
├── ROI
│   ├── nome
│   ├── tipo
│   └── geometria
│
├── misure
│   ├── ROI sul range
│   ├── uniformità
│   ├── bad pixel
│   └── ghost
│
└── esportazioni
```

La sessione viene aggiornata automaticamente quando vengono eseguite le analisi.

Caricando una nuova serie viene inizializzata una nuova sessione, evitando di mescolare i risultati di controlli differenti.

---

## Esportazione dei dati

Nel pannello **Dati sessione / report** sono disponibili due comandi.

### Copia tutte le misure per Excel

Genera un unico blocco TSV contenente, quando presenti:

```text
ROI SUL RANGE
NON UNIFORMITA
BAD PIXEL
GHOST
```

Il testo può essere incollato direttamente in Excel.

### Esporta dati JSON

Genera:

```text
FLC_QC_sessione.json
```

contenente l'intera struttura dati della sessione.

Questo JSON è pensato anche come interfaccia fra:

```text
analisi delle immagini
        ↓
dati strutturati
        ↓
generazione del report
```

---

## Flusso di lavoro consigliato

### 1. Aprire la cartella

Usare:

```text
▣ Carica cartella / range
```

oppure, quando serve prima sistemare i nomi:

```text
✎ Rinomina immagini
```

### 2. Verificare l'ordine

Controllare:

- Acquisition Date/Time;
- kVp;
- mAs;
- anteprima.

Eventuali ripetizioni diventano così immediatamente riconoscibili.

### 3. Selezionare il range

Impostare:

```text
Da slice
A slice
```

e premere **Applica range**.

### 4. Preparare le ROI

Disegnare le ROI oppure richiamare un template precedentemente salvato.

### 5. Eseguire l'analisi

Dal pannello **Analisi CQ** scegliere:

- ROI sul range;
- Non uniformità DR;
- Ghost.

### 6. Esportare

A seconda del flusso di lavoro:

- copiare i singoli blocchi in Excel;
- copiare tutte le misure;
- esportare la sessione JSON.

---

## Privacy e funzionamento locale

Il programma è un singolo file HTML.

Non richiede:

- server;
- database;
- installazione;
- upload delle immagini;
- connessione a servizi esterni per l'analisi.

I file radiologici vengono letti localmente dal browser.

Questo approccio è particolarmente utile per dati tecnici o sanitari che non devono essere inviati a servizi remoti.

---

## Avvio

Non è necessaria una procedura di installazione.

1. scaricare il file HTML;
2. aprirlo con un browser moderno;
3. caricare una singola immagine o una cartella.

Per la funzione di **rinomina diretta dei file** utilizzare preferibilmente:

- Microsoft Edge;
- Google Chrome.

---

## Struttura del progetto

Attualmente il progetto è volutamente distribuito come **single-file application**:

```text
SIEMENS_TLC_CalculatorHDR_v19_DatiReport_Excel.html
```

HTML, CSS, JavaScript e risorse dell'interfaccia sono incorporati nello stesso file.

Vantaggi:

- distribuzione semplice;
- nessuna dipendenza da installare;
- utilizzo offline;
- facile archiviazione insieme alla documentazione CQ.

---

## Stato del progetto

### Implementato

- [x] Visualizzazione Siemens FLC
- [x] Visualizzazione DICOM non compresso
- [x] Lettura automatica matrice e pixel spacing quando disponibili
- [x] Lettura Acquisition Date/Time
- [x] Lettura kVp e mAs
- [x] Anteprima delle acquisizioni
- [x] Ordinamento temporale
- [x] Rinomina guidata `1, 2, 3…`
- [x] Gestione coordinata HDR + RAW + PP
- [x] ROI rettangolari e circolari
- [x] Template ROI
- [x] Analisi ROI sul range
- [x] Copia dati per Excel
- [x] Non uniformità DR
- [x] Ricerca bad pixel
- [x] Ghost / immagini latenti
- [x] Struttura dati `FLC-QC-SESSION`
- [x] Export JSON della sessione

### Roadmap

- [ ] Generazione automatica del report CQ
- [ ] Inserimento dei dati identificativi dell'apparecchiatura nel report
- [ ] Riepilogo automatico delle acquisizioni utilizzate
- [ ] Tabelle e risultati formattati automaticamente
- [ ] Esportazione del report in formato stampabile
- [ ] Eventuale supporto ai DICOM compressi
- [ ] Ulteriore semplificazione del flusso guidato

---

## Note sul formato Siemens FLC

La lettura di alcuni parametri tecnici Siemens FLC utilizza informazioni presenti nell'header proprietario `FLC_V7`.

Il parser applica controlli di plausibilità sui valori ricavati, in particolare per:

- kVp;
- tempo di esposizione;
- mAs;
- corrente ricostruita.

Poiché il formato è proprietario, nuovi layout o revisioni dell'header Siemens potrebbero richiedere ulteriori verifiche.

Per questo motivo i parametri mostrati dal programma devono essere sempre confrontati con il contesto dell'acquisizione quando vengono utilizzati per attività di controllo qualità.

---

## Uso previsto

Questo software è uno strumento di supporto al controllo di qualità e all'organizzazione delle misure.

Non sostituisce:

- le procedure aziendali;
- i protocolli di controllo qualità applicabili;
- la valutazione professionale del fisico medico;
- la verifica dei risultati prima della loro registrazione ufficiale.

---

## Sviluppo

Il progetto è in evoluzione.

La v19 segna il passaggio da un semplice viewer/strumento di misura a una struttura più completa:

```text
ACQUISIZIONE
     ↓
VERIFICA E RINOMINA
     ↓
ANALISI
     ↓
DATI STRUTTURATI
     ↓
REPORT
```

Il prossimo obiettivo è utilizzare direttamente `FLC-QC-SESSION` per produrre il report finale, evitando di ricalcolare o ricostruire manualmente i risultati già ottenuti durante la sessione.
