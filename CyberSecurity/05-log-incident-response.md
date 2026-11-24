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
