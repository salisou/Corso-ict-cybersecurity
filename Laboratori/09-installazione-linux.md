# Installazione di Ubuntu e Kali Linux

Questa guida descrive due metodi: **macchine virtuali VirtualBox** e **WSL 2 su Windows**. VirtualBox offre maggior controllo sull'isolamento e sugli snapshot; WSL è più rapido da configurare e integra Linux con Windows.

> **Attenzione:** le procedure VirtualBox usano un disco virtuale nuovo. Non selezionare opzioni come *Erase disk*, *Install alongside* o partizionamento manuale sul computer fisico. WSL non richiede un ISO né il partizionamento del disco.

## Alternativa: installare Ubuntu e Kali con WSL 2

WSL (Windows Subsystem for Linux) consente di usare distribuzioni Linux da Windows senza gestire una VM tradizionale. La procedura seguente è per Windows 10 aggiornato (versione 2004, build 19041 o successiva) o Windows 11; i nomi delle distribuzioni disponibili possono variare.

### Installare WSL

1. Apri **PowerShell come amministratore**.
2. Installa WSL senza installare subito una distribuzione predefinita:

```powershell
wsl --install --no-distribution
```

3. Riavvia Windows se richiesto.
4. Controlla che la versione predefinita sia WSL 2:

```powershell
wsl --status
wsl --set-default-version 2
```

Se WSL era già installato, aggiorna il componente prima di procedere:

```powershell
wsl --update
```

### Installare Ubuntu

Visualizza prima i nomi esatti delle distribuzioni disponibili:

```powershell
wsl --list --online
```

Installa Ubuntu. Se `Ubuntu-24.04` non compare nell'elenco, sostituiscilo con il nome Ubuntu mostrato dal comando precedente:

```powershell
wsl --install --distribution Ubuntu-24.04
```

Avvia la distribuzione e verifica la versione di Linux:

```powershell
wsl --distribution Ubuntu-24.04
```

Al primo avvio viene chiesto di creare un utente Linux, se non già configurato.

### Installare Kali Linux

Controlla che Kali sia presente nell'elenco:

```powershell
wsl --list --online
```

Se compare con il nome `kali-linux`, installala così:

```powershell
wsl --install --distribution kali-linux
```

Al termine, apri Kali dal menu Start oppure esegui:

```powershell
wsl --distribution kali-linux
```

Se il nome non è disponibile o l'installazione non parte, consulta la procedura WSL aggiornata sul sito ufficiale Kali: <https://www.kali.org/docs/wsl/>.

### Verificare e aggiornare le distribuzioni

Da PowerShell:

```powershell
# Elenca distribuzioni installate e versione WSL
wsl --list --verbose

# Ferma tutte le distribuzioni WSL in esecuzione
wsl --shutdown
```

Dentro il terminale Ubuntu o Kali:

```bash
# Identifica distribuzione e kernel
cat /etc/os-release
uname -a

# Aggiorna l'elenco dei pacchetti e installa aggiornamenti
sudo apt update
sudo apt upgrade
```

Usa `sudo` solo per i comandi che richiedono privilegi amministrativi. Leggi il riepilogo del gestore pacchetti prima di confermare gli aggiornamenti.

### Backup di una distribuzione WSL

Da PowerShell, sostituisci il nome con quello riportato da `wsl --list --verbose`:

```powershell
wsl --export Ubuntu-24.04 "$env:USERPROFILE\Ubuntu-24.04-backup.tar"
wsl --export kali-linux "$env:USERPROFILE\kali-linux-backup.tar"
```

I file di backup possono contenere dati personali e credenziali: conservali in un luogo protetto. **Non eseguire `wsl --unregister <Distribuzione>` come comando di pulizia**: rimuove la distribuzione e i suoi dati locali.

### Limiti di WSL per il laboratorio

- WSL non equivale a una VM con una rete interna isolata: può comunicare con destinazioni raggiungibili secondo la configurazione di rete e firewall di Windows.
- Le distribuzioni WSL possono accedere ai file Windows, spesso sotto `/mnt/c`; evita di eseguire esercizi su file personali o di sistema.
- WSL non offre lo stesso controllo sull'hardware virtuale, sul boot e sugli snapshot di VirtualBox.
- Alcuni strumenti che richiedono accesso diretto a hardware, driver o funzioni specifiche del kernel possono non funzionare in WSL.
- Kali WSL fornisce un ambiente Linux Kali, ma non è una VM Kali completa con tutte le caratteristiche di un desktop installato.

Per esercizi che richiedono isolamento di rete o che potrebbero interagire con altri host, usa una VM con rete **Internal Network** e limita le prove ai sistemi del laboratorio autorizzato.

## 1. Preparazione

1. Installa VirtualBox dal sito ufficiale: <https://www.virtualbox.org/>.
2. Scarica le immagini esclusivamente dai siti ufficiali:
   - Ubuntu Desktop: <https://ubuntu.com/download/desktop>
   - Kali Linux: <https://www.kali.org/get-kali/>
3. Per Kali, se disponibile per la tua versione di VirtualBox, puoi scegliere l'immagine **prebuilt VirtualBox**. In alternativa scarica l'ISO dell'installer.
4. Verifica checksum e firma secondo le istruzioni della pagina ufficiale prima di usare l'immagine.
5. Controlla che la virtualizzazione hardware (Intel VT-x o AMD-V) sia abilitata nel firmware UEFI/BIOS, se richiesto.

Per una VM didattica con interfaccia grafica, una configurazione iniziale ragionevole è 2 CPU virtuali, 4 GB di RAM e disco virtuale da almeno 25 GB. Kali con strumenti aggiuntivi o Ubuntu con molte applicazioni può richiedere più spazio. Non assegnare alla VM tutta la memoria o tutti i processori del computer host.

## 2. Creare una VM vuota in VirtualBox

1. Seleziona **Nuova** e assegna un nome, ad esempio `Ubuntu-Lab` o `Kali-Lab`.
2. Scegli il tipo **Linux** e la versione appropriata (Ubuntu 64-bit o Debian 64-bit per Kali).
3. Assegna memoria e CPU in base alle risorse disponibili sul computer.
4. Crea un disco virtuale nuovo, in formato VDI, allocazione dinamica. Un disco dinamico cresce quando serve, ma non supera la dimensione massima scelta.
5. Apri **Impostazioni > Archiviazione** e collega l'ISO scaricata al lettore ottico virtuale.
6. In **Impostazioni > Rete**, lascia inizialmente l'adattatore su **NAT**. Questa modalità consente alla VM di accedere alla rete tramite l'host, ma non impedisce alla VM di avviare connessioni verso altri sistemi raggiungibili.
7. Avvia la VM.

I nomi dei menu possono cambiare leggermente tra versioni di VirtualBox.

## 3. Installare Ubuntu Desktop

1. Avvia la VM con l'ISO Ubuntu collegata.
2. Seleziona **Try or Install Ubuntu**, poi **Install Ubuntu**.
3. Scegli lingua, tastiera e rete. Per un corso è sufficiente la configurazione di rete predefinita.
4. Seleziona l'installazione interattiva e il set di applicazioni desiderato.
5. Quando richiesto il tipo di installazione, usa il disco virtuale vuoto creato per la VM. Verifica con attenzione che il disco selezionato sia quello virtuale, non quello del computer host.
6. Imposta fuso orario, nome utente e una password robusta. Non riutilizzare la password personale o quella di altri account.
7. Attendi la copia dei file e riavvia la VM quando richiesto.
8. Se il sistema riparte nell'installer, spegni la VM e rimuovi l'ISO da **Impostazioni > Archiviazione**.

## 4. Installare Kali Linux

### Opzione A: immagine VirtualBox prebuilt

1. Nella pagina ufficiale Kali scarica l'immagine VirtualBox adatta alla tua architettura.
2. Segui le istruzioni ufficiali per importare l'appliance o estrarre e aprire il file VM.
3. Controlla RAM, CPU e rete prima del primo avvio.
4. Imposta una password personale durante la configurazione iniziale, se richiesta, e non conservare credenziali predefinite.

Un'immagine prebuilt evita i passaggi dell'installer, ma è comunque opportuno verificare provenienza e hash secondo le istruzioni ufficiali.

### Opzione B: ISO installer

1. Crea una VM Debian 64-bit e collega l'ISO Kali.
2. Avvia la VM e scegli l'installazione grafica (*Graphical install*) o testuale.
3. Seleziona lingua, località, tastiera e configurazione di rete.
4. Imposta hostname, eventuale dominio di laboratorio, utente e password robusta.
5. Per il partizionamento, seleziona il disco virtuale nuovo. Conferma la scrittura delle modifiche solo dopo aver verificato il disco selezionato.
6. Scegli il desktop environment e i componenti proposti per l'uso didattico.
7. Installa il boot loader sul disco virtuale della VM quando richiesto.
8. Al termine, riavvia e rimuovi l'ISO dall'unità ottica virtuale se l'installer riparte.

## 5. Primo avvio e aggiornamenti

In Ubuntu o Kali, apri un terminale e aggiorna gli indici dei pacchetti e il sistema:

```bash
# Aggiorna l'elenco dei pacchetti disponibili
sudo apt update

# Visualizza gli aggiornamenti disponibili
apt list --upgradable

# Installa gli aggiornamenti
sudo apt upgrade
```

In Kali è comune usare `sudo apt full-upgrade` per aggiornamenti che possono richiedere modifiche alle dipendenze. Leggi il riepilogo prima di confermare. Non interrompere l'aggiornamento mentre il gestore pacchetti sta modificando il sistema.

Controlla la versione installata:

```bash
# Distribuzione
cat /etc/os-release

# Kernel e architettura
uname -a

# Spazio disponibile sui filesystem
df -h
```

Riavvia dopo aggiornamenti importanti, in particolare quando viene aggiornato il kernel:

```bash
sudo reboot
```

## 6. Rete isolata per gli esercizi

- **NAT**: scelta iniziale più semplice; la VM può uscire verso Internet e avviare connessioni verso destinazioni raggiungibili. Non è un isolamento completo: limita soprattutto le connessioni in ingresso non richieste verso la VM.
- **Host-only**: crea una rete tra host e VM senza accesso diretto a Internet; utile per esercizi tra VM configurate sulla stessa rete host-only.
- **Internal Network**: collega solo le VM sulla stessa rete interna; è una buona scelta per un laboratorio isolato.
- **Bridged Adapter**: collega la VM direttamente alla rete fisica. Evitalo per le esercitazioni di sicurezza, a meno che non sia espressamente necessario e autorizzato.

Per un laboratorio con Ubuntu e Kali:

1. Spegni entrambe le VM.
2. In VirtualBox assegna a ciascuna un secondo adattatore di tipo **Internal Network** con lo stesso nome, ad esempio `cyber-lab`.
3. Lascia l'adattatore NAT solo se serve installare aggiornamenti.
4. Avvia le VM e controlla gli indirizzi assegnati:

```bash
ip addr
ip route
```

Se l'interfaccia interna non riceve un indirizzo automaticamente, configura indirizzi di laboratorio distinti nella stessa sottorete, evitando indirizzi già usati. Verifica la connettività soltanto tra le tue VM:

```bash
ping -c 4 <IP_ALTRA_VM_DEL_LAB>
```

## 7. Snapshot e ripristino

Prima di installare strumenti o svolgere un esercizio:

1. Spegni la VM o portala in uno stato stabile.
2. In VirtualBox apri **Snapshot** e crea uno snapshot, per esempio `installazione-pulita`.
3. Dopo l'esercizio, ripristina lo snapshot se vuoi tornare allo stato iniziale.

Gli snapshot sono utili per esercizi didattici, ma non sostituiscono backup indipendenti dei dati importanti.

## Risoluzione dei problemi

| Problema | Controllo |
|---|---|
| VM a 64 bit non disponibile | Verifica VT-x/AMD-V nel firmware e che non sia disabilitato da altre impostazioni di virtualizzazione. |
| Schermata nera o avvio bloccato | Verifica l'integrità dell'ISO, il tipo di sistema scelto e le risorse assegnate. |
| Installer riparte a ogni riavvio | Rimuovi l'ISO dall'unità ottica virtuale. |
| Nessuna connessione Internet nella VM | Controlla che l'adattatore NAT sia abilitato e che il sistema host sia online. |
| Disco virtuale pieno | Controlla `df -h` nel guest e la dimensione del disco virtuale in VirtualBox. |

Per errori specifici, annota il messaggio completo e la versione di VirtualBox e dell'immagine, evitando di pubblicare credenziali o dati personali.
