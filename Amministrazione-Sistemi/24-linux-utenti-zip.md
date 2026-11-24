# Linux - utenti, gruppi e archivi ZIP

Gli account e i gruppi definiscono identità usate dal sistema per applicare permessi. ZIP raccoglie file in un archivio, ma non è cifratura né un backup affidabile di ACL e proprietari.

## Preparare la cartella di laboratorio

```bash
LAB=/tmp/CorsoLinuxUtentiZip
mkdir -p "$LAB/dati"
chmod 755 "$LAB"
printf 'Documento di laboratorio\n' > "$LAB/dati/note.txt"
chmod 640 "$LAB/dati/note.txt"
chmod 750 "$LAB/dati"
```

`/tmp` è usata qui per rendere attraversabile il percorso a un account di test; i file sono fittizi. `chmod` modifica realmente i permessi, quindi non sostituire i percorsi con cartelle personali o di sistema.

## Identità e gruppi

```bash
# Account e gruppi della sessione corrente
whoami
id

# Voce account e gruppo primario correnti
getent passwd "$(id -un)"
getent group "$(id -gn)"

# Permessi e proprietà dell'area di laboratorio
ls -ld "$LAB" "$LAB/dati"
ls -l "$LAB/dati/note.txt"
```

`getent` legge le fonti account configurate. La voce `/etc/passwd` contiene dati dell'account, non la password in chiaro. `id` mostra i gruppi effettivi della sessione.

## Utenti e gruppi di prova (facoltativo)

La gestione degli account modifica il sistema e richiede privilegi amministrativi. Eseguila solo in una VM dedicata, con nomi che non esistono già:

```bash
# Prima controlla i nomi
getent passwd corso_lab
getent group corso_team

# Se sono liberi, crea l'account senza home e un gruppo di laboratorio
sudo useradd -M -s /bin/bash corso_lab
sudo groupadd corso_team

# Imposta una password interattiva, se serve nel tuo esercizio
sudo passwd corso_lab

# Aggiunge il gruppo senza sostituire le appartenenze esistenti
sudo usermod -aG corso_team corso_lab

# Verifica identità e gruppi configurati
id corso_lab
getent group corso_team
```

Salta la creazione se i nomi esistono. `-aG` aggiunge un gruppo supplementare; omettere `-a` può sostituire gli altri gruppi supplementari. Gli account non sono amministratori e non vanno abilitati per SSH in questo esercizio.

## Gruppo condiviso, SGID e umask

```bash
# Assegna la cartella al gruppo di prova
sudo chgrp corso_team "$LAB/dati"

# Proprietario e gruppo hanno accesso; SGID fa ereditare il gruppo
sudo chmod 2770 "$LAB/dati"
ls -ld "$LAB/dati"

# Crea un nuovo file con umask 007 senza cambiare la shell corrente
(umask 007; touch "$LAB/dati/condiviso.txt")
ls -l "$LAB/dati/condiviso.txt"
```

`2770` è setgid più `770`: i nuovi elementi ereditano il gruppo della directory. La umask `007` rimuove i permessi per "altri"; per un file nuovo il modo atteso è `660`, salvo ACL predefinite o impostazioni specifiche. SGID seleziona il gruppo, umask limita i permessi: sono meccanismi distinti.

Verifica l'accesso come utente di prova, solo se è stato creato:

```bash
# 0 significa che il test è riuscito; un valore diverso da 0 indica che il test è fallito
sudo -u corso_lab test -r "$LAB/dati/note.txt"
echo "Lettura: $?"
sudo -u corso_lab test -w "$LAB/dati"
echo "Scrittura nella cartella: $?"
```

I controlli `test -r` e `test -w` verificano l'accesso del processo, non tutte le capacità di una sessione interattiva.

## Backup e ripristino ACL

```bash
# Salva ACL e metadati riportati per l'albero dati
getfacl -R "$LAB/dati" > "$LAB/acl-backup.txt"

# Legge l'ACL corrente
getfacl "$LAB/dati" "$LAB/dati/note.txt"

# Ripristina dallo stesso backup
sudo setfacl --restore="$LAB/acl-backup.txt"
getfacl "$LAB/dati" "$LAB/dati/note.txt"
```

Il backup ACL non contiene il contenuto dei file. Proteggi il file di backup e ripristina solo sul percorso previsto; il ripristino di proprietari o gruppi può richiedere privilegi.

Per aggiungere o rimuovere una voce nominata:

```bash
# Esempio: concede lettura a un utente esistente
setfacl -m u:<UTENTE>:r "$LAB/dati/note.txt"
getfacl "$LAB/dati/note.txt"

# Rimuove quella voce nominata, non l'account dal sistema
setfacl -x u:<UTENTE> "$LAB/dati/note.txt"
```

Controlla anche la `mask` in `getfacl`: può limitare i diritti effettivi di utenti e gruppi nominati. Per dettagli su bit, directory e ACL, consulta [04-linux-security.md](../CyberSecurity/04-linux-security.md#permessi).

## Creare e verificare un archivio ZIP

Installa `zip` e `unzip` solo se mancanti e secondo la policy della VM. Poi:

```bash
# Crea un archivio nuovo a partire dalla cartella dati
cd "$LAB"
zip -r dati.zip dati

# Elenca i membri e controlla l'integrità CRC
unzip -l dati.zip
unzip -t dati.zip

# Estrae in una directory nuova senza sovrascrivere file esistenti
mkdir -p estratti
unzip -n dati.zip -d estratti
```

Se `dati.zip` esiste già, scegli un nome nuovo: `zip` può aggiornare un archivio esistente. `unzip -t` controlla la coerenza dei dati archiviati, ma CRC non autentica l'autore e non prova l'assenza di malware. Non estrarre archivi non attendibili con `sudo`.

Confronta contenuto originale ed estratto:

```bash
sha256sum dati/note.txt estratti/dati/note.txt
cmp dati/note.txt estratti/dati/note.txt
echo "Codice cmp: $?"

# Controlla permessi dell'estratto separatamente
ls -l estratti/dati/note.txt
getfacl estratti/dati/note.txt
```

Hash uguali e codice `cmp` pari a `0` indicano che i byte confrontati sono identici. Non dimostrano che ZIP abbia conservato proprietario, ACL o tutti i permessi originali.

## Chiusura dell'esercizio

Se hai creato un account temporaneo e vuoi bloccarne la password:

```bash
sudo usermod -L corso_lab
```

Il blocco password non revoca sessioni già aperte, chiavi SSH o altri metodi di accesso. Non cancellare account o gruppi non creati per il laboratorio.
