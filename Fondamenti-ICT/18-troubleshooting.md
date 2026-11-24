# Troubleshooting ICT

Il troubleshooting è un metodo per identificare e correggere un problema in modo controllato. Evita modifiche casuali: raccogli i sintomi, formula ipotesi e prova una modifica alla volta.

## Metodo in sei passi

1. **Definisci il problema**: cosa non funziona, da quando e per chi?
2. **Raccogli dati**: messaggio di errore, orario, dispositivo, rete e modifiche recenti.
3. **Isola il livello**: hardware, sistema operativo, rete, servizio o applicazione.
4. **Formula un'ipotesi verificabile**.
5. **Esegui un test non distruttivo** e registra il risultato.
6. **Applica una modifica controllata**, verifica il funzionamento e documenta il ripristino.

Prima di riavviare o modificare una macchina operativa, controlla impatto, backup e procedure del servizio.

## Rete non raggiungibile

Controlla nell'ordine:

1. Stato dell'interfaccia e indirizzo IP.
2. Gateway e route.
3. DNS, se il problema riguarda un nome.
4. Raggiungibilità dell'host.
5. Porta e servizio previsto.

Windows:

```powershell
ipconfig /all
route print
Resolve-DnsName <NOME_HOST>
Test-NetConnection <IP_TUA_VM> -Port 8080
```

Linux:

```bash
ip addr
ip route
getent hosts <NOME_HOST>
curl -I --connect-timeout 5 http://<IP_TUA_VM>:8080/
```

Il fallimento di `ping` non dimostra da solo che un host sia spento: ICMP può essere filtrato.

## Problemi di spazio o prestazioni

```powershell
# Spazio disponibile
Get-Volume | Select-Object DriveLetter, Size, SizeRemaining

# Processi più attivi in CPU
Get-Process | Sort-Object CPU -Descending |
  Select-Object -First 10 ProcessName, Id, CPU
```

```bash
# Spazio filesystem e memoria
df -h
free -h

# Processi attivi
top
```

Non terminare processi o cancellare file di sistema per liberare risorse senza averne verificato funzione e impatto.

## Servizio che non parte

Windows:

```powershell
Get-Service -Name <NOME_SERVIZIO>
Get-WinEvent -LogName System -MaxEvents 30 |
  Select-Object TimeCreated, Id, Message
```

Linux:

```bash
systemctl status <NOME_SERVIZIO>
sudo journalctl -u <NOME_SERVIZIO> -n 100 --no-pager
```

Controlla prima stato, dipendenze, percorso del programma e log. Cambia una sola impostazione e ripeti il test.

## Hardware o periferica

1. Verifica alimentazione, cavi e collegamento fisico.
2. Controlla lo stato del dispositivo in Gestione dispositivi (Windows) o con `lsusb`/`lspci` (Linux).
3. Leggi eventuali codici errore e log.
4. Non aprire alimentatori o componenti elettrici; scollega il dispositivo prima degli interventi consentiti.

## Verbale di diagnosi

Registra:

| Campo | Esempio |
|---|---|
| Sistema | VM Ubuntu di laboratorio |
| Sintomo | Pagina locale non raggiungibile |
| Ora e contesto | Data/ora con fuso, rete usata |
| Test | Comando e risultato |
| Ipotesi | Servizio non in ascolto |
| Modifica | Avvio del servizio di test |
| Esito | Pagina raggiungibile, test ripetibile |

Evita di includere password, token o dati personali nei report.

## Esercizio guidato

Nella tua VM, avvia un server locale su `127.0.0.1:8080`, poi interrompilo. Per ogni stato:

1. Controlla se il processo esiste.
2. Controlla se la porta è in ascolto.
3. Prova la richiesta HTTP con `curl`.
4. Scrivi una diagnosi basata sulle osservazioni, senza cambiare firewall o rete.
