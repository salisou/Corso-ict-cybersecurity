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
