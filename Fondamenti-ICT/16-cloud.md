# Cloud computing

Il cloud fornisce risorse informatiche tramite servizi gestiti e reti. Le risorse possono essere create, dimensionate e monitorate da portali o API; l'uso può generare costi.

## Modelli di servizio

- **IaaS**: macchine virtuali, rete e storage; il cliente gestisce gran parte del sistema operativo e delle applicazioni.
- **PaaS**: piattaforma gestita per distribuire applicazioni; il provider gestisce più componenti infrastrutturali.
- **SaaS**: applicazione pronta all'uso gestita dal fornitore.
- **Public, private, hybrid** descrivono modelli di distribuzione, non livelli di sicurezza automatici.

La responsabilità è condivisa: il provider protegge componenti del servizio, mentre il cliente resta responsabile di identità, configurazione, dati e accessi secondo il servizio scelto.

## Concetti fondamentali

- **Tenant/account/subscription**: confini amministrativi e di fatturazione.
- **Region/zone**: posizione geografica e area di disponibilità delle risorse.
- **IAM/RBAC**: identità e autorizzazioni assegnate secondo il principio del privilegio minimo.
- **VPC/VNet**: reti virtuali e relative subnet.
- **Storage**: oggetti, file o dischi con diversi modelli di accesso e disponibilità.
- **Monitoring e cost management**: metriche, log, budget e avvisi.

Prima di creare risorse, controlla account, regione, autorizzazioni, costi stimati e policy dell'organizzazione. Non inserire chiavi API o segreti nei file del corso.

## CLI in sola lettura

Gli esempi seguenti presuppongono che la CLI sia installata e che tu sia già autenticato nell'account autorizzato.

### Azure CLI

```powershell
# Versione della CLI
az version

# Visualizza l'account attivo
az account show --output table

# Elenca i gruppi di risorse accessibili
az group list --output table
```

### AWS CLI

```powershell
# Identifica l'account e l'identità attivi senza mostrare chiavi segrete
aws sts get-caller-identity
```

I comandi di elenco possono mostrare nomi interni e dati dell'organizzazione: non incollarne l'output in spazi pubblici. Se non hai un account di laboratorio, studia i concetti senza autenticarti.

## Costi e protezione degli account

- Usa MFA per gli account amministrativi e non condividere identità personali.
- Preferisci ruoli temporanei e permessi minimi rispetto a credenziali statiche.
- Imposta budget/avvisi e controlla la regione prima di avviare servizi a pagamento.
- Elimina risorse di test solo seguendo una procedura che verifichi proprietà e dipendenze.
- Abilita log e monitoraggio secondo le policy del progetto.

## Esercizio

Con un account cloud didattico autorizzato, identifica l'account attivo e la regione predefinita senza creare o modificare risorse. Redigi una scheda con modello di servizio, ruolo dell'utente, rete e costi potenziali. Non includere ID sensibili o token.
