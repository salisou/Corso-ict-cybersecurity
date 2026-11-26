# Log e incident response

I log aiutano a ricostruire eventi, ma non sono una registrazione completa di ogni attività. La raccolta dipende dalla configurazione, dai permessi, dalla sincronizzazione dell'orologio e dalla conservazione.

## Metodo iniziale

Quando osservi un possibile incidente:

1. Registra chi sta eseguendo la raccolta, data, ora e fuso orario.
2. Annota sistema, hostname, sintomo e momento in cui è stato notato.
3. Segui la procedura di risposta dell'organizzazione e avvisa il referente.
4. Evita riavvii, pulizie, scansioni o modifiche non concordate: possono alterare dati utili.
5. Conserva gli originali, lavora su copie e registra chi accede alle evidenze.

In ambienti reali, non improvvisare contenimento o spegnimento: la decisione può avere impatti operativi e legali. Gli esercizi qui sotto si svolgono solo in VM di laboratorio.

## Eventi Windows

```powershell
# Ultimi eventi System
Get-WinEvent -LogName System -MaxEvents 50 |
  Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message

# Eventi Security: accessi falliti nelle ultime 24 ore
Get-WinEvent -FilterHashtable @{
  LogName = "Security"
  Id = 4625
  StartTime = (Get-Date).AddHours(-24)
} -MaxEvents 100

# Eventi PowerShell operativi, se il log è abilitato
Get-WinEvent -LogName "Microsoft-Windows-PowerShell/Operational" `
  -MaxEvents 50
```

L'evento 4625 indica un accesso non riuscito; confronta account, ora, tipo di accesso e origine. Le informazioni disponibili dipendono dalle impostazioni di audit. Gli eventi PowerShell potrebbero non essere abilitati o conservati.

Esporta un registro per analizzarlo in seguito:

```powershell
# Esporta il registro System in formato EVTX
wevtutil epl System .\System-lab.evtx

# Calcola un hash per documentare il file esportato
Get-FileHash .\System-lab.evtx -Algorithm SHA256
```

## Log Linux

```bash
# Log del servizio SSH: il nome può essere ssh o sshd
sudo journalctl -u ssh -n 100 --no-pager
sudo journalctl -u sshd -n 100 --no-pager

# Eventi kernel recenti
sudo journalctl -k -n 100 --no-pager

# Eventi nell'intervallo di tempo
sudo journalctl --since "2026-01-01 10:00:00" \
  --until "2026-01-01 11:00:00" --no-pager

# Log tradizionali Debian/Ubuntu, se presenti
sudo grep -i "failed password" /var/log/auth.log*

# Log tradizionali RHEL/Rocky/Fedora, se presenti
sudo grep -i "failed password" /var/log/secure*
```

Il nome del servizio e il percorso dei log variano tra distribuzioni. `journalctl` usa l'orario del sistema; registrare anche il fuso orario evita ambiguità tra macchine.

## Filtrare e ordinare

```powershell
# Filtra gli eventi Security per ID e intervallo temporale
Get-WinEvent -FilterHashtable @{
  LogName = "Security"
  Id = 4625
  StartTime = (Get-Date).AddHours(-6)
} | Sort-Object TimeCreated |
  Select-Object TimeCreated, Id, Message
```

```bash
# Cerca una parola chiave nei log del journal
sudo journalctl --since today --no-pager | grep -i "error"

# Conta le righe che contengono una parola chiave
sudo journalctl --since today --no-pager |
  grep -ic "failed"
```

Una ricerca testuale può perdere eventi espressi con campi o messaggi differenti. Parti da un intervallo di tempo ristretto e amplia solo se necessario.

## Integrità e catena di custodia

```powershell
# Hash SHA-256 di un file di evidenza
Get-FileHash .\evidenza.log -Algorithm SHA256
```

```bash
# Crea e verifica un manifest di hash
sha256sum evidenza.log > evidenza.sha256
sha256sum -c evidenza.sha256
```

Registra almeno: identificativo dell'evidenza, nome e origine, data e ora di acquisizione, persona che l'ha raccolta, metodo, hash e ogni passaggio di custodia. L'hash aiuta a rilevare modifiche al file; non prova da solo chi lo ha creato o quando.

## Esercizio guidato in VM

1. Annota hostname e orario della VM.
2. Genera alcuni eventi innocui, ad esempio avvia e arresta un servizio di test.
3. Cerca l'intervallo temporale corrispondente nel journal o nel registro System.
4. Esporta o copia il log secondo le regole del laboratorio.
5. Calcola SHA-256 e registra il risultato in un verbale.
6. Copia il file in un'area di analisi e verifica l'hash della copia.

Non generare deliberatamente tentativi di accesso ripetuti su account reali. Se si studiano gli eventi di login, usare un account e una VM creati apposta.
## Spiegazione riga per riga dei comandi

Un **log** è una registrazione di eventi generata da sistema operativo, applicazioni o servizi. L'analisi dei log serve a ricostruire cosa è accaduto, quando e su quale componente. Non equivale a una prova completa: possono mancare eventi o esserci differenze di orario e configurazione.

### Eventi Windows
- `Get-WinEvent -LogName System -MaxEvents 50`: legge gli eventi più recenti dal registro `System`; `-LogName` sceglie il registro e `-MaxEvents 50` limita il numero a 50.
- `Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message`: mostra solo data/ora, identificativo, livello, origine e messaggio dell'evento, evitando colonne non necessarie.
- `Get-WinEvent -FilterHashtable @{ ... }`: filtra gli eventi direttamente alla fonte usando una tabella di criteri. `LogName = "Security"` seleziona il registro Sicurezza; `Id = 4625` seleziona gli accessi non riusciti; `StartTime = (Get-Date).AddHours(-24)` calcola il momento di 24 ore prima; `-MaxEvents 100` limita il risultato.
- `Get-WinEvent -LogName "Microsoft-Windows-PowerShell/Operational" -MaxEvents 50`: legge gli ultimi 50 eventi del registro operativo di PowerShell, se il registro è abilitato.
- `wevtutil epl System .\\System-lab.evtx`: usa l'utilità Windows Event Log; `epl` esporta un registro; `System` è il registro sorgente e il percorso finale è il file EVTX creato.
- `Get-FileHash .\\System-lab.evtx -Algorithm SHA256`: calcola l'impronta SHA-256 del file esportato per poter controllare in seguito se è cambiato.

### Log Linux
- `sudo journalctl -u ssh -n 100 --no-pager`: consulta il journal del servizio `ssh`; `-u` filtra per unità systemd, `-n 100` mostra gli ultimi 100 eventi e `--no-pager` stampa direttamente il risultato. Su alcune distribuzioni il servizio si chiama `sshd`.
- `sudo journalctl -u sshd -n 100 --no-pager`: esegue lo stesso controllo usando il nome alternativo dell'unità.
- `sudo journalctl -k -n 100 --no-pager`: mostra gli ultimi 100 messaggi del kernel; `-k` limita la ricerca ai messaggi del kernel.
- `sudo journalctl --since "2026-01-01 10:00:00" --until "2026-01-01 11:00:00" --no-pager`: restringe il journal all'intervallo indicato; `--since` è l'inizio e `--until` la fine. Sostituisci data e ora con quelle del caso di laboratorio.
- `sudo grep -i "failed password" /var/log/auth.log*`: cerca la frase nei log di autenticazione Debian/Ubuntu; `-i` ignora maiuscole/minuscole e `*` include i file ruotati che corrispondono al nome.
- `sudo grep -i "failed password" /var/log/secure*`: esegue la stessa ricerca nel percorso tradizionale usato da alcune distribuzioni RHEL-like.

### Filtrare e ordinare
- `Sort-Object TimeCreated`: ordina gli eventi Windows in base alla data di creazione, dal più vecchio al più recente.
- `Select-Object TimeCreated, Id, Message`: riduce l'output ai campi utili per la revisione.
- `sudo journalctl --since today --no-pager | grep -i "error"`: il journal fornisce gli eventi da oggi; la pipe `|` passa il testo a `grep`; `-i` cerca senza distinguere maiuscole e minuscole.
- `sudo journalctl --since today --no-pager | grep -ic "failed"`: come sopra, ma `-c` conta le righe corrispondenti invece di stamparle.

### Integrità delle evidenze
- `Get-FileHash .\\evidenza.log -Algorithm SHA256`: calcola l'hash del file Windows indicato.
- `sha256sum evidenza.log > evidenza.sha256`: calcola SHA-256 e redirige il risultato nel manifest; `>` crea o sovrascrive il manifest.
- `sha256sum -c evidenza.sha256`: legge il manifest e ricalcola l'hash per verificare se il file corrisponde al valore registrato.

**Nota forense:** conserva il file originale, registra chi lo ha raccolto, quando, da quale sistema e con quale procedura. Un hash corrispondente aiuta a verificare l'integrità, ma non dimostra da solo provenienza o autenticità.
