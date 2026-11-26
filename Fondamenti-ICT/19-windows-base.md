# Windows - Prompt dei comandi (cmd.exe)

Questa guida introduce `cmd.exe`, il Prompt dei comandi. I comandi e le virgolette di questa pagina non sono intercambiabili con PowerShell. Per PowerShell consulta [20-powershell-base.md](20-powershell-base.md).

## Preparare un laboratorio

Apri il Prompt dei comandi, senza privilegi amministrativi:

```cmd
if not exist "%TEMP%\CorsoCMD" mkdir "%TEMP%\CorsoCMD"
cd /d "%TEMP%\CorsoCMD"
cd
```

`%TEMP%` indica la cartella temporanea dell'utente. `cd /d` cambia percorso e, se serve, unità. Le virgolette proteggono percorsi che contengono spazi. Controlla sempre `cd` prima di creare o modificare file.

## Orientamento e cartelle

```cmd
help
dir /?
cls
ver
dir /a
tree /f
```

- `help` elenca i comandi integrati; `comando /?` mostra la guida.
- `cls` pulisce lo schermo, non elimina file.
- `ver` mostra la versione Windows.
- `dir /a` include anche gli elementi nascosti.
- `tree /f` visualizza cartelle e file.

```cmd
mkdir documenti
cd documenti
cd ..
```

## Creare, leggere e cercare file di testo

```cmd
echo Prima riga>note.txt
echo Seconda riga>>note.txt
type note.txt
findstr /i "riga" note.txt
```

`>` crea o sovrascrive il file; `>>` aggiunge testo senza cancellare quello precedente. `type` legge testo nel terminale. `findstr /i` cerca senza distinguere maiuscole e minuscole.

## Copiare, rinominare e spostare

```cmd
copy note.txt copia.txt
ren copia.txt archivio.txt
mkdir archivio
move archivio.txt archivio\
dir archivio
```

La copia lascia l'originale al suo posto; `ren` cambia il nome; `move` cambia percorso. Se la destinazione esiste, leggi la richiesta prima di confermare.

## Informazioni di sistema e rete locale

```cmd
whoami
tasklist
ipconfig
ping -n 4 127.0.0.1
```

`whoami` mostra l'account corrente; `tasklist` elenca i processi; `ipconfig` mostra gli indirizzi locali. `127.0.0.1` è il loopback: verifica lo stack di rete locale, non Internet.

## Pulizia solo dei file di prova

```cmd
del archivio\archivio.txt
rmdir archivio
```

`del` non usa normalmente il Cestino. `rmdir` senza opzioni rimuove solo una cartella vuota. Prima di ogni eliminazione controlla percorso e nome con `cd` e `dir`; evita wildcard e `/s`.

## Esercizio

Nel laboratorio, crea `note.txt` con due righe, cercane una con `findstr`, copialo, sposta la copia e verifica la struttura con `tree /f`. Elimina solo la copia e lascia intatto l'originale.
## Spiegazione riga per riga dei comandi

`cmd.exe` è l'interprete dei comandi tradizionale di Windows. Le opzioni qui sotto appartengono al Prompt dei comandi e non devono essere copiate automaticamente in PowerShell.

### Preparare un laboratorio
- `if not exist "%TEMP%\\CorsoCMD" mkdir "%TEMP%\\CorsoCMD"`: `if not exist` controlla che il percorso non esista; solo in quel caso esegue `mkdir`; `%TEMP%` è la cartella temporanea dell'utente. In questo modo non si crea una nuova cartella se è già presente.
- `cd /d "%TEMP%\\CorsoCMD"`: `cd` cambia directory; `/d` permette di cambiare anche unità (per esempio da C: a D:); le virgolette proteggono i percorsi con spazi.
- `cd`: senza argomenti visualizza la directory corrente.

### Orientamento
- `help`: mostra l'elenco dei comandi interni disponibili in `cmd.exe`.
- `dir /?`: apre la guida del comando `dir`; `/?` è la convenzione di aiuto di molti comandi CMD.
- `cls`: pulisce la schermata, ma non cancella file.
- `ver`: visualizza la versione di Windows.
- `dir /a`: elenca file e cartelle includendo anche gli elementi nascosti; `/a` richiede tutti gli attributi.
- `tree /f`: mostra la struttura delle directory; `/f` include anche i nomi dei file.
- `mkdir documenti`: crea la directory `documenti`.
- `cd documenti`: entra nella directory appena creata.
- `cd ..`: torna alla directory genitore.

### Creare e cercare testo
- `echo Prima riga>note.txt`: `echo` scrive il testo; `>` lo reindirizza nel file e lo crea o sovrascrive.
- `echo Seconda riga>>note.txt`: `>>` aggiunge il testo in fondo al file senza cancellare il contenuto precedente.
- `type note.txt`: visualizza il contenuto testuale del file.
- `findstr /i "riga" note.txt`: cerca la parola nel file; `/i` ignora la differenza tra maiuscole e minuscole.

### Copiare e spostare
- `copy note.txt copia.txt`: crea `copia.txt` partendo da `note.txt`, lasciando intatto l'originale.
- `ren copia.txt archivio.txt`: cambia il nome del file senza spostarlo in un'altra directory.
- `mkdir archivio`: crea la directory che conterrà la copia rinominata.
- `move archivio.txt archivio\\`: sposta il file nella directory `archivio`; la barra finale rende evidente che la destinazione è una cartella.
- `dir archivio`: elenca il contenuto della directory per verificare lo spostamento.

### Sistema e rete
- `whoami`: mostra l'account Windows con cui è in esecuzione il terminale.
- `tasklist`: elenca i processi attivi e i relativi identificativi.
- `ipconfig`: mostra la configurazione IP delle interfacce di rete.
- `ping -n 4 127.0.0.1`: invia quattro richieste ICMP all'indirizzo di loopback; `-n 4` imposta il numero di richieste e `127.0.0.1` verifica lo stack TCP/IP locale, non la connessione Internet.

### Pulizia
- `del archivio\\archivio.txt`: elimina il file indicato; non lo sposta normalmente nel Cestino.
- `rmdir archivio`: elimina la directory solo se è vuota. Se contiene ancora file, il comando non la rimuove.

Controlla sempre la directory con `cd` e `dir` prima di usare `del`. Non aggiungere opzioni ricorsive e non usare caratteri jolly per esercizi introduttivi.
