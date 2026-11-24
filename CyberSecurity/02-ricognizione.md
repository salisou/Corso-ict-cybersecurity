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
