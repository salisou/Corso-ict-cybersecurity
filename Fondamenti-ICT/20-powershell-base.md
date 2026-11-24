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
