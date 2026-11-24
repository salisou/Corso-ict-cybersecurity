# Networking

Questa scheda introduce i controlli di rete utili per capire come un computer comunica. Esegui i test verso la tua macchina o una VM del laboratorio.

## Concetti essenziali

- **Indirizzo IP**: identifica un'interfaccia nella rete, ad esempio `192.168.56.10`.
- **Subnet mask/prefix**: definisce quali indirizzi appartengono alla stessa rete, ad esempio `/24`.
- **Gateway**: router a cui vengono inoltrati i pacchetti diretti fuori dalla rete locale.
- **DNS**: traduce un nome, come `lab.example`, in uno o più indirizzi IP.
- **Porta**: identifica un servizio di rete sul computer. TCP e UDP sono protocolli di trasporto distinti.

Un indirizzo `127.0.0.1` è il loopback: indica il computer stesso. Un servizio in ascolto solo su `127.0.0.1` non è normalmente raggiungibile dagli altri computer della rete.

## Configurazione locale in Windows

Apri PowerShell:

```powershell
# Indirizzi, gateway, DNS e dettagli degli adattatori
ipconfig /all

# Elenco degli adattatori e del loro stato
Get-NetAdapter

# Indirizzi IP configurati
Get-NetIPAddress |
  Select-Object InterfaceAlias, AddressFamily, IPAddress, PrefixLength

# Tabella di routing
route print
```

In `ipconfig /all`, cerca l'adattatore effettivamente in uso. L'indirizzo IPv4, il gateway predefinito e i server DNS aiutano a capire come è configurato il computer. Un indirizzo `169.254.x.x` spesso indica che il computer non ha ricevuto una configurazione DHCP.

Visualizza connessioni e processi locali:

```powershell
# Porte TCP in ascolto
Get-NetTCPConnection -State Listen |
  Select-Object LocalAddress, LocalPort, OwningProcess

# Mappa il PID di una porta al processo
Get-Process -Id <PID>

# Alternativa che mostra anche il PID
netstat -ano
```

## Configurazione locale in Linux

```bash
# Interfacce e indirizzi
ip addr

# Gateway e rotte
ip route

# Socket in ascolto
sudo ss -tulpn

# Stato dei resolver DNS (se supportato)
resolvectl status
```

La rotta `default` indica il gateway usato per le destinazioni non locali. In `ss`, un servizio associato a `127.0.0.1:PORTA` ascolta solo in loopback; `0.0.0.0:PORTA` indica che ascolta su tutte le interfacce IPv4.

## DNS

```powershell
# Risolvi un nome
Resolve-DnsName example.com

# Richiedi record a un DNS specifico
nslookup example.com 1.1.1.1

# Svuota la cache DNS locale
ipconfig /flushdns
```

```bash
# Interroga il resolver configurato
getent hosts example.com

# Se disponibile, esegui una query DNS dettagliata
dig example.com

# Elenca i server DNS configurati con systemd-resolved
resolvectl status
```

Tipi di record comuni:

- `A`: indirizzo IPv4.
- `AAAA`: indirizzo IPv6.
- `CNAME`: alias di un altro nome DNS.
- `MX`: server di posta del dominio.
- `TXT`: testo usato, tra gli altri scopi, per verifiche e configurazioni di posta.

Una risposta DNS dimostra che il nome è stato risolto, non che il servizio web o il computer sia raggiungibile.

## Verificare la connettività

```powershell
# Verifica se la VM risponde a ping
ping <IP_TUA_VM>

# Traccia il percorso di rete
tracert <IP_TUA_VM>

# Testa una porta TCP specifica senza scansione
Test-NetConnection <IP_TUA_VM> -Port 22
```

```bash
# Invia quattro richieste ICMP
ping -c 4 <IP_TUA_VM>

# Traccia il percorso, se tracepath è installato
tracepath <IP_TUA_VM>

# Verifica una connessione TCP applicativa
curl -I --connect-timeout 5 http://<IP_TUA_VM>:8080
```

`ping` usa ICMP; un timeout può dipendere dal firewall e non prova da solo che il computer sia spento. `Test-NetConnection -Port` verifica una singola porta TCP precisa. `tracert` e `tracepath` possono mostrare hop incompleti quando i router filtrano le risposte.

## Porte locali e processi

```powershell
# Trova il PID che ascolta sulla porta 8080
Get-NetTCPConnection -State Listen -LocalPort 8080 |
  Select-Object LocalAddress, LocalPort, OwningProcess

# Sostituisci il valore con il PID restituito
Get-Process -Id <PID>
```

```bash
# Trova il processo associato alla porta 8080
sudo ss -tulpn | grep ':8080'

# Alternativa, se lsof è installato
sudo lsof -iTCP:8080 -sTCP:LISTEN
```

Prima di modificare firewall o servizi, identifica quale programma usa la porta e perché. Non terminare processi sconosciuti.

## Esercizio guidato: server solo locale

Su una VM Linux di laboratorio:

```bash
# Avvia un server HTTP di test accessibile solo dalla VM stessa
python3 -m http.server 8080 --bind 127.0.0.1
```

In un secondo terminale della stessa VM:

```bash
# La richiesta locale dovrebbe ricevere una risposta
curl -I http://127.0.0.1:8080

# Controlla l'indirizzo su cui il server è in ascolto
ss -tulpn | grep ':8080'
```

Interrompi il server con `Ctrl+C`. La prova mostra la differenza tra un servizio locale e uno esposto su un'interfaccia di rete.

## Esercizio guidato: diagnosi ordinata

Per una VM del laboratorio che non risponde:

1. Controlla indirizzi e interfaccia (`ipconfig` oppure `ip addr`).
2. Controlla gateway e rete (`route print` oppure `ip route`).
3. Prova il ping senza interpretare il fallimento come prova definitiva.
4. Se conosci il servizio, testa solo la porta prevista.
5. Controlla il firewall e che il servizio sia effettivamente in ascolto.

Annota comando, ora, indirizzi sorgente/destinazione e risultato. Non cambiare più parametri insieme: una modifica alla volta rende la diagnosi ripetibile.
