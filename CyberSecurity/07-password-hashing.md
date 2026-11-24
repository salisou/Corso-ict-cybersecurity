# Password e hashing

Questa scheda illustra come proteggere password, verificare l'integrità dei file e usare la crittografia. Non contiene procedure per indovinare o recuperare password di account.

## Password, passphrase e MFA

- Usa password uniche per ogni servizio; un password manager aiuta a crearle e conservarle.
- Preferisci password lunghe e casuali o passphrase non prevedibili.
- Abilita MFA quando disponibile e conserva i codici di recupero in un luogo protetto.
- Non riutilizzare password aziendali, personali e di laboratorio.
- Non condividere password in chat, ticket, file di configurazione o righe di comando.

## Generare casualità

Su Windows PowerShell, usa un generatore crittografico:

```powershell
# Genera 24 byte casuali crittograficamente sicuri (compatibile con Windows PowerShell)
$bytes = New-Object byte[] 24
$rng = [System.Security.Cryptography.RandomNumberGenerator]::Create()
$rng.GetBytes($bytes)
$rng.Dispose()
[Convert]::ToBase64String($bytes)
```

Su Linux, OpenSSL può generare byte casuali:

```bash
# Genera 24 byte casuali in formato Base64
openssl rand -base64 24
```

Conserva il risultato in un password manager, non in un file in chiaro o nella cronologia del terminale. La Base64 può contenere caratteri speciali; quando la usi in una shell, racchiudila tra virgolette o incollala nel campo previsto.

## Hashing e integrità dei file

Un hash è un'impronta di lunghezza fissa. SHA-256 è adatto a rilevare se un file è cambiato quando si confronta con un valore di riferimento fidato.

```powershell
# Calcola SHA-256
Get-FileHash .\documento.txt -Algorithm SHA256
```

```bash
# Calcola SHA-256
sha256sum documento.txt

# Crea un manifest e verifica in seguito
sha256sum documento.txt > documento.sha256
sha256sum -c documento.sha256
```

Se il file cambia anche di un solo byte, l'hash risulterà diverso. L'hash non autentica da solo l'autore: per fidarsi del valore di riferimento, serve una fonte attendibile o una firma digitale.

## Perché non usare SHA-256 direttamente per le password

Le password richiedono algoritmi di derivazione lenti e specifici, che usano un **salt** casuale per ogni password e un costo di calcolo regolabile. Esempi di algoritmi adatti sono Argon2id, scrypt, bcrypt e PBKDF2 configurati secondo indicazioni aggiornate.

- Un **salt** è unico per ogni registrazione e non deve essere segreto.
- Il **costo** rende più onerose le verifiche di password.
- L'applicazione deve usare una libreria consolidata e conservare l'algoritmo, il costo e il salt insieme al valore derivato.
- Non inventare un algoritmo, non usare cifrature reversibili per conservare password e non codificarle in Base64 aspettandosi protezione.

Per applicazioni reali, segui la documentazione della piattaforma e le raccomandazioni di sicurezza correnti. Non implementare manualmente il formato dell'hash.

## Crittografia di file con GPG

Per cifrare un file destinato a un'altra persona serve la sua chiave pubblica verificata:

```bash
# Elenca le chiavi pubbliche disponibili
gpg --list-keys

# Cifra il file per il destinatario indicato
gpg --encrypt --recipient <CHIAVE-ID> documento.txt

# Decifra il file con la chiave privata corrispondente
gpg --decrypt documento.txt.gpg
```

Prima di cifrare, verifica l'impronta della chiave pubblica del destinatario attraverso un canale indipendente. La chiave privata deve restare segreta e protetta da una passphrase robusta.

Per cifrare un file per uso personale con una passphrase:

```bash
# Crea documento.txt.gpg e richiede una passphrase interattiva
gpg --symmetric documento.txt

# Decifra in un nuovo file di prova
gpg --output documento-recuperato.txt --decrypt documento.txt.gpg
```

Il file originale `documento.txt` resta in chiaro. Proteggi la passphrase e il file cifrato; non digitare la passphrase in uno script o nella riga di comando.

## Firma e verifica

```bash
# Firma un file con la propria chiave privata
gpg --sign documento.txt

# Verifica una firma
gpg --verify documento.txt.gpg
```

La cifratura protegge la riservatezza; la firma verifica integrità e associazione alla chiave firmataria. La verifica è significativa solo se la chiave pubblica è stata autenticata correttamente.

## Esercizi guidati

### Verificare una modifica

```bash
printf 'test laboratorio\n' > campione.txt
sha256sum campione.txt
printf 'seconda riga\n' >> campione.txt
sha256sum campione.txt
```

Confronta i due hash e spiega perché sono diversi.

### Confrontare file in Windows

```powershell
"test laboratorio" | Set-Content .\campione.txt
Get-FileHash .\campione.txt -Algorithm SHA256
Add-Content .\campione.txt "seconda riga"
Get-FileHash .\campione.txt -Algorithm SHA256
```

Usa solo dati fittizi. Non inserire password vere in script, esempi o file di esercitazione.
