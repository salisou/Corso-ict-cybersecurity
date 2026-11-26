# Linux - comandi essenziali in Bash

Questi esempi funzionano in Bash su Linux o WSL e non richiedono `sudo`. Usano una cartella di laboratorio e file fittizi.

## Preparare il laboratorio

```bash
mkdir -p "$HOME/corso-linux"
cd "$HOME/corso-linux"
pwd
```

`$HOME` è la home dell'utente. `pwd` mostra il percorso corrente: controllalo prima di modificare o cancellare file.

## Orientamento e file

```bash
# Elenco dettagliato, inclusi i nomi nascosti
ls -la

# Crea un file di due righe e leggilo
printf 'Prima riga\nSeconda riga\n' > note.txt
cat note.txt

# Copia e rinomina senza sovrascrivere silenziosamente
cp -i note.txt copia.txt
mv -i copia.txt archivio.txt
```

`>` crea o sovrascrive il file; verifica il nome prima di eseguirlo. `cp -i` e `mv -i` chiedono conferma in caso di sovrascrittura.

## Ricerca, estratti e conteggi

```bash
# Cerca una parola e mostra il numero di riga
grep -n "riga" note.txt

# Cerca file di testo sotto la cartella corrente
find . -type f -name "*.txt"

# Prime e ultime righe
head -n 1 note.txt
tail -n 1 note.txt

# Conta le righe
wc -l note.txt
```

Le virgolette in `find` impediscono alla shell di espandere `*.txt` prima che il comando lo riceva.

## Permessi e informazioni di sistema

```bash
# Rende il file leggibile e modificabile solo dal proprietario
chmod 600 note.txt
ls -l note.txt

# Processi dell'utente corrente
ps -u "$USER"

# Spazio filesystem e dimensione della cartella
df -h .
du -sh .
```

Per una spiegazione completa di `rwx`, gruppi, ACL e permessi delle directory consulta [04-linux-security.md](../CyberSecurity/04-linux-security.md#permessi).

## Rimozione controllata

```bash
# Chiede conferma prima di rimuovere il file di prova
rm -i archivio.txt

# Rimuove solo una directory vuota
mkdir vuota
rmdir vuota
```

`rm` non usa normalmente il Cestino. Evita `rm -rf`, wildcard e percorsi non controllati; non usare `sudo` per aggirare un errore durante gli esercizi.

## Esercizio

Crea due righe in un file, cerca la parola `riga`, conta le righe, applica `chmod 600` e verifica il risultato con `ls -l`. Rimuovi solo la copia con `rm -i`.
## Spiegazione riga per riga dei comandi

Questa sezione riprende **ogni comando** usato negli esempi precedenti. La spiegazione parte dal significato dello strumento e chiarisce poi opzioni, argomenti e risultato.

### Preparare il laboratorio
- `mkdir -p "$HOME/corso-linux"`: `mkdir` crea una directory; `-p` crea anche eventuali directory intermedie e non segnala errore se la cartella esiste già; `$HOME` è la home dell'utente. Serve a tenere gli esercizi separati dai documenti personali.
- `cd "$HOME/corso-linux"`: `cd` cambia la directory di lavoro. Le virgolette proteggono il percorso se contiene spazi.
- `pwd`: significa *print working directory* e stampa il percorso in cui Bash eseguirà i comandi successivi.

### Orientamento e file
- `ls -la`: `ls` elenca file e directory; `-l` mostra dettagli come permessi, proprietario, dimensione e data; `-a` include anche i nomi nascosti che iniziano con un punto.
- `printf 'Prima riga\\nSeconda riga\\n' > note.txt`: `printf` stampa testo; `\\n` inserisce un ritorno a capo; `>` reindirizza il testo nel file, creandolo o sovrascrivendolo. Usalo solo dopo aver verificato il nome del file.
- `cat note.txt`: `cat` legge e visualizza il contenuto del file nel terminale.
- `cp -i note.txt copia.txt`: `cp` copia il file; il primo argomento è la sorgente e il secondo la destinazione; `-i` chiede conferma prima di sovrascrivere.
- `mv -i copia.txt archivio.txt`: `mv` sposta o rinomina; qui cambia il nome nella stessa directory; `-i` protegge da una sovrascrittura involontaria.

### Ricerca e conteggi
- `grep -n "riga" note.txt`: `grep` cerca testo; `"riga"` è la stringa da trovare; `note.txt` è il file; `-n` mostra anche il numero della riga.
- `find . -type f -name "*.txt"`: `find` cerca nel filesystem; `.` indica la directory corrente; `-type f` limita ai file normali; `-name "*.txt"` seleziona i nomi che terminano in `.txt`. Le virgolette impediscono alla shell di espandere il carattere jolly prima che lo riceva `find`.
- `head -n 1 note.txt`: `head` mostra l'inizio del file; `-n 1` limita l'output alla prima riga.
- `tail -n 1 note.txt`: `tail` mostra la fine del file; `-n 1` visualizza l'ultima riga.
- `wc -l note.txt`: `wc` conta; `-l` richiede il conteggio delle righe.

### Permessi e sistema
- `chmod 600 note.txt`: `chmod` modifica i permessi; `600` assegna lettura e scrittura al proprietario e nessun permesso a gruppo e altri. È adatto a un file privato di laboratorio.
- `ls -l note.txt`: verifica i permessi e gli altri dettagli del file dopo la modifica.
- `ps -u "$USER"`: `ps` mostra i processi; `-u` filtra per utente; `$USER` viene sostituito con il nome dell'utente corrente.
- `df -h .`: `df` mostra lo spazio dei filesystem; `-h` usa unità leggibili (KB, MB, GB); `.` indica il filesystem che contiene la directory corrente.
- `du -sh .`: `du` stima lo spazio occupato; `-s` mostra un totale invece dei dettagli di ogni elemento; `-h` rende la dimensione leggibile; `.` indica la directory corrente.

### Rimozione controllata
- `rm -i archivio.txt`: `rm` elimina un file; `-i` chiede conferma. Linux normalmente non sposta il file nel Cestino.
- `mkdir vuota`: crea una directory chiamata `vuota`, utile per provare la rimozione di una directory senza contenuti.
- `rmdir vuota`: rimuove la directory solo se è vuota; è più prudente di una cancellazione ricorsiva.

**Attenzione:** non sostituire questi percorsi con directory di sistema o cartelle personali. Evita `rm -rf` e `sudo` negli esercizi introduttivi.
