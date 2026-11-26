# Web Security

Questa scheda tratta controlli di base su applicazioni proprie o ambienti didattici deliberatamente predisposti. Non eseguire scansioni o test su servizi di terzi senza autorizzazione esplicita.

## HTTP con curl

```bash
# Richiedi solo gli header di una pagina di laboratorio
curl -I --connect-timeout 5 http://<IP_TUA_VM>:8080/

# Mostra header e corpo della risposta
curl -i --connect-timeout 5 http://<IP_TUA_VM>:8080/

# Salva la risposta in un file di laboratorio
curl --output risposta.html http://<IP_TUA_VM>:8080/
```

Codici HTTP frequenti:

- `200`: richiesta riuscita.
- `301` o `302`: redirect; controlla attentamente il valore di `Location`.
- `400`: richiesta non valida.
- `401`: autenticazione richiesta.
- `403`: richiesta rifiutata.
- `404`: risorsa non trovata.
- `500`: errore interno del server.

Un codice HTTP da solo non indica se l'applicazione è sicura. Analizza insieme header, contenuto, log e comportamento previsto.

Per un form di test della tua app, invia dati fittizi:

```bash
# Solo endpoint e account fittizi della tua applicazione di laboratorio
curl -i -X POST \
  -H "Content-Type: application/x-www-form-urlencoded" \
  --data "username=studente&password=esempio-non-reale" \
  http://<IP_TUA_VM>:8080/login
```

Non inserire password vere nella riga di comando: la cronologia della shell e i log dei processi possono conservarle.

## TLS e certificati

```bash
# Connessione TLS e informazioni sul certificato
openssl s_client -connect <HOST_LAB>:443 -servername <HOST_LAB>

# Controlla la risposta HTTPS e gli header
curl -I https://<HOST_LAB>

# Ispeziona un certificato già salvato nel laboratorio
openssl x509 -in cert.pem -text -noout
```

Non usare `curl -k` come soluzione abituale: disabilita la verifica del certificato e nasconde errori di fiducia. In un lab con certificato autofirmato, documenta il motivo del test e verifica il certificato tramite il canale previsto.

La lettura di un certificato mostra campi e date, ma da sola non verifica fiducia della catena, corrispondenza hostname o revoca.

## Header e cookie di sicurezza

Controlla se l'applicazione usa in modo appropriato:

- `Strict-Transport-Security` (HSTS), solo dopo aver verificato HTTPS.
- `Content-Security-Policy` (CSP), per ridurre l'esecuzione di contenuti non previsti.
- `X-Content-Type-Options: nosniff`.
- Cookie di sessione con attributi `Secure`, `HttpOnly` e `SameSite`.
- Redirect HTTP verso HTTPS e nessun dato sensibile negli URL.

Esempio per leggere gli header di risposta:

```bash
curl -sS -D - -o /dev/null http://<IP_TUA_VM>:8080/
```

L'assenza di un singolo header non dimostra automaticamente una vulnerabilità: considera tipo di app, architettura e requisiti.

## OWASP ZAP Baseline in laboratorio

ZAP Baseline esegue controlli passivi e può visitare URL collegate all'applicazione. Usalo solo contro un'applicazione di test isolata e controlla che redirect e link non portino fuori dal perimetro.

```bash
# Esempio Docker: target di test locale
docker run --rm -t ghcr.io/zaproxy/zaproxy:stable zap-baseline.py \
  -t http://host.docker.internal:8080
```

`host.docker.internal` è disponibile in molti ambienti Docker Desktop; su Linux la risoluzione può richiedere configurazione diversa. Verifica che il target sia davvero la tua app prima di avviare lo scanner. Il report contiene osservazioni da verificare manualmente e non equivale a una certificazione di sicurezza.

## Controlli di progettazione

- **Autenticazione**: MFA dove appropriato, sessioni invalidate al logout, messaggi di errore che non rivelino segreti.
- **Autorizzazione**: il server verifica i permessi per ogni operazione e risorsa, senza fidarsi dei soli controlli nell'interfaccia.
- **Validazione**: dati e tipi sono verificati lato server; gli errori non mostrano stack trace o dettagli interni.
- **Gestione segreti**: token e password non sono nel codice, nei log o nei repository.
- **Dipendenze**: versioni e aggiornamenti sono mantenuti e verificati con strumenti del progetto.
- **Logging**: si registrano eventi utili senza salvare password, token o dati personali non necessari.

## Esercizio guidato

1. Avvia una web app deliberatamente predisposta in una VM di laboratorio.
2. Usa `curl -I` e `curl -i` per registrare status code e header.
3. Verifica se HTTP redirige a HTTPS e controlla gli attributi dei cookie di test.
4. Esegui, se autorizzato, il solo baseline scan contro l'URL locale.
5. Classifica ogni rilievo come confermato, falso positivo o da approfondire e annota le prove.

Non tentare payload di exploit o accessi a dati di altri utenti. Per studiare vulnerabilità, usa un corso o una piattaforma didattica isolata e segui le istruzioni specifiche del relativo laboratorio.
## Spiegazione riga per riga dei comandi

Questa scheda usa strumenti client per osservare la risposta di un'applicazione web. Esegui le richieste soltanto verso la tua app o un target di laboratorio esplicitamente autorizzato.

### Richieste HTTP con curl
- `curl -I --connect-timeout 5 http://<IP_TUA_VM>:8080/`: `curl` effettua una richiesta HTTP; `-I` richiede gli header senza scaricare normalmente il corpo; `--connect-timeout 5` limita a cinque secondi il tempo per stabilire la connessione.
- `curl -i --connect-timeout 5 http://<IP_TUA_VM>:8080/`: `-i` include gli header nella risposta mostrata insieme al corpo.
- `curl --output risposta.html http://<IP_TUA_VM>:8080/`: scarica il corpo della risposta nel file indicato da `--output`.
- `curl -i -X POST ...`: `-X POST` imposta il metodo HTTP POST; `-H "Content-Type: ..."` invia l'header che dichiara il formato dei dati; `--data "..." ` invia i campi del form nel corpo della richiesta; l'URL finale indica l'endpoint. Nell'esempio usare esclusivamente credenziali fittizie.

### TLS e certificati
- `openssl s_client -connect <HOST_LAB>:443 -servername <HOST_LAB>`: apre una connessione TLS verso la porta 443; `-connect` indica host e porta; `-servername` invia il nome SNI, utile quando più siti condividono lo stesso server.
- `curl -I https://<HOST_LAB>`: richiede gli header della risorsa HTTPS.
- `openssl x509 -in cert.pem -text -noout`: legge il certificato X.509 dal file PEM; `-text` mostra i campi in forma leggibile e `-noout` evita di ristampare il certificato codificato.

### Header HTTP
- `curl -sS -D - -o /dev/null http://<IP_TUA_VM>:8080/`: `-sS` riduce i messaggi di avanzamento ma mostra gli errori; `-D -` stampa gli header di risposta sul terminale; `-o /dev/null` scarta il corpo su Linux; l'URL identifica il server di laboratorio. Su Windows il dispositivo di scarto ha un nome diverso, quindi questo esempio è Bash/Linux.

### OWASP ZAP Baseline
- `docker run --rm -t ghcr.io/zaproxy/zaproxy:stable zap-baseline.py -t http://host.docker.internal:8080`: `docker run` avvia un container; `--rm` lo elimina al termine; `-t` assegna un terminale; l'immagine indica la distribuzione ZAP; `zap-baseline.py` avvia il controllo passivo; `-t` seguito dall'URL specifica l'applicazione da esaminare. Verifica prima che l'URL punti alla tua app di test.

I risultati di ZAP sono avvisi da verificare: un report automatico non certifica che un'applicazione sia sicura o vulnerabile.
