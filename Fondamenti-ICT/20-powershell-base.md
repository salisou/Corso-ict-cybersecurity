# PowerShell - fondamenti

PowerShell usa **cmdlet** con nomi del tipo `Verbo-Nome` e restituisce oggetti con proprietà. Questa guida usa PowerShell 5.1 o 7 su Windows; non eseguire i comandi in `cmd.exe`.

## Preparare l'ambiente

```powershell
$lab = Join-Path $env:TEMP "CorsoPS"
New-Item -ItemType Directory -Path $lab -Force | Out-Null
Set-Location $lab
Get-Location
```

`$env:TEMP` è la cartella temporanea dell'utente. `Get-Location` verifica la directory corrente.

## Scoprire comandi e guida

```powershell
Get-Help Get-ChildItem
Get-Command *File*
```

`Get-Command` trova i comandi disponibili. `Get-Help` spiega sintassi e parametri; alcune guide complete potrebbero non essere installate localmente.

## File e cartelle di prova

```powershell
# Elenco, inclusi elementi nascosti
Get-ChildItem -Force

# Crea e legge un file
Set-Content -Path .\note.txt -Value "Prima riga"
Add-Content -Path .\note.txt -Value "Seconda riga"
Get-Content -Path .\note.txt

# Copia, rinomina e cerca testo
Copy-Item -Path .\note.txt -Destination .\copia.txt
Rename-Item -Path .\copia.txt -NewName "archivio.txt"
Select-String -Path .\note.txt -Pattern "riga"
```

`Set-Content` sostituisce il contenuto; `Add-Content` aggiunge. `Copy-Item` crea un'altra copia, mentre `Rename-Item` cambia il nome. Controlla il percorso e la destinazione prima di scrivere.

## Pipeline: filtrare e ordinare oggetti

```powershell
# File non vuoti
Get-ChildItem -File |
  Where-Object { $_.Length -gt 0 }

# Ordina per dimensione e mostra colonne selezionate
Get-ChildItem -File |
  Sort-Object Length -Descending |
  Select-Object Name, Length
```

La pipeline `|` passa gli oggetti al cmdlet successivo. `Where-Object` filtra; `Sort-Object` ordina; `Select-Object` sceglie proprietà. `Length` è la dimensione in byte. `$_` indica l'oggetto corrente in un filtro.

## Esportare e importare dati CSV

```powershell
Get-ChildItem -File |
  Select-Object Name, Length |
  Export-Csv -Path .\elenco.csv -NoTypeInformation -Encoding UTF8

Import-Csv -Path .\elenco.csv
```

Un CSV conserva valori tabellari, non tutti i dettagli degli oggetti originali. `Export-Csv` crea o sovrascrive il file indicato.

## Hash e simulazione

```powershell
Get-FileHash -Path .\note.txt -Algorithm SHA256

# Simula la rimozione: non elimina il file
Remove-Item -Path .\archivio.txt -WhatIf
```

Un hash è un'impronta del contenuto e va confrontato con un riferimento attendibile. `-WhatIf` mostra cosa farebbe un comando senza eseguire l'azione; verifica comunque il percorso.

## Esercizio

1. Crea `note.txt` con `Set-Content` e `Add-Content`.
2. Esportane nome e dimensione in CSV e rileggi il CSV.
3. Calcola l'hash, aggiungi una riga e ricalcola: i valori dovrebbero differire.
4. Usa `Remove-Item -WhatIf` sulla copia di prova, poi verifica che esista ancora.
## Spiegazione riga per riga dei comandi

PowerShell è una shell e un linguaggio di automazione: i suoi comandi (cmdlet) lavorano principalmente con oggetti e proprietà, non soltanto con testo. Di seguito trovi il significato di ogni riga degli esempi.

### Preparare l'ambiente
- `$lab = Join-Path $env:TEMP "CorsoPS"`: crea una variabile chiamata `$lab`; `Join-Path` combina due parti di un percorso; `$env:TEMP` legge la variabile d'ambiente della cartella temporanea; `"CorsoPS"` è il nome della sottocartella.
- `New-Item -ItemType Directory -Path $lab -Force | Out-Null`: `New-Item` crea un elemento; `-ItemType Directory` specifica una cartella; `-Path` indica dove crearla; `-Force` evita l'errore se la cartella è già presente; `| Out-Null` non visualizza l'oggetto restituito.
- `Set-Location $lab`: sposta la sessione nella cartella di laboratorio.
- `Get-Location`: mostra il percorso corrente per verificare dove si sta lavorando.

### Scoprire i comandi
- `Get-Help Get-ChildItem`: apre la guida del cmdlet `Get-ChildItem`; il nome dopo `Get-Help` indica l'argomento da documentare.
- `Get-Command *File*`: cerca i comandi disponibili il cui nome corrisponde al modello `*File*`; l'asterisco rappresenta una sequenza qualsiasi di caratteri.

### File e cartelle
- `Get-ChildItem -Force`: elenca gli elementi della directory corrente; `-Force` include anche elementi nascosti o normalmente non mostrati.
- `Set-Content -Path .\\note.txt -Value "Prima riga"`: crea il file o sostituisce il contenuto esistente; `-Path` indica il file e `-Value` il testo da scrivere.
- `Add-Content -Path .\\note.txt -Value "Seconda riga"`: aggiunge testo alla fine del file senza sostituire la riga già presente.
- `Get-Content -Path .\\note.txt`: legge e mostra il contenuto del file.
- `Copy-Item -Path .\\note.txt -Destination .\\copia.txt`: crea una copia; `-Path` è la sorgente e `-Destination` la destinazione.
- `Rename-Item -Path .\\copia.txt -NewName "archivio.txt"`: rinomina l'elemento indicato; `-NewName` contiene il nuovo nome.
- `Select-String -Path .\\note.txt -Pattern "riga"`: cerca il testo indicato da `-Pattern` nel file specificato da `-Path` e restituisce le righe corrispondenti.

### Pipeline, filtri e ordinamento
- `Get-ChildItem -File`: elenca soltanto i file, escludendo le directory.
- `|`: la pipeline passa gli oggetti prodotti dal comando a sinistra al comando a destra.
- `Where-Object { $_.Length -gt 0 }`: mantiene gli oggetti per cui la condizione è vera; `$_` rappresenta l'oggetto corrente, `.Length` è la dimensione in byte e `-gt 0` significa “maggiore di zero”.
- `Sort-Object Length -Descending`: ordina gli oggetti usando la proprietà `Length`; `-Descending` ordina dal valore più grande al più piccolo.
- `Select-Object Name, Length`: mostra soltanto le proprietà `Name` e `Length`, rendendo l'output più leggibile.

### CSV
- `Export-Csv -Path .\\elenco.csv -NoTypeInformation -Encoding UTF8`: salva gli oggetti ricevuti dalla pipeline in un file CSV; `-Path` indica la destinazione, `-NoTypeInformation` evita una riga aggiuntiva con il tipo .NET e `-Encoding UTF8` sceglie la codifica.
- `Import-Csv -Path .\\elenco.csv`: legge il CSV e ricrea oggetti PowerShell con colonne utilizzabili nei comandi successivi.

### Hash e simulazione
- `Get-FileHash -Path .\\note.txt -Algorithm SHA256`: calcola l'impronta del file; `-Algorithm SHA256` sceglie l'algoritmo SHA-256.
- `Remove-Item -Path .\\archivio.txt -WhatIf`: normalmente elimina l'elemento; `-WhatIf` simula l'azione e mostra cosa verrebbe fatto senza cancellare il file.

Prima di usare cmdlet che modificano o eliminano dati, controlla sempre `Get-Location`, `-Path` e `-Destination`. Non usare percorsi di sistema negli esercizi.
