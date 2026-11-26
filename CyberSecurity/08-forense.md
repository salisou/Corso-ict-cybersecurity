# Forense digitale

L'analisi forense mira a raccogliere e interpretare dati mantenendone documentate origine e integrità. In un caso reale, segui le procedure dell'organizzazione e le leggi applicabili; questa scheda è per copie e immagini di laboratorio.

## Principi di acquisizione

- Identifica il supporto o file sorgente e annota chi lo ha consegnato.
- Riduci le scritture sull'originale; usa blocco in scrittura dove previsto.
- Crea una copia di lavoro e calcola hash dell'originale disponibile e della copia.
- Registra strumenti, versione, ora, fuso orario, comandi e anomalie.
- Mantieni l'originale protetto e non analizzarlo direttamente quando è possibile lavorare su una copia.

Un hash corrispondente supporta la verifica che due file siano identici; da solo non dimostra provenienza o autenticità.

## Hash e manifest

```bash
# Calcola l'hash di un'immagine già acquisita
sha256sum immagine.dd

# Salva il valore in un manifest
sha256sum immagine.dd > immagine.sha256

# Verifica successivamente il manifest
sha256sum -c immagine.sha256
```

Su Windows:

```powershell
Get-FileHash .\immagine.dd -Algorithm SHA256
```

Conserva il manifest separatamente dalla copia analizzata e documenta come è stato trasferito.

## Esaminare un'immagine con The Sleuth Kit

I comandi seguenti leggono un file immagine preparato per il laboratorio:

```bash
# Mostra informazioni generali del filesystem
fsstat immagine.dd

# Elenca la tabella delle partizioni, se presente
mmls immagine.dd

# Elenca file e directory
fls -r immagine.dd
```

`mmls` aiuta a individuare partizioni e relativi offset. `fsstat` descrive il filesystem. `fls` può mostrare voci cancellate ancora presenti nei metadati, ma non garantisce che i contenuti siano recuperabili.

## Timeline

```bash
# Crea il body file a partire dall'immagine
fls -r -m "/" immagine.dd > timeline-body.txt

# Ordina gli eventi in una timeline leggibile
mactime -b timeline-body.txt > timeline.csv
```

I timestamp possono rappresentare momenti diversi (modifica, accesso, cambio metadati e creazione, se supportata). Considera fuso orario, orologio del sistema e possibili modifiche intenzionali o automatiche.

## Analisi di file in copia

```bash
# Identifica il tipo rilevato
file <FILE_COPIA>

# Legge metadati EXIF, se presenti
exiftool <FILE_COPIA>

# Estrae un campione di stringhe ASCII stampabili
strings -a <FILE_COPIA> | head -n 100

# Hash del file analizzato
sha256sum <FILE_COPIA>
```

I metadati possono mancare o essere modificati. Le stringhe trovate sono indizi, non prova del contesto o dell'autore.

## Raccolta di dati volatili in Linux

I dati volatili cambiano rapidamente. In un esercizio, raccogli le informazioni nella VM di laboratorio e annota che la raccolta stessa può modificare lo stato del sistema.

```bash
# Ora e identificazione della macchina
date -Is
hostname
uname -a
uptime

# Utenti e processi correnti
who
ps auxww

# Connessioni e interfacce di rete
sudo ss -tunap
ip addr
ip route
```

Non copiare le uscite in una cartella condivisa senza verificare che non contengano account, indirizzi o altri dati sensibili.

## Raccolta di dati Windows

```powershell
# Identità e orario
Get-Date -Format o
$env:COMPUTERNAME
whoami

# Processi e connessioni TCP
Get-Process
Get-NetTCPConnection

# Esporta il registro System in un file di laboratorio
wevtutil epl System .\System-lab.evtx

# Hash del file esportato
Get-FileHash .\System-lab.evtx -Algorithm SHA256
```

L'accesso ai registri e ai dettagli di processo dipende dai permessi e dalle policy di audit.

## Catena di custodia

Registra in un verbale, senza inserire segreti:

| Campo | Esempio |
|---|---|
| ID evidenza | `LAB-2026-001` |
| Descrizione e origine | `Immagine disco di una VM didattica` |
| Data e ora | In formato ISO 8601, con fuso orario |
| Operatore | Nome o identificativo previsto dalla procedura |
| Metodo e strumenti | Comando e versione utilizzata |
| Hash | SHA-256 del file acquisito |
| Passaggi di custodia | Trasferimenti, luogo di conservazione e destinatari |

Conserva gli originali con accesso limitato e documenta ogni trasferimento. Per esercitazioni, usa dati sintetici e non immagini di dispositivi personali o di terzi.

## Esercizio guidato

1. Usa un'immagine disco didattica già predisposta dal corso.
2. Calcola SHA-256 e salva il risultato in un verbale.
3. Esegui `mmls`, `fsstat` e `fls` sull'immagine.
4. Genera una timeline e annota tre eventi con il relativo timestamp.
5. Copia l'immagine in una seconda directory e verifica l'hash della copia.
6. Confronta i risultati e indica quali conclusioni sono supportate dai dati e quali restano ipotesi.

Non usare strumenti di acquisizione su dischi fisici senza autorizzazione, competenza e procedure adeguate: una scelta errata del dispositivo di destinazione può sovrascrivere dati.
## Spiegazione riga per riga dei comandi

La **forense digitale** raccoglie e interpreta dati digitali preservandone integrità e tracciabilità. Gli strumenti seguenti devono essere applicati a immagini o copie di laboratorio, non a dispositivi di terzi senza autorizzazione e procedura.

### Hash e manifest
- `sha256sum immagine.dd`: calcola l'impronta SHA-256 dell'immagine disco.
- `sha256sum immagine.dd > immagine.sha256`: scrive il risultato in un file manifest; `>` crea o sovrascrive il file di destinazione.
- `sha256sum -c immagine.sha256`: ricalcola e confronta l'hash con il valore del manifest.
- `Get-FileHash .\\immagine.dd -Algorithm SHA256`: esegue il calcolo SHA-256 equivalente in PowerShell.

### The Sleuth Kit
- `fsstat immagine.dd`: mostra le informazioni sul filesystem contenuto nell'immagine, come tipo e parametri.
- `mmls immagine.dd`: elenca la struttura delle partizioni e gli offset presenti nell'immagine.
- `fls -r immagine.dd`: elenca ricorsivamente file e directory; `-r` richiede la discesa nelle directory. Le voci elencate non garantiscono che i dati siano recuperabili.

### Timeline
- `fls -r -m "/" immagine.dd > timeline-body.txt`: elenca le voci in modo ricorsivo; `-m "/" ` imposta il prefisso del percorso per il body file usato da The Sleuth Kit; `>` salva il risultato in `timeline-body.txt`.
- `mactime -b timeline-body.txt > timeline.csv`: legge il body file con `-b` e genera una timeline; `>` salva il risultato in CSV.

### Analisi dei file
- `file <FILE_COPIA>`: prova a identificare il tipo del file in base al contenuto, non soltanto all'estensione.
- `exiftool <FILE_COPIA>`: legge metadati incorporati, se presenti, come informazioni di formato e timestamp.
- `strings -a <FILE_COPIA> | head -n 100`: `strings` estrae sequenze di caratteri stampabili; `-a` analizza l'intero file; la pipe passa il risultato a `head`; `-n 100` limita la visualizzazione alle prime 100 righe.
- `sha256sum <FILE_COPIA>`: calcola l'hash del file esaminato per documentare quale copia è stata analizzata.

### Dati volatili Linux
- `date -Is`: registra data e ora in formato ISO 8601 con indicazione del fuso orario.
- `hostname`: mostra il nome della macchina.
- `uname -a`: mostra informazioni sul kernel e sull'architettura.
- `uptime`: mostra da quanto tempo il sistema è acceso e il carico medio.
- `who`: elenca gli utenti con sessioni attive.
- `ps auxww`: mostra i processi; `a` include processi associati a terminali, `u` usa un formato dettagliato per utente, `x` include processi senza terminale e `ww` evita di troncare la riga di comando.
- `sudo ss -tunap`: mostra socket e connessioni; `-t` TCP, `-u` UDP, `-n` valori numerici, `-a` tutti i socket e `-p` processo associato.
- `ip addr`: mostra interfacce e indirizzi.
- `ip route`: mostra la tabella di instradamento, utile per documentare gateway e percorsi di rete.

### Raccolta Windows
- `Get-Date -Format o`: stampa data e ora in formato round-trip ISO 8601.
- `$env:COMPUTERNAME`: legge la variabile d'ambiente contenente il nome del computer.
- `whoami`: mostra l'account corrente.
- `Get-Process`: elenca i processi attivi.
- `Get-NetTCPConnection`: mostra le connessioni TCP e le porte in ascolto.
- `wevtutil epl System .\\System-lab.evtx`: esporta il registro System nel file EVTX indicato.
- `Get-FileHash .\\System-lab.evtx -Algorithm SHA256`: calcola l'hash del file EVTX esportato.

La raccolta dei dati può modificare lo stato del sistema e i timestamp. Annota sempre l'orario, i privilegi, lo strumento e il comando effettivamente usato.
