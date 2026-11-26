# Analisi del traffico in laboratorio

La cattura di rete registra pacchetti che attraversano un'interfaccia. Può contenere indirizzi, dati personali o credenziali: cattura solo traffico generato da te in un laboratorio e proteggi i file `.pcap`.

## Identificare l'interfaccia locale

```bash
# Crea la cartella di laboratorio se non esiste
mkdir -p "$HOME/corso-linux"

# Elenca le interfacce visibili a tcpdump
sudo tcpdump -D

# Stato e indirizzi delle interfacce
ip addr
```

Usa l'interfaccia loopback (`lo`) per un test locale tra processi della stessa VM. Non scegliere interfacce aziendali o di rete condivisa.

## Cattura limitata su loopback

```bash
# Cattura al massimo 20 pacchetti senza risolvere nomi
sudo tcpdump -i lo -n -c 20

# Salva al massimo 100 pacchetti di loopback in un file del laboratorio
sudo tcpdump -i lo -n -c 100 -w "$HOME/corso-linux/lab.pcap"
```

`-i` sceglie l'interfaccia, `-n` evita risoluzioni DNS durante la visualizzazione, `-c` limita il numero di pacchetti, `-w` salva una cattura binaria. Il file può comunque contenere dati sensibili.

Per generare traffico innocuo, avvia un server locale in un terminale:

```bash
python3 -m http.server 8080 --bind 127.0.0.1
```

Da un secondo terminale della stessa VM effettua una richiesta di prova:

```bash
curl http://127.0.0.1:8080/
```

Interrompi il server con `Ctrl+C` e ripeti la cattura. Non inserire password o dati reali nella richiesta.

## Leggere una cattura con TShark

Se Wireshark/TShark è installato:

```bash
# Legge i pacchetti dal file e mostra un riepilogo
tshark -r "$HOME/corso-linux/lab.pcap"

# Cerca richieste HTTP riconosciute nella cattura del test locale
tshark -r "$HOME/corso-linux/lab.pcap" -Y 'http.request'
```

I filtri di visualizzazione selezionano pacchetti già presenti nel file; non effettuano una scansione della rete. Il traffico HTTPS è cifrato e normalmente non mostra il contenuto HTTP.

## Buone pratiche

- Cattura il minimo indispensabile e limita tempo/interfaccia/numero di pacchetti.
- Inizia e termina la cattura in momenti documentati.
- Conserva file e report in un'area con permessi adeguati.
- Elimina le catture secondo le regole del corso dopo aver concluso l'analisi.
- Non condividere file `.pcap` senza verificare che non contengano dati di terzi.

## Esercizio

1. Avvia il server locale sulla loopback.
2. Cattura un numero limitato di pacchetti su `lo`.
3. Genera una richiesta HTTP al server locale.
4. Leggi la cattura con TShark e identifica la richiesta di test.
5. Spiega perché una cattura su una scheda di rete condivisa richiede un'autorizzazione diversa.
## Spiegazione riga per riga dei comandi

L'**analisi del traffico** osserva i pacchetti che attraversano un'interfaccia di rete. È utile per capire quali protocolli e comunicazioni sono presenti, ma una cattura può contenere dati personali: in questo corso si usa soltanto traffico generato nella VM di laboratorio.

### Identificare l'interfaccia
- `mkdir -p "$HOME/corso-linux"`: crea la cartella di lavoro nella home; `-p` crea eventuali directory mancanti e non fallisce se esiste già.
- `sudo tcpdump -D`: `tcpdump` è uno strumento da terminale per catturare e leggere pacchetti; `-D` elenca le interfacce disponibili. `sudo` è necessario in molte configurazioni per accedere alla cattura.
- `ip addr`: mostra interfacce, stato e indirizzi IP assegnati.

### Catturare pacchetti
- `sudo tcpdump -i lo -n -c 20`: cattura sull'interfaccia loopback `lo`; `-i` sceglie l'interfaccia, `-n` evita la risoluzione di indirizzi e porte in nomi, `-c 20` termina dopo 20 pacchetti.
- `sudo tcpdump -i lo -n -c 100 -w "$HOME/corso-linux/lab.pcap"`: cattura al massimo 100 pacchetti; `-w` scrive i pacchetti nel file PCAP invece di mostrarli in formato leggibile. Il file va trattato come potenzialmente sensibile.
- `python3 -m http.server 8080 --bind 127.0.0.1`: avvia un piccolo server HTTP di test sulla porta 8080; `127.0.0.1` limita l'ascolto alla sola VM locale, a differenza di `0.0.0.0`.
- `curl http://127.0.0.1:8080/`: invia una richiesta HTTP al server locale; serve a generare traffico innocuo da osservare nella cattura.

### Leggere la cattura
- `tshark -r "$HOME/corso-linux/lab.pcap"`: TShark è la versione da terminale di Wireshark; `-r` legge un file di cattura già esistente e mostra i pacchetti riconosciuti.
- `tshark -r "$HOME/corso-linux/lab.pcap" -Y 'http.request'`: legge lo stesso file e applica il filtro di visualizzazione `http.request`, mostrando i pacchetti che Wireshark interpreta come richieste HTTP.

**Differenza importante:** `-w` crea una cattura; `-r` la legge. Il filtro `-Y` seleziona pacchetti già registrati e non avvia una scansione della rete. HTTPS cifra il contenuto applicativo, quindi normalmente non è possibile leggere il testo della richiesta senza le condizioni e le chiavi di decrittazione appropriate.
