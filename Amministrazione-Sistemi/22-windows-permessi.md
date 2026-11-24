# Windows - permessi NTFS e ACL

NTFS controlla chi può leggere, modificare o amministrare file e cartelle. Una **ACL** è un elenco di regole associate a un oggetto; la **DACL** contiene le concessioni e le negazioni di accesso.

Gli accessi via rete hanno due livelli: i permessi della condivisione e quelli NTFS. Per accedere via SMB devono consentirlo entrambi; questa guida prova solo ACL NTFS locali.

## Preparare i file di prova

Apri PowerShell non elevato:

```powershell
$lab = Join-Path $env:TEMP "PermessiWin"
$data = Join-Path $lab "Dati"
New-Item -ItemType Directory -Path $data -Force | Out-Null
Set-Content -LiteralPath (Join-Path $data "note.txt") -Value "Documento di prova"
Set-Location $lab
```

La cartella è sotto il profilo temporaneo dell'utente. Non usare dati reali o percorsi di sistema.

## Identità, proprietario e voci ACL

```powershell
# Identità e SID dell'account corrente
whoami /user
whoami /groups

# Legge proprietario e voci accesso come oggetti PowerShell
Get-Acl -LiteralPath "$data\note.txt" |
  Format-List Owner, AccessToString

# Visualizza ACL con icacls
icacls "$data\note.txt"
```

Un SID identifica un account anche se il nome viene rinominato. `Owner` indica il proprietario, non prova da solo che l'utente possa leggere il contenuto. I gruppi, le negazioni e le voci ereditate influenzano l'accesso effettivo.

Diritti `icacls` comuni:

- `F`: controllo completo, include la possibilità di cambiare autorizzazioni.
- `M`: modifica, include scrittura ed eliminazione.
- `RX`: lettura ed esecuzione/attraversamento.
- `R`: lettura.
- `W`: scrittura.
- `(I)`: voce ereditata dal contenitore.

## Salvare DACL e provarne il ripristino

```powershell
# Salva le DACL del contenuto di Dati prima delle modifiche
icacls .\Dati /save .\permessi-backup.acl /t

# Verifica che il file di backup esista
Get-Item .\permessi-backup.acl
```

Il backup contiene le DACL, non il contenuto dei file, il proprietario o le SACL. Conservalo fuori dalla cartella modificata e proteggilo.

## Concessioni e diritti effettivi

```powershell
$account = [System.Security.Principal.WindowsIdentity]::GetCurrent().Name

# Sostituisce le concessioni esplicite di questo account con una voce di lettura
icacls .\Dati\note.txt /grant:r "${account}:(R)"

# Rilegge le ACL e il contenuto
icacls .\Dati\note.txt
Get-Content .\Dati\note.txt
```

`/grant:r` sostituisce le concessioni **esplicite** per l'identità indicata, non revoca diritti ottenuti tramite gruppi o voci ereditate. Una concessione `R` non garantisce quindi che l'accesso effettivo sia solo lettura.

Per vedere l'effetto di una regola di modifica su un file di prova:

```powershell
icacls .\Dati\note.txt /grant:r "${account}:(M)"
icacls .\Dati\note.txt
```

`M` permette modifica ed eliminazione. Non assegnare `F` o `M` a gruppi ampi. Ripristina la DACL dopo l'esercizio.

## Ereditarietà

Le cartelle possono trasmettere ACL a file e sottocartelle. Le opzioni seguenti cambiano l'ereditarietà sul file di laboratorio:

```powershell
# Disabilita l'ereditarietà copiando le voci ereditate come esplicite
icacls .\Dati\note.txt /inheritance:d

# Riabilita l'ereditarietà dal contenitore
icacls .\Dati\note.txt /inheritance:e
```

Riabilitare l'ereditarietà non cancella le voci esplicite create in precedenza. `/inheritance:r` rimuove le voci ereditate e può bloccare l'accesso: non usarlo in questo esercizio.

## Ripristinare e verificare

```powershell
# Esegui dalla cartella padre usata per salvare il backup
icacls . /restore .\permessi-backup.acl

# Controlla ACL e struttura
icacls .\Dati\note.txt
icacls .\Dati /verify /t
```

`/verify` controlla struttura e ordinamento delle ACL, non stabilisce se la policy sia corretta o se un utente specifico possa accedere. Ripristinare le DACL non ripristina il contenuto del file, il proprietario o le SACL.

Per valutare l'accesso complessivo, apri **Proprietà > Sicurezza > Avanzate > Accesso effettivo** e seleziona l'account di prova. Considera i gruppi e gli eventuali diritti di condivisione, se l'accesso avviene via rete.

## Esercizio

1. Salva la DACL iniziale.
2. Applica la concessione di lettura, poi quella di modifica, solo al file di prova.
3. Confronta `icacls` prima e dopo e descrivi le voci ereditate.
4. Ripristina la DACL e verifica il risultato.

Non aggiungere `Everyone:F`, non cambiare ACL in `C:\Windows` e non usare `takeown` su file di sistema.
