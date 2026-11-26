# Windows Security

Questa guida usa PowerShell per controlli locali su Windows 10/11. Alcuni comandi richiedono un terminale elevato; usa privilegi amministrativi solo quando necessario e solo su sistemi che amministri.

## Account, gruppi e privilegi

```powershell
# Elenca gli account locali
Get-LocalUser

# Informazioni sull'identità corrente
whoami
whoami /groups
whoami /priv

# Membri del gruppo degli amministratori locali
Get-LocalGroupMember -Group "Administrators"

# Gruppi locali
Get-LocalGroup
```

Usa un account standard per le attività quotidiane. Un account amministratore può modificare configurazioni di sistema e dovrebbe essere impiegato solo per operazioni che lo richiedono.

Per verificare le policy password locali:

```powershell
net accounts
```

Il risultato dipende dalla versione e dalle policy gestite dal dominio. Su computer aziendali, non modificare le policy senza autorizzazione dell'amministratore.

## Aggiornamenti e Microsoft Defender

```powershell
# Versione e build di Windows
Get-ComputerInfo |
  Select-Object WindowsProductName, WindowsVersion, OsBuildNumber

# Verifica aggiornamenti tramite i componenti Windows disponibili
Get-Service -Name wuauserv

# Stato di Microsoft Defender, se il modulo è disponibile
Get-MpComputerStatus |
  Select-Object AntivirusEnabled, RealTimeProtectionEnabled,
    AntivirusSignatureLastUpdated
```

Mantieni Windows e le firme antimalware aggiornati tramite Windows Update o gli strumenti di gestione approvati dall'organizzazione. Non disattivare la protezione in tempo reale per eseguire esercizi.

## Processi e servizi

```powershell
# Processi che usano più CPU
Get-Process |
  Sort-Object CPU -Descending |
  Select-Object -First 15 Id, ProcessName, CPU

# Dettagli di un processo specifico
Get-CimInstance Win32_Process |
  Where-Object ProcessId -eq <PID> |
  Select-Object ProcessId, Name, ExecutablePath, CommandLine

# Servizi in esecuzione
Get-Service | Where-Object Status -eq "Running"

# Cerca un servizio specifico
Get-Service -Name <NOME_SERVIZIO>
```

Un processo sconosciuto non è automaticamente malevolo. Verifica nome, percorso, firma digitale, processo padre e contesto prima di intervenire.

```powershell
# Informazioni sulla firma digitale di un eseguibile locale
Get-AuthenticodeSignature -FilePath "C:\Percorso\programma.exe"
```

Non terminare servizi di sistema senza averne identificato funzione e impatto.

## Firewall di Windows

```powershell
# Stato dei profili firewall
Get-NetFirewallProfile |
  Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction

# Regole abilitate in ingresso
Get-NetFirewallRule -Direction Inbound -Enabled True |
  Select-Object DisplayName, Action, Profile
```

Non disabilitare il firewall per eseguire test. Se serve un servizio di esempio, consenti una singola porta limitando la regola al profilo e all'indirizzo del laboratorio.

```powershell
# Regola temporanea, limitata all'IP della VM autorizzata
New-NetFirewallRule -DisplayName "LAB TCP 8080" -Direction Inbound `
  -Protocol TCP -LocalPort 8080 -RemoteAddress <IP_TUA_VM> `
  -Profile Private -Action Allow

# Elimina la regola al termine
Remove-NetFirewallRule -DisplayName "LAB TCP 8080"
```

Controlla che il profilo attivo sia quello previsto e che `<IP_TUA_VM>` sia l'indirizzo corretto. Non usare questa regola su una rete pubblica.

## Connessioni e porte

```powershell
# Connessioni TCP stabilite
Get-NetTCPConnection -State Established |
  Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, OwningProcess

# Porte TCP in ascolto
Get-NetTCPConnection -State Listen |
  Select-Object LocalAddress, LocalPort, OwningProcess

# Dettagli del processo che possiede la porta
Get-Process -Id <PID>

# Test di una singola porta sulla VM del laboratorio
Test-NetConnection <IP_TUA_VM> -Port 8080
```

Registra ora, indirizzo remoto e processo prima di classificare una connessione come anomala. Connessioni legittime a servizi cloud o aggiornamenti possono usare indirizzi variabili.

## Log di sicurezza

```powershell
# Ultimi eventi del registro System
Get-WinEvent -LogName System -MaxEvents 30 |
  Select-Object TimeCreated, Id, LevelDisplayName, Message

# Ultimi tentativi di accesso non riusciti (evento 4625)
Get-WinEvent -FilterHashtable @{
  LogName = "Security"
  Id = 4625
  StartTime = (Get-Date).AddDays(-1)
} -MaxEvents 50
```

La raccolta degli eventi di sicurezza dipende dalla policy di audit e dai permessi. Un singolo evento 4625 non dimostra un attacco: controlla ora, account, origine e ricorrenza.

## Esercizio guidato: inventario locale

1. Esegui `whoami /all` e annota l'account e i gruppi.
2. Elenca le porte in ascolto con `Get-NetTCPConnection`.
3. Per un PID specifico, individua processo e percorso.
4. Controlla stato del firewall e del servizio associato.
5. Ripeti il controllo dopo aver avviato e fermato un servizio di test nella tua VM.

Salva solo i dati necessari e non includere account, indirizzi o percorsi sensibili in report pubblici.
## Spiegazione riga per riga dei comandi

Questa guida usa PowerShell per verificare la configurazione di Windows. I comandi di inventario leggono informazioni; quelli che creano o rimuovono regole firewall modificano invece la configurazione e vanno eseguiti solo nella VM autorizzata.

### Account e privilegi
- `Get-LocalUser`: elenca gli account locali configurati sul computer.
- `whoami`: mostra il nome dell'account che sta eseguendo la sessione.
- `whoami /groups`: elenca i gruppi di sicurezza dell'account; serve a capire a quali autorizzazioni può appartenere.
- `whoami /priv`: mostra i privilegi assegnati al token di sicurezza corrente.
- `Get-LocalGroupMember -Group "Administrators"`: elenca i membri del gruppo locale degli amministratori; il parametro `-Group` indica il gruppo da esaminare.
- `Get-LocalGroup`: elenca i gruppi locali.
- `net accounts`: mostra criteri locali relativi agli account e alle password; in un PC gestito da dominio le policy effettive possono essere diverse.

### Aggiornamenti e Defender
- `Get-ComputerInfo`: raccoglie informazioni sul computer e sul sistema operativo.
- `Select-Object WindowsProductName, WindowsVersion, OsBuildNumber`: mostra solo nome prodotto, versione e numero build.
- `Get-Service -Name wuauserv`: verifica lo stato del servizio Windows Update, identificato dal nome interno `wuauserv`.
- `Get-MpComputerStatus`: legge lo stato di Microsoft Defender, se il modulo e i relativi cmdlet sono disponibili.
- `Select-Object AntivirusEnabled, RealTimeProtectionEnabled, AntivirusSignatureLastUpdated`: mostra stato antivirus, protezione in tempo reale e data dell'ultimo aggiornamento delle firme.

### Processi e servizi
- `Get-Process`: elenca i processi attivi.
- `Sort-Object CPU -Descending`: ordina per tempo CPU in ordine decrescente; non è una percentuale istantanea di utilizzo.
- `Select-Object -First 15 Id, ProcessName, CPU`: limita l'output ai primi 15 processi e mostra identificativo, nome e CPU.
- `Get-CimInstance Win32_Process`: interroga la classe WMI/CIM che descrive i processi Windows.
- `Where-Object ProcessId -eq <PID>`: mantiene il processo con identificativo uguale al valore sostituito a `<PID>`; `-eq` significa “uguale”.
- `Select-Object ProcessId, Name, ExecutablePath, CommandLine`: mostra ID, nome, percorso dell'eseguibile e riga di avvio.
- `Get-Service`: elenca i servizi Windows.
- `Where-Object Status -eq "Running"`: filtra i servizi il cui stato è `Running`.
- `Get-Service -Name <NOME_SERVIZIO>`: cerca un servizio specifico; sostituisci il segnaposto con il nome interno corretto.
- `Get-AuthenticodeSignature -FilePath "C:\\Percorso\\programma.exe"`: verifica la firma Authenticode del file indicato; il percorso è un esempio da sostituire con un eseguibile locale da controllare.

### Firewall
- `Get-NetFirewallProfile`: legge la configurazione dei profili firewall.
- `Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction`: mostra nome del profilo, stato e azioni predefinite per traffico in ingresso e in uscita.
- `Get-NetFirewallRule -Direction Inbound -Enabled True`: elenca le regole abilitate che controllano il traffico in ingresso.
- `Select-Object DisplayName, Action, Profile`: mostra nome leggibile, azione (consenti/blocca) e profilo associato.
- `New-NetFirewallRule ...`: crea una nuova regola firewall. `-DisplayName` assegna un nome descrittivo; `-Direction Inbound` riguarda il traffico in ingresso; `-Protocol TCP` seleziona TCP; `-LocalPort 8080` limita la porta locale; `-RemoteAddress <IP_TUA_VM>` limita l'origine all'IP della VM; `-Profile Private` la applica al profilo privato; `-Action Allow` consente il traffico corrispondente.
- `Remove-NetFirewallRule -DisplayName "LAB TCP 8080"`: rimuove la regola con quel nome. Eseguilo solo dopo aver verificato che sia la regola temporanea creata per il laboratorio.

### Connessioni e porte
- `Get-NetTCPConnection -State Established`: elenca le connessioni TCP nello stato Established.
- `Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, OwningProcess`: mostra estremi locali/remoti e PID proprietario della connessione.
- `Get-NetTCPConnection -State Listen`: mostra le porte TCP in ascolto.
- `Get-Process -Id <PID>`: identifica il processo corrispondente al PID trovato; sostituisci il segnaposto con un numero reale.
- `Test-NetConnection <IP_TUA_VM> -Port 8080`: verifica la connettività TCP verso l'IP e la porta indicati; non esegue una scansione generale.

### Log
- `Get-WinEvent -LogName System -MaxEvents 30`: legge gli ultimi 30 eventi del registro System.
- `Select-Object TimeCreated, Id, LevelDisplayName, Message`: mostra data/ora, ID, livello e messaggio.
- `Get-WinEvent -FilterHashtable @{ ... }`: filtra gli eventi usando criteri strutturati; `LogName = "Security"` sceglie il registro, `Id = 4625` seleziona gli accessi non riusciti e `StartTime = (Get-Date).AddDays(-1)` imposta l'inizio a 24 ore prima; `-MaxEvents 50` limita i risultati.

Non considerare automaticamente malevolo un processo o un evento isolato: verifica percorso, firma, contesto, orario e ricorrenza.
