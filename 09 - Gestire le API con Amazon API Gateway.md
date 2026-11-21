# Capitolo 9 – Gestire le API con Amazon API Gateway

## 9.1 Obiettivi del capitolo

Le funzioni Lambda del capitolo precedente contengono la logica dell'applicazione, ma per essere usate da un browser o da un'app mobile devono essere raggiungibili tramite un'**API**. Questo capitolo è dedicato ad **Amazon API Gateway**, il servizio che svolge proprio questo ruolo.

Si vedranno i **componenti chiave** di API Gateway e come sviluppare risorse API che si **integrano con i servizi AWS**. Si imparerà a **configurare le richieste e le risposte** degli endpoint dell'applicazione, a **testare** le risorse e a **distribuire** l'API. Con la distribuzione si ottiene un endpoint pubblico, e si vedrà come interagire con esso. Un laboratorio completo mostra come creare un'API che scrive direttamente in DynamoDB, senza alcuna funzione Lambda intermedia.

## 9.2 Che cos'è Amazon API Gateway

### L'API come contratto

In un'architettura orientata ai servizi o a microservizi, i servizi di backend vanno **esposti** a chi li consuma. Spesso c'è un client, come un browser o un'app, che deve interagire con il backend, ma non si vuole esporre il backend direttamente. Si crea invece un'**API**, che si può pensare come un **contratto**: stabilisce come interagire con il backend, quali input sono attesi e quali output verranno restituiti. Client e backend condividono così un'interfaccia comune.

**Amazon API Gateway** è il servizio con cui in AWS si **creano, pubblicano, mantengono, monitorano e proteggono** le API.

### Le integrazioni

API Gateway si integra con molti tipi di backend.

**AWS Lambda** è l'integrazione più comune e quella più usata in questo libro: una funzione Lambda viene posta "dietro" API Gateway, che la invoca a ogni richiesta.

Gli **altri servizi AWS** possono essere chiamati **direttamente**. Per esempio, per inserire un item in una tabella DynamoDB serve chiamare l'API `PutItem` con un input di una struttura precisa. API Gateway può ricevere la richiesta del client, costruire quell'input e passarlo al servizio, senza codice intermedio.

Infine, API Gateway può fare da facciata ad **API ospitate su EC2**, a **servizi di terze parti** e, in generale, a **qualunque endpoint HTTP**.

### I tre tipi di API

API Gateway supporta tre tipi di API: **REST API**, **HTTP API** e **WebSocket API**.

Le **REST API** e le **HTTP API** usano entrambe HTTP e svolgono funzioni simili, ma con un diverso insieme di funzionalità. Le **HTTP API** sono progettate per essere **più semplici**. Hanno meno funzionalità, ma sono più veloci e più economiche. Sono ideali per fare da **proxy** verso un backend come una funzione Lambda: il payload inviato dal client arriva al backend così com'è, senza modifiche. Supportano l'autenticazione **OIDC/OAuth 2.0** con authorizer JWT e la configurazione **CORS**.

Le **REST API** offrono **più funzionalità**, ma richiedono anche più configurazione. In particolare permettono di controllare la **forma dei dati** tra client e backend tramite **model** e **mapping template**.

Le **WebSocket API** mantengono una **connessione bidirezionale persistente** tra client e backend. Il client, un browser o un'app, apre la connessione, e da quel momento sia il client sia il servizio possono inviare messaggi in qualunque momento. API Gateway gestisce le connessioni tramite **route** predefinite: `$connect`, all'apertura della connessione, `$disconnect`, alla chiusura, e `$default`, per i messaggi che non corrispondono ad altre route. A queste si aggiungono le route personalizzate. Le WebSocket API sono la scelta per le applicazioni **in tempo reale**, come le chat.

### API Gateway in Polly Notes

Nell'applicazione di riferimento, API Gateway è al centro dell'architettura con una **REST API**. La risorsa è `/notes` e su di essa sono definiti diversi metodi HTTP: GET per elencare le note, POST per crearle, un altro GET per la ricerca, DELETE per cancellarle e un ulteriore GET per recuperare l'audio. Ogni endpoint è collegato a una **funzione Lambda diversa**, quindi l'applicazione è in pratica un piccolo insieme di microservizi, e API Gateway ne è la **porta d'ingresso**. Su questa porta si può configurare l'**autorizzazione**. Inizialmente l'API sarà aperta e pubblica, e la si proteggerà nel Capitolo 11.

### Le funzionalità delle REST API

Le REST API offrono un ricco insieme di funzionalità:

- **Più versioni in parallelo**: se una modifica rischia di rompere i client esistenti, si possono ospitare contemporaneamente due versioni dell'API su **stage** ed endpoint diversi. Ogni client passa alla nuova versione quando è pronto.
- **API key**: si possono creare chiavi per **limitare** (*throttling*) e **misurare** (*metering*) le richieste di ciascun client, tramite *usage plan*.
- **Controllo degli accessi**: si stabilisce chi può chiamare l'API. Si può configurare la funzione Lambda in modo che solo API Gateway possa invocarla, e API Gateway in modo che solo determinati utenti o client possano chiamarlo.
- **Trasformazione dei dati**: se il backend si aspetta un formato diverso da quello inviato dal client, la richiesta si può trasformare con il **Velocity Template Language (VTL)**.
- **Generazione di SDK**: dalla Console si può generare un SDK client per l'API, per esempio in JavaScript, che i client possono scaricare e usare.
- **Mock integration**: l'API risponde con dati simulati senza essere collegata a un backend reale.
- **Caching delle risposte**: si abilita una cache con un **TTL** (*time to live*) configurabile. Se molti client chiedono gli stessi dati, solo la prima richiesta raggiunge il backend e le successive, entro il TTL, vengono servite dalla cache. Ne risultano **minore latenza** per gli utenti, **meno carico** sul backend e, spesso, **costi inferiori**.

## 9.3 Lavorare con API Gateway

Da qui in avanti il capitolo si concentra sulle **REST API**.

### Il percorso di una richiesta

Una richiesta che arriva ad API Gateway attraversa diverse fasi, tutte configurabili:

1. **Autorizzazione**: se è configurato un *authorizer*, API Gateway verifica che il chiamante sia autorizzato. Un'API può essere pubblica, ma è più comune proteggerla. Esistono diversi tipi di authorizer: IAM, Cognito e Lambda.
2. **Validazione**: un **model** verifica che la struttura della richiesta sia quella attesa.
3. **Metering e throttling**: si applicano i limiti associati alla API key o le soglie di throttling configurate.
4. **Trasformazione e mapping**: la richiesta viene trasformata nel formato atteso dal backend.
5. **Backend**: il servizio di destinazione, cioè il codice o un servizio AWS, esegue l'elaborazione.
6. **Risposta**: anche la risposta può essere validata e trasformata prima di essere restituita al client.

### Risorse e metodi

In una REST API una **risorsa** è un'entità logica a cui si accede tramite un **percorso** (*resource path*). Raggruppa gli endpoint di backend che appartengono alla stessa categoria. In Polly Notes tutte le API riguardano le note, quindi la risorsa è `/notes`. Un URL di esempio è:

```
https://a1b2c3d4e5.execute-api.us-east-1.amazonaws.com/prod/notes/student-a
```

Qui si chiama la risorsa `/notes` passando nel percorso un **parametro**, l'identificativo dell'utente, che verrà inoltrato al backend per eseguire la ricerca. Una stessa API può avere **molte risorse**.

Su ogni risorsa si definiscono i **metodi**, cioè i metodi HTTP supportati: GET, POST, PUT, DELETE, HEAD e così via. Per ciascun metodo si configurano validazione, trasformazione e integrazione.

### I tipi di endpoint

Un'API API Gateway appartiene a una **Region**, come si vede dall'URL. Esistono tre **tipi di endpoint**:

- **Regional**: l'API vive in una sola Region, e i client vi arrivano attraverso Internet. È adatto quando i client sono nella stessa area geografica.
- **Edge-optimized**: le richieste vengono instradate attraverso la rete di **edge location di Amazon CloudFront**, il punto di presenza più vicino al client. È adatto a client distribuiti in tutto il mondo.
- **Private**: l'API è accessibile **solo dall'interno di un VPC**, tramite un *interface VPC endpoint*. È adatto ad API interne, chiamate per esempio da applicazioni su EC2.

### I tipi di integrazione

Il backend collegato a un metodo si chiama **integration endpoint**, e il comportamento di API Gateway dipende dal **tipo di integrazione** scelto.

| Tipo | Comportamento | Quando usarlo |
|---|---|---|
| **Proxy** | Il payload della richiesta passa **invariato** al backend, che restituisce direttamente la risposta. Non è possibile modificare il contenuto. | È semplice da configurare. È ideale con Lambda quando validazione e logica stanno tutte nella funzione (*Lambda proxy integration*). |
| **Non-proxy** (custom) | Si configurano esplicitamente l'**integration request** e l'**integration response**, con i relativi mapping. | Serve quando client e backend usano formati diversi o si integra direttamente un servizio AWS. |
| **Mock** | API Gateway risponde con dati simulati, **senza chiamare alcun backend**. | Per test, sviluppo in parallelo e design API-first. |

### I casi d'uso delle mock integration

Le mock integration sono utili in due situazioni.

La prima è lo **sviluppo in parallelo tra team**. Supponiamo che un team stia scrivendo una funzione che deve chiamare un'API, dietro la quale ci sarà un'altra funzione ancora in sviluppo da parte di un secondo team. Il primo team vuole comunque scrivere e provare il proprio *happy path*, cioè il codice per il caso in cui tutto va bene: chiamare l'API, ricevere la risposta, analizzarla e usarla. Con una mock integration, API Gateway non chiama la funzione non ancora pronta ma restituisce una risposta simulata predefinita, e i due team possono lavorare in parallelo.

La seconda è il **design API-first**. Prima di iniziare lo sviluppo del backend si definiscono tutte le risorse e i metodi dell'API, li si fa rispondere con dati simulati e si verifica che le risposte siano quelle attese. Il contratto è quindi stabilito e validato prima di scrivere il codice.

Le risposte simulate non sono limitate al successo: si possono simulare anche errori come 404 o 500.

## 9.4 L'elaborazione di richieste e risposte

### Le quattro fasi

In una REST API ogni chiamata attraversa quattro fasi, visibili nella Console come *Method Execution*:

1. **Method request**: è il contratto con il client. Qui si configurano l'autorizzazione, la richiesta di una API key, i parametri attesi e la **validazione** della richiesta tramite un **model**. Un model è uno **JSON Schema** che descrive la forma dei dati, per esempio quali campi sono obbligatori e di che tipo.
2. **Integration request**: è la richiesta verso il backend. Qui si applicano i **mapping template**, che trasformano i dati del client nel formato richiesto dal backend. È indispensabile quando si integra direttamente un servizio AWS senza Lambda intermedia.
3. **Integration response**: è la risposta del backend, che si può mappare e trasformare.
4. **Method response**: è la risposta verso il client, con eventuali **response model** che ne definiscono la forma.

Questo meccanismo è utile quando client e backend seguono formati di comunicazione diversi e non si vuole modificare il codice di nessuno dei due: è API Gateway ad adattare i dati. La validazione tramite model, però, non sostituisce del tutto quella nel frontend e nel backend, e in molte applicazioni conviene mantenerle tutte.

### Laboratorio: un'API che scrive direttamente in DynamoDB

Il laboratorio costruisce una REST API il cui metodo POST su `/notes` scrive una nota nella tabella DynamoDB `Notes` **senza alcuna funzione Lambda**: è una **AWS Service integration**. API Gateway riceve il payload del client, lo trasforma nel formato dell'API `PutItem` e chiama direttamente DynamoDB.

**Creazione dell'API.** Nella Console di API Gateway si sceglie *Create API*. Vengono proposti quattro tipi:

- **HTTP API**, per API a bassa latenza ed economiche, senza model e mapping, con supporto OIDC/OAuth 2.0 e CORS;
- **WebSocket API**, per connessioni persistenti in tempo reale;
- **REST API**, con pieno controllo su model e mapping;
- **REST API Private**, accessibile solo da un VPC.

Si sceglie **REST API**, si chiama l'API `notes` e si lascia il tipo di endpoint **Regional**.

**Risorsa e metodo.** Si crea la risorsa `notes` abilitando **CORS** e su di essa il metodo **POST**. Come tipo di integrazione si sceglie **AWS Service**, con questi parametri:

- Region: `us-east-1`;
- servizio: **DynamoDB**;
- metodo HTTP: **POST**, perché le API di DynamoDB si chiamano sempre in POST;
- action: **PutItem**;
- **execution role**: l'ARN di un ruolo IAM che API Gateway assumerà per chiamare DynamoDB.

Il ruolo, chiamato per esempio `APIGatewayDynamoDB`, ha una policy che consente l'accesso a DynamoDB e una **trust relationship** con il servizio API Gateway:

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": { "Service": "apigateway.amazonaws.com" },
    "Action": "sts:AssumeRole"
  }]
}
```

È uno dei *service role* visti nel Capitolo 3: permette a un servizio AWS di chiamarne un altro.

**Il model.** Nella sezione *Models* si crea un model `Note` con content type `application/json` e il seguente schema, coerente con la tabella `Notes`:

```json
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "title": "Note",
  "type": "object",
  "properties": {
    "UserId": { "type": "string" },
    "NoteId": { "type": "number" },
    "Note":   { "type": "string" }
  },
  "required": ["UserId", "NoteId", "Note"]
}
```

**La method request.** Nella method request si configurano i seguenti parametri:

- **Authorization**: `NONE`, cioè API pubblica. In alternativa `AWS_IAM` richiederebbe al chiamante credenziali IAM;
- **Request validator**: *Validate body*;
- **API key required**: `false`;
- **Request body**: per il content type `application/json` si associa il model `Note`.

**L'integration request.** Nella sezione *Mapping Templates* dell'integration request si sceglie l'opzione *When there are no templates defined* e si aggiunge un template per `application/json`. Il template, scritto in **VTL**, costruisce il corpo della richiesta `PutItem`:

```velocity
{
  "TableName": "Notes",
  "Item": {
    "UserId": { "S": "$input.path('$.UserId')" },
    "NoteId": { "N": "$input.path('$.NoteId')" },
    "Note":   { "S": "$input.path('$.Note')" }
  }
}
```

La struttura è quella attesa da `PutItem`: il nome della tabella e l'item con i descrittori di tipo DynamoDB. L'espressione **`$input.path('$.campo')`** estrae un valore dal **corpo** della richiesta tramite un'espressione JSONPath.

> **Box – Le variabili principali dei mapping template**
>
> Nei template VTL di API Gateway si usano spesso alcune variabili.
>
> - **`$input.body`** restituisce l'intero corpo della richiesta come stringa.
> - **`$input.json('$.x')`** restituisce una porzione del corpo come stringa JSON.
> - **`$input.path('$.x')`** restituisce una porzione del corpo come oggetto, utile per accedere ai singoli valori.
> - **`$input.params('x')`** restituisce il valore del **parametro della richiesta** `x`, cercandolo nel percorso, nella query string o negli header. È la variabile da usare per leggere, per esempio, un parametro di path come `{userId}`. `$input.params().path` restituisce invece la mappa di tutti i parametri di path.
> - **`$context`** dà accesso al contesto della richiesta, come l'ID della richiesta e l'identità del chiamante.
> - **`$stageVariables`** dà accesso alle stage variable (paragrafo 9.6).
>
> La distinzione tra `$input.path` (corpo) e `$input.params` (parametri) è una domanda d'esame ricorrente.

**Il test.** Il pulsante **TEST** della Console permette di verificare il metodo senza distribuirlo, impostando query string, header e corpo della richiesta. Non sostituisce i test reali, ma è un controllo rapido della configurazione.

Con un **payload non valido**, privo di `UserId`, `NoteId` e `Note`, la risposta è `Invalid request body`. I log del test mostrano tutto il percorso: il corpo della method request prima delle trasformazioni e poi l'errore di validazione *"Request body does not match model schema for content type application/json"*, con l'elenco delle tre proprietà obbligatorie mancanti. Il model funziona come previsto. Quando si fa troubleshooting, è in questi log che si trovano le informazioni su ogni fase.

Con un **payload valido**, come `{"UserId": "StudentM", "NoteId": 1, "Note": "first note"}`, i log mostrano:

- il corpo della richiesta prima della trasformazione;
- la validazione riuscita;
- la richiesta all'endpoint di DynamoDB in `us-east-1` con `Action=PutItem`;
- il **corpo dopo la trasformazione**, già nel formato DynamoDB;
- lo status **200**.

In *Explore table items* della Console DynamoDB compare la nuova nota: StudentM, 1, "first note".

L'esempio integra direttamente un servizio AWS, ma nella maggior parte dei casi dietro API Gateway ci sarà del codice: una funzione Lambda, un servizio su EC2 o un server remoto.

## 9.5 La progettazione delle API come codice

Dopo l'*Infrastructure as Code* c'è l'**API design as code**. Un'API funzionante è un buon risultato, ma come faranno gli altri a sapere com'è fatta e come integrarsi?

API Gateway aderisce allo standard **OpenAPI**, lo stesso usato da strumenti come **Swagger**. Un'API si descrive in un file **YAML** o JSON, leggibile sia dalle persone sia dalle macchine, in cui si definiscono **percorsi**, **risorse**, **metodi** e **tipi di integrazione**:

```yaml
openapi: 3.0.1
info:
  title: notes
  version: "1.0"
paths:
  /notes:
    post:
      requestBody:
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/Note"
      responses:
        "200":
          description: Nota salvata
      x-amazon-apigateway-integration:
        type: aws
        httpMethod: POST
        uri: arn:aws:apigateway:us-east-1:dynamodb:action/PutItem
        credentials: arn:aws:iam::123456789012:role/APIGatewayDynamoDB
components:
  schemas:
    Note:
      type: object
      required: [UserId, NoteId, Note]
      properties:
        UserId: { type: string }
        NoteId: { type: number }
        Note:   { type: string }
```

Le estensioni `x-amazon-apigateway-*` contengono le configurazioni specifiche di API Gateway, come l'integrazione. Il file si può usare in due direzioni. In un approccio *design first* si progetta l'API nel file e poi la si **importa** in API Gateway. Al contrario, si può progettare l'API nella Console e poi **esportarla** in formato OpenAPI o Swagger per condividerla.

I vantaggi sono diversi:

- il file è una **documentazione** dell'API, che chiunque può leggere per capire che cosa fa e come usarla;
- molti strumenti OpenAPI **generano codice** a partire dal file, in particolare i client per chiamare l'API, accelerando lo sviluppo;
- il file va conservato in un **repository di codice**. Come per l'Infrastructure as Code, per modificare l'API non si interviene a mano nella Console: si modifica il file di configurazione e lo si importa per creare la nuova versione. Le modifiche sono così tracciate, revisionabili e ripetibili.

## 9.6 Distribuire le API

### Stage e invoke URL

Un'API progettata e testata deve essere **distribuita** (*deployed*) su uno **stage** per essere invocabile. La distribuzione produce un **invoke URL** con questa forma:

```
https://{api-id}.execute-api.{region}.amazonaws.com/{stage}
```

L'URL contiene l'identificativo dell'API, `execute-api`, la Region e il **nome dello stage**. Gli stage possono rappresentare ambienti, come `dev`, `qa` e `prod`, oppure versioni dell'API, come `v1` e `v2`. Il nome dello stage nell'URL indirizza la richiesta allo stage desiderato.

Un'API distribuita si può invocare dalla **Console**, dalla **CLI**, dagli **SDK** o da strumenti di test come **Postman**.

### La configurazione per stage

Ogni stage ha una propria configurazione:

- **Caching**: la cache delle risposte si attiva per stage. Poiché ha un costo, è comune abilitarla solo in produzione.
- **Throttling**: si possono impostare limiti diversi per ogni stage.
- **Logging**: log di esecuzione e log di accesso in CloudWatch, e tracing con X-Ray.
- **Stage variable**: variabili di configurazione specifiche dello stage.
- **Canary release**: rilascio graduale di una nuova versione.

### Le stage variable

Le **stage variable** sono coppie **nome-valore** associate a uno stage. Funzionano come variabili d'ambiente e si possono usare sia nei mapping template VTL sia nella configurazione dell'API.

Un esempio tipico combina le stage variable con gli **alias** delle funzioni Lambda visti nel Capitolo 8. L'API `notes` ha uno stage `prod` e uno stage `dev`. La funzione Lambda ha un alias `prod`, che punta alla versione di produzione, e un alias `dev`, che punta alla versione di sviluppo, e ciascuno lavora sulla rispettiva tabella. Si definisce in ogni stage una variabile, per esempio `lambdaAlias`, con valore `prod` o `dev`, e nell'integrazione si indica come funzione:

```
arn:aws:lambda:us-east-1:123456789012:function:notes-function:${stageVariables.lambdaAlias}
```

Lo stesso metodo invoca quindi la versione di produzione dallo stage `prod` e quella di sviluppo dallo stage `dev`. Le stage variable si prestano anche a scenari di tipo **feature flag**, in cui uno stage punta a una versione del codice con una nuova funzionalità mentre un altro continua a usare la versione precedente.

### Le canary release

Una **canary release** permette di rilasciare una nuova versione dell'API in modo graduale. Si mantengono in esercizio la **versione stabile**, collaudata e in produzione da tempo, e la **nuova versione** con le nuove funzionalità. Invece di passare tutto il traffico alla nuova versione in un colpo solo, se ne invia una **piccola percentuale** al *canary* e se ne osserva il comportamento. Se tutto va bene si aumenta gradualmente la quota, fino al passaggio completo. A quel punto la vecchia versione si può dismettere.

I passi sono questi:

1. si sviluppa la modifica, eventualmente con cambiamenti all'API;
2. la si distribuisce come canary su uno stage;
3. si configurano le impostazioni specifiche, come cache e stage variable;
4. si abilita il canary;
5. si stabilisce la percentuale di traffico, per esempio **10%** al canary e **90%** alla versione corrente;
6. quando si è soddisfatti si **promuove** il canary al 100%.

La ripartizione è **statistica**: si controlla la percentuale di traffico, non quale richiesta o quale cliente va a quale versione.

### Laboratorio: distribuire e chiamare l'API

Nella Console si sceglie *Actions → Deploy API*, si crea un nuovo stage `prod` e si conferma. Si apre lo **stage editor**, da cui si configurano tutte le impostazioni viste sopra:

- i **log** di CloudWatch, a livello *Errors* o *Info*, e l'**access logging**;
- il **tracing con X-Ray** (Capitolo 13);
- le **stage variable**, accessibili nei mapping template tramite `$stageVariables`. Si ricordi che `$input` riguarda il payload e `$context` il contesto della richiesta;
- la **generazione dell'SDK** per la piattaforma scelta;
- l'**esportazione** dell'API in formato OpenAPI 3 o Swagger;
- la **cronologia dei deployment** dello stage;
- la configurazione del **canary**.

Dallo stage editor si copia l'**invoke URL** e lo si usa in **Postman** per chiamare l'API dall'esterno della Console. Si invia una richiesta **POST** a `<invoke-url>/notes` con questo corpo:

```json
{ "UserId": "StudentF", "NoteId": 13, "Note": "Hello world from StudentF" }
```

La risposta è **200 OK**. In DynamoDB compare la nuova nota: StudentF, 13, "Hello world from StudentF".

## 9.7 Caso di studio: endpoint Regional o Edge-optimized?

La scelta del tipo di endpoint dipende dalla distribuzione degli utenti. Se gli utenti sono **sparsi nel mondo**, un endpoint **Edge-optimized** porta l'API vicino a ciascuno di loro. Se gli utenti sono **concentrati in un'area**, basta un endpoint **Regional**, purché si scelga la **Region giusta**. Ma qual è la Region giusta?

Un modo rapido e approssimativo per scoprirlo è misurare il tempo di risposta di ciascuna Region dal punto in cui si trovano gli utenti. Il programma seguente, in .NET, sfrutta le funzionalità asincrone del linguaggio:

```csharp
var regions = new[] {
    RegionEndpoint.USWest1, RegionEndpoint.USWest2, RegionEndpoint.USEast2,
    RegionEndpoint.CACentral1, RegionEndpoint.EUWest1, RegionEndpoint.APSoutheast2
};
var clients = regions.Select(r => new AmazonEC2Client(r)).ToArray();

var tasks = clients.Select(async client =>
{
    var stopwatch = Stopwatch.StartNew();
    Console.WriteLine("Getting regions...");
    await client.DescribeRegionsAsync(new DescribeRegionsRequest());
    Console.WriteLine($"{client.Config.RegionEndpoint}: {stopwatch.ElapsedMilliseconds} ms");
}).ToArray();

Task.WaitAll(tasks);
```

Il programma crea un client EC2 per ogni Region. Con `Select` esegue per ciascun client, **in parallelo**, una funzione asincrona che avvia un cronometro, chiama un'API qualsiasi, qui `DescribeRegionsAsync`, perché interessa solo misurare il tempo di andata e ritorno, e stampa i millisecondi trascorsi. `Task.WaitAll`, una funzionalità del *Task-based Asynchronous Pattern* di .NET, impedisce al programma di terminare prima che tutte le chiamate si siano concluse.

Eseguito da Seattle con `dotnet run`, il programma stampa subito tutte le righe "Getting regions...", perché le chiamate partono quasi contemporaneamente, e poi le risposte man mano che arrivano. In testa compaiono N. California, Oregon, Ohio e Canada. Sydney è tra le ultime. In una seconda esecuzione vince l'Oregon: altri fattori influenzano i tempi, e una singola misurazione non basta.

Il test va interpretato con attenzione:

- per risultati affidabili bisogna **ripetere molte misurazioni e farne la media**;
- la scelta della Region dipende anche dalla **conformità normativa**: potrebbe essere obbligatorio mantenere i dati nel proprio Paese;
- la geografia può sorprendere. Per un'applicazione destinata allo Stato del Vermont, la Region di **Montréal** (Canada) è molto più vicina di quella della Virginia del Nord. In assenza di vincoli normativi potrebbe essere la scelta migliore;
- **la Region più veloce non è necessariamente la più vicina**: dipende da come passano i cavi di rete.

## 9.8 Verifica dei concetti

**"Con Amazon API Gateway gli sviluppatori possono creare, pubblicare, mantenere, monitorare e proteggere le API."** *Vero.* Si creano risorse, si pubblica su uno stage, si monitora con le metriche di CloudWatch e si protegge con authorizer IAM, Cognito o Lambda.

**"In una canary release il traffico totale dell'API è suddiviso tra la release corrente e la canary secondo uno schema prevedibile."** *Falso.* Si controlla la **percentuale** di traffico inviata al canary, non quale singola richiesta o quale cliente vi viene indirizzato. La distribuzione delle singole richieste non è prevedibile.

**"Le mock integration rispondono solo con lo status code 200."** *Falso.* Si possono simulare anche risposte di errore, come 404 o 500.

**"Una risorsa è un'entità logica a cui un'applicazione può accedere tramite un resource path."** *Vero.* Per esempio una risorsa *customer* è raggiungibile tramite un percorso come `/customer/{customerId}`.

**"Le stage variable sono coppie nome-valore definibili come attributi di configurazione associati a uno stage di deployment di una REST API."** *Vero.* Si definiscono nello stage, per esempio uno per `dev` e uno per `prod`, e si usano nei template VTL e in altre parti della configurazione.

**"Per gestire in modo intelligente un insieme eterogeneo di chiamate API si può usare una funzione Lambda come backend CRUD."** *Vero.* Si può avere una funzione per ogni operazione, come in Polly Notes, oppure un'unica funzione che gestisce più operazioni in base ai parametri ricevuti.
