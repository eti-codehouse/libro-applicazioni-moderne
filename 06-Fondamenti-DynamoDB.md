# Capitolo 6
# Fondamenti di DynamoDB e Modellazione dei Dati NoSQL

## Introduzione

Le applicazioni moderne generano e gestiscono grandi quantità di dati strutturati. La scelta del database influenza direttamente prestazioni, scalabilità, costi operativi e complessità architetturale. Nell'ecosistema AWS sono disponibili differenti tecnologie per soddisfare esigenze specifiche: database relazionali, sistemi NoSQL, database a grafo e piattaforme di caching distribuito.

Tra queste soluzioni, Amazon DynamoDB occupa una posizione centrale nello sviluppo cloud-native grazie alla sua natura serverless, alla scalabilità automatica e alla capacità di fornire latenze nell'ordine dei millisecondi anche sotto carichi elevati. Basato su un modello NoSQL key-value e documentale, DynamoDB rappresenta uno dei componenti fondamentali delle architetture moderne costruite su AWS. 

---

## Le principali tipologie di database in AWS

AWS mette a disposizione un portafoglio estremamente ampio di tecnologie per la persistenza dei dati.

Le basi di dati relazionali sono rappresentate principalmente da Amazon RDS e Amazon Redshift. Questi sistemi utilizzano tabelle, righe, colonne, chiavi primarie e chiavi esterne. Lo schema viene definito preventivamente e i dati vengono interrogati mediante SQL.

I database NoSQL adottano invece un approccio differente. Invece di concentrarsi sulle relazioni tra tabelle, privilegiano prestazioni elevate, semplicità delle query e scalabilità orizzontale.

Per scenari che richiedono la rappresentazione di relazioni complesse tra entità, AWS offre Amazon Neptune, una piattaforma specializzata nei database a grafo.

Per il caching ad alta velocità sono disponibili Amazon ElastiCache, Redis e Memcached, utilizzati per ridurre il carico sui database principali e migliorare le prestazioni complessive delle applicazioni.

---

## Database relazionali e NoSQL a confronto

I database relazionali memorizzano i dati sotto forma di righe e colonne. Lo schema è rigido e deve essere definito prima dell'inserimento dei dati. Questo approccio offre elevata consistenza e potenti funzionalità di interrogazione.

I database NoSQL adottano invece strutture più flessibili. I dati vengono archiviati come documenti o insiemi di attributi associati a una chiave.

In un sistema relazionale la tabella Cliente deve possedere una struttura definita. In DynamoDB due record appartenenti alla stessa tabella possono invece contenere attributi differenti.

Dal punto di vista della scalabilità emergono ulteriori differenze. I database relazionali tendono a crescere verticalmente attraverso hardware più potente. Le piattaforme NoSQL sono invece progettate per distribuire automaticamente i dati su più nodi e crescere orizzontalmente.

---

## Perché scegliere Amazon DynamoDB

DynamoDB è progettato per garantire prestazioni prevedibili su qualunque scala.

Le applicazioni possono ottenere tempi di risposta nell'ordine dei millisecondi anche in presenza di volumi estremamente elevati di richieste.

Uno dei principali vantaggi consiste nella natura completamente gestita del servizio. Gli sviluppatori non devono amministrare server, applicare patch o pianificare capacità hardware.

Il servizio integra inoltre funzionalità avanzate di sicurezza tramite IAM e permette di definire autorizzazioni molto granulari sui dati memorizzati.

---

## Concetti fondamentali di DynamoDB

La struttura logica di DynamoDB ruota attorno a pochi concetti fondamentali: tabelle, item, attributi e chiavi.

Una tabella rappresenta il contenitore principale dei dati.

Gli item corrispondono ai record archiviati nella tabella.

Gli attributi rappresentano invece le informazioni contenute all'interno di ciascun item.

A differenza dei database relazionali, DynamoDB non impone uno schema rigido sugli attributi. Ogni item può contenere un insieme differente di informazioni purché siano presenti gli elementi che compongono la chiave primaria.

---

## Partition Key e Sort Key

La progettazione della chiave primaria rappresenta una delle decisioni più importanti nella modellazione dei dati.

La Partition Key determina il modo in cui DynamoDB distribuisce i dati all'interno delle partizioni fisiche della piattaforma.

Tutti gli elementi che condividono la stessa Partition Key vengono memorizzati nella medesima partizione logica.

Quando una tabella utilizza anche una Sort Key, la combinazione tra Partition Key e Sort Key identifica univocamente ogni item.

Questa struttura prende il nome di chiave primaria composta.

### Best Practice

Una buona Partition Key deve presentare elevata distribuzione statistica. Chiavi troppo concentrate generano partizioni molto utilizzate, denominate hot partition, che possono limitare le prestazioni complessive.

---

## Attributi e tipi di dato

DynamoDB supporta differenti tipologie di attributo.

Le più comuni comprendono stringhe, numeri e valori booleani.

Il sistema supporta inoltre dati binari, liste e mappe.

Le mappe e le liste possono essere annidate ricorsivamente, permettendo di modellare documenti complessi molto simili a strutture JSON.

Questa caratteristica rende DynamoDB particolarmente adatto ad applicazioni moderne che scambiano informazioni tramite API REST e payload JSON.

---

## Throughput e Capacity Unit

Le prestazioni di una tabella DynamoDB sono misurate mediante Read Capacity Unit e Write Capacity Unit.

Una Read Capacity Unit consente una lettura fortemente consistente al secondo per item fino a 4 KB.

Le letture a consistenza eventuale richiedono metà della capacità e possono quindi raddoppiare il throughput disponibile.

Una Write Capacity Unit permette invece una scrittura al secondo per item fino a 1 KB.

Questi parametri consentono di pianificare il comportamento del database e controllare i costi operativi.

---

## Consistenza forte ed eventuale

Uno dei compromessi tipici dei sistemi distribuiti riguarda il modello di consistenza.

DynamoDB offre sia letture fortemente consistenti sia letture eventualmente consistenti.

Con la consistenza eventuale le modifiche possono richiedere un breve intervallo di tempo prima di essere visibili a tutte le repliche.

Nella maggior parte delle applicazioni aziendali questa differenza è trascurabile e permette di ottenere prestazioni superiori.

Quando necessario è comunque possibile richiedere esplicitamente letture fortemente consistenti.

---

## Query e Scan

Le modalità principali di recupero dei dati sono Query e Scan.

Query rappresenta l'operazione più efficiente perché utilizza direttamente la chiave primaria o un indice secondario.

Scan esamina invece l'intero contenuto della tabella e richiede una quantità di risorse significativamente superiore.

Le applicazioni ben progettate cercano di utilizzare prevalentemente Query, riducendo al minimo il ricorso alle scansioni complete.

---

## Local Secondary Index

Un Local Secondary Index condivide la stessa Partition Key della tabella principale ma utilizza una differente Sort Key.

Questo meccanismo permette di eseguire interrogazioni alternative mantenendo la stessa distribuzione dei dati.

Gli LSI devono essere definiti durante la creazione della tabella e vengono aggiornati in modo sincrono.

---

## Global Secondary Index

I Global Secondary Index permettono di ridefinire completamente il modello di accesso ai dati.

Un GSI può utilizzare chiavi totalmente differenti rispetto a quelle della tabella principale.

La struttura viene memorizzata e partizionata indipendentemente, consentendo query altamente efficienti su nuovi criteri di ricerca.

I GSI possono essere creati successivamente alla definizione della tabella e vengono aggiornati in modo asincrono.

---

## Partizionamento e Adaptive Capacity

La distribuzione automatica dei dati rappresenta uno dei punti di forza di DynamoDB.

AWS suddivide internamente le tabelle in partizioni fisiche gestite automaticamente.

Per migliorare la gestione dei carichi sbilanciati, DynamoDB implementa la funzionalità Adaptive Capacity.

Questa caratteristica consente di trasferire capacità inutilizzata verso partizioni che stanno ricevendo un numero elevato di richieste, riducendo il rischio di colli di bottiglia.

---

## Strumenti di accesso per gli sviluppatori

Gli sviluppatori possono lavorare con DynamoDB mediante differenti strumenti.

La AWS Management Console permette la gestione visuale delle tabelle.

La AWS CLI consente automazione e scripting.

PartiQL introduce un linguaggio simile a SQL che semplifica alcune operazioni di interrogazione.

DynamoDB Local permette invece di sviluppare e testare le applicazioni localmente anche in assenza di connettività verso AWS.

---

## Sviluppo locale con DynamoDB Local

DynamoDB Local può essere eseguito direttamente sul computer dello sviluppatore o all'interno di container Docker.

Questo approccio consente di creare tabelle, caricare dati e verificare il comportamento dell'applicazione senza interagire con risorse cloud reali.

L'utilizzo di ambienti locali accelera lo sviluppo e riduce i costi associati ai test.

---

## NoSQL Workbench

NoSQL Workbench è uno strumento multipiattaforma progettato specificamente per DynamoDB.

Permette di modellare tabelle, creare indici, eseguire query PartiQL, generare codice e visualizzare strutture dati complesse.

Per molti sviluppatori rappresenta uno strumento prezioso nella fase di progettazione e ottimizzazione del modello dati.

---

## Modelli di programmazione

Gli SDK AWS offrono differenti livelli di astrazione.

L'interfaccia Low Level fornisce una corrispondenza diretta con le API DynamoDB.

L'interfaccia Document permette di lavorare con documenti simili a JSON.

L'Object Persistence Model consente infine di associare classi e oggetti ai record della tabella mediante annotazioni e mapping automatico.

La scelta dipende dal linguaggio utilizzato e dal livello di controllo richiesto.

---

## Comprendere richieste e risposte HTTP

Dietro ogni operazione eseguita tramite SDK si trova una chiamata HTTP verso le API DynamoDB.

Le richieste includono intestazioni di autenticazione, identificazione dell'operazione e payload JSON che descrive il comando richiesto.

Le risposte restituiscono codici HTTP, metadati e contenuti serializzati.

Sebbene gli sviluppatori raramente operino a questo livello, la comprensione del protocollo risulta estremamente utile durante le attività di troubleshooting.

---

## Gestione degli errori

Le API DynamoDB utilizzano codici HTTP standard.

Gli errori della famiglia 4xx indicano generalmente problemi legati alla richiesta del client, come tabelle inesistenti o parametri non validi.

Gli errori della famiglia 5xx identificano invece problematiche lato servizio.

Una corretta gestione delle eccezioni costituisce un requisito fondamentale nelle applicazioni di produzione.

---

## Caso di studio: Sistema di gestione delle note

Un'applicazione per la gestione delle note può utilizzare UserId come Partition Key e NoteId come Sort Key.

Questo modello consente di recuperare rapidamente tutte le note appartenenti a uno specifico utente e, contemporaneamente, identificare in maniera univoca ciascuna nota.

Se emerge la necessità di cercare le note preferite, un Local Secondary Index può utilizzare il campo Favorite come nuova Sort Key.

Se invece diventa necessario cercare direttamente una nota tramite il suo identificativo globale, è possibile introdurre un Global Secondary Index dedicato.

---

## Conclusioni

Amazon DynamoDB rappresenta uno dei servizi più importanti per lo sviluppo cloud-native su AWS. La combinazione di scalabilità automatica, prestazioni elevate, modello serverless e flessibilità dello schema consente di progettare applicazioni moderne estremamente efficienti.

La corretta comprensione di chiavi, partizioni, throughput e indici costituisce il fondamento per progettare modelli dati performanti e prepararsi agli argomenti avanzati relativi alle operazioni, all'ottimizzazione delle query e alle strategie di caching che verranno affrontati nei capitoli successivi.
