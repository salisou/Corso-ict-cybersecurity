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
