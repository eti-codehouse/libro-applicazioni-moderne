# Capitolo 7 – Operare su DynamoDB dal codice

## 7.1 Obiettivi del capitolo

Questo capitolo prosegue lo studio di DynamoDB e si concentra sulle API che uno sviluppatore usa ogni giorno. Si vedrà come scrivere programmi che usano DynamoDB tramite gli AWS SDK, per **creare, modificare ed eliminare tabelle** e per eseguire le operazioni **CRUD** (*create, read, update, delete*) sugli item. Si presenteranno le **best practice** di progettazione e di sviluppo, perché DynamoDB ha alcune peculiarità che uno sviluppatore deve conoscere. Si chiuderà con le opzioni di **caching** per migliorarne ulteriormente le prestazioni.

Le operazioni su DynamoDB si dividono in due categorie. Le **operazioni di control plane** agiscono sulle **tabelle**: creazione, aggiornamento, cancellazione, descrizione ed elenco. Le **operazioni di data plane** agiscono sugli **item**: inserimento, aggiornamento, lettura e cancellazione. Entrambe si possono eseguire dal codice, dalla CLI o dalla Console.

Prima di scrivere qualunque riga di codice, però, c'è un passo che con DynamoDB è più importante di tutti gli altri: la **pianificazione**.

## 7.2 Progettare le tabelle

### Un approccio rovesciato rispetto al relazionale

In un database relazionale come MySQL si hanno molte tabelle legate da chiavi esterne, e le query possono unire con le *join* i dati di più tabelle. Si progetta prima lo schema e poi si scrivono le query di cui l'applicazione ha bisogno.

In DynamoDB le cose funzionano diversamente. Non c'è un database che raccoglie tabelle collegate tra loro: ci sono **tabelle indipendenti**, ciascuna con la sua partition key, l'eventuale sort key e gli altri attributi. Le **query** sono possibili solo sulle chiavi. Per arrivare ai dati negli altri attributi bisogna ricorrere a una **scan**, meno efficiente e quindi più costosa. Per interrogare altri attributi si possono creare **indici secondari**: un **LSI** sceglie una nuova sort key, un **GSI** una nuova partition key e una nuova sort key. Ma gli LSI si possono creare **solo insieme alla tabella**.

La conseguenza è decisiva: bisogna **conoscere le query dell'applicazione prima di creare la tabella**. Se non si conoscono gli *access pattern*, cioè i modi in cui l'applicazione leggerà e scriverà i dati, non si può progettare una tabella che li serva tutti in modo efficiente. Prima di passare alla costruzione bisogna quindi capire tre cose:

- **quali dati** si memorizzano;
- **quanto** si leggono e si scrivono, cioè volumi e frequenze;
- **quali sono gli access pattern** dell'applicazione.

Rispetto al mondo relazionale l'ordine è invertito: **prima si definiscono gli access pattern, poi si progettano tabella e indici**.

### Scegliere la partition key

La partition key incide sia sulle **prestazioni** sia sugli **access pattern** possibili. Le regole per sceglierla sono due.

La prima: la partition key deve essere un valore **comune agli access pattern** e **noto all'applicazione** nel momento in cui interroga i dati. In Polly Notes l'access pattern principale è *"recupera tutte le note di un utente"*. Una tabella con `NoteId` come partition key non lo serve, perché l'applicazione parte dall'utente, non dalla singola nota. La scelta giusta è **`UserId` come partition key**. Poiché un utente ha molte note, `UserId` da solo non identifica un item in modo univoco. Si usa quindi una **chiave composita**: `UserId` (partition key) + `NoteId` (sort key). La combinazione dei due valori identifica univocamente ogni nota.

La seconda: la partition key deve avere un'**alta cardinalità**, cioè un alto grado di unicità. La partition key determina la partizione fisica in cui sono memorizzati i dati, e valori numerosi e ben distribuiti spalmano il carico su tutte le partizioni, evitando le *hot partition*.

### Progettare gli indici

Definita la chiave della tabella, si progettano gli indici per gli altri access pattern. Se un'esigenza comune è *"mostra le note preferite di un utente"*, si crea un indice secondario con chiave `UserId` + `Favorite`. Sono decisioni da prendere in anticipo, durante la pianificazione.

### Sort key composite

La sort key si può usare in modo creativo, costruendo una **sort key composita** (o *sovraccaricata*) che codifica una gerarchia. In una tabella di indirizzi si può usare il nome della persona come partition key e una sort key del tipo:

```
USA#Washington#King#Seattle
ITA#Lombardia#Milano#Milano
```

Gli operatori sulla sort key, in particolare `begins_with`, permettono poi di interrogare a qualunque livello della gerarchia: tutti gli indirizzi negli Stati Uniti (`USA#`), nello Stato di Washington (`USA#Washington#`) e così via.

### La modalità di capacità

Alla creazione della tabella si sceglie anche la **modalità di capacità**.

La modalità **provisioned** richiede di indicare quante RCU e WCU servono. È adatta a un **traffico prevedibile**: se si sa quanto si legge e si scrive in ogni momento, è l'opzione più economica. La capacità è però limitata a quanto provisionato. Si può attivare l'auto scaling, ma resta necessario conoscere approssimativamente il carico.

La modalità **on-demand** non richiede pianificazione: la tabella accetta il traffico che arriva e si paga a consumo, per richiesta. È la scelta migliore per un **traffico imprevedibile** o discontinuo.

> **Box – Single-table design**
>
> Una tecnica avanzata ma molto diffusa è il **single-table design**: invece di una tabella per ogni entità, come si farebbe in un database relazionale, si memorizzano **entità diverse nella stessa tabella**. Si usano chiavi generiche (`PK`, `SK`) e valori prefissati (`USER#123`, `NOTE#42`), così che una sola query possa restituire insieme un utente e le sue note. Il vantaggio è servire molti access pattern con poche query efficienti, evitando le scan e le "join" eseguite nel codice applicativo. Il prezzo è una progettazione più impegnativa, che richiede una conoscenza precisa degli access pattern, ancora una volta da definire prima di creare la tabella.

## 7.3 Lavorare con le tabelle in Java

L'esempio seguente mostra il ciclo di vita completo di una tabella con l'SDK for Java 2.x: elenco, creazione, aggiornamento e cancellazione.

```java
DynamoDbClient ddb = DynamoDbClient.builder().build();
String tableName = "notes2";
String key = "studentId";
```

Il client viene creato con il builder senza personalizzazioni, quindi con Region e credenziali predefinite.

### Elencare le tabelle

```java
static void listAllTables(DynamoDbClient ddb) {
    ListTablesRequest request = ListTablesRequest.builder().build();
    ListTablesResponse response = ddb.listTables(request);
    List<String> tableNames = response.tableNames();

    if (tableNames.size() > 0) {
        tableNames.forEach(System.out::println);
    } else {
        System.out.println("No tables found!");
    }
}
```

`ListTablesRequest` non richiede parametri. La risposta espone `tableNames()`, la lista dei nomi delle tabelle nella Region.

### Creare una tabella con un waiter

```java
static boolean createTable(DynamoDbClient ddb, String tableName, String key) {
    DynamoDbWaiter waiter = ddb.waiter();

    CreateTableRequest request = CreateTableRequest.builder()
            .attributeDefinitions(AttributeDefinition.builder()
                    .attributeName(key)
                    .attributeType(ScalarAttributeType.S)
                    .build())
            .keySchema(KeySchemaElement.builder()
                    .attributeName(key)
                    .keyType(KeyType.HASH)
                    .build())
            .provisionedThroughput(ProvisionedThroughput.builder()
                    .readCapacityUnits(10L)
                    .writeCapacityUnits(10L)
                    .build())
            .tableName(tableName)
            .build();

    ddb.createTable(request);

    DescribeTableRequest describe = DescribeTableRequest.builder()
            .tableName(tableName)
            .build();
    WaiterResponse<DescribeTableResponse> waiterResponse = waiter.waitUntilTableExists(describe);
    waiterResponse.matched().response().ifPresent(System.out::println);
    return true;
}
```

Alcuni dettagli sono importanti.

Le **attribute definitions** dichiarano solo gli attributi che compongono le chiavi. Il resto dello schema non va definito, perché DynamoDB è *schemaless* al di fuori della chiave. Il tipo `ScalarAttributeType.S` indica una stringa.

Il **key schema** usa i termini storici di DynamoDB: **`HASH`** corrisponde alla **partition key** e **`RANGE`** alla **sort key**. Sono sinonimi che si incontrano spesso. Ogni item inserito nella tabella **deve** contenere gli attributi del key schema, quindi conviene scegliere attributi che ogni item possiede davvero.

La **capacità** è impostata a 10 RCU e 10 WCU in modalità provisioned.

La creazione di una tabella richiede qualche secondo prima che passi allo stato `ACTIVE`. Invece di scrivere un ciclo con `DescribeTable`, si usa il **waiter** integrato nell'SDK: `waitUntilTableExists`.

### Aggiornare la tabella senza waiter

Supponiamo di aver sovrastimato il traffico e di voler dimezzare la capacità:

```java
static void updateTable(DynamoDbClient ddb, String tableName) throws InterruptedException {
    ProvisionedThroughput throughput = ProvisionedThroughput.builder()
            .readCapacityUnits(5L)
            .writeCapacityUnits(5L)
            .build();

    UpdateTableRequest request = UpdateTableRequest.builder()
            .tableName(tableName)
            .provisionedThroughput(throughput)
            .build();
    ddb.updateTable(request);

    DescribeTableRequest describe = DescribeTableRequest.builder().tableName(tableName).build();
    while (true) {
        String status = ddb.describeTable(describe).table().tableStatusAsString();
        System.out.println(status);
        if ("ACTIVE".equals(status)) break;
        Thread.sleep(1000);
    }
}
```

Per l'aggiornamento di una tabella **non esiste un waiter** integrato. Bisogna quindi scrivere un ciclo che descriva la tabella e ne controlli lo stato con `tableStatusAsString()`, finché non torna `ACTIVE`. Il controllo è necessario: se si tentasse di cancellare la tabella mentre è in stato `UPDATING`, si otterrebbe un errore.

### Eliminare la tabella

```java
static void deleteTable(DynamoDbClient ddb, String tableName) {
    ddb.deleteTable(DeleteTableRequest.builder().tableName(tableName).build());
    System.out.println(tableName + " was successfully deleted!");
}
```

All'esecuzione l'output mostra in sequenza la creazione della tabella, con il programma in attesa finché non diventa attiva, e la `DescribeTableResponse`. Poi una serie di stati `UPDATING` seguiti da `ACTIVE`, e infine la conferma della cancellazione.

## 7.4 Caricare e leggere gli item

### PutItem: creare o sostituire

L'API **PutItem** inserisce un item:

```bash
aws dynamodb put-item \
    --table-name Notes \
    --item '{"UserId": {"S": "StudentD"}, "NoteId": {"N": "42"}, "Note": {"S": "Test note"}}'
```

Il comportamento va compreso bene. Se non esiste un item con quella chiave, PutItem lo **crea**. Se esiste già, lo **sostituisce interamente** con quello nuovo. Si rischia quindi di perdere dati senza accorgersene: gli attributi del vecchio item che non compaiono nel nuovo vengono cancellati.

### BatchWriteItem: scritture multiple

**BatchWriteItem** permette di inserire o cancellare **fino a 25 item** con una sola richiesta. È utile per esempio per popolare una tabella in fase di sviluppo o di proof of concept, senza inviare decine di richieste separate. Il comando riceve un file che descrive le operazioni:

```bash
aws dynamodb batch-write-item --request-items file://items.json
```

La risposta indica quali item **non sono stati elaborati**, nel campo `UnprocessedItems`, per esempio perché una partizione ha esaurito la capacità. Questi item vanno ritentati dal codice.

In alternativa esiste una funzionalità più recente: l'**importazione da S3**. Si carica un file in un bucket e lo si importa in una nuova tabella DynamoDB. Per popolare una tabella in sviluppo può essere più semplice. All'interno di uno script o di un programma, BatchWriteItem resta la scelta naturale.

### GetItem: leggere un item

**GetItem** restituisce **un singolo item** e richiede la **chiave completa**: partition key e, se la tabella ne ha una, sort key.

```bash
aws dynamodb get-item \
    --table-name Notes \
    --key '{"UserId": {"S": "StudentA"}, "NoteId": {"N": "11"}}'
```

Per default GetItem esegue una lettura **eventualmente consistente**. Si può richiedere una lettura **fortemente consistente** con `--consistent-read`.

### Query: leggere una collezione di item

Per recuperare tutte le note di uno studente si usa **Query**:

```bash
aws dynamodb query \
    --table-name Notes \
    --key-condition-expression "UserId = :userid" \
    --expression-attribute-values '{":userid": {"S": "StudentA"}}'
```

La **key condition expression** contiene le condizioni sulle chiavi. `:userid` è un **segnaposto**, il cui valore effettivo è fornito in `--expression-attribute-values`. Sulla partition key è ammesso solo l'operatore **di uguaglianza** (`=`). Sulla sort key si può aggiungere una seconda condizione con uno di questi operatori:

| Operatore | Esempio |
|---|---|
| `=`, `<`, `<=`, `>`, `>=` | `NoteId > :n` |
| `BETWEEN` | `NoteId BETWEEN :a AND :b` |
| `begins_with` | `begins_with(Address, :prefix)`, per esempio `"USA#Washington"` |

Conoscere bene questi operatori rende molto più semplice lavorare con DynamoDB. Le key condition expression si usano **solo nelle query** e **solo sugli attributi chiave**: non si può interrogare per attributi che non sono chiave. Per quelli servono gli indici secondari o, in ultima istanza, una scan.

### La paginazione

Una query restituisce **al massimo 1 MB di dati per volta**. Se i risultati sono di più, vengono suddivisi in **pagine**. Ogni pagina contiene, oltre agli item, un **`LastEvaluatedKey`**: lo si passa alla richiesta successiva, come `ExclusiveStartKey`, per ottenere la pagina seguente. Si continua finché la risposta non contiene più un `LastEvaluatedKey`. I paginator degli SDK automatizzano questo processo.

### Scan: leggere tutta la tabella

Quando l'informazione cercata non è in una chiave, si ricorre a **Scan** con una **filter expression**:

```bash
aws dynamodb scan \
    --table-name Notes \
    --filter-expression "Favorite = :fav" \
    --expression-attribute-values '{":fav": {"S": "Yes"}}'
```

La scan esamina **ogni item della tabella**, procedendo in sequenza partizione dopo partizione. La filter expression viene applicata **dopo** la lettura, quindi la capacità consumata dipende dagli item letti, non da quelli restituiti. Ne derivano due problemi. La scan consuma **molto più throughput** di una query. Inoltre si concentra su **una partizione alla volta**, che diventa calda mentre le altre restano inattive: non si sfrutta la capacità provisionata in modo efficiente.

### Query o scan?

La **query** lavora sui valori della chiave primaria, della tabella o di un indice, ed è efficiente. La **scan** legge ogni item della tabella o dell'indice. Le scan sono meno efficienti e talvolta sono inevitabili, ma con una progettazione attenta, con indici secondari adeguati ed eventualmente un single-table design, se ne possono evitare molte.

Quando una scan è davvero necessaria su una tabella grande, si può usare la **parallel scan**: il codice suddivide la tabella in **segmenti** (parametri `Segment` e `TotalSegments`) e li scansiona contemporaneamente con più thread o processi. Così si distribuisce il carico su più partizioni e si riducono i tempi.

## 7.5 Aggiornare e cancellare gli item

### UpdateItem

Per aggiornare un item si usa **UpdateItem**. A differenza di PutItem, che sostituisce l'intero item, UpdateItem modifica **solo gli attributi indicati** nella richiesta. Gli aggiornamenti sono quindi più granulari e si scrivono meno dati per ogni chiamata.

### Le scritture condizionali e l'optimistic locking

Le scritture possono essere **condizionali**: vengono eseguite solo se una condizione sull'item esistente è vera. Un'applicazione importante di questo meccanismo è l'**optimistic locking**, una tecnica che ogni sviluppatore dovrebbe conoscere. Si usa quando molti client leggono e scrivono gli stessi item, perché DynamoDB **non blocca gli item** come fanno i database relazionali.

Il meccanismo funziona così:

1. ogni item ha un attributo **versione**;
2. il client legge l'item e ne memorizza la versione;
3. al momento dell'aggiornamento il client incrementa la versione e invia una **condition expression** che verifica che la versione sull'item sia ancora quella letta;
4. se la condizione è vera, nessun altro ha modificato l'item nel frattempo e l'aggiornamento va a buon fine;
5. se è falsa, un altro client ha già scritto. La richiesta fallisce con `ConditionalCheckFailedException` e l'applicazione deve riconciliare il conflitto, per esempio rileggendo l'item e riprovando.

### Esempio: non modificare le note preferite

In Polly Notes si vuole impedire di modificare il testo di una nota contrassegnata come preferita:

```bash
aws dynamodb update-item \
    --table-name Notes \
    --key '{"UserId": {"S": "StudentD"}, "NoteId": {"N": "42"}}' \
    --update-expression "SET Note = :newnote" \
    --condition-expression "Favorite <> :yes" \
    --expression-attribute-values file://values.json
```

con `values.json`:

```json
{
  ":newnote": {"S": "Testo aggiornato della nota"},
  ":yes":     {"S": "Yes"}
}
```

La **key** individua la nota 42 di StudentD. L'**update expression** stabilisce che cosa cambia, cioè il testo della nota. La **condition expression** stabilisce quando la modifica è permessa: solo se `Favorite` è diverso da `Yes`. I valori dei segnaposto sono letti da file.

### DeleteItem

**DeleteItem** cancella un item indicando la tabella e la **chiave completa**: con una chiave composita servono sia la partition key sia la sort key, perché si cancella un item specifico.

```bash
aws dynamodb delete-item \
    --table-name Notes \
    --key '{"UserId": {"S": "StudentD"}, "NoteId": {"N": "42"}}'
```

Anche le cancellazioni accettano una condition expression, per cancellare un item solo in determinate circostanze.

## 7.6 Laboratorio: gestire gli item con l'AWS CLI

Questo laboratorio esegue tutte le operazioni sugli item in **AWS CloudShell**, la CLI nel browser. CloudShell è particolarmente comodo perché non richiede di configurare credenziali locali: usa automaticamente le credenziali del ruolo con cui si è entrati nella Console.

Nella Console DynamoDB la tabella `Notes` contiene già alcuni dati, visibili con *Explore table items*. Le operazioni del laboratorio sono le seguenti.

**Inserimento.** Il comando `put-item` del paragrafo 7.4 scrive la nota 42 di StudentD con testo "Test note". Nessun errore significa che la scrittura è riuscita.

**Scrittura batch.** BatchWriteItem richiede un file in un formato preciso. Invece di ricostruirlo dalla documentazione, si fa generare alla CLI lo scheletro:

```bash
aws dynamodb batch-write-item --generate-cli-skeleton
```

L'output mostra la struttura: un oggetto `RequestItems` con, per ogni tabella, una lista di `PutRequest` e `DeleteRequest`, più i parametri `ReturnConsumedCapacity` e `ReturnItemCollectionMetrics`. Si prepara un file con alcune note di StudentA, StudentB e StudentD e `"ReturnConsumedCapacity": "TOTAL"`, e lo si passa con:

```bash
aws dynamodb batch-write-item --cli-input-json file://batch.json
```

Il parametro `--cli-input-json` indica che il file segue il formato generato dallo scheletro. La risposta riporta **zero `UnprocessedItems`** e un consumo di **4 WCU**. Se una partizione avesse esaurito la capacità e rifiutato alcuni item, questi sarebbero comparsi tra gli `UnprocessedItems`, pronti per essere ritentati.

**Lettura di un item.** `get-item` con la chiave completa di StudentD restituisce la singola nota.

**Query.** La query con `UserId = :userid` e il valore `StudentD` restituisce le due note di StudentD, con `"Count": 2`.

**Scan.** La scan con filtro `Favorite = :fav` e valore `Yes` esamina tutta la tabella e restituisce le due note preferite.

**Aggiornamento.** `update-item` imposta a `Yes` l'attributo `Favorite` della nota 42 di StudentD.

**Aggiornamento condizionale.** Si esegue l'aggiornamento del paragrafo 7.5, che modifica il testo solo se la nota **non** è preferita. Il comando fallisce con il messaggio *"The conditional request failed"*. È il comportamento atteso, perché la nota è appena stata marcata come preferita. Dalla Console si imposta `Favorite` a `No`, si riesegue il comando e questa volta l'aggiornamento riesce.

**Cancellazione.** Infine `delete-item` con la chiave completa elimina la nota.

## 7.7 Le interfacce di alto livello in Java: il DynamoDB Enhanced Client

Costruire richieste, impostare attributi con i loro descrittori di tipo e poi mappare le risposte sugli oggetti del proprio codice è un lavoro ripetitivo. Per Java esiste un'interfaccia di alto livello che fa tutto questo automaticamente: il **DynamoDB Enhanced Client**, che mappa gli **oggetti Java** sugli **item DynamoDB**.

### La classe annotata

```java
@DynamoDbBean
public class Note {
    private String userId;
    private Integer noteId;
    private String note;
    private String favorite;

    @DynamoDbPartitionKey
    @DynamoDbAttribute("UserId")
    public String getUserId() { return userId; }
    public void setUserId(String userId) { this.userId = userId; }

    @DynamoDbSortKey
    @DynamoDbAttribute("NoteId")
    public Integer getNoteId() { return noteId; }
    public void setNoteId(Integer noteId) { this.noteId = noteId; }

    @DynamoDbAttribute("Note")
    public String getNote() { return note; }
    public void setNote(String note) { this.note = note; }

    @DynamoDbAttribute("Favorite")
    public String getFavorite() { return favorite; }
    public void setFavorite(String favorite) { this.favorite = favorite; }
}
```

L'annotazione **`@DynamoDbBean`** dichiara che la classe è mappata su un item DynamoDB. Sui getter, **`@DynamoDbPartitionKey`** e **`@DynamoDbSortKey`** identificano le chiavi. **`@DynamoDbAttribute("...")`** indica il nome dell'attributo corrispondente nella tabella. Il nome è **case sensitive** e deve coincidere esattamente con quello usato nella tabella.

### Creare l'Enhanced Client

```java
DynamoDbClient ddb = DynamoDbClient.builder()
        .region(Region.US_EAST_1)
        .build();

DynamoDbEnhancedClient enhancedClient = DynamoDbEnhancedClient.builder()
        .dynamoDbClient(ddb)
        .build();
```

L'Enhanced Client si costruisce sopra il client normale, che gli viene passato. Da quel momento in poi si usa solo l'Enhanced Client.

### Leggere un item

```java
static Note getNote(DynamoDbEnhancedClient enhancedClient) {
    DynamoDbTable<Note> table =
            enhancedClient.table("Notes", TableSchema.fromBean(Note.class));

    Key key = Key.builder()
            .partitionValue("StudentD")
            .sortValue(42)
            .build();

    return table.getItem(r -> r.key(key));
}
```

`TableSchema.fromBean(Note.class)` costruisce lo schema a partire dalle annotazioni, e `enhancedClient.table(...)` restituisce un `DynamoDbTable<Note>`, una tabella **tipizzata**. La chiave si costruisce con `Key.builder()`. `getItem` restituisce **direttamente un oggetto `Note`**, senza una `GetItemResponse` da analizzare e senza conversioni manuali.

### Aggiungere un item

```java
static void addNote(DynamoDbEnhancedClient enhancedClient) {
    DynamoDbTable<Note> table =
            enhancedClient.table("Notes", TableSchema.fromBean(Note.class));

    Note note = new Note();
    note.setUserId("StudentA");
    note.setNoteId(99);
    note.setNote("Nota creata con l'Enhanced Client");
    note.setFavorite("No");

    table.putItem(note);
}
```

Si istanzia l'oggetto, se ne impostano i campi con i setter e lo si passa a `putItem`. È molto più semplice che costruire la richiesta con i builder di basso livello.

### Interrogare con una condizione

Per mostrare le note preferite di un utente si combinano una **query** sulla partition key e un'**espressione di filtro** sull'attributo `Favorite`:

```java
static void displayFavorites(DynamoDbEnhancedClient enhancedClient) {
    DynamoDbTable<Note> table =
            enhancedClient.table("Notes", TableSchema.fromBean(Note.class));

    AttributeValue yes = AttributeValue.builder().s("Yes").build();
    Map<String, AttributeValue> expressionValues = new HashMap<>();
    expressionValues.put(":value", yes);

    Expression filter = Expression.builder()
            .expression("Favorite = :value")
            .expressionValues(expressionValues)
            .build();

    QueryConditional queryConditional = QueryConditional.keyEqualTo(
            Key.builder().partitionValue("StudentA").build());

    PageIterable<Note> results = table.query(QueryEnhancedRequest.builder()
            .queryConditional(queryConditional)
            .filterExpression(filter)
            .build());

    results.items().forEach(n -> System.out.println(n.getNote()));
}
```

La mappa `expressionValues` associa ogni **segnaposto** dell'espressione al suo valore. Se l'espressione avesse più segnaposto, la mappa avrebbe più voci. `QueryConditional.keyEqualTo(...)` imposta la condizione sulla partition key. La richiesta restituisce un **`PageIterable<Note>`**, che gestisce automaticamente la paginazione e da cui si ottengono direttamente gli oggetti `Note`.

## 7.8 Il caching per DynamoDB

DynamoDB offre latenze nell'ordine dei **millisecondi a una cifra**. Alcune applicazioni però hanno bisogno di latenze nell'ordine dei **microsecondi**, e per loro conviene aggiungere una **cache**. Le opzioni principali sono due.

### Amazon ElastiCache

**Amazon ElastiCache** crea una cache in memoria basata su **Redis** o **Memcached**. Con ElastiCache l'interazione con la cache è **interamente gestita dall'applicazione**. È il codice a stabilire la strategia di caching, per esempio *lazy loading*, che legge dal database solo in caso di *cache miss* e poi popola la cache, oppure *write-through*, che scrive contemporaneamente su database e cache. È sempre il codice a decidere quando interrogare la cache e quando il database, e a mantenere sincronizzati i dati. È una soluzione flessibile e potente, valida per qualunque database.

### DynamoDB Accelerator (DAX)

**DynamoDB Accelerator (DAX)** è una cache in memoria progettata specificamente per DynamoDB, con tempi di risposta nell'ordine dei **microsecondi**. Si posiziona **tra l'applicazione e DynamoDB** e si integra molto più facilmente di ElastiCache. Nel codice basta usare un **client DAX**, che punta all'endpoint del cluster DAX anziché a quello di DynamoDB. Poiché DAX è **compatibile con l'API di DynamoDB**, le modifiche al codice sono minime e le operazioni CRUD restano sostanzialmente le stesse.

DAX supporta le principali operazioni:

| Tipo | Operazioni |
|---|---|
| **Lettura** (servite dalla cache) | `GetItem`, `BatchGetItem`, `Query`, `Scan` |
| **Scrittura** (*write-through*: scritte su DynamoDB e aggiornate in cache) | `PutItem`, `UpdateItem`, `DeleteItem`, `BatchWriteItem` |

DAX serve letture **eventualmente consistenti**. Le letture fortemente consistenti non vengono servite dalla cache ma inoltrate a DynamoDB, quindi non beneficiano dell'accelerazione. DAX non è la soluzione giusta per carichi che richiedono soprattutto letture fortemente consistenti. Per moltissimi casi d'uso, però, la consistenza eventuale è perfettamente accettabile.

> **Box – ElastiCache o DAX?**
>
> Si sceglie **DAX** quando l'obiettivo è accelerare le letture di DynamoDB con il minimo sforzo di sviluppo e la consistenza eventuale è accettabile. Si sceglie **ElastiCache** quando serve una cache sotto il pieno controllo dell'applicazione, per esempio per memorizzare risultati elaborati, sessioni o dati provenienti da più sorgenti, o quando si vogliono usare le strutture dati avanzate di Redis.

## 7.9 Verifica dei concetti

**"I concetti di progettazione delle chiavi NoSQL includono dimensione, forma e velocità."** *Vero.* La "velocità" (*velocity*) non ha a che fare con i chilometri orari: indica il comportamento di lettura e scrittura, cioè con quale frequenza e intensità si accede ai dati. È un fattore da conoscere in fase di progettazione, insieme alla dimensione (*size*) e alla forma (*shape*) dei dati.

**"Le partition key vanno progettate in base agli access pattern comuni e al loro grado di unicità tra gli item della tabella."** *Vero.* Una chiave con alta unicità, o entropia, distribuisce gli item uniformemente tra le partizioni e garantisce le migliori prestazioni. Una chiave "calda" crea invece una hot partition.

**"Gli sviluppatori dovrebbero impostare la modalità di capacità provisioned se prevedono un traffico incostante."** *Falso.* Con un traffico imprevedibile è difficile sapere quanta capacità provisionare. L'alternativa migliore è la modalità **on-demand**: con la modalità provisioned si paga la capacità riservata anche quando non si usa, mentre on-demand segue l'andamento effettivo degli accessi.

**"Per default le operazioni di scrittura PutItem, UpdateItem e DeleteItem sono condizionali."** *Falso.* Le scritture possono essere condizionali, come nell'optimistic locking, ma **non lo sono per default**. La condizione va specificata esplicitamente.

**"L'API BatchWriteItem non può aggiornare gli item: per farlo bisogna usare UpdateItem."** *Vero.* BatchWriteItem supporta solo richieste di put e di delete. Un put può **sostituire** un item esistente, ma non lo aggiorna in modo granulare come UpdateItem.

**"Per come sono progettate, le scan sono più efficienti delle query."** *Falso.* Una scan accede a ogni item della tabella, mentre una query accede direttamente alla partizione giusta. Interrogare per partizione è sempre più efficiente.
