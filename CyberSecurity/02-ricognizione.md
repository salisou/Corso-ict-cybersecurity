# Scansione e ricognizione

La ricognizione serve a documentare i sistemi e i servizi di un ambiente autorizzato. Ogni scansione invia traffico: eseguila soltanto su macchine proprie o su un intervallo assegnato esplicitamente per il laboratorio.

## Prima di iniziare

Definisci per iscritto:

1. Gli indirizzi IP o hostname autorizzati.
2. L'intervallo e l'orario consentiti.
3. Gli strumenti e i tipi di test consentiti.
4. Il punto di contatto in caso di anomalie.

Non sostituire `<RETE_LAB>` con la rete di casa, aziendale o pubblica. Un indirizzo CIDR come `192.168.56.0/24` descrive un blocco di indirizzi, ma il valore da usare deve corrispondere al tuo laboratorio.

## Ricognizione locale

```powershell
# Mostra le voci ARP note al computer
arp -a

# Mostra l'hostname e gli indirizzi configurati
hostname
ipconfig

# Risolvi un nome della tua VM
Resolve-DnsName <NOME_VM>
```

```bash
# Mostra le voci neighbor/ARP osservate
ip neigh

# Mostra hostname e indirizzi
hostname
ip addr
```

Le tabelle ARP e neighbor contengono solo dispositivi osservati di recente nella rete locale; non sono inventari completi.

## Nmap nel laboratorio

Installa Nmap dalla fonte ufficiale o dal gestore pacchetti della distribuzione. Prima controlla la versione:

```bash
nmap --version
```

### Rilevare gli host autorizzati

```bash
# Ping scan limitato alla sottorete del laboratorio
nmap -sn <RETE_LAB>/24
```

`-sn` tenta di individuare host attivi senza effettuare la normale scansione delle porte. Un host può non rispondere ai probe di rilevamento, quindi l'assenza di risposta non prova che sia spento.

### Verificare poche porte note

```bash
# Verifica solo le porte indicate su una VM di laboratorio
nmap -sT -p 22,80,443 <IP_TUA_VM>
```

`-sT` usa normali tentativi di connessione TCP. `-p` limita la prova alle porte specificate. Gli stati più comuni sono:

- `open`: un servizio accetta connessioni su quella porta.
- `closed`: l'host risponde, ma non c'è un servizio in ascolto.
- `filtered`: un filtro impedisce di determinare lo stato.

### Identificare i servizi con moderazione

Solo se previsto dall'autorizzazione del laboratorio:

```bash
# Prova a riconoscere il servizio sulle sole porte specificate
nmap -sT -sV --version-light -p 22,80,443 <IP_TUA_VM>
```

Il rilevamento di versione invia richieste applicative aggiuntive e può generare log. I risultati sono indizi, non una prova definitiva della versione o della configurazione.

### Salvare e confrontare i risultati

```bash
# Salva risultati leggibili, XML e grepable
nmap -sT -p 22,80,443 -oA risultati-vm-lab <IP_TUA_VM>

# Leggi il formato testuale
cat risultati-vm-lab.nmap
```

Conserva i risultati con data, target autorizzato, versione dello strumento e opzioni usate. I file possono contenere informazioni di rete: non pubblicarli senza autorizzazione.

## Interpretare senza saltare alle conclusioni

- Una porta aperta non prova una vulnerabilità.
- Un banner può essere nascosto, generico o alterato.
- Una porta filtrata non equivale a una porta sicura.
- La configurazione può variare in base a firewall, proxy, NAT e momento della scansione.
- Conferma i risultati con la configurazione del sistema e con chi amministra il servizio.

Per un servizio sulla tua VM, puoi confrontare la scansione con il comando locale:

```bash
sudo ss -tulpn
```

## Esercizio guidato

1. Avvia un servizio HTTP di prova su una VM che controlli:

   ```bash
   python3 -m http.server 8080 --bind 0.0.0.0
   ```

2. Da una seconda VM della rete interna, verifica esclusivamente il suo IP e la porta 8080:

   ```bash
   nmap -sT -p 8080 <IP_TUA_VM>
   ```

3. Registra lo stato della porta e confrontalo con `ss -tulpn` sul sistema che ospita il server.
4. Interrompi il server con `Ctrl+C` e ripeti il controllo.

Usa `0.0.0.0` soltanto nella rete interna del laboratorio: il servizio si mette in ascolto su tutte le interfacce IPv4 della VM.

## Regole di sicurezza

- Non usare opzioni di scansione aggressiva o script di exploit.
- Non aumentare la velocità per scansioni su reti condivise.
- Non provare credenziali o accessi ai servizi rilevati.
- Interrompi l'attività se osservi instabilità o traffico non previsto.
- Mantieni la scansione limitata agli IP esplicitamente autorizzati.
## Spiegazione riga per riga dei comandi

La ricognizione è la fase in cui si raccolgono informazioni tecniche sugli host e sui servizi **esclusivamente nel perimetro autorizzato**. Nmap genera traffico di rete: prima di eseguire una scansione devi conoscere IP, intervallo, orario e test autorizzati.

### Ricognizione locale
- `arp -a`: visualizza la cache ARP di Windows, cioè le associazioni IP-indirizzo MAC apprese dal computer nella rete locale.
- `hostname`: stampa il nome del computer, utile per identificare la macchina nel verbale del laboratorio.
- `ipconfig`: mostra indirizzi IP, subnet mask e gateway delle interfacce Windows.
- `Resolve-DnsName <NOME_VM>`: interroga il sistema DNS per risolvere il nome della VM; sostituisci il segnaposto con un nome realmente configurato nel laboratorio.
- `ip neigh`: su Linux mostra la tabella dei vicini, equivalente moderno delle informazioni ARP per IPv4 e NDP per IPv6.
- `ip addr`: elenca le interfacce Linux e gli indirizzi assegnati.

### Verificare Nmap
- `nmap --version`: mostra la versione installata e alcune informazioni di compilazione; serve a documentare lo strumento usato e verificare che sia disponibile.
- `nmap -sn <RETE_LAB>/24`: esegue il rilevamento degli host senza la scansione ordinaria delle porte; `-sn` disabilita la scansione delle porte e il target CIDR identifica la sottorete autorizzata. Il suffisso `/24` rappresenta la maschera di rete, ma va usato solo se corrisponde davvero al laboratorio.
- `nmap -sT -p 22,80,443 <IP_TUA_VM>`: `-sT` usa connessioni TCP complete; `-p` limita il controllo alle porte 22, 80 e 443; il segnaposto deve essere l'IP della VM autorizzata.
- `nmap -sT -sV --version-light -p 22,80,443 <IP_TUA_VM>`: oltre alla connessione TCP, `-sV` prova a identificare i servizi; `--version-light` usa un insieme meno intenso di probe; `-p` mantiene limitato l'elenco delle porte. Può generare richieste applicative e log.
- `nmap -sT -p 22,80,443 -oA risultati-vm-lab <IP_TUA_VM>`: esegue il controllo limitato e salva tre formati di output; `-oA` usa il nome base `risultati-vm-lab` per creare i file Nmap, XML e grepable.
- `cat risultati-vm-lab.nmap`: legge il report testuale prodotto da Nmap.

### Verifica locale dei servizi
- `sudo ss -tulpn`: `ss` mostra socket e porte; `-t` seleziona TCP, `-u` UDP, `-l` socket in ascolto, `-p` processi associati e `-n` indirizzi e porte numerici; `sudo` consente di vedere anche informazioni di processo protette. Usalo sulla VM che amministri.

### Esercizio con server HTTP
- `python3 -m http.server 8080 --bind 0.0.0.0`: avvia il server HTTP semplice incluso in Python; `-m http.server` esegue il modulo come programma, `8080` è la porta TCP, `--bind 0.0.0.0` lo mette in ascolto su tutte le interfacce IPv4 della VM. Per questo motivo va eseguito solo in una rete interna di laboratorio.
- `nmap -sT -p 8080 <IP_TUA_VM>`: dalla seconda VM controlla soltanto la porta 8080 dell'host indicato.
- `Ctrl+C`: interrompe il server in primo piano nel terminale.

**Interpretazione:** `open` indica che la porta accetta connessioni; `closed` che l'host risponde ma non ha un servizio in ascolto; `filtered` che un filtro impedisce di determinare lo stato. Nessuno di questi risultati, da solo, dimostra una vulnerabilità.
