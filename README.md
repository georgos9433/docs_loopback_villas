- [Guida Loopback Test (Delay Measurement)](#guida-loopback-test-delay-measurement)
  - [Prerequisiti](#prerequisiti)
  - [Configurazione VILLASnode](#configurazione-villasnode)
  - [Configurazione Modello](#configurazione-modello)
  - [Avvio di VILLASnode](#avvio-di-villasnode)
  - [Avvio della simulazione](#avvio-della-simulazione)
    - [Monitoraggio](#monitoraggio)
    - [Reset della simulazione tramite RT-Lab](#reset-della-simulazione-tramite-rt-lab)


# Guida Loopback Test (Delay Measurement)

## Prerequisiti
Per eseguire correttamente il test di loopback tra VILLASnode e RT-Lab, assicurarsi di avere:
1. **Docker Desktop** (o Docker Engine) installato e **attivo** sul proprio PC.
2. Un'istanza funzionante di **VILLASnode** pronta per l'esecuzione tramite Docker.
3. Accesso di rete tra il simulatore OPAL-RT e il PC che ospita VILLASnode (senza firewall che blocchino le porte UDP).

---

## Configurazione VILLASnode
Nella cartella `dockerVillasNode` (o quella scelta per l'installazione di VILLASnode), crea il file `loopback_local.conf` con il seguente contenuto:

```json  
nodes = {
    sock_local={
        type ="socket",
        layer="udp",
        format="raw",
        in = {
            # This node only received messages on this IP:Port pair
            address = "*:12001",
        },
        out = {
            # This node sends outgoing messages to this IP:Port pair
            address = "xxx.xxx.xxx.xxx:12001",
        }
    }
}

paths=(
    {
        in = "sock_local",
        out = "sock_local",
        hooks=(
            {
                type="print",
            }
            

        )
    }
    
)
```
dove "xxx.xxx.xxx.xxx" deve essere sostituito con l'indirizzo IP del simulatore.

## Configurazione Modello
1. Importare il progetto Simulink contenuto nel file zip `loopback_simulink.zip` in RT-Lab. Assicurarsi che vengano importati tutti i file contenuti nell'archivio:
    ```text
    loopback_simulink
    ├── loopback_model.slx
    ├── AsyncIP.c
    ├── AsyncIP.mk
    └── AsyncIPUtils.h 
    ```  
2. Inserire la seguente stringa nella tab Development/Compiler:
    ```text
    make -f /usr/opalrt/common/bin/opalmodelmk
    ```  
    come mostrato in figura
![alt text](loopback_test_img/compiler.png)


3. **Opzionale se necessario** Nella tab variables se non presente per qualche ragione il compilatore sul sistema opal (si hanno errori riguardo al compiler GNU o gcc), impostare la variabile in figura (presente tra le predefinite premendo su "Add")
 ![alt text](loopback_test_img/compiler_var.png)

4. Nella tab files devono essere aggiunti i file precedentemente importati nella creazione del progetto come in figura seguente, assicurarsi che il transfer time sia impostato su "Before compilation":
 ![alt text](loopback_test_img/files.png)
 i file possono essere importati per mezzo del tasto "Add" selezionando i file presenti nella root folder del progetto.

    **ATTENZIONE**

    Talvolta premendo "Add" di default si viene portati nella cartella da cui si è importato il progetto e di conseguenza i file, assicurarsi di importare la copia dei file presenti nella root folder del progetto RT-Lab, presente nel workspace RT-Lab.


5. Editando il progetto è necessario impostare IP e porte di comunicazione. La porta 12001 preimpostata dovrebbe essere libera sul PC, per cui l'esempio sarà presentato con tale impostazione. L'IP da impostare sarà invece quello del PC su cui viene eseguita l'istanza VILLASnode. Nell'esempio in figura sostituire l'indirizzo evidenziato con quello corretto per il proprio setup.
![alt text](loopback_test_img/ipset.png)


6. Salvare il progetto simulink e chiudere il file
7. Procedere con il Build su RT-Lab
8. Procedere con il load sul simulatore

## Avvio di VILLASnode
Assicurarsi che Docker Desktop sia aperto e funzionante. Aprire un terminale PowerShell nella cartella `dockerVillasNode` ed eseguire il comando seguente.

**Nota:** Questa finestra del terminale deve restare aperta per tutta la durata del test. Il terminale rimarrà in attesa (come mostrato in figura) durante il funzionamento
```powershell
docker run -it --rm -p 12001:12001/udp -p 8000:8000 -v "${PWD}:/data" --entrypoint /bin/bash villas-node -c "villas node loopback_local.conf"
``` 
![alt text](loopback_test_img/terminal.png)

    


## Avvio della simulazione
Eseguire la simulazione in RT-Lab premendo il pulsante "Execute". Nel terminale di VILLASnode dovrebbero iniziare a comparire i dati inviati da e verso il simulatore ![alt text](loopback_test_img/terminal_execute.png) Per fermare VILLASnode è sufficiente premere `Ctrl+C`.


### Monitoraggio
La console della simulazione (riportata di seguito) mostrerà l'andamento del test.

![alt text](loopback_test_img/console.png)

**IMPORTANTE:** Assicurarsi che VILLASnode sia già in esecuzione e in attesa prima di avviare la registrazione dei dati su RT-Lab, altrimenti le misurazioni del ritardo iniziale verrebbero falsate o perse.

### Reset della simulazione tramite RT-Lab

Al termine della simulazione il file con le misure dei delay verrà salvato nella cartella root del progetto nel workspace RT-Lab in un percorso simile a:

```text
loopback_simulink/loopback_simulink_sm_master/OpREDHAWKtarget/delays.mat
``` 

È possibile spostare il file "delays.mat" per aprirlo ed analizzarlo. Il file è composto da una tabella di sole due righe, nella prima il tempo della simulazione corrispondente al sample, nella seconda riga sono riportati i sample dei delay misurati in secondi. Facendo la media/mediana/moda della seconda riga sulla sua lunghezza si ottiene la metrica per analizzare il ritardo sullo specifico percorso di rete.

    


   