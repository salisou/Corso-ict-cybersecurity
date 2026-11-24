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
