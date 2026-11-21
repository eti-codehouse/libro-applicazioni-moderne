# Parte III – I dati applicativi con Amazon DynamoDB

# Capitolo 6 – Introduzione ai database

## 6.1 Obiettivi del capitolo

Oltre ai file, che S3 gestisce egregiamente, le applicazioni devono conservare **dati strutturati**: utenti, ordini, note, configurazioni. Questo capitolo esamina le opzioni di database offerte da AWS e si concentra su **Amazon DynamoDB**, il database NoSQL che costituisce il cuore dei dati di Polly Notes.

Prima si confrontano i diversi tipi di database e si chiarisce quando conviene un database relazionale e quando uno non relazionale. Poi si introducono i componenti chiave di DynamoDB: tabelle, item, attributi, chiavi, capacità di lettura e scrittura, operazioni di *scan* e *query* e indici secondari. Si esplorano i diversi modi di accedere a DynamoDB: la Console, il codice, la CLI, il linguaggio PartiQL e strumenti locali come DynamoDB Local e NoSQL Workbench. Il capitolo si chiude con le richieste e le risposte che viaggiano sul filo quando si dialoga con DynamoDB.

## 6.2 Le opzioni di database su AWS

### I tipi di database

Le opzioni per memorizzare i dati di un'applicazione si raggruppano in alcune grandi famiglie.

Un **database relazionale** organizza i dati in un insieme di **tabelle** collegate da **chiavi primarie** e **chiavi esterne**. Lo **schema** delle tabelle si definisce in anticipo e i dati si interrogano con un linguaggio come **SQL**. In AWS i carichi relazionali si affidano ad **Amazon RDS** (e ad **Amazon Aurora**) per i database transazionali e ad **Amazon Redshift** per il data warehousing analitico.

Un **database chiave-valore**, spesso chiamato **NoSQL**, è orientato alle **prestazioni**. Il tipo di interrogazione è diverso: non si fanno query di reportistica con molte *join*, *group by* e funzioni di aggregazione, ma letture e scritture rapide per chiave. **Amazon DynamoDB** appartiene a questa famiglia.

Un **database a grafo** rappresenta esplicitamente le **relazioni** tra gli elementi. Per i dati di un social network, dove contano i collegamenti tra persone, la scelta naturale è **Amazon Neptune**.

Una **cache in memoria** conserva i dati nella RAM per tempi di accesso minimi. **Amazon ElastiCache** crea e gestisce cluster di cache basati su **Memcached** o **Redis**, e verrà ripreso nel Capitolo 7.

### Relazionale e NoSQL a confronto

| Aspetto | Database relazionale | Database NoSQL (chiave-valore) |
|---|---|---|
| **Memorizzazione** | Righe e colonne | Item identificati da una chiave, con un insieme arbitrario di attributi (un "documento") |
| **Schema** | Fisso, definito alla creazione della tabella | Non imposto all'inserimento, a parte la chiave |
| **Interrogazione** | SQL, con join, group by e aggregazioni | Lettura per chiave, query sulle chiavi, scansione della tabella |
| **Scalabilità** | Prevalentemente **verticale**: un nodo più grande, eventualmente con repliche di lettura | **Orizzontale**: i dati sono partizionati su più nodi |
| **Transazioni** | ACID complete | Supporto variabile, spesso più semplice |
| **Consistenza** | Forte, grazie al nodo singolo | Spesso **eventuale**, con opzione di lettura fortemente consistente |

Alcuni punti meritano un approfondimento.

**Lo schema.** In un database relazionale, la tabella `customer` ha colonne decise una volta per tutte. In un database NoSQL si memorizza un record con chiave `customerId = 99` e, insieme alla chiave, un documento di attributi che il database non verifica al momento dell'inserimento. Il valore di un attributo può essere a sua volta un documento gerarchico, con mappe e liste annidate.

**La scalabilità.** Un database relazionale tipicamente lavora su un singolo nodo principale. Per scalare gli si danno più CPU, memoria e disco, e si possono affidare le letture a una *read replica*. Scalare un database relazionale è spesso un'arte delicata. I database NoSQL sono invece progettati fin dall'inizio per scalare **orizzontalmente**: i dati vengono suddivisi in **partizioni**, distribuite su nodi diversi del cluster. Da qui derivano le loro prestazioni.

**Le transazioni.** In settori come quello finanziario è essenziale poter bloccare i dati ed eseguire operazioni in transazioni con garanzie **ACID** (atomicità, consistenza, isolamento, durabilità), così che il database sia sempre in uno stato coerente. Nei database NoSQL il supporto varia. Spesso si ha un concetto più semplice, come "aggiorna tutti questi record: o vanno tutti a buon fine o non ne va a buon fine nessuno". DynamoDB, per esempio, offre transazioni su più item con le API `TransactWriteItems` e `TransactGetItems`.

**La consistenza.** Il **teorema CAP** afferma che un sistema distribuito non può garantire contemporaneamente *consistency*, *availability* e *partition tolerance*. Un database relazionale su nodo singolo offre agevolmente **consistenza forte**. In un database NoSQL distribuito un record appena scritto potrebbe non essere ancora stato replicato su tutti i nodi. Leggendolo con **consistenza eventuale** si potrebbe ottenere un dato vecchio di qualche centinaio di millisecondi, e per molte applicazioni non è un problema. Quando invece serve, si può richiedere una **lettura fortemente consistente**: la risposta arriva solo quando i nodi concordano sul valore più recente.

### Perché scegliere Amazon DynamoDB

Diverse ragioni rendono DynamoDB una scelta frequente per le applicazioni moderne:

- **Prestazioni su larga scala**: DynamoDB è progettato per offrire tempi di risposta nell'ordine dei **millisecondi a una cifra** a qualunque scala.
- **Pronto per l'enterprise**: è usato da moltissime grandi aziende e offre crittografia, backup, ripristino point-in-time e replica globale.
- **Query a bassa latenza**: il modello di interrogazione, basato sulla chiave, si presta a moltissimi carichi di lavoro.
- **Serverless e completamente gestito**: gira sull'infrastruttura AWS senza server da aggiornare o installare. Lo sviluppatore lo usa tramite un'API di alto livello.
- **Controllo degli accessi granulare**: con le policy IAM si può stabilire che cosa può fare ogni principal sulla tabella, fino al livello dei **singoli attributi**. Per esempio si può consentire a una policy di leggere solo alcuni attributi di una tabella.

## 6.3 I concetti chiave di Amazon DynamoDB

### Tabelle, item e attributi

In DynamoDB i dati vivono in **tabelle**. Una tabella contiene **item**, l'equivalente dei record, e ogni item è composto da **attributi**. Ogni item è identificato in modo univoco dalla sua **chiave primaria**. Oltre alla chiave, un item può avere un numero arbitrario di attributi, e item diversi della stessa tabella possono avere attributi diversi.

Consideriamo un item della tabella delle note di Polly Notes:

```json
{
  "UserId":   "StudentA",
  "NoteId":   13,
  "Note":     "Ripassare i GSI",
  "Favorite": "Yes",
  "Active":   true,
  "Year":     2022,
  "Tags":     ["dynamodb", "esame"]
}
```

La combinazione `UserId` + `NoteId` costituisce la chiave primaria e deve essere univoca nella tabella. Tutti gli altri attributi sono facoltativi: DynamoDB **non** ne impone la presenza. Ne consegue che è il **codice** a doversi far carico della validazione. Leggendo un item bisogna verificare che l'attributo `Note` esista davvero prima di usarlo, oppure il team deve stabilire una convenzione condivisa su che cosa si memorizza.

### La chiave primaria: partition key e sort key

Alla creazione di una tabella si sceglie il tipo di chiave primaria. Ci sono due possibilità.

La **chiave semplice** è composta dalla sola **partition key**.

La **chiave composita** è composta da **partition key** e **sort key**.

La **partition key** determina **in quale partizione** viene memorizzato un item. DynamoDB applica una funzione di hash al suo valore, e tutti gli item con la stessa partition key finiscono nella stessa partizione. Se immaginiamo la tabella come una pizza divisa in fette, tutti gli item con `UserId = StudentA` stanno nella stessa fetta. La cosa diventa importante quando si interrogano i dati.

DynamoDB funziona meglio quando la partition key ha **alta entropia**, cioè molti valori diversi usati in modo uniforme, così che i dati si distribuiscano equamente tra le partizioni. Una partition key con pochi valori, o con un valore molto più "popolare" degli altri, concentra il traffico su poche partizioni.

La **sort key**, facoltativa, determina **l'ordine** degli item all'interno di una partizione e abilita, in fase di query, operatori aggiuntivi come maggiore di, compreso tra e "inizia con".

### I tipi di dato

Gli attributi possono contenere diversi tipi di dato:

- **scalari**: **String**, **Number**, **Boolean**, **Binary** (dati binari) e Null;
- **documento**: **Map**, cioè coppie chiave-valore come un dizionario, e **List**, una collezione ordinata simile a un array;
- **insiemi**: String Set, Number Set e Binary Set.

Mappe e liste sono **ricorsive**: il valore associato a una chiave di una mappa può essere un'altra mappa o un'altra lista, e così via. Ne risulta esattamente la struttura di un documento **JSON**. Tutto ciò che si può esprimere in JSON si può memorizzare in un attributo DynamoDB. Questa caratteristica è alla base dell'interfaccia *document* degli SDK, presentata nel paragrafo 6.7.

### La capacità di lettura e scrittura

Le prestazioni di una tabella in modalità *provisioned* si configurano con due grandezze:

- **Read Capacity Unit (RCU)**: **1 RCU** consente **una lettura fortemente consistente al secondo** di un item fino a **4 KB**, oppure **due letture eventualmente consistenti al secondo** dello stesso item. Le letture eventualmente consistenti consumano infatti **metà** della capacità.
- **Write Capacity Unit (WCU)**: **1 WCU** consente **una scrittura al secondo** di un item fino a **1 KB**.

> **Box – Calcolare RCU e WCU**
>
> I calcoli di capacità sono un classico delle domande d'esame. Le dimensioni si arrotondano sempre per eccesso al blocco successivo, di 4 KB per le letture e di 1 KB per le scritture.
>
> - *Leggere 10 item al secondo da 6 KB con consistenza forte*: 6 KB si arrotonda a 8 KB, cioè 2 blocchi da 4 KB. Si ottiene 10 × 2 = **20 RCU**.
> - *Stessa lettura con consistenza eventuale*: 20 / 2 = **10 RCU**.
> - *Scrivere 5 item al secondo da 2,5 KB*: 2,5 KB si arrotonda a 3 KB, cioè 3 WCU per item. Si ottiene 5 × 3 = **15 WCU**.
>
> Le letture e le scritture **transazionali** consumano il doppio delle unità: 2 RCU per 4 KB letti e 2 WCU per 1 KB scritto.

La scelta della capacità può sembrare delicata, ma non è definitiva. La capacità si può **modificare** in seguito e si può affidare all'**auto scaling**. Esistono anche diverse **modalità di fatturazione**: oltre alla modalità *provisioned* c'è la modalità **on-demand**, in cui si paga per richiesta senza pianificare la capacità (Capitolo 7).

### Scan e query

Ci sono due modi per leggere più item da una tabella.

Una **scan** legge **l'intera tabella**, item per item, ed eventualmente filtra i risultati. È un'operazione **costosa**, perché consuma capacità in proporzione alla dimensione della tabella, non a quella dei risultati.

Una **query** legge gli item che hanno un determinato **valore della partition key**, con eventuali condizioni sulla sort key. Va direttamente alla partizione giusta ed è quindi molto più efficiente.

La regola generale è **interrogare per chiave**. Ma che cosa succede se serve interrogare i dati in base a un attributo che non fa parte della chiave primaria?

### Gli indici secondari

Un **indice secondario** fornisce una **chiave alternativa** con cui interrogare i dati. Oltre alla nuova chiave, l'indice può **proiettare** solo un sottoinsieme degli attributi della tabella: se una query ha bisogno di due attributi soltanto, l'indice può contenere solo quelli, riducendo spazio e costi. Ci sono due tipi di indice.

Un **Local Secondary Index (LSI)** ha la **stessa partition key** della tabella e una **sort key diversa**. Se la tabella delle note ha chiave `UserId` + `NoteId`, un LSI con chiave `UserId` + `Favorite` permette di trovare in modo efficiente le note preferite di un utente. Gli LSI hanno alcune caratteristiche distintive:

- si creano **solo al momento della creazione della tabella**;
- poiché condividono la partition key, i loro dati sono memorizzati **nella stessa partizione** degli item della tabella;
- di conseguenza **consumano la capacità della tabella** (RCU e WCU);
- sono aggiornati in modo **sincrono** e sono sempre allineati alla tabella, quindi supportano anche letture fortemente consistenti.

Un **Global Secondary Index (GSI)** può avere una **chiave completamente diversa**, sia partition key sia sort key. Per esempio `NoteId` + `UserId`, invertendo la chiave della tabella. Un GSI "ri-chiava" i dati per rendere efficienti nuovi tipi di query. Anche il GSI ha caratteristiche proprie:

- si può creare **in qualunque momento**, anche dopo la tabella;
- può essere **sparso**: contiene solo gli item che possiedono gli attributi della sua chiave;
- ha una **propria capacità** (RCU e WCU) separata da quella della tabella;
- i suoi dati sono memorizzati **separatamente** dalla tabella, perché partizionati su una chiave diversa;
- è aggiornato in modo **asincrono**, quindi le letture sono solo eventualmente consistenti.

| | **LSI** | **GSI** |
|---|---|---|
| Partition key | Uguale alla tabella | Qualsiasi |
| Sort key | Diversa da quella della tabella | Qualsiasi (facoltativa) |
| Creazione | Solo con la tabella | In qualunque momento |
| Capacità | Condivisa con la tabella | Propria |
| Aggiornamento | Sincrono | Asincrono |
| Letture fortemente consistenti | Sì | No |

### Partizioni e adaptive capacity

Dietro le quinte i dati di una tabella sono distribuiti su più **partizioni**. È un dettaglio di implementazione gestito da AWS e invisibile allo sviluppatore. In passato la capacità della tabella veniva divisa in parti uguali tra le partizioni: una tabella con 400 WCU su quattro partizioni offriva 100 WCU a ciascuna. Se una partizione riceveva più traffico delle altre, diventava un collo di bottiglia, la cosiddetta **hot partition**, anche se la tabella nel complesso aveva capacità inutilizzata.

DynamoDB ha migliorato questo comportamento in due modi. L'**adaptive capacity**, attiva per default su tutte le tabelle, ridistribuisce automaticamente la capacità verso le partizioni più sollecitate. La **burst capacity** permette di accumulare la capacità non utilizzata negli **ultimi cinque minuti**: se le partizioni ne hanno consumate solo 50 su 100, la differenza resta "in banca" e può essere spesa da una partizione che riceve un picco.

Queste funzionalità non esonerano dal progettare bene le chiavi. Se si partiziona per mese, e dicembre è il periodo in cui tutti caricano i dati fiscali, la partizione di dicembre riceverà molto più traffico delle altre. Una buona entropia della partition key resta il fondamento delle prestazioni.

## 6.4 Laboratorio: Amazon DynamoDB in azione

Una piccola applicazione Python mette in pratica i concetti visti. I concetti valgono identici per Java e .NET: cambiano solo i dettagli sintattici dell'SDK.

L'applicazione crea una tabella `CityPopulation` con le città degli Stati Uniti:

```python
dynamodb.create_table(
    TableName="CityPopulation",
    KeySchema=[
        {"AttributeName": "StateCode", "KeyType": "HASH"},   # partition key
        {"AttributeName": "CityName",  "KeyType": "RANGE"},  # sort key
    ],
    AttributeDefinitions=[
        {"AttributeName": "StateCode",   "AttributeType": "S"},
        {"AttributeName": "CityName",    "AttributeType": "S"},
        {"AttributeName": "AirportCode", "AttributeType": "S"},
    ],
    LocalSecondaryIndexes=[{
        "IndexName": "StateAirportIndex",
        "KeySchema": [
            {"AttributeName": "StateCode",   "KeyType": "HASH"},
            {"AttributeName": "AirportCode", "KeyType": "RANGE"},
        ],
        "Projection": {"ProjectionType": "ALL"},
    }],
    GlobalSecondaryIndexes=[{
        "IndexName": "AirportIndex",
        "KeySchema": [{"AttributeName": "AirportCode", "KeyType": "HASH"}],
        "Projection": {"ProjectionType": "ALL"},
        "ProvisionedThroughput": {"ReadCapacityUnits": 5, "WriteCapacityUnits": 5},
    }],
    ProvisionedThroughput={"ReadCapacityUnits": 5, "WriteCapacityUnits": 5},
)
```

Il **key schema** della tabella è `StateCode` + `CityName`. Le **attribute definitions** dichiarano il tipo dei soli attributi usati nelle chiavi, della tabella o degli indici. L'**LSI** usa `StateCode` + `AirportCode` come chiave alternativa con la stessa partition key. Il **GSI** usa la sola `AirportCode` e ha una capacità propria. La tabella viene poi popolata con circa 500 città.

Si eseguono quindi quattro interrogazioni, misurando ogni volta la capacità consumata con il parametro `ReturnConsumedCapacity`.

**1. Scan con filtro**: tutte le città chiamate Portland.

```python
table.scan(
    FilterExpression=Attr("CityName").eq("Portland"),
    ReturnConsumedCapacity="TOTAL",
)
```

Consumo: **1,5 unità**. La scan legge tutta la tabella e poi scarta gli item che non corrispondono. Con 500 città il costo è contenuto, ma se la tabella contenesse tutte le città del mondo il consumo crescerebbe in proporzione, rendendo l'operazione inefficiente.

**2. Query sulla tabella**: le città dell'Oregon.

```python
table.query(
    KeyConditionExpression=Key("StateCode").eq("OR"),
    ReturnConsumedCapacity="TOTAL",
)
```

Risultato: Eugene, Portland, Salem. Consumo: **0,5 unità**. La query va dritta alla partizione `OR`.

**3. Query sull'LSI**: le città dell'Oregon servite dall'aeroporto KPDX.

```python
table.query(
    IndexName="StateAirportIndex",
    KeyConditionExpression=Key("StateCode").eq("OR") & Key("AirportCode").eq("KPDX"),
    ReturnConsumedCapacity="TOTAL",
)
```

Consumo: **0,5 unità**. L'LSI condivide la partition key `StateCode`, ma permette di filtrare per aeroporto.

**4. Query sul GSI**: tutte le città servite da KPDX, **senza** specificare lo Stato.

```python
table.query(
    IndexName="AirportIndex",
    KeyConditionExpression=Key("AirportCode").eq("KPDX"),
    ReturnConsumedCapacity="TOTAL",
)
```

Consumo: **0,5 unità**. Il risultato contiene un'informazione nuova: **Vancouver, nello Stato di Washington**, servita anch'essa dall'aeroporto di Portland. Le query precedenti non potevano trovarla, perché richiedevano lo Stato come partition key. Il GSI, ripartizionato per aeroporto, risponde a un tipo di domanda che la tabella non poteva soddisfare senza una scan.

## 6.5 Le modalità di accesso a DynamoDB per lo sviluppatore

Lo sviluppatore ha molti modi per lavorare con DynamoDB:

- la **AWS Management Console**, per creare tabelle e indici ed eseguire interrogazioni;
- **DynamoDB Local**, un'implementazione di DynamoDB eseguibile in locale per lo sviluppo e i test;
- **PartiQL**, un linguaggio in stile SQL per interrogare DynamoDB;
- la **AWS CLI**, con i comandi `aws dynamodb` (Capitolo 7);
- gli **SDK**, che offrono più interfacce: quella di basso livello, quella *document* e quella a oggetti (paragrafo 6.7).

### La Console

Nella Console la tabella `CityPopulation` mostra molte informazioni. La scheda **Monitor** riporta le metriche di lettura e scrittura nel tempo. Si vede che durante il caricamento iniziale la capacità provisionata era stata superata, ma la **burst capacity** ha assorbito il picco senza errori.

La funzione **Explore table items** permette di eseguire scan e query scegliendo la tabella o uno degli indici. La Console conosce la chiave della tabella e chiede il valore di `StateCode`. Inserendo `OR` si ottengono le città dell'Oregon con un consumo di 0,5 RCU. Con `MA` si ottengono Boston, Springfield e Worcester. Sulla sort key sono disponibili operatori come *greater than*, *between* e *begins with*. Quest'ultimo è utile con le sort key di tipo stringa.

### PartiQL

**PartiQL** permette di interrogare DynamoDB con una sintassi praticamente identica a SQL:

```sql
SELECT * FROM CityPopulation WHERE CityName = 'Portland'
```

C'è però un'avvertenza importante, riportata anche nella documentazione: per evitare che una `SELECT` si traduca in una **scansione completa della tabella**, la clausola `WHERE` deve contenere una condizione sulla **partition key**. PartiQL somiglia a SQL ma sotto c'è sempre DynamoDB: una `WHERE` su un attributo che non fa parte della chiave diventa una scan, con tutti i costi che ne derivano. Conviene quindi essere consapevoli del tipo di operazione che ogni istruzione PartiQL genera.

### DynamoDB Local

**DynamoDB Local** è una versione di DynamoDB eseguibile sul proprio computer, scaricabile oppure disponibile come **container Docker**:

```bash
docker run -p 8000:8000 amazon/dynamodb-local
```

Il servizio risponde sulla porta **8000**. Nel codice basta cambiare l'**endpoint URL** per indirizzare le chiamate all'istanza locale:

```python
dynamodb = boto3.resource("dynamodb", endpoint_url="http://localhost:8000")
```

Rieseguendo l'applicazione, la creazione della tabella e il caricamento dei dati avvengono in pochi istanti, senza costi e senza bisogno di connessione ad AWS. È lo strumento ideale per lo sviluppo e per i test automatici.

## 6.6 Laboratorio: NoSQL Workbench

**NoSQL Workbench** è un'applicazione desktop multipiattaforma per lavorare con DynamoDB in locale. L'esperienza è simile a quella della Console, ma dal proprio computer. Ci si collega con un profilo AWS configurato, per esempio verso la Region `us-west-2`, e compare la tabella `CityPopulation` con il suo GSI e il suo LSI.

Lo strumento offre diverse funzioni utili:

- un editor **PartiQL**, dove eseguire istruzioni come `SELECT * FROM CityPopulation WHERE StateCode = 'CA'` e vederne subito i risultati;
- la **generazione di codice** in Python, Node.js e Java a partire dalle operazioni eseguite;
- l'**operation builder**, un'interfaccia guidata per le operazioni. Comprende sia quelle di *control plane* (creare, aggiornare, eliminare tabelle) sia quelle sui dati. Per esempio, una query sulla tabella con `StateCode = NV` e nessuna condizione sulla sort key restituisce le città del Nevada con i loro aeroporti: Las Vegas, North Las Vegas, Henderson e Reno.

Anche dall'operation builder si può generare il codice corrispondente. Il codice Python prodotto include commenti di documentazione, la creazione del client e la query già impostata: un ottimo punto di partenza per l'applicazione.

> **Box – NoSQL Workbench come strumento di progettazione**
>
> Oltre a interrogare tabelle esistenti, NoSQL Workbench include un **data modeler**: si possono progettare tabelle e indici, popolarli con dati di esempio e visualizzare come i diversi access pattern verrebbero soddisfatti, prima ancora di creare una risorsa su AWS. È particolarmente utile per il *single-table design* discusso nel Capitolo 7.

## 6.7 Programmare con Amazon DynamoDB

Gli SDK offrono **tre interfacce** per lavorare con DynamoDB. Tutte e tre finiscono per chiamare la **stessa API REST**: cambia solo il livello di astrazione con cui lo sviluppatore scrive il codice.

L'**interfaccia di basso livello** ha una corrispondenza uno a uno con le API del servizio. È quella usata negli esempi Python di questo capitolo. Gli oggetti di richiesta e risposta somigliano molto ai documenti che viaggiano sul filo, compresi i **descrittori di tipo** degli attributi.

L'**interfaccia document** permette di lavorare con **documenti** in stile JSON: chiavi e valori, dove un valore può essere a sua volta una mappa o una lista. Non è necessario specificare i descrittori di tipo, perché l'SDK li deduce.

L'**interfaccia a oggetti** (*object persistence*, o *high-level*) permette di lavorare con le **classi** del proprio linguaggio. Si definisce per esempio una classe `Person` e la si **annota** per indicare come i suoi campi corrispondono agli attributi della tabella. Si istanzia l'oggetto come qualunque altro e lo si passa a un "contesto" DynamoDB che sa come persisterlo, leggerlo, aggiornarlo e interrogarlo. In Java è il ruolo del DynamoDB Enhanced Client, in .NET del `DynamoDBContext`.

Le interfacce document e a oggetti sono **astrazioni** costruite sopra quella di basso livello. La scelta dipende dallo stile di programmazione preferito. Molti sviluppatori trovano più naturale lavorare direttamente con oggetti e classi. Il Capitolo 7 approfondisce queste interfacce con esempi in Java.

## 6.8 Richieste e risposte

### La richiesta

Lavorare con l'interfaccia di basso livello significa manipolare oggetti molto simili a quelli che viaggiano sulla rete. Una richiesta `GetItem` ha questo aspetto:

```http
POST / HTTP/1.1
Host: dynamodb.us-west-2.amazonaws.com
Content-Length: 85
User-Agent: aws-sdk-java/2.x
Content-Type: application/x-amz-json-1.0
Authorization: AWS4-HMAC-SHA256 Credential=..., SignedHeaders=..., Signature=...
X-Amz-Target: DynamoDB_20120810.GetItem

{
  "TableName": "Notes",
  "Key": {
    "UserId": { "S": "StudentA" },
    "NoteId": { "N": "13" }
  }
}
```

È una **POST HTTP** con le intestazioni consuete. Ci sono l'host, la lunghezza del contenuto, lo *user agent*, il tipo di contenuto e, come per quasi tutte le API AWS, l'intestazione **Authorization** con la firma SigV4. L'intestazione **`X-Amz-Target`** indica l'**operazione** richiesta, qui `GetItem`. Il **payload** contiene il nome della tabella e la chiave dell'item.

### La risposta

La risposta contiene lo **status code**, il **RequestId**, un **checksum CRC32**, il tipo e la lunghezza del contenuto, la data e il payload:

```json
{
  "Item": {
    "UserId":   { "S": "StudentA" },
    "NoteId":   { "N": "13" },
    "Note":     { "S": "Ripassare i GSI" },
    "Favorite": { "S": "Yes" }
  }
}
```

Il payload è un documento JSON in cui ogni attributo è accompagnato dal suo **descrittore di tipo**: `S` per le stringhe, `N` per i numeri, che viaggiano come stringhe per non perdere precisione, `BOOL` per i booleani, `L` per le liste, `M` per le mappe e `B` per i dati binari.

È raro dover guardare questi dettagli, perché l'SDK fa tutto il necessario. Diventano però preziosi in due situazioni: durante il **debug e il tracing** di un problema e quando si deve **scrivere una policy IAM** per un'applicazione che non si conosce bene. Sapere quali operazioni compaiono in `X-Amz-Target` dice esattamente quali permessi concedere, come visto nel laboratorio del Capitolo 3.

### Gli errori

Come ogni servizio HTTP, DynamoDB segnala gli errori con gli status code. Gli errori **4xx** indicano un problema nella richiesta del client. Gli errori **5xx** indicano un problema lato servizio. Un esempio tipico di 4xx è l'accesso a una tabella inesistente:

```json
HTTP/1.1 400 Bad Request

{
  "__type": "com.amazonaws.dynamodb.v20120810#ResourceNotFoundException",
  "message": "Requested resource not found: Table: UserNote not found"
}
```

L'eccezione **`ResourceNotFoundException`** è un errore del client: il client ha chiesto una tabella che il servizio non conosce. Altri errori 4xx frequenti sono `ProvisionedThroughputExceededException`, quando si supera la capacità, `ConditionalCheckFailedException`, quando una condizione di scrittura non è soddisfatta (Capitolo 7), e `AccessDeniedException`, quando mancano i permessi IAM.

## 6.9 Verifica dei concetti

**"I database relazionali non hanno uno schema fisso: record diversi possono avere attributi diversi."** *Falso.* Nei database relazionali lo schema è fisso. Un campo si può rendere *nullable*, ma la colonna esiste comunque per ogni riga, e omettere un campo obbligatorio genera un errore. È nei database NoSQL che gli item possono avere attributi diversi.

**"Amazon DynamoDB memorizza i dati in item e suddivide gli item di una tabella in più partizioni in base al valore della *sort key*."** *Falso.* La suddivisione in partizioni avviene in base alla **partition key**. La sort key determina l'ordine degli item all'interno di una partizione.

**"Ogni attributo DynamoDB ha un nome, un tipo di dato e un valore, e un item può avere un numero qualsiasi di attributi."** *Vero.* La chiave o le chiavi sono obbligatorie, mentre per il resto DynamoDB non impone uno schema fisso.

**"Una Read Capacity Unit (RCU) corrisponde al numero di letture fortemente consistenti al secondo di item fino a 4 KB."** *Vero.* È un valore da ricordare per l'esame: 4 KB per le letture, 1 KB per le scritture, e due letture eventualmente consistenti per RCU.

**"Sviluppare con Amazon DynamoDB richiede di testare le applicazioni accedendo al servizio web DynamoDB."** *Falso.* Si può testare anche su una tabella di sviluppo nel cloud, ma esiste **DynamoDB Local**, scaricabile o eseguibile in un container Docker, che permette di sviluppare e testare interamente sul proprio computer.

**"Con l'interfaccia document dell'SDK per DynamoDB non è necessario specificare i descrittori di tipo dei dati."** *Vero.* Con l'interfaccia di basso livello bisogna indicare il tipo di ogni attributo, mentre l'interfaccia document lo deduce. Nell'interfaccia a oggetti l'informazione sui tipi è contenuta nelle **annotazioni** delle classi anziché nel codice che esegue le operazioni.
