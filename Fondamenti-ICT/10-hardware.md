# Hardware e componenti del computer

L'hardware è l'insieme dei componenti fisici di un sistema. Conoscerne le caratteristiche aiuta a scegliere software compatibile, diagnosticare rallentamenti e dimensionare macchine virtuali.

## Componenti principali

- **CPU**: esegue istruzioni. Frequenza, core e thread influenzano il tipo di carico gestibile.
- **RAM**: memoria temporanea usata da sistema e programmi; i dati si perdono allo spegnimento.
- **SSD/HDD**: archiviazione persistente. Gli SSD sono in genere più rapidi; gli HDD possono offrire capacità maggiore a costo inferiore.
- **Scheda madre e chipset**: collegano CPU, memoria, archiviazione e periferiche.
- **GPU**: elabora grafica e calcoli paralleli, secondo il modello e i driver.
- **NIC/scheda di rete**: collega il dispositivo a reti cablate o wireless.
- **Alimentatore e raffreddamento**: forniscono energia e mantengono le temperature entro i limiti previsti.

## Inventario in Windows

Apri PowerShell:

```powershell
# Modello del computer, produttore e versione Windows
Get-ComputerInfo |
  Select-Object CsManufacturer, CsModel, WindowsProductName, WindowsVersion

# Processori
Get-CimInstance Win32_Processor |
  Select-Object Name, NumberOfCores, NumberOfLogicalProcessors

# Moduli di memoria
Get-CimInstance Win32_PhysicalMemory |
  Select-Object Manufacturer, Capacity, Speed

# Dischi fisici e volumi
Get-PhysicalDisk | Select-Object FriendlyName, MediaType, Size, HealthStatus
Get-Volume | Select-Object DriveLetter, FileSystem, Size, SizeRemaining
```

`Capacity` e `Size` sono espressi in byte. Per convertire in GB dividili per `1GB` in una sessione PowerShell, ad esempio:

```powershell
$ram = Get-CimInstance Win32_PhysicalMemory | Measure-Object Capacity -Sum
"RAM installata: {0:N1} GB" -f ($ram.Sum / 1GB)
```

## Inventario in Linux

```bash
# CPU e architettura
lscpu

# Memoria totale e disponibile
free -h

# Dischi, partizioni e filesystem montati
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
df -h

# Dispositivi PCI, se pciutils è installato
lspci

# Informazioni USB, se usbutils è installato
lsusb
```

`lsblk` mostra i dispositivi a blocchi; `df -h` mostra lo spazio usato nei filesystem montati. Un disco visibile in `lsblk` potrebbe non avere filesystem o mount point.

## Stato e temperature

In Windows usa **Gestione attività > Prestazioni** oppure **Gestione dispositivi** per osservare uso delle risorse e stato dei driver. In Linux, `top` o `htop` (se installato) mostrano il carico dei processi; `sensors` può mostrare temperature se il supporto è disponibile.

```bash
# Carico e processi in tempo reale
top

# Sensori, se lm-sensors è installato e configurato
sensors
```

Non interpretare una temperatura isolata senza conoscere modello, carico e limiti del produttore. Non smontare un dispositivo acceso e non intervenire sull'alimentatore.

## Diagnosi di spazio

```powershell
# Spazio disponibile sui volumi Windows
Get-Volume | Sort-Object SizeRemaining |
  Select-Object DriveLetter, FileSystem, Size, SizeRemaining
```

```bash
# Uso dei filesystem
df -h

# Dimensione di una cartella scelta
du -sh <PERCORSO>
```

## Esercizio

1. Raccogli modello, CPU, RAM e tipo di disco del tuo computer o della VM.
2. Controlla quanto spazio è libero.
3. Confronta la RAM assegnata alla VM con quella disponibile sull'host.
4. Scrivi quale componente potrebbe limitare un'attività che usa molte VM simultanee.

Non pubblicare numeri di serie, identificativi hardware o informazioni aziendali senza autorizzazione.
