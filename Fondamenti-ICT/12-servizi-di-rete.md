# Protocolli e servizi di rete

Un protocollo definisce come dispositivi e applicazioni scambiano dati. Un servizio di rete è un programma che attende richieste su un indirizzo e una porta.

## Protocolli comuni

| Protocollo | Uso tipico | Note |
|---|---|---|
| DHCP | Assegna configurazione IP ai client | La configurazione può includere gateway e DNS. |
| DNS | Risolve nomi in indirizzi | Una risposta DNS non prova che il servizio sia raggiungibile. |
| HTTP/HTTPS | Pagine e API web | HTTPS protegge il trasporto se certificato e configurazione sono validi. |
| SSH | Amministrazione remota cifrata | Autorizza solo accessi previsti e usa credenziali protette. |
| SMB | Condivisione file Windows | Non esporre condivisioni a reti non fidate. |
| NTP | Sincronizzazione dell'orario | Orari coerenti aiutano log e diagnosi. |

Le porte più note (per esempio HTTPS su TCP 443) sono convenzioni, non garanzie: un servizio può essere configurato su una porta diversa.

## DNS e HTTP in pratica

```powershell
# Risolve il nome e mostra i record disponibili
Resolve-DnsName example.com

# Verifica una sola connessione TCP HTTPS
Test-NetConnection example.com -Port 443

# Legge gli header HTTP della tua app di laboratorio
curl.exe -I http://<IP_TUA_VM>:8080/
```

```bash
# Risoluzione tramite il sistema
getent hosts example.com

# Query DNS, se dig è installato
dig example.com

# Segue la catena di deleghe DNS per un nome di esempio
dig +trace example.com

# Richiesta HTTP alla sola app del laboratorio
curl -I --connect-timeout 5 http://<IP_TUA_VM>:8080/
```

`dig +trace` interroga in sequenza i server DNS per seguire le deleghe; è una consultazione informativa, non un test di sicurezza del dominio. Per diagnosticare, separa i passaggi: risoluzione nome, raggiungibilità del server, connessione alla porta e risposta dell'applicazione.

## SSH e file condivisi

```bash
# Connessione SSH a una macchina autorizzata
ssh <UTENTE>@<IP_TUA_VM>

# Copia un file verso una VM autorizzata
scp <FILE> <UTENTE>@<IP_TUA_VM>:<PERCORSO_DESTINAZIONE>
```

Verifica host, utente e percorso prima di trasferire file. Non copiare segreti in cartelle condivise non protette.

Su Windows, controlla eventuali connessioni SMB già aperte:

```powershell
Get-SmbConnection |
  Select-Object ServerName, ShareName, UserName, Dialect
```

## VLAN e segmentazione

Una VLAN separa logicamente dispositivi a livello di rete; il traffico tra VLAN richiede normalmente routing e policy. La configurazione si esegue su switch/router gestiti e deve seguire il piano di rete dell'organizzazione. Non cambiare VLAN o trunk su reti operative senza autorizzazione.

Per documentare la rete di un laboratorio, annota subnet, gateway, DNS e VLAN assegnati. Non dedurre la VLAN solo dall'indirizzo IP.

## Esercizio guidato

1. Risolvi il nome di una VM o di un servizio di test.
2. Verifica una sola porta prevista con `Test-NetConnection` o `curl`.
3. Confronta il risultato con il servizio in ascolto sulla VM (`ss -tulpn` o `Get-NetTCPConnection`).
4. Registra se il problema avviene nel DNS, nella rete, nella porta o nell'applicazione.

Esegui i test soltanto su host e servizi del tuo laboratorio.
