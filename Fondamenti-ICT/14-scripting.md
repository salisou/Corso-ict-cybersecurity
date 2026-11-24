# Scripting e automazione

Uno script raccoglie comandi in un file eseguibile o interpretato. Automatizza attività ripetitive, rende i passaggi riproducibili e riduce gli errori manuali. Prima di automatizzare, prova i comandi su dati fittizi.

## PowerShell: primo script

Crea `inventario-lab.ps1`:

```powershell
$ErrorActionPreference = "Stop"

"Data: $(Get-Date -Format o)"
"Computer: $env:COMPUTERNAME"
"Utente: $(whoami)"

Get-Volume |
  Select-Object DriveLetter, FileSystem, SizeRemaining |
  Format-Table -AutoSize
```

Eseguilo nella cartella in cui l'hai creato:

```powershell
.\inventario-lab.ps1
```

Le policy di esecuzione PowerShell dipendono dal computer. Non modificarle globalmente per eseguire script sconosciuti; esamina il contenuto e segui le regole del tuo dispositivo.

## Bash: primo script

Crea `inventario-lab.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

printf 'Data: %s\n' "$(date -Is)"
printf 'Computer: %s\n' "$(hostname)"
printf 'Utente: %s\n' "$(id -un)"
df -h
```

Eseguilo:

```bash
chmod 700 inventario-lab.sh
./inventario-lab.sh
```

`set -euo pipefail` aiuta a far emergere alcuni errori: uscita non riuscita, variabili non definite e fallimenti in pipeline. Non sostituisce la verifica dei risultati.

## Variabili e input

```bash
PERCORSO="$HOME/permessi-lab"
printf 'Analizzo: %s\n' "$PERCORSO"
test -d "$PERCORSO" && echo "La directory esiste"
```

Metti tra virgolette le variabili che rappresentano percorsi o testo: spazi e caratteri speciali possono altrimenti essere interpretati dalla shell.

In PowerShell:

```powershell
$percorso = Join-Path $HOME "permessi-lab"
if (Test-Path -LiteralPath $percorso -PathType Container) {
    "La directory esiste: $percorso"
}
```

## Pipeline e filtri

```powershell
# Filtra processi e seleziona colonne utili
Get-Process |
  Where-Object CPU -gt 10 |
  Select-Object ProcessName, Id, CPU
```

```bash
# Mostra le prime righe del filesystem
df -h | head

# Cerca una parola in un file di esempio
grep -n "errore" ./log-lab.txt
```

Le pipeline passano dati da un comando al successivo. Verifica il tipo di dati e non inviare output sensibile a servizi esterni.

## Registrare output

```powershell
Get-Date -Format o | Out-File -FilePath .\report-lab.txt
Get-Volume | Out-File -FilePath .\report-lab.txt -Append
```

```bash
date -Is > report-lab.txt
df -h >> report-lab.txt
```

`>` crea o sovrascrive un file; `>>` aggiunge in coda. Verifica il percorso prima di usare la redirezione: `>` può cancellare il contenuto precedente.

## Errori e manutenzione

- Usa nomi descrittivi e commenti per spiegare decisioni non ovvie.
- Gestisci errori in modo esplicito; non ignorare codici di uscita.
- Evita password e token hard-coded. Usa i sistemi di credenziali approvati.
- Limita permessi dello script e dei file di output.
- Testa casi con spazi, file assenti e input vuoto.

## Esercizio

Modifica lo script inventario per salvare data, hostname e spazio libero in un file nella cartella di laboratorio. Eseguilo due volte, controlla il contenuto e spiega la differenza tra sovrascrivere (`>`) e aggiungere (`>>`).
