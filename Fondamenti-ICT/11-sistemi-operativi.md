# Amministrazione di base dei sistemi operativi

Questa scheda introduce attività quotidiane su Windows e Linux: identità, processi, servizi, spazio disco e aggiornamenti. I comandi che modificano il sistema vanno eseguiti solo su dispositivi che amministri.

## Identità e sistema

```powershell
# Utente corrente e gruppi
whoami
whoami /groups

# Informazioni su sistema operativo e build
Get-ComputerInfo |
  Select-Object WindowsProductName, WindowsVersion, OsBuildNumber
```

```bash
# Identità e gruppi correnti
whoami
id

# Distribuzione e kernel
cat /etc/os-release
uname -r
```

## File e spazio

```powershell
# Elenco file e cartelle
Get-ChildItem -LiteralPath <PERCORSO>

# Spazio disponibile
Get-Volume | Select-Object DriveLetter, Size, SizeRemaining
```

```bash
# Elenco dettagliato
ls -la <PERCORSO>

# Spazio filesystem e dimensione directory
df -h
du -sh <PERCORSO>
```

Prima di spostare o cancellare file, verifica il percorso e mantieni una copia dei dati importanti.

## Processi

```powershell
# Processi che usano più CPU
Get-Process | Sort-Object CPU -Descending |
  Select-Object -First 10 Id, ProcessName, CPU

# Dettagli di un processo
Get-Process -Id <PID>
```

```bash
# Processi e proprietari
ps aux

# Attività interattiva, se installato
top
```

Identifica programma e funzione prima di arrestare un processo. I processi di sistema potrebbero essere necessari anche se il loro nome non è familiare.

## Servizi

Un servizio è un processo gestito dal sistema operativo, spesso avviato automaticamente. Prima di cambiarne lo stato, verifica dipendenze e impatto.

```powershell
# Cerca un servizio
Get-Service -Name <NOME_SERVIZIO>

# Stato e modalità di avvio
Get-CimInstance Win32_Service |
  Where-Object Name -eq "<NOME_SERVIZIO>" |
  Select-Object Name, State, StartMode, PathName
```

```bash
# Stato e dettagli di un servizio systemd
systemctl status <NOME_SERVIZIO>

# Servizi che hanno segnalato un errore
systemctl --failed

# Log recente del servizio
sudo journalctl -u <NOME_SERVIZIO> -n 50 --no-pager
```

Su Linux, `systemctl stop` ferma il servizio in esecuzione, mentre `systemctl disable` ne modifica l'avvio automatico; sono azioni diverse. Usale solo dopo aver verificato il servizio.

## Aggiornamenti

Installa aggiornamenti da Windows Update o dal gestore pacchetti della distribuzione. Su Linux:

```bash
# Debian/Ubuntu
sudo apt update
apt list --upgradable
sudo apt upgrade
```

```bash
# Fedora/RHEL compatibili
sudo dnf check-update
sudo dnf upgrade
```

Leggi l'elenco dei pacchetti e le richieste di conferma; pianifica il riavvio per i sistemi in uso. Non interrompere gli aggiornamenti mentre il gestore pacchetti opera.

## Esercizio: scheda di un sistema

Per una VM didattica, registra hostname, sistema operativo, kernel/build, memoria, spazio libero e servizi in errore. Esegui gli stessi controlli dopo un aggiornamento e annota eventuali differenze.
