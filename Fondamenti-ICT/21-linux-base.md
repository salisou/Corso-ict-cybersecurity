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
