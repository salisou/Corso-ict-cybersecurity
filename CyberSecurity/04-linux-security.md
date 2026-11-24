# Linux Security

Questi comandi sono pensati per distribuzioni Linux di laboratorio.

Prima di iniziare, distinguere tre tipi di informazione:

- **Identita**: quale utente e quali privilegi usa il processo.
- **Autorizzazione**: quali permessi, ACL o regole SELinux/AppArmor consentono un'azione.
- **Contesto**: kernel, distribuzione, servizi, rete, pacchetti e configurazione installata.

L'elevazione dei privilegi con `sudo` va usata solo quando necessario. Non avviare una shell amministrativa e tenerla aperta per intere sessioni se basta eseguire un singolo comando con privilegi.

## Informazioni di sistema

```bash
# Utente, kernel e distribuzione
whoami
uname -a
cat /etc/os-release

# Distribuzione e release su sistemi con systemd
hostnamectl

# Architettura della CPU
uname -m

# Lista degli utenti con una shell interattiva
getent passwd | grep -E '/(bash|zsh|sh)$'
```

`whoami` mostra l'identita effettiva della shell. Su `uname -a`, le parti principali sono kernel, hostname e architettura. `/etc/os-release` identifica nome e versione della distribuzione.

Esempio di interpretazione:

- `Linux labvm 6.8.0-... x86_64`: il sistema usa kernel 6.8 su architettura a 64 bit.
- `PRETTY_NAME="Ubuntu 24.04 LTS"`: la distribuzione e Ubuntu 24.04.

## Utenti, gruppi e privilegi

```bash
# Informazioni sull'utente corrente
id

# Informazioni su un utente specifico
id <UTENTE>

# Utenti presenti nel sistema
cut -d: -f1 /etc/passwd

# Gruppi presenti nel sistema
cut -d: -f1 /etc/group

# Membri di un gruppo
getent group <GRUPPO>

# Ultimi accessi
last -n 20

# Sessioni correnti
who
```

`id` mostra UID, GID principale e gruppi supplementari. Un UID diverso da `0` indica normalmente un utente non privilegiato; l'utente `root` ha UID `0`.

Esempio pratico:

```text
uid=1001(alice) gid=1001(alice) gruppi=1001(alice),27(sudo)
```

Alice usa l'utente `alice`, appartiene al gruppo principale `alice` e al gruppo `sudo`. La sola appartenenza al gruppo sudo permette normalmente di richiedere privilegi con autenticazione.

## Permessi

I permessi Linux decidono **chi può leggere, modificare o eseguire un file** e **chi può entrare o creare elementi in una directory**. Servono a proteggere dati, separare gli utenti e limitare l'impatto di errori o processi compromessi. Una buona regola è concedere solo l'accesso necessario.

Ogni file ha un proprietario e un gruppo proprietario. I permessi si applicano a tre classi:

1. **u (user)**: il proprietario del file.
2. **g (group)**: gli utenti appartenenti al gruppo proprietario.
3. **o (others)**: tutti gli altri utenti.

Con i permessi standard, il sistema controlla la classe del proprietario; se l'utente non è il proprietario, controlla il gruppo; altrimenti usa la classe "altri". Le classi non si sommano: avere `r` nella classe gruppo non dà accesso se il controllo applicabile è quello del proprietario. Le ACL aggiungono regole e una maschera che possono cambiare il calcolo effettivo.

Un output come `-rw-r-----` si legge così:

```text
- rw- r-- ---
|  |   |   |
|  |   |   +-- altri: nessun permesso
|  |   +------ gruppo: lettura
|  +---------- proprietario: lettura e scrittura
+------------- tipo: - file, d directory, l collegamento simbolico
```

Le lettere sono `r` (read, lettura), `w` (write, scrittura) e `x` (execute/search, esecuzione o attraversamento); `-` indica il permesso non concesso. Per un file `x` permette di eseguirlo se è un programma/script valido; per una directory `x` permette di attraversarla e accedere ai suoi elementi, se se ne conosce il nome.

```bash
# Permessi del file
ls -l <FILE>

# Permessi in formato ottale
stat -c '%A %a %n' <FILE>

# Legge le ACL estese
getfacl <FILE>

# Proprietario e gruppo
ls -ln <FILE>
```

Esempi di permessi:

- `755`: il proprietario può leggere, modificare ed eseguire; gli altri possono leggere ed eseguire.
- `640`: il proprietario può leggere/scrivere; il gruppo può leggere; gli altri non hanno accesso.
- `600`: solo il proprietario può leggere e modificare il file.
- `700`: solo il proprietario può accedere alla directory o eseguire il contenuto.

Tre cifre ottali corrispondono rispettivamente a proprietario, gruppo e altri:

| Valore | Lettura | Scrittura | Esecuzione |
|---|---:|---:|---:|
| `4` | Sì | No | No |
| `2` | No | Sì | No |
| `1` | No | No | Sì |
| `6` | Sì | Sì | No |
| `7` | Sì | Sì | Sì |

Esempio di modifica sicura:

```bash
# Imposta proprietario e gruppo
sudo chown alice:secure-team <FILE>

# Imposta permessi solo per proprietario e gruppo
sudo chmod 640 <FILE>

# Elenca il risultato
ls -l <FILE>
```

Nelle directory, il permesso di esecuzione permette di attraversare una directory, mentre quello di scrittura permette di creare o eliminare elementi. La lettura permette di elencare i nomi, ma non garantisce l'accesso ai singoli file.

Esempio di ricerca di permessi potenzialmente pericolosi:

```bash
# File con permesso setuid
find / -xdev -perm -4000 -type f -ls 2>/dev/null

# File con permesso setgid
find / -xdev -perm -2000 -type f -ls 2>/dev/null

# Directory modificabili da tutti gli utenti
find / -xdev -type d -perm -0002 -ls 2>/dev/null
```

`setuid` e `setgid` possono elevare temporaneamente i privilegi. Non significa necessariamente che siano dannosi, ma ogni occorrenza inattesa richiede verifica.

### Come interpretare `r`, `w` e `x` su una directory

I permessi di una directory non coincidono con quelli dei file che contiene:

- `r`: consente di elencare i nomi delle voci.
- `x`: consente di attraversare la directory. Senza `x`, i nomi potrebbero non essere accessibili anche se la directory è leggibile.
- `w` insieme a `x`: consente di creare, rinominare o rimuovere voci nella directory.

Per questo, la cancellazione di un file dipende principalmente dai permessi sulla directory che lo contiene, non dal permesso `w` del file stesso. Per raggiungere un file, l'utente deve poter attraversare **ogni directory** del percorso.

Esempio di prova sicura:

```bash
# Crea una piccola area di esercitazione nella home dell'utente corrente
mkdir -p ~/permessi-lab/condivisa
cd ~/permessi-lab
printf 'test didattico\n' > condivisa/nota.txt

# La directory è leggibile e attraversabile dal proprietario e dal gruppo
chmod 750 condivisa

# -d mostra la directory stessa; il secondo comando mostra il contenuto
ls -ld condivisa
ls -l condivisa

# Mostra permessi, proprietario e gruppo in forma compatta
stat -c '%A %a %U:%G %n' condivisa condivisa/nota.txt
```

Se una directory del percorso non concede `x`, i permessi permissivi del file finale non sono sufficienti per accedervi.

### Cambiare permessi con `chmod`

`chmod` accetta sia numeri ottali sia notazione simbolica. Nella forma simbolica, `u`, `g`, `o`, `a` indicano proprietario, gruppo, altri e tutti; `+` aggiunge, `-` rimuove e `=` imposta esattamente i bit specificati.

```bash
cd ~/permessi-lab
printf 'appunti privati\n' > appunti.txt
printf '#!/bin/sh\necho test di laboratorio\n' > script.sh

# Solo il proprietario legge e modifica (rw-------)
chmod 600 appunti.txt

# Proprietario legge/modifica, gruppo legge, altri non accedono (rw-r-----)
chmod 640 appunti.txt

# Aggiunge esecuzione al proprietario dello script
chmod u+x script.sh

# Script privato: proprietario può leggere, modificare ed eseguire
chmod 700 script.sh

# Rimuove la scrittura da gruppo e altri
chmod go-w appunti.txt

# Imposta esplicitamente: proprietario rw, nessun accesso per gruppo/altri
chmod u=rw,go= appunti.txt

# Controlla ogni risultato
ls -l appunti.txt
stat -c '%A %a %n' appunti.txt
```

I numeri si ottengono sommando i bit `r=4`, `w=2`, `x=1` per ciascuna classe. Per esempio `7 = 4+2+1 = rwx`, `5 = 4+1 = r-x`, `6 = 4+2 = rw-`.

| Valore | Permessi | Significato pratico |
|---:|:---:|---|
| `0` | `---` | nessun accesso |
| `1` | `--x` | esecuzione/attraversamento |
| `2` | `-w-` | scrittura senza lettura o esecuzione |
| `3` | `-wx` | scrittura ed esecuzione |
| `4` | `r--` | sola lettura |
| `5` | `r-x` | lettura ed esecuzione |
| `6` | `rw-` | lettura e scrittura |
| `7` | `rwx` | lettura, scrittura ed esecuzione |

Esempi comuni:

- `600`: file personale o chiave privata leggibile/modificabile solo dal proprietario.
- `640`: proprietario legge e modifica; gruppo legge; altri non hanno accesso.
- `644`: proprietario modifica; gruppo e altri leggono.
- `700`: directory o script accessibile solo al proprietario.
- `750`: proprietario ha accesso completo; il gruppo può leggere e attraversare; gli altri non accedono.
- `755`: file eseguibile o directory leggibile/attraversabile da tutti, ma modificabile solo dal proprietario.
- `777`: tutti possono leggere, scrivere ed eseguire/attraversare; in genere è troppo permissivo e non va usato come soluzione automatica.

La scelta dipende dal contenuto e dal servizio che lo usa. Per esempio, un file di configurazione con segreti non dovrebbe essere `644` se altri utenti del sistema non devono leggerlo.

> Evita `chmod -R` e `chown -R` finché non hai verificato il percorso: applicano i cambiamenti a tutto ciò che si trova sotto la directory. Non usarli su `/` o su directory di sistema per tentativi.

### Proprietario e gruppo

`chown` cambia il proprietario e, se indicato, il gruppo. `chgrp` cambia solo il gruppo. Prima di usare `sudo`, verifica che l'utente e il gruppo esistano e che la modifica sia necessaria.

```bash
# Controlla identità e gruppi dell'utente corrente
id

# Mostra proprietario numerico e gruppo
ls -ln ~/permessi-lab/appunti.txt

# Esempio: assegna il file all'utente e al gruppo previsti
sudo chown <UTENTE>:<GRUPPO> ~/permessi-lab/appunti.txt

# Oppure cambia solo il gruppo
sudo chgrp <GRUPPO> ~/permessi-lab/appunti.txt

# Verifica l'esito
stat -c '%U:%G %A %a %n' ~/permessi-lab/appunti.txt
```

Un percorso o un nome errato con `sudo` può cambiare la proprietà di file importanti e causare malfunzionamenti. Sostituisci sempre i segnaposto con nomi effettivamente presenti nel laboratorio.

### Umask: permessi dei nuovi file

La **umask** sottrae permessi dai valori massimi usati alla creazione: normalmente `666` per i file (nessuna esecuzione automatica) e `777` per le directory. Per esempio, `umask 027` produce comunemente file `640` e directory `750`.

```bash
# Visualizza la umask corrente
umask

# Imposta 027 solo per questa shell e i processi avviati da essa
umask 027

# Crea elementi di prova
touch ~/permessi-lab/nuovo-file
mkdir ~/permessi-lab/nuova-directory

# Verifica i permessi risultanti
stat -c '%A %a %n' ~/permessi-lab/nuovo-file ~/permessi-lab/nuova-directory
```

La umask non aggiunge permessi: li rimuove. L'impostazione fatta in un terminale non è necessariamente permanente; la configurazione persistente dipende dalla distribuzione e dall'account.

### Bit speciali

La notazione ottale può avere una quarta cifra, prima dei permessi normali:

- **setuid (`4xxx`)** su un eseguibile: il processo può usare l'identità effettiva del proprietario del file. Se il proprietario è `root`, il programma opera con privilegi elevati; è una configurazione sensibile. Linux ignora il setuid sugli script in molte configurazioni: non usarlo per elevare script.
- **setgid (`2xxx`)** su un eseguibile: il processo può usare il gruppo effettivo del file. Su una directory, fa ereditare il gruppo della directory ai nuovi elementi.
- **sticky (`1xxx`)** su una directory condivisa e scrivibile: limita la rimozione/rinomina delle voci, di norma, al proprietario della voce, al proprietario della directory o all'amministratore. `/tmp` è un esempio comune.

Per esempio, `2770` su una directory significa setgid (`2`) più `770` (`rwxrwx---`). Ecco come applicarlo a una directory di laboratorio:

```bash
# Crea la cartella di progetto
mkdir -p ~/permessi-lab/progetto

# Assegna il gruppo di progetto (sostituiscilo con un gruppo esistente)
sudo chgrp <GRUPPO> ~/permessi-lab/progetto

# Proprietario e gruppo hanno accesso completo; setgid fa ereditare il gruppo
chmod 2770 ~/permessi-lab/progetto

# Controlla il bit e il gruppo
stat -c '%A %a %U:%G %n' ~/permessi-lab/progetto
```

I collaboratori devono appartenere al gruppo indicato. Se il gruppo o l'appartenenza non sono corretti, i collaboratori potrebbero non accedere ai file. Per controllare i bit speciali senza crearli:

```bash
# /tmp normalmente usa sticky bit; controlla la configurazione di questa macchina
stat -c '%A %a %U:%G %n' /tmp

# Cerca file setuid e setgid sul filesystem corrente
find / -xdev -perm -4000 -type f -ls 2>/dev/null
find / -xdev -perm -2000 -type f -ls 2>/dev/null
```

Un risultato non è automaticamente una vulnerabilità: alcuni programmi li usano legittimamente. Verifica proprietario, pacchetto e necessità prima di intervenire; non rimuovere bit speciali dai file di sistema senza capirne l'impatto.

### ACL

Le ACL consentono permessi specifici per utenti o gruppi oltre alle normali tre classi.

```bash
# Assegna lettura a un utente specifico
setfacl -m u:<UTENTE>:r <FILE>

# Assegna scrittura al gruppo
setfacl -m g:<GRUPPO>:rw <FILE>

# Rimuove tutte le ACL aggiuntive
setfacl -b <FILE>

# Controlla il risultato
getfacl <FILE>
```

Un ACL aggiuntivo può comparire come `+` nell'output di `ls -l`. L'uso di `chmod` può modificare la maschera ACL, quindi ricontrolla i permessi dopo operazioni sul file.

Un caso d'uso: il file appartiene ad `alice`, il gruppo proprietario è `team`, ma soltanto `anna` deve poterlo leggere. Una ACL può dare accesso ad Anna senza cambiare il gruppo del file:

```bash
# Sostituisci i nomi con utenti e gruppi esistenti nel laboratorio
setfacl -m u:<UTENTE>:r <FILE>
getfacl <FILE>
```

In `getfacl`, una riga come `user:anna:r--` mostra una voce ACL per Anna. La **mask** limita i permessi effettivi delle voci di gruppo e degli utenti/gruppi nominati; perciò controlla anche le annotazioni `effective:` nell'output. Il simbolo `+` accanto ai permessi in `ls -l` segnala ACL estese.

Una ACL predefinita su una directory viene usata come base per i nuovi elementi creati al suo interno:

```bash
# Imposta una ACL predefinita per il gruppo di progetto
setfacl -d -m g:<GRUPPO>:rwx ~/permessi-lab/condivisa

# Crea un file e verifica i permessi effettivi ereditati
touch ~/permessi-lab/condivisa/nuovo.txt
getfacl ~/permessi-lab/condivisa/nuovo.txt
```

Per rimuovere le ACL estese di accesso da un file usa `setfacl -b <FILE>`; per rimuovere le ACL predefinite da una directory usa `setfacl -k <DIRECTORY>`. Dopo `chmod` o `setfacl`, ricontrolla sempre con `getfacl`, perché la maschera può cambiare i permessi effettivi.

### Esercizio riepilogativo

```bash
cd ~/permessi-lab

# File privato: proprietario soltanto
printf 'dati fittizi\n' > privato.txt
chmod 600 privato.txt
stat -c '%A %a %U:%G %n' privato.txt

# File pubblicabile: lettura per tutti, scrittura solo per il proprietario
printf 'appunti non sensibili\n' > appunti-pubblici.txt
chmod 644 appunti-pubblici.txt
stat -c '%A %a %U:%G %n' appunti-pubblici.txt

# Directory privata
mkdir -p privata
chmod 700 privata
stat -c '%A %a %U:%G %n' privata
```

Per ogni risultato, spiega quali azioni possono fare proprietario, gruppo e altri. Non usare dati reali o account di altre persone per gli esercizi.

## Attività, processi e cron

```bash
# Processi con PID e utente
ps aux

# Albero dei processi
ps -ef --forest

# Processo con un nome specifico
pgrep -a <NOME_PROCESSO>

# Informazioni su un PID
ps -p <PID> -o pid,user,stat,etime,cmd

# File aperti da un processo
sudo lsof -p <PID>

# Attività pianificate per l'utente
crontab -l

# Sistema di pianificazione globale
sudo ls -l /etc/cron*
```

Esempio di analisi:

```bash
# Individua un processo sospetto
pgrep -a python

# Verifica il percorso dell'eseguibile
readlink -f /proc/<PID>/exe

# Leggi gli argomenti e il working directory
sudo cat /proc/<PID>/cmdline | tr '\0' ' '
sudo readlink -f /proc/<PID>/cwd
```

Stati comuni di `ps`:

- `R`: in esecuzione.
- `S`: in attesa interrompibile.
- `D`: attesa non interrompibile.
- `Z`: zombie, non ha processo eseguibile ma conserva una voce nella tabella dei processi.
- `T`: fermato.

## Processi e rete

```bash
# Processi con PID e utente
ps aux

# Porte in ascolto
sudo ss -tulpn

# Socket TCP collegati
sudo ss -tnp

# Socket UDP collegati
sudo ss -unp

# Mappa una porta al processo
sudo lsof -iTCP:<PORTA> -sTCP:LISTEN

# Verifica integrità dei pacchetti installati su Debian/Ubuntu
debsums -s
```

Esempio di lettura:

```text
tcp LISTEN 0 4096 127.0.0.1:8080 0.0.0.0:* users:(("python3",pid=1234,fd=3))
```

Significa che `python3` con PID `1234` ascolta sulla porta TCP `8080` solo sull'indirizzo locale `127.0.0.1`, quindi non è raggiungibile direttamente dalla rete esterna.

Comandi di rete autorizzati nel laboratorio:

```bash
# Interfacce e indirizzi
ip addr

# Tabelle di routing
ip route

# Regole DNS configurate
cat /etc/resolv.conf

# Connettività verso una VM di laboratorio
ping -c 4 <IP_TUA_VM>

# Percorso verso una destinazione
tracepath <IP_TUA_VM>
```

## Firewall

Le distribuzioni possono usare `ufw`, `nftables` o `firewalld`. Prima di modificare le regole verifica quale sistema è effettivamente attivo.

```bash
# Stato di ufw
sudo ufw status verbose

# Stato di firewalld
sudo firewall-cmd --state
sudo firewall-cmd --list-all

# Regole nftables
sudo nft list ruleset

# Regole della tabella filter di iptables, senza risolvere nomi
sudo iptables -L -v -n
```

`iptables -L` mostra la tabella filter predefinita, non necessariamente tutte le tabelle o l'intero ruleset del sistema. Alcune distribuzioni usano nftables o un frontend come UFW/firewalld.

Esempio locale con `ufw`:

```bash
# Consenti SSH solo dalla rete di laboratorio
sudo ufw allow from <RETE_LAB>/24 to any port 22 proto tcp

# Controlla le regole
sudo ufw status numbered

# NON usare su sistemi remoti senza verifica:
# sudo ufw disable
```

Non disabilitare il firewall remoto senza un controllo alternativo. In caso di modifica errata potresti perdere completamente l'accesso alla macchina.

## Servizi e avvio

```bash
# Stato di un servizio
systemctl status <SERVIZIO>

# Elenca i servizi falliti
systemctl --failed

# Elenca i servizi abilitati all'avvio
systemctl list-unit-files --type=service --state=enabled

# Disattiva temporaneamente un servizio solo per diagnosi
sudo systemctl stop <SERVIZIO>

# Riavvia il servizio
sudo systemctl start <SERVIZIO>

# Mantiene le modifiche permanenti disabilitando l'avvio
sudo systemctl disable <SERVIZIO>
```

`stop` modifica lo stato corrente. `disable` modifica l'avvio automatico. Eseguire prima la diagnosi con `status` e controllare i log.

```bash
# Log del servizio
sudo journalctl -u <SERVIZIO> -n 100 --no-pager

# Log dall'avvio precedente
sudo journalctl -b -1 --no-pager
```

## Sicurezza SSH

I comandi seguenti descrivono controlli e configurazioni da usare su una macchina di laboratorio. Effettua sempre un backup della configurazione prima di modificarla.

```bash
# Configurazione efficace
sudo sshd -T

# Verifica il file principale
sudo grep -Ev '^[[:space:]]*(#|$)' /etc/ssh/sshd_config

# Backup della configurazione
sudo cp -a /etc/ssh/sshd_config /etc/ssh/sshd_config.backup

# Verifica della sintassi dopo una modifica
sudo sshd -t

# Riavvia SSH solo dopo una verifica positiva
sudo systemctl restart ssh
```

Controlli raccomandati nel file `/etc/ssh/sshd_config`:

```text
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
X11Forwarding no
AllowUsers alice lab-user
```

Prima di disabilitare l'autenticazione tramite password:

1. Crea una coppia di chiavi sul client.
2. Copia la chiave pubblica sul server con `ssh-copy-id`.
3. Testa una nuova connessione in una seconda sessione.
4. Mantieni una console o una sessione esistente aperta come rollback.

```bash
# Dal client
ssh-keygen -t ed25519 -f ~/.ssh/lab_ed25519
ssh-copy-id -i ~/.ssh/lab_ed25519.pub <UTENTE>@<IP_TUA_VM>
ssh -i ~/.ssh/lab_ed25519 <UTENTE>@<IP_TUA_VM>
```

## Integrita dei pacchetti e dei file

Su Debian e Ubuntu:

```bash
# Verifica i file controllati dai pacchetti
debsums -s

# Mostra le firme dei pacchetti
debsums -c

# Verifica se un singolo eseguibile appartiene a un pacchetto
dpkg -S /usr/bin/<COMANDO>
```

Su RHEL, Rocky Linux e Fedora:

```bash
# Verifica tutti i file installati dai pacchetti RPM
sudo rpm -Va

# Informazioni sul pacchetto che ha installato un file
rpm -qf /usr/bin/<COMANDO>
```

`rpm -Va` può segnalare anche modifiche legittime effettuate dalla configurazione locale. Analizza il significato delle modifiche, non considerare ogni riga come prova di compromissione.

## Aggiornamenti

```bash
# Debian/Ubuntu
sudo apt update
sudo apt list --upgradable
sudo apt full-upgrade

# RHEL/Fedora
sudo dnf check-update
sudo dnf upgrade --refresh

# Distribuzioni con pacchetti Snap
sudo snap refresh
```

Prima degli aggiornamenti, registrare kernel e pacchetti installati:

```bash
# Pacchetti Debian/Ubuntu
dpkg -l

# Pacchetti RHEL/Fedora
rpm -qa

# Versione dei pacchetti critici
dpkg -l openssh-server sudo
rpm -q openssh-server sudo
```

## Log essenziali

```bash
# Autenticazioni e privilegi
sudo journalctl _COMM=sudo -n 100
sudo journalctl _COMM=sshd -n 100

# Accessi SSH falliti
sudo journalctl _COMM=sshd | grep -i "failed"

# Log tradizionali
sudo tail -n 100 /var/log/auth.log
sudo tail -n 100 /var/log/syslog
```

Su distribuzioni senza `journalctl` o con log tradizionali, controlla anche `/var/log/secure` su RHEL/Rocky/Fedora.

## Esercizi guidati

### Esercizio 1: individuare una porta

1. Avvia un servizio nella VM di laboratorio sulla porta 8080.
2. Esegui `sudo ss -tulpn`.
3. Identifica PID, utente ed eseguibile associati.
4. Interrompi il servizio e verifica che la porta sia scomparsa.

Esempio:

```bash
python3 -m http.server 8080 --bind 127.0.0.1
sudo ss -tulpn | grep 8080
```

`--bind 127.0.0.1` limita il server al solo localhost ed è un comportamento più sicuro per i primi test.

### Esercizio 2: correggere i permessi

1. Crea un file di esempio.
2. Impostalo inizialmente con permessi troppo aperti.
3. Analizza il rischio.
4. Riduci i permessi.
5. Confronta i risultati con `ls -l` e `stat`.

```bash
touch laboratorio.txt
echo "Dato sensibile di laboratorio" > laboratorio.txt
chmod 777 laboratorio.txt
ls -l laboratorio.txt
chmod 600 laboratorio.txt
ls -l laboratorio.txt
```

Risultato atteso dopo il secondo comando: `-rw-------`, cioè sola lettura e scrittura per il proprietario.

### Esercizio 3: verifica di un processo

1. Avvia un processo controllato.
2. Ottieni il PID.
3. Verifica comando, eseguibile e working directory.
4. Termina il processo con il PID corretto.

```bash
python3 -m http.server 8080 --bind 127.0.0.1 &
PID=$!
ps -p "$PID" -o pid,user,stat,cmd
readlink -f "/proc/$PID/exe"
readlink -f "/proc/$PID/cwd"
kill "$PID"
```

Usa sempre il PID raccolto dal tuo esercizio e non terminare processi a caso.
