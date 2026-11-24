# Virtualizzazione e container

La virtualizzazione consente di eseguire sistemi isolati sullo stesso hardware fisico. È utile per provare sistemi operativi, creare ambienti didattici e ripristinare uno stato noto.

## VM e container

- **Macchina virtuale (VM)**: emula hardware e avvia un sistema operativo guest completo. Offre controllo su kernel, boot e rete virtuale.
- **Container**: isola processi e filesystem usando il kernel dell'host; in genere è più leggero di una VM, ma non equivale a un sistema operativo completo.
- **WSL 2**: esegue un ambiente Linux integrato con Windows tramite virtualizzazione; non va considerato una rete di laboratorio completamente isolata.

VirtualBox e l'installazione di Ubuntu/Kali sono trattati in [09-installazione-linux.md](../Laboratori/09-installazione-linux.md).

## Comandi Docker di osservazione

I comandi seguenti presuppongono che Docker sia installato e che il servizio sia attivo:

```bash
# Versioni del client e del motore
docker version

# Container in esecuzione
docker ps

# Tutti i container, inclusi quelli terminati
docker ps -a

# Immagini locali
docker image ls

# Informazioni dettagliate su un container
docker inspect <ID_O_NOME_CONTAINER>
```

Un container non è necessariamente sicuro perché è breve o leggero: immagini, volumi, privilegi e porte esposte influenzano l'isolamento.

## Esempio locale controllato

Questo esempio avvia una pagina di test e pubblica la porta **solo su localhost**:

```bash
docker run --rm --name web-lab \
  -p 127.0.0.1:8080:80 nginx:stable
```

In un altro terminale della stessa macchina:

```bash
curl -I http://127.0.0.1:8080
```

Interrompi il container con `Ctrl+C`; `--rm` rimuove il container quando termina, non l'immagine scaricata. Non rimuovere container o volumi non identificati.

## Reti e volumi

```bash
# Elenca reti Docker
docker network ls

# Elenca volumi
docker volume ls

# Log di un container di test
docker logs <ID_O_NOME_CONTAINER>
```

La pubblicazione generica `-p 8080:80` può rendere il servizio raggiungibile da altre interfacce. Per un'esercitazione locale, preferisci `-p 127.0.0.1:8080:80`. I volumi possono contenere dati persistenti anche dopo la rimozione del container: verifica prima di cancellarli.

## Snapshot e ciclo di vita VM

Prima di modificare una VM, crea uno snapshot tramite l'hypervisor e assegna un nome descrittivo. Uno snapshot non sostituisce un backup dei file importanti. Alloca CPU, RAM e disco in modo che l'host mantenga risorse sufficienti.

## Esercizio

1. Avvia il container web locale.
2. Verifica che risponda su `127.0.0.1:8080`.
3. Controlla il mapping con `docker ps`.
4. Ferma il container e verifica che non sia più in esecuzione.
5. Descrivi una differenza tra il container e una VM Linux completa.
