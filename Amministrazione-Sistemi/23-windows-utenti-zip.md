# Windows - utenti locali e archivi ZIP

Questa guida mostra come osservare account locali e creare un archivio didattico. La creazione di utenti o gruppi è opzionale e richiede PowerShell come amministratore. Non aggiungere account o gruppi agli amministratori.

## Preparare l'area file

```powershell
$lab = Join-Path $env:TEMP "FileZipWin"
$data = Join-Path $lab "Dati"
New-Item -ItemType Directory -Path $data -Force | Out-Null
Set-Location $lab
Set-Content -LiteralPath (Join-Path $data "note.txt") -Value "Documento di laboratorio"
```

## Ispezionare utenti e gruppi

```powershell
# Account locali e stato
Get-LocalUser | Select-Object Name, Enabled

# Gruppi locali
Get-LocalGroup | Select-Object Name

# Proprietario e ACL del file e della cartella
Get-Acl .\Dati\note.txt | Format-List Owner, AccessToString
Get-Acl .\Dati | Format-List Owner, AccessToString
```

I nomi dei gruppi predefiniti possono essere localizzati. L'appartenenza a un gruppo contribuisce ai diritti, ma non dimostra da sola l'accesso effettivo a un percorso.

## Account di laboratorio (facoltativo)

Solo in una VM di laboratorio e con autorizzazione amministrativa:

```powershell
# Verifica prima che i nomi non siano già usati
Get-LocalUser -Name "corso_lab" -ErrorAction SilentlyContinue
Get-LocalGroup -Name "CorsoFileLab" -ErrorAction SilentlyContinue

# Crea una password interattiva: non salvarla nel file del corso
$pw = Read-Host "Password per corso_lab" -AsSecureString
New-LocalUser -Name "corso_lab" -Password $pw `
  -Description "Account temporaneo di laboratorio"

# Crea un gruppo non amministrativo e aggiungi l'account
New-LocalGroup -Name "CorsoFileLab" -Description "Gruppo di prova"
Add-LocalGroupMember -Group "CorsoFileLab" -Member "$env:COMPUTERNAME\corso_lab"

# Verifica l'appartenenza
Get-LocalGroupMember -Group "CorsoFileLab"
```

Salta la creazione se uno dei nomi esiste già. Per assegnare una lettura di esempio:

```powershell
$group = "$env:COMPUTERNAME\CorsoFileLab"
icacls .\Dati /grant "${group}:(OI)(CI)R"
```

`OI` propaga la voce ai file e `CI` alle cartelle figlie compatibili. Verifica l'accesso effettivo con **Sicurezza > Avanzate > Accesso effettivo**. Non confondere una singola concessione `R` con l'insieme dei diritti effettivi dell'account.

## Backup ACL (non è un backup dei file)

```powershell
# Salva le DACL e controlla che il file sia stato creato
icacls .\Dati /save .\dacl-backup.txt /t
Get-Item .\dacl-backup.txt
```

Il file contiene regole DACL, non contenuti, proprietari o SACL. Per ripristinarlo, dalla cartella padre:

```powershell
icacls . /restore .\dacl-backup.txt
```

## Creare, elencare e testare uno ZIP

```powershell
# Scegli un nome nuovo per evitare di aggiornare un archivio esistente
$zip = Join-Path $lab "dati.zip"
Compress-Archive -Path .\Dati -DestinationPath $zip

# Elenca i membri senza estrarre
Add-Type -AssemblyName System.IO.Compression.FileSystem
$archive = [System.IO.Compression.ZipFile]::OpenRead($zip)
try {
    $archive.Entries | Select-Object FullName, Length
}
finally {
    $archive.Dispose()
}
```

Uno ZIP non è cifrato automaticamente e non è un backup completo delle ACL NTFS. `Compress-Archive` può omettere file nascosti. Controlla nomi e dimensioni prima di estrarre archivi non attendibili.

Estrai in una cartella nuova e confronta l'hash:

```powershell
$extract = Join-Path $lab "Estratti"
Expand-Archive -Path $zip -DestinationPath $extract

$originalHash = (Get-FileHash .\Dati\note.txt -Algorithm SHA256).Hash
$copyHash = (Get-FileHash .\Estratti\Dati\note.txt -Algorithm SHA256).Hash
$originalHash -eq $copyHash
```

`True` indica che i contenuti confrontati sono identici; non confronta ACL, proprietario o altri metadati. Non usare `-Force` o una destinazione già popolata per gli esercizi.

## Chiusura dell'esercizio

Se hai creato l'account e il gruppo temporanei, disabilita l'account e rimuovilo dal gruppo:

```powershell
Disable-LocalUser -Name "corso_lab"
Remove-LocalGroupMember -Group "CorsoFileLab" -Member "$env:COMPUTERNAME\corso_lab"
```

Questi comandi non eliminano il profilo o i file. Usali solo se hai creato proprio questi oggetti nel laboratorio; non cancellare utenti o gruppi esistenti.
