# Capitolo 7
# Operazioni Avanzate, Prestazioni e Caching in Amazon DynamoDB

## Introduzione

Dopo aver compreso i fondamenti di Amazon DynamoDB e i principi della modellazione dei dati NoSQL, è necessario approfondire le modalità operative attraverso cui le applicazioni interagiscono quotidianamente con il database.

In questo capitolo verranno analizzate le API principali di DynamoDB, le strategie di progettazione delle tabelle, le operazioni CRUD, le tecniche di ottimizzazione delle prestazioni e le soluzioni di caching che consentono di raggiungere latenze estremamente ridotte.

---

## Control Plane e Data Plane

Le operazioni di DynamoDB possono essere divise in due grandi categorie.

Le operazioni di Control Plane riguardano la gestione delle tabelle. Appartengono a questa categoria la creazione, modifica, descrizione ed eliminazione delle tabelle.

Le operazioni di Data Plane riguardano invece i dati archiviati nella tabella. In questo gruppo rientrano inserimenti, letture, aggiornamenti e cancellazioni degli item.

Questa distinzione è importante perché le due categorie presentano differenti caratteristiche operative, livelli di autorizzazione e requisiti prestazionali.

---

## L'importanza della progettazione preliminare

In DynamoDB la progettazione della tabella rappresenta l'attività più importante dell'intero ciclo di sviluppo.

A differenza dei database relazionali tradizionali, dove il modello dati viene progettato prima delle query, DynamoDB adotta un approccio inverso. Lo sviluppatore deve conoscere in anticipo tutti gli access pattern dell'applicazione e modellare i dati di conseguenza.

Ogni decisione relativa a Partition Key, Sort Key e indici secondari influenza direttamente prestazioni, costi e scalabilità.

### Approfondimento: Access Pattern First Design

Un errore comune consiste nel progettare una tabella basandosi esclusivamente sulla struttura dei dati.

In DynamoDB occorre invece partire dalle domande che l'applicazione porrà al database:

- quali dati verranno letti più frequentemente;
- quali attributi saranno utilizzati nelle ricerche;
- quali relazioni dovranno essere recuperate rapidamente;
- quali volumi di traffico sono previsti.

Solo dopo aver risposto a queste domande è possibile progettare correttamente la struttura della tabella.

---

## Progettazione della Partition Key

La scelta della Partition Key determina sia le modalità di accesso ai dati sia la distribuzione interna delle partizioni.

Una buona Partition Key deve essere coerente con gli access pattern più frequenti e presentare elevata cardinalità.

L'elevata cardinalità implica che il valore della chiave assuma molte varianti differenti, distribuendo il traffico in modo uniforme.

Nel caso di un'applicazione per la gestione delle note, utilizzare UserId come Partition Key rappresenta generalmente una scelta migliore rispetto a NoteId, poiché la maggior parte delle operazioni richiede il recupero di tutte le note di uno specifico utente.

---

## Chiavi composte e unicità degli item

Quando la sola Partition Key non è sufficiente a identificare in modo univoco un record, è possibile introdurre una Sort Key.

La combinazione tra Partition Key e Sort Key costituisce una chiave primaria composta.

Nel sistema Notes, ad esempio, uno studente può possedere molte note differenti. UserId non è quindi sufficiente per identificare un singolo record.

La combinazione UserId + NoteId garantisce invece l'unicità di ogni item.

---

## Progettazione degli indici secondari

Gli indici secondari consentono di supportare access pattern aggiuntivi.

I Local Secondary Index permettono di utilizzare una Sort Key alternativa mantenendo la stessa Partition Key della tabella.

I Global Secondary Index consentono invece di ridefinire completamente la struttura delle chiavi, permettendo interrogazioni basate su nuovi criteri.

Una progettazione efficace degli indici riduce drasticamente la necessità di effettuare scansioni complete.

---

## Capacità Provisioned e On-Demand

DynamoDB offre due modalità principali di gestione della capacità.

La modalità Provisioned richiede la definizione anticipata di Read Capacity Unit e Write Capacity Unit. Questa opzione è particolarmente efficace quando il traffico è prevedibile e relativamente stabile.

La modalità On-Demand elimina invece qualsiasi attività di pianificazione preventiva. Il database scala automaticamente in funzione del traffico e l'organizzazione paga esclusivamente per il consumo effettivo.

Le applicazioni con carichi molto variabili tendono a beneficiare particolarmente della modalità On-Demand.

---

## Creazione e gestione delle tabelle

Le API di gestione consentono di creare, aggiornare ed eliminare tabelle direttamente mediante codice.

Durante la creazione vengono definiti:

- nome della tabella;
- schema della chiave primaria;
- tipologia degli attributi della chiave;
- throughput iniziale;
- eventuali indici secondari.

Una volta completata la creazione è necessario attendere che la tabella raggiunga lo stato Active prima di eseguire operazioni sui dati.

---

## Utilizzo dei Waiter

Gli SDK AWS mettono a disposizione meccanismi denominati Waiter.

I Waiter verificano automaticamente il raggiungimento di uno stato desiderato evitando la scrittura manuale di cicli di polling.

Questa funzionalità risulta particolarmente utile durante la creazione delle tabelle, quando è necessario attendere che la risorsa diventi completamente operativa.

---

## Inserimento degli item con PutItem

L'API PutItem consente di inserire nuovi item all'interno di una tabella.

È importante comprendere che PutItem non si limita alla creazione di nuovi record. Se la chiave esiste già, l'operazione sostituisce integralmente l'item precedente.

Questa caratteristica richiede particolare attenzione durante le attività di aggiornamento dei dati.

---

## Operazioni BatchWriteItem

Quando è necessario caricare numerosi record contemporaneamente, DynamoDB offre l'API BatchWriteItem.

Una singola richiesta può contenere fino a venticinque operazioni.

L'API restituisce gli eventuali elementi non elaborati, consentendo all'applicazione di effettuare tentativi successivi senza dover riprocessare l'intero batch.

Questa funzionalità è particolarmente utile nelle attività di popolamento iniziale delle tabelle.

---

## Recupero dei dati con GetItem

L'API GetItem consente il recupero di un singolo elemento identificato dalla chiave primaria.

Quando la tabella utilizza una chiave composta è necessario fornire sia Partition Key sia Sort Key.

Per impostazione predefinita DynamoDB utilizza letture eventualmente consistenti, ma è possibile richiedere letture fortemente consistenti quando richiesto dai requisiti applicativi.

---

## Query: il metodo di accesso preferenziale

Le Query rappresentano il meccanismo più efficiente per recuperare i dati.

Le interrogazioni sfruttano direttamente le chiavi della tabella o degli indici secondari e consentono di ottenere prestazioni molto elevate.

La Query può utilizzare operatori specifici sulla Sort Key quali:

- equal;
- less than;
- greater than;
- between;
- begins_with.

Queste funzionalità permettono di realizzare modelli di interrogazione estremamente sofisticati.

---

## Strategie avanzate con la Sort Key

Una tecnica particolarmente efficace consiste nell'utilizzare la Sort Key per rappresentare strutture gerarchiche.

Ad esempio è possibile memorizzare valori nel formato:

Italia#Lazio#Roma

oppure:

USA#Washington#Seattle

Successivamente l'operatore begins_with permette di interrogare interi rami della gerarchia mantenendo elevate prestazioni.

---

## Scan e impatto sulle prestazioni

Le Scan consentono di filtrare attributi non presenti nella chiave primaria.

L'operazione legge però tutti gli elementi della tabella e successivamente applica il filtro richiesto.

Questo approccio comporta costi e tempi di risposta superiori rispetto alle Query.

Per questa ragione l'utilizzo delle Scan dovrebbe essere limitato ai casi strettamente necessari.

---

## Parallel Scan

Quando le Scan risultano inevitabili è possibile migliorarne le prestazioni utilizzando Parallel Scan.

La tabella viene suddivisa logicamente in segmenti differenti che vengono elaborati simultaneamente.

Questo approccio consente di sfruttare più efficacemente la capacità disponibile riducendo la durata complessiva dell'operazione.

---

## Aggiornamento selettivo con UpdateItem

L'API UpdateItem consente di modificare soltanto gli attributi necessari.

A differenza di PutItem, non richiede la sostituzione dell'intero record.

Questo riduce il volume dei dati trasferiti e migliora l'efficienza dell'applicazione.

---

## Scritture condizionali

Una delle funzionalità più potenti di DynamoDB consiste nelle Condition Expression.

Le condizioni permettono di eseguire aggiornamenti soltanto quando determinati requisiti risultano soddisfatti.

Ad esempio è possibile aggiornare una nota soltanto se il campo Favorite non possiede uno specifico valore.

Le scritture condizionali rappresentano uno strumento fondamentale per garantire coerenza applicativa nei sistemi distribuiti.

---

## Optimistic Locking

DynamoDB non implementa i tradizionali meccanismi di lock tipici dei database relazionali.

Per gestire aggiornamenti concorrenti viene spesso utilizzato l'Optimistic Locking.

Ogni record contiene un numero di versione.

Prima dell'aggiornamento l'applicazione verifica che la versione letta sia ancora valida. Se nel frattempo un altro client ha modificato l'item, l'operazione viene rifiutata e il conflitto può essere gestito esplicitamente.

Questo approccio permette di mantenere elevate prestazioni evitando lock pesanti.

---

## Eliminazione degli item

L'API DeleteItem consente di rimuovere singoli elementi dalla tabella.

Anche le eliminazioni possono utilizzare condizioni, permettendo la cancellazione esclusivamente in presenza di specifici requisiti aziendali.

---

## Programmazione ad alto livello con DynamoDB Enhanced Client

Gli SDK AWS forniscono librerie di alto livello che semplificano notevolmente lo sviluppo.

Nel caso di Java, il DynamoDB Enhanced Client permette di mappare automaticamente gli item della tabella in oggetti del dominio applicativo.

Le classi vengono annotate con metadati che definiscono:

- Partition Key;
- Sort Key;
- mapping degli attributi.

Questo approccio elimina gran parte del codice necessario alla serializzazione e deserializzazione dei dati.

---

## Object Mapping e DynamoDbBean

L'utilizzo dell'annotazione DynamoDbBean trasforma una normale classe Java in una rappresentazione diretta degli elementi memorizzati nel database.

Lo sviluppatore può quindi lavorare direttamente con oggetti, migliorando leggibilità, manutenibilità e produttività.

Questa tecnica risulta particolarmente vantaggiosa nei progetti enterprise caratterizzati da modelli dati complessi.

---

## Strategie di caching

Sebbene DynamoDB fornisca latenze nell'ordine dei millisecondi, alcuni scenari richiedono prestazioni ancora superiori.

In questi casi è possibile introdurre un livello di caching.

AWS mette a disposizione due soluzioni principali:

- Amazon ElastiCache;
- DynamoDB Accelerator (DAX).

---

## Amazon ElastiCache

ElastiCache permette di creare cluster Redis o Memcached completamente gestiti.

La logica di interazione tra applicazione, cache e database viene implementata direttamente nel codice.

Questo approccio offre grande flessibilità ma richiede una maggiore complessità progettuale.

---

## DynamoDB Accelerator (DAX)

DAX rappresenta una soluzione specializzata per DynamoDB.

Si posiziona tra applicazione e database fornendo una cache in memoria completamente compatibile con le API DynamoDB.

L'integrazione richiede modifiche minime al codice applicativo.

Le latenze possono ridursi dall'ordine dei millisecondi a quello dei microsecondi.

---

## Operazioni supportate da DAX

DAX supporta le principali operazioni di lettura:

- GetItem;
- BatchGetItem;
- Query;
- Scan.

Per le operazioni di scrittura utilizza una strategia write-through che mantiene sincronizzati cache e database.

Sono supportate:

- PutItem;
- UpdateItem;
- DeleteItem;
- BatchWriteItem.

---

## Best Practice per DynamoDB

Una corretta progettazione dovrebbe seguire alcuni principi fondamentali:

- progettare partendo dagli access pattern;
- scegliere Partition Key ad alta cardinalità;
- preferire Query rispetto a Scan;
- utilizzare gli indici secondari per supportare nuovi scenari di accesso;
- sfruttare UpdateItem invece di riscrivere interamente gli item;
- implementare Optimistic Locking nelle operazioni concorrenti;
- utilizzare DAX o ElastiCache quando i requisiti prestazionali lo richiedono.

---

## Conclusioni

La vera potenza di DynamoDB emerge quando la progettazione del modello dati viene combinata con una corretta gestione delle capacità, delle Query e degli indici. Le API avanzate, le strategie di accesso ottimizzate, i meccanismi di concorrenza e le soluzioni di caching consentono di costruire applicazioni cloud-native altamente scalabili e capaci di gestire milioni di richieste mantenendo tempi di risposta estremamente contenuti.
