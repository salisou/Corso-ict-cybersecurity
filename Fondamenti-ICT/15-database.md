# Basi di dati e SQL

Un database organizza dati in modo consultabile e aggiornabile. In un database relazionale, le tabelle contengono righe (record) e colonne (campi); una chiave primaria identifica ogni riga.

## SQLite in locale

SQLite è un database contenuto in un file, utile per esercizi e applicazioni locali. Se `sqlite3` è installato, crea un database di laboratorio:

```bash
sqlite3 corso-lab.db
```

Nel prompt SQLite:

```sql
CREATE TABLE studenti (
    id INTEGER PRIMARY KEY,
    nome TEXT NOT NULL,
    corso TEXT NOT NULL
);

INSERT INTO studenti (nome, corso)
VALUES ('Ada', 'ICT');

SELECT id, nome, corso
FROM studenti;

UPDATE studenti
SET corso = 'CyberSecurity'
WHERE id = 1;

SELECT * FROM studenti;
.quit
```

`CREATE TABLE` definisce lo schema; `INSERT` aggiunge righe; `SELECT` legge; `UPDATE` modifica. `WHERE` limita le righe interessate: senza `WHERE`, un `UPDATE` o un `DELETE` può agire su tutta la tabella.

## Query e filtri

```sql
SELECT nome
FROM studenti
WHERE corso = 'ICT'
ORDER BY nome;

SELECT COUNT(*) AS totale
FROM studenti;
```

Prima di modificare dati, esegui una `SELECT` con la stessa condizione `WHERE` per controllare quali righe saranno interessate.

## Usare parametri invece di concatenare testo

Le applicazioni dovrebbero passare i valori come parametri, non costruire una query concatenando input. Esempio Python con la libreria standard:

```python
import sqlite3

connection = sqlite3.connect("corso-lab.db")
try:
    connection.execute(
        "INSERT INTO studenti (nome, corso) VALUES (?, ?)",
        ("Luca", "ICT"),
    )
    connection.commit()
finally:
    connection.close()
```

I `?` sono segnaposto associati ai valori separatamente. La parametrizzazione riduce errori e rischi quando i dati provengono da input.

## Esportare e copiare il database

```bash
# Esporta schema e dati in SQL leggibile
sqlite3 corso-lab.db .dump > corso-lab.sql

# Copia il file del database, dopo aver chiuso le applicazioni che lo usano
cp corso-lab.db corso-lab-backup.db
```

Le copie possono contenere dati personali: usa dati fittizi e conserva il backup in una posizione protetta.

## Concetti da ricordare

- **Schema**: struttura di tabelle, colonne e vincoli.
- **Primary key**: identificatore univoco di una riga.
- **Foreign key**: relazione tra righe di tabelle diverse.
- **Indice**: accelera alcune ricerche, ma occupa spazio e rende più costose le modifiche.
- **Transazione**: raggruppa operazioni che devono essere confermate o annullate insieme.
- **Least privilege**: applicazioni e account database devono avere solo i permessi necessari.

## Esercizio

1. Crea una tabella `dispositivi` con ID, nome e tipo.
2. Inserisci tre dispositivi fittizi.
3. Elenca solo quelli di un tipo scelto, ordinandoli per nome.
4. Esegui una query di conteggio e descrivi il risultato.
5. Crea un dump e verifica che contenga schema e righe inserite.
