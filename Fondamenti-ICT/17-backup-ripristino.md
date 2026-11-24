# Backup e ripristino

Un backup è una copia separata dei dati, utilizzabile per recuperare informazioni dopo guasti, errori o incidenti. Una sincronizzazione non è sempre un backup: una cancellazione o un file cifrato può propagarsi anche alla copia sincronizzata.

## Strategia 3-2-1

- Mantieni almeno **3 copie** dei dati importanti.
- Usa almeno **2 tipi di supporto o sistemi** differenti.
- Conserva almeno **1 copia separata/offline** o protetta da credenziali indipendenti.

Definisci anche frequenza, retention, cifratura, responsabili e tempi di ripristino. Testa periodicamente il recupero: un backup non verificato potrebbe essere inutilizzabile.

## Copiare dati di prova in Windows

Usa una sorgente e una destinazione create per il laboratorio:

```powershell
$source = Join-Path $HOME "backup-lab\origine"
$destination = Join-Path $HOME "backup-lab\copia"
New-Item -ItemType Directory -Force -Path $source, $destination
"dati fittizi" | Set-Content (Join-Path $source "esempio.txt")

# /E copia le sottocartelle, incluse quelle vuote
# Non usare /MIR per un primo esercizio: può cancellare file nella destinazione
robocopy $source $destination /E /R:2 /W:2
```

Robocopy può restituire codici diversi da zero anche quando la copia è riuscita; consulta la documentazione Microsoft per interpretare il codice di uscita.

## Copiare dati di prova in Linux

```bash
mkdir -p ~/backup-lab/origine ~/backup-lab/copia
printf 'dati fittizi\n' > ~/backup-lab/origine/esempio.txt

# Prima simula l'operazione senza copiare
rsync -av --dry-run ~/backup-lab/origine/ ~/backup-lab/copia/

# Dopo aver verificato sorgente e destinazione, esegui la copia
rsync -av ~/backup-lab/origine/ ~/backup-lab/copia/
```

La barra finale sulla sorgente `origine/` significa copiare il contenuto della cartella nella destinazione. Controlla sempre entrambi i percorsi prima di lanciare il comando.

## Verifica delle copie

```powershell
Get-FileHash "$HOME\backup-lab\origine\esempio.txt" -Algorithm SHA256
Get-FileHash "$HOME\backup-lab\copia\esempio.txt" -Algorithm SHA256
```

```bash
sha256sum ~/backup-lab/origine/esempio.txt
sha256sum ~/backup-lab/copia/esempio.txt
```

Hash uguali indicano che i contenuti dei due file sono identici al momento del controllo. Non provano che il backup sia completo o che il supporto sia integro nel lungo periodo.

## Ripristinare in una cartella di prova

Ripristina prima in un percorso separato, non sopra i dati originali:

```bash
mkdir -p ~/backup-lab/ripristino
rsync -av ~/backup-lab/copia/ ~/backup-lab/ripristino/
diff -r ~/backup-lab/origine ~/backup-lab/ripristino
```

Su Windows, copia la cartella di backup in una nuova directory e confronta i file prima di sostituire quelli originali. Se i dati sono aziendali, segui la procedura di ripristino e verifica con il proprietario dei dati.

## Esercizio

1. Crea due file fittizi in una cartella di laboratorio.
2. Copiali in una destinazione separata con dry-run prima della copia effettiva.
3. Verifica almeno un file con SHA-256.
4. Ripristina la copia in una nuova cartella e confronta sorgente e risultato.
5. Documenta cosa accadrebbe se la cartella sorgente venisse cancellata.

Non usare `/MIR`, `--delete` o comandi di rimozione per esercizi iniziali: possono cancellare dati nella destinazione.
