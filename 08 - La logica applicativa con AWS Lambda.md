# Parte IV – Logica applicativa e API

# Capitolo 8 – La logica applicativa con AWS Lambda

## 8.1 Obiettivi del capitolo

Dopo aver visto dove conservare file e dati, resta da capire **dove eseguire il codice** dell'applicazione. Questo capitolo presenta i servizi di calcolo di AWS e si concentra su **AWS Lambda**, il servizio serverless che esegue tutta la logica di Polly Notes.

Il percorso parte dal **funzionamento di Lambda**, con un primo esempio nella Console. Si passa poi allo **sviluppo** di funzioni che usano gli SDK, come qualunque altro codice, per chiamare i servizi AWS. Si vedrà come configurare **trigger e permessi**: chi può invocare una funzione, tramite API o in risposta a un evento come il caricamento di un oggetto in un bucket, e che cosa la funzione può fare. Il capitolo copre poi il ciclo completo di uno sviluppatore. Si **testa** la funzione nella Console, in locale e con unit test. Si **distribuisce** impacchettandola con le sue dipendenze. Infine la si **monitora** una volta in produzione, per sapere se funziona correttamente.

## 8.2 I servizi di calcolo su AWS

Un servizio di calcolo è, in sostanza, un servizio che mette a disposizione della CPU su cui eseguire applicazioni e codice. AWS ne offre diversi, a livelli di astrazione crescenti.

**Amazon EC2** fornisce **macchine virtuali**: hardware virtualizzato che avvia un sistema operativo Linux o Windows, su cui installare le proprie applicazioni.

**Amazon ECS** e **Amazon EKS** sono servizi di **orchestrazione di container**. Un container, per esempio creato con Docker, è un'applicazione impacchettata insieme a tutto ciò che le serve per funzionare. Viene eseguito in un ambiente isolato che "crede" di avere un proprio sistema operativo. Quando si hanno molti container da distribuire su una flotta di server, serve uno strumento di orchestrazione. **ECS** è l'orchestratore di Amazon e può eseguire i container su host EC2 oppure in modalità serverless con **AWS Fargate**. **EKS** offre lo stesso tipo di servizio basandosi sul progetto open source **Kubernetes**.

**AWS Lambda** è il servizio di **calcolo serverless**: si impacchetta il codice di una funzione e lo si consegna ad AWS, che lo esegue in risposta a un **evento** o quando lo si **invoca** esplicitamente.

| | **EC2** | **ECS / EKS** | **Lambda** |
|---|---|---|---|
| **Livello di astrazione** | Hardware (macchina virtuale) | Sistema operativo (container) | Runtime (Java, Python, Node.js, Ruby…) |
| **Pacchetto** | Amazon Machine Image (AMI) | Immagine di container | Codice della funzione (zip) o immagine di container |
| **Modello di prezzo** | Consumo di infrastruttura: si paga finché l'host è acceso | Consumo di infrastruttura | Per richiesta e per durata di esecuzione |
| **Scalabilità** | Progettata dall'utente (Auto Scaling, load balancer) | Progettata dall'utente (numero di container e di host) | Implicita, gestita dal servizio |

Alcune voci della tabella meritano una spiegazione. EC2 astrae l'**hardware**: un'**AMI** è in pratica un'immagine del disco che la macchina virtuale avvia. ECS ed EKS astraggono il **sistema operativo**, perché i container sfruttano funzionalità del kernel Linux per isolarsi. Un'immagine di container contiene, per esempio, l'interprete Python, il codice e le sue dipendenze. Lambda astrae il **runtime**: si sceglie un linguaggio tra quelli disponibili e si fornisce solo il codice, eventualmente con le dipendenze in un file zip.

Anche il modello di prezzo è molto diverso. Un'istanza EC2 accesa per 40 ore costa 40 ore di virtualizzazione, che venga usata o no. Con Lambda si paga per ogni esecuzione: una funzione eseguita cinque volte costa cinque esecuzioni, in base alla loro durata e alla memoria configurata. È un modello molto efficiente per i carichi intermittenti.

Lo stesso vale per la scalabilità. Con EC2 ed ECS lo sviluppatore **progetta** come l'applicazione scalerà: istanze dietro un load balancer, regole di Auto Scaling, numero di container e di host. Con Lambda la scalabilità è **implicita**: il servizio esegue tante copie della funzione quante ne servono, come avviene per molti servizi serverless.

## 8.3 Come funziona AWS Lambda

### Event source e destinazioni

Al centro c'è la **funzione Lambda**: il codice fornito dallo sviluppatore, eseguito sull'infrastruttura AWS. Come qualunque codice, una funzione può chiamare servizi AWS, per esempio per salvare record in DynamoDB, oppure servizi su Internet, come un *payment gateway* esterno.

A invocare la funzione è una **event source**, che può essere di diversi tipi:

- un **cambiamento di dati o di stato** di una risorsa, come un oggetto caricato in un bucket S3. Un esempio già incontrato: una funzione che genera la miniatura di ogni immagine caricata e la salva in un altro bucket;
- una **richiesta HTTP** verso un endpoint, tipicamente un'API REST ospitata in API Gateway, a cui la funzione fornisce la risposta;
- una **chiamata API** tramite SDK o CLI, cioè l'operazione `Invoke`.

### Le integrazioni con gli altri servizi

Lambda è profondamente integrato con il resto di AWS, e l'integrazione va in due direzioni: eventi di altri servizi possono attivare una funzione, e molte funzionalità di altri servizi si basano su funzioni Lambda. Alcuni esempi:

- **Amazon S3**: un oggetto caricato in un bucket attiva la funzione;
- **DynamoDB Streams**: si può configurare uno *stream* su una tabella che registra ogni modifica agli item, e una funzione Lambda può elaborarla. È l'equivalente dei *trigger* di un database relazionale;
- **Amazon Kinesis**: la funzione elabora flussi di dati in streaming;
- **Amazon API Gateway**: la funzione implementa la logica dietro un'API REST (Capitolo 9);
- **AWS CloudFormation**: per l'infrastruttura che CloudFormation non sa gestire si scrive una **custom resource**, implementata come funzione Lambda, il cui codice può stare direttamente nel template;
- **AWS CodeCommit**: un *push* sul repository può attivare una funzione come *post-commit hook*, per esempio per eseguire una scansione delle vulnerabilità a ogni commit.

### L'anatomia di una funzione

Una funzione Lambda è composta da diversi elementi:

- **Permessi di accesso e trigger**: le policy stabiliscono quali principal, inclusi altri servizi, possono invocare la funzione, e i trigger collegano la funzione alle event source.
- **Configurazione**: la **concorrenza** desiderata o riservata, la **memoria** e il **timeout**, cioè la durata massima prevista. La memoria si regola su una scala continua, e la **CPU assegnata cresce in proporzione alla memoria**. Serve quindi un lavoro di *right-sizing* per trovare la quantità giusta.
- **Codice**: contiene la funzione **handler**, il punto d'ingresso che il servizio chiama a ogni invocazione.
- **Runtime**: Lambda offre runtime gestiti per i linguaggi più diffusi (Java, .NET, Node.js, Python, Ruby) e permette di fornire **runtime personalizzati**.
- **Layer**: file aggiuntivi resi disponibili nel file system della funzione, tipicamente librerie condivise tra più funzioni (paragrafo 8.6).

### Laboratorio: una prima funzione nella Console

Nella Console Lambda si sceglie *Create function* e si inseriscono il nome (`Demo`), il runtime (Python) e l'architettura della CPU, **x86** oppure **ARM** (Graviton). Lasciando le impostazioni predefinite, Lambda crea automaticamente anche un **ruolo di esecuzione**. Come a un'istanza EC2 si associa un ruolo, a ogni funzione Lambda è associato un ruolo che stabilisce che cosa può fare in AWS.

Il codice generato è un handler con due parametri:

```python
def lambda_handler(event, context):
    print(event)
    return {
        "statusCode": 200,
        "body": "Hello from my Lambda function"
    }
```

Il parametro **`event`** è un documento JSON passato alla funzione al momento dell'invocazione. Il parametro **`context`** contiene informazioni sull'ambiente di esecuzione. Per provare la funzione nella Console si configura un **test event**, un qualunque documento JSON, anche quello di esempio proposto. Lo si salva con un nome, per esempio `Demo`, si fa il *Deploy* del codice e si esegue il *Test*. La Console mostra il risultato, con la risposta "Hello from my Lambda function", e i log, in cui compare il JSON dell'evento stampato da `print(event)`. Se a invocare la funzione fosse un altro servizio, l'evento conterrebbe le informazioni su ciò che è accaduto in quel servizio.

## 8.4 I modelli di invocazione di Lambda

### Sincrono, asincrono e basato su polling

Una funzione Lambda può essere invocata in tre modi.

Nell'**invocazione sincrona** il chiamante invoca la funzione e **attende la risposta**. L'esempio tipico è API Gateway. Un client chiama la route `/order` di un'API REST, API Gateway invoca la funzione e restituisce al client la risposta prodotta.

Nell'**invocazione asincrona** il chiamante chiede a Lambda di eseguire la funzione e riceve subito una conferma di presa in carico, non il risultato. L'evento viene messo in una **coda interna** e la funzione viene eseguita appena possibile. È il modello usato da servizi come **S3** e **SNS** quando invocano una funzione per conto dell'utente. Per l'invocazione asincrona si possono configurare tre cose:

- le **destination**, dove inviare l'esito dell'esecuzione: una coda SQS, un topic SNS, un'altra funzione Lambda o un event bus EventBridge, separatamente per i successi e per i fallimenti;
- i **retry**: per default, in caso di errore, Lambda **ritenta l'esecuzione due volte**. Il numero è configurabile;
- una **dead-letter queue (DLQ)**: quando una funzione continua a fallire, l'evento che ha causato il problema viene scritto in una coda, così da poterlo analizzare.

Nell'**invocazione basata su polling** (*poll-based*, tramite *event source mapping*) è **Lambda stesso** a leggere i dati da uno **stream** o da una **coda**: Amazon Kinesis, DynamoDB Streams, Amazon MSK (Managed Streaming for Apache Kafka) o Amazon SQS. Si configura quanti eventi raccogliere in ogni **batch** o quanto attendere tra un batch e l'altro. Lambda risparmia così allo sviluppatore il codice che altrimenti servirebbe per leggere dallo stream.

| Modello | Esempi | Gestione dei retry |
|---|---|---|
| **Sincrono** | API Gateway, invocazione diretta con `RequestResponse` | **Lambda non ritenta**. Può farlo il servizio chiamante, secondo la sua integrazione |
| **Asincrono** | S3, SNS, EventBridge, invocazione con `Event` | **2 retry** per default, configurabili. Destination e DLQ |
| **Polling** | Kinesis, DynamoDB Streams, MSK, SQS | Dipende dalla sorgente: va verificato nella documentazione di ciascuna integrazione |

> **Box – Retry di Lambda: una distinzione da esame**
>
> Una domanda frequente chiede quanti retry esegue Lambda "per default". La risposta corretta dipende dal modello. Nelle invocazioni **asincrone** sono **due** retry, quindi fino a tre tentativi in totale. Nelle invocazioni **sincrone**, come quelle di API Gateway, Lambda **non ritenta**: l'errore torna al chiamante, che decide se riprovare. Nelle invocazioni **da stream** il record viene ritentato finché non va a buon fine o non scade, salvo diversa configurazione, e può bloccare l'elaborazione della sua partizione o shard.

### Il ciclo di vita dell'ambiente di esecuzione

Quando si invoca una funzione, Lambda crea un **ambiente di esecuzione** (*execution environment*) in cui eseguirla. L'ambiente non viene distrutto subito dopo l'esecuzione: può restare attivo e servire **più invocazioni successive**. Il suo ciclo di vita ha quattro fasi:

1. **Download**: se è la prima esecuzione, Lambda scarica il codice della funzione e gli eventuali layer.
2. **Init**: l'ambiente viene inizializzato, cioè si avvia il runtime e si esegue tutto il codice che sta **fuori dall'handler**. Questa fase avviene **una volta sola** per ciclo di vita, quindi è il posto giusto per operazioni costose come creare i client SDK, aprire connessioni o popolare una cache. Le invocazioni successive troveranno l'ambiente nello stato lasciato dall'inizializzazione.
3. **Invoke**: Lambda chiama l'handler. Questa fase si ripete a ogni invocazione servita dall'ambiente.
4. **Shutdown**: dopo un periodo di inattività l'ambiente viene chiuso. Eventuali *extension* possono eseguire operazioni di chiusura.

Le fasi di download e init costituiscono il cosiddetto **cold start**: il tempo aggiuntivo che paga la prima invocazione servita da un nuovo ambiente. Le invocazioni successive sullo stesso ambiente (*warm start*) ne sono esenti.

### Concorrenza e cold start

Con più richieste contemporanee la situazione si complica. Un ambiente può servire **una sola invocazione alla volta**. Supponiamo che arrivi un primo evento: si crea un ambiente, con un cold start, e la funzione viene eseguita. Se un secondo evento arriva quando il primo è già terminato, riutilizza lo stesso ambiente senza cold start. Se invece un evento arriva mentre l'ambiente sta ancora elaborando la richiesta precedente, Lambda crea un **secondo ambiente**, con un nuovo cold start. Dopo un periodo di inattività gli ambienti inutilizzati vengono chiusi, e le richieste successive vengono servite da quelli rimasti attivi o da nuovi ambienti. Il numero di ambienti attivi in un dato momento è la **concorrenza** della funzione.

### Ridurre i cold start

Il cold start richiede tempo per scaricare il codice ed eseguire l'inizializzazione, mentre il riutilizzo degli ambienti porta grandi benefici. Per ridurre i cold start ci sono due approcci principali.

Il primo è tenere "calde" le funzioni con **Amazon EventBridge**. Si crea una regola con sintassi in stile *cron* che invoca la funzione a intervalli regolari, così da avere sempre almeno un ambiente pronto.

Il secondo è la **provisioned concurrency**: si chiede a Lambda di mantenere **pre-inizializzato** un certo numero di ambienti, per esempio 100 o 200, pronti a servire le invocazioni senza cold start.

La scelta è soprattutto una questione di **costi e di utilizzo**. Quanto costa la provisioned concurrency, e quanto sarà effettivamente usata? Per molte applicazioni i cold start non sono un problema, e non serve alcuno dei due accorgimenti.

> **Box – Altri strumenti contro i cold start**
>
> Per le funzioni Java esiste anche **Lambda SnapStart**, che crea un'istantanea dell'ambiente già inizializzato e la riprende alle invocazioni successive, riducendo drasticamente i tempi di avvio. In generale tutto ciò che alleggerisce l'inizializzazione riduce il cold start: pacchetti più piccoli, meno dipendenze e client SDK creati una sola volta fuori dall'handler.

## 8.5 I permessi di una funzione Lambda

Per una funzione Lambda contano **due tipi di permessi**, ed è fondamentale non confonderli.

I **permessi di invocazione** stabiliscono **chi può eseguire la funzione**: un principal AWS, un ruolo o un altro servizio, a seconda del tipo di integrazione. Si esprimono con una **resource-based policy** associata alla funzione. Quando qualcuno chiede di eseguirla, AWS controlla questa policy per verificare che sia autorizzato.

I **permessi di esecuzione** stabiliscono **che cosa può fare la funzione** mentre è in esecuzione, per esempio scrivere un messaggio in una coda SQS o un item in una tabella DynamoDB. Sono concessi dal **ruolo di esecuzione IAM** (*execution role*) associato alla funzione.

### Modello push

Nel modello push un servizio, come S3 o API Gateway, o un chiamante diretto invoca la funzione. Le policy in gioco sono due:

- la **resource policy della funzione**, che autorizza l'invocazione;
- il **ruolo di esecuzione**, che autorizza le azioni della funzione, per esempio inserire item in DynamoDB.

> **Box – Un classico del debug: nessun log**
>
> Tutto l'output di una funzione Lambda viene inviato a **CloudWatch Logs**, ed è il **ruolo di esecuzione** a consentirlo. Il ruolo predefinito creato dalla Console include i permessi necessari. Una situazione frequente: si sta scrivendo una funzione, non funziona, si va a guardare i log e… non c'è niente. Molto spesso la causa è un ruolo di esecuzione che non permette di scrivere su CloudWatch Logs, cioè che manca delle azioni `logs:CreateLogGroup`, `logs:CreateLogStream` e `logs:PutLogEvents`. La policy gestita `AWSLambdaBasicExecutionRole` le contiene.

### Modello pull

Nel modello pull è Lambda a leggere da uno stream: Kinesis, MSK o DynamoDB Streams. In questo caso il **ruolo di esecuzione** è usato per **due scopi**: autorizzare Lambda a **leggere dalla sorgente** e autorizzare le **azioni successive** della funzione, per esempio scrivere in DynamoDB. Lo stesso ruolo deve quindi contenere sia i permessi per leggere dallo stream sia quelli per la tabella di destinazione.

## 8.6 Sviluppare funzioni Lambda

### Gli strumenti

Una funzione si può scrivere direttamente nella **Console**, ma solo per funzioni piccole. Un'applicazione vera va messa sotto controllo di versione, coperta da unit test e inserita in un flusso di sviluppo completo. Gli strumenti disponibili sono molti:

- la **AWS CLI**, per impacchettare e distribuire le funzioni;
- gli **SDK**, usati come dipendenza dentro la funzione per chiamare i servizi AWS;
- **AWS SAM** (Capitolo 12), che automatizza molte attività dello sviluppo serverless e permette di simulare Lambda in locale, così da scrivere e provare molto codice prima di distribuirlo;
- l'**AWS Toolkit** per gli IDE, come Visual Studio Code o IntelliJ, integrato con SAM per eseguire e fare il debug delle funzioni in locale;
- il caricamento diretto di **pacchetti zip**.

### Il modello di programmazione

Programmare per Lambda è un po' diverso dalla programmazione tradizionale, soprattutto a causa del ciclo di vita visto prima. L'ambiente viene creato, serve alcune invocazioni e, dopo un periodo di inattività, scompare. All'interno dell'ambiente il codice vede quello che sembra un sistema operativo completo: può avviare **processi** e **thread** e può scrivere nella cartella **`/tmp`**. Bisogna però ricordare sempre che l'ambiente **può sparire in qualunque momento**.

La cartella `/tmp` è adatta come **cache**. Nel codice di inizializzazione si controlla se in `/tmp` c'è già quello che serve. Se non c'è, lo si scarica e lo si salva. Se l'ambiente scompare, il nuovo ambiente ripopolerà la cache.

Diverso è il caso dello **stato**. Lambda è **stateless**: non si deve memorizzare nell'ambiente di esecuzione alcuno stato che non si possa perdere. Lo stato va affidato a un altro servizio, come S3, DynamoDB o RDS. In un'applicazione di e-commerce, quando un utente aggiunge un prodotto al carrello, il carrello va salvato in DynamoDB, non in memoria nella funzione. Non c'è inoltre alcuna **affinità con l'infrastruttura sottostante**, a cui non si ha accesso: la richiesta successiva potrebbe essere servita da un ambiente completamente diverso.

### Monitoraggio e logging

Una funzione in produzione va osservata, e Lambda offre diversi strumenti:

- **metriche** visibili nella Console e in CloudWatch: numero di invocazioni, errori, durata, *throttling* (quando si superano i limiti di concorrenza) e concorrenza;
- **log in CloudWatch Logs**: tutto ciò che la funzione scrive nei log viene inviato automaticamente a CloudWatch Logs. Durante il debug è comodo seguirli in tempo reale da un terminale con `aws logs tail /aws/lambda/nome-funzione --follow`, mentre da un altro si invoca la funzione;
- **integrazione con AWS X-Ray**: X-Ray ricostruisce in un'unica *trace* tutte le chiamate generate da un'attività dell'applicazione e mostra quale componente introduce più latenza. Si attiva il tracing sulla funzione e si include l'SDK di X-Ray nel codice (Capitolo 13).

### L'handler: event e context

L'handler riceve sempre due parametri.

Il primo è l'**event**, un documento JSON che descrive ciò che ha causato l'invocazione. Se si invoca la funzione direttamente, il contenuto lo decide il chiamante. Se la invoca un servizio, l'evento ne descrive l'accaduto. Un evento S3, per esempio, contiene la Region, l'ora, il tipo di evento (`ObjectCreated:Put`), il principal che ha eseguito l'azione, l'indirizzo IP di origine, il nome del bucket, la chiave e la dimensione dell'oggetto:

```json
{
  "Records": [{
    "eventSource": "aws:s3",
    "awsRegion": "eu-south-1",
    "eventTime": "2022-10-01T12:00:00.000Z",
    "eventName": "ObjectCreated:Put",
    "userIdentity": { "principalId": "AWS:AIDA..." },
    "requestParameters": { "sourceIPAddress": "203.0.113.10" },
    "s3": {
      "bucket": { "name": "photos-bucket" },
      "object": { "key": "holiday/beach.jpg", "size": 1048576 }
    }
  }]
}
```

Una funzione che crea miniature leggerà da qui bucket e chiave, scaricherà l'immagine, genererà la miniatura e la salverà in un altro bucket. Esistono formati di evento per moltissimi servizi AWS.

Il secondo è il **context**, un oggetto con metodi e proprietà che descrivono l'invocazione e l'ambiente:

- il **tempo rimanente** prima del timeout. Il timeout massimo è di **15 minuti**, e una funzione lunga può controllare quanto tempo le resta per salvare lo stato o passare il lavoro a un'altra esecuzione;
- il **nome della funzione**, la **memoria** disponibile e altri dati utili al debug;
- l'**identità del chiamante**: quando la funzione è invocata tramite API Gateway con autenticazione Cognito, qui compaiono le informazioni sull'utente.

### I layer

I **layer** permettono di **condividere codice e dipendenze** tra funzioni. Alcune librerie sono molto pesanti, come l'SDK di Amazon SageMaker, e appesantiscono il pacchetto di ogni funzione che le usa. Invece di includerle nel pacchetto, si pubblicano in un layer, che poi si associa a tutte le funzioni che ne hanno bisogno. Un layer è, in pratica, un insieme di file aggiuntivi montati nel file system della funzione. Ci sono tre fonti di layer:

- i layer **creati dallo sviluppatore**;
- i layer **gestiti da AWS**, che contengono librerie molto diffuse;
- i layer **condivisi da altri account AWS**.

La morale di questo paragrafo è semplice: il codice in Lambda è codice come un altro, ma l'ambiente di esecuzione ha caratteristiche proprie, e bisogna conoscerle.

## 8.7 Laboratorio: sviluppare una funzione in Java

Questo laboratorio crea, distribuisce e configura una funzione Java interamente dalla riga di comando.

**Il codice.** La funzione legge un saluto dalla variabile d'ambiente `GREETING`, usa "Hello" come valore predefinito e restituisce la frase "<saluto> from Lambda":

```java
package example;

import com.amazonaws.services.lambda.runtime.Context;
import com.amazonaws.services.lambda.runtime.RequestHandler;
import java.util.Map;

public class Hello implements RequestHandler<Map<String, Object>, String> {
    @Override
    public String handleRequest(Map<String, Object> event, Context context) {
        String greeting = System.getenv().getOrDefault("GREETING", "Hello");
        return greeting + " from Lambda";
    }
}
```

**La build.** Le dipendenze sono dichiarate nel file `build.gradle`. Lo stesso risultato si potrebbe ottenere con Maven e un `pom.xml`. Il comando `gradle build` compila il codice ed esegue un task che crea l'archivio da consegnare a Lambda. Aprendo l'archivio si trovano la classe compilata e il file manifest.

**La creazione della funzione.** Serve un ruolo di esecuzione già esistente, il cui ARN viene salvato in una variabile d'ambiente:

```bash
export ROLE_ARN=arn:aws:iam::123456789012:role/lambda-basic-execution

aws lambda create-function \
    --function-name hello-java \
    --runtime java17 \
    --zip-file fileb://build/distributions/hello-java.zip \
    --handler example.Hello::handleRequest \
    --role $ROLE_ARN
```

L'**handler** indica pacchetto, classe e metodo nella forma `package.Class::method`. Il **ruolo** autorizza tutte le operazioni AWS eseguite dalla funzione, compresa la scrittura dei log su CloudWatch.

**L'invocazione.**

```bash
aws lambda invoke --function-name hello-java out.txt
cat out.txt
# "Hello from Lambda"
```

L'ultimo parametro è il file in cui viene scritta la risposta. La CLI riporta `StatusCode: 200`.

**La configurazione.** Le funzioni Lambda ricevono molta della loro configurazione tramite **variabili d'ambiente**, come il nome della tabella DynamoDB da usare. Si aggiorna la configurazione:

```bash
aws lambda update-function-configuration \
    --function-name hello-java \
    --environment "Variables={GREETING=Hola}"

aws lambda invoke --function-name hello-java out.txt
cat out.txt
# "Hola from Lambda"
```

Il codice non è cambiato, ma la funzione reagisce alla nuova configurazione.

## 8.8 Testare le funzioni Lambda

### Gli obiettivi dei test

Una funzione Lambda è codice come un altro e va testata **prima** di distribuirla. Il pacchetto di distribuzione, eventualmente affiancato da layer, offre un vantaggio in fase di test: si controlla con precisione quali versioni delle dipendenze vengono usate. Il codice si può verificare con **unit test** eseguiti in locale, simulando i servizi con cui comunica.

Oltre alla correttezza, i test di una funzione Lambda hanno obiettivi specifici:

- **testare le prestazioni** in relazione alla **memoria** e al **timeout**, che fanno parte della configurazione;
- **conoscere i limiti** del servizio (paragrafo 8.9);
- fare **right-sizing**: distribuire la funzione, simulare un carico di produzione e leggere da CloudWatch quanta memoria viene usata, per capire se beneficerebbe di più memoria. Ricordiamo che **più memoria significa anche più CPU**, e a volte aumentare la memoria riduce la durata fino a far diminuire il costo complessivo.

### Le opzioni di test

Ci sono diverse strade, da combinare:

- la **Console**: si definisce un test event e lo si esegue. È un test che avviene **dopo** la distribuzione;
- la **CLI**: `aws lambda invoke` accetta un payload. Nel repository si può tenere una cartella di eventi di esempio, da usare per provare la funzione distribuita;
- gli **SDK con mock**: un vero unit test verifica il codice in isolamento, quindi le chiamate ai servizi AWS vanno simulate. Esistono librerie che simulano i servizi e permettono di configurare risposte specifiche, per verificare come si comporta il codice in ogni caso. Per Python la più nota è **Moto**;
- **AWS SAM**: crea una struttura di progetto completa, con codice di esempio e **test già pronti** per Python, .NET e Java, e permette di invocare la funzione in locale.

### Invocare una funzione con un payload

```bash
aws lambda invoke \
    --function-name hello-java \
    --invocation-type Event \
    --cli-binary-format raw-in-base64-out \
    --payload '{"name": "test"}' \
    response.json
```

Il parametro `--invocation-type` stabilisce il modello di invocazione. **`RequestResponse`**, il predefinito, è **sincrono**. **`Event`** è **asincrono**. **`DryRun`** verifica solo i permessi senza eseguire la funzione. Il **payload** è il JSON passato come primo parametro, l'`event`, all'handler. L'ultimo parametro è il file in cui viene salvata la risposta.

### ARN, versioni e alias

Per invocare una funzione o per scrivere policy che la riguardano si usa spesso il suo ARN:

```
arn:aws:lambda:<region>:<account-id>:function:<nome>[:<versione>|:<alias>]
```

Il suffisso facoltativo indica una **versione** o un **alias**. Una **versione** è un'istantanea immutabile del codice e della configurazione della funzione. Un **alias** è un **puntatore** a una versione. Con gli alias si possono avere, per esempio, `dev`, `staging`, `test` e `prod`, ciascuno puntato a una versione diversa. I servizi chiamanti fanno riferimento all'alias, e per promuovere una nuova versione basta spostare il puntatore.

### La gestione degli errori

Gli errori di Lambda vanno distinti con attenzione.

Gli **errori del client (4xx)** si verificano prima dell'esecuzione del codice. Le cause tipiche sono la funzione inesistente, i parametri errati, una richiesta troppo grande, un **JSON non valido** come payload, la mancanza di permessi per invocare la funzione o il superamento dei **limiti** dell'account, come il numero di istanze o di richieste concorrenti.

Gli **errori del server (5xx)** sono problemi interni del servizio.

Gli **errori della funzione** (*function error*) nascono nel codice, per esempio da un'eccezione non gestita. In questo caso l'invocazione risponde comunque con **200**, ma il payload contiene i dettagli dell'errore, compreso lo **stack trace**, e la risposta riporta l'intestazione `X-Amz-Function-Error`. Per Lambda non è un suo errore, è un errore del codice, e il servizio fornisce tutto il necessario per analizzarlo.

Gli **errori di runtime** comprendono gli errori di sintassi, che emergono subito con il messaggio dell'interprete, per esempio Python, e i **timeout**, quando la funzione supera la durata configurata.

Serve quindi una **strategia di gestione degli errori**. Il consiglio pratico è di **loggare tutto il possibile** e costruire dashboard e allarmi sui log, così da sapere subito quando le funzioni iniziano a fallire.

## 8.9 Distribuire le funzioni Lambda

### Zip o container

Una funzione si può distribuire in due forme. La prima è un **archivio zip** con il codice e tutte le sue dipendenze. La seconda è un'**immagine di container**, costruita per esempio con Docker.

Le dipendenze vanno controllate con cura. Nel pacchetto si possono includere anche **dipendenze binarie**, ma devono essere compilate per l'ambiente in cui girerà la funzione, cioè **Amazon Linux**. Strumenti come SAM avviano un container Docker che simula quell'ambiente proprio per compilare correttamente le dipendenze binarie.

Due best practice valgono sempre:

- **ridurre al minimo la dimensione del pacchetto**: più byte devono essere trasferiti sull'host che eseguirà la funzione, più lungo è il cold start;
- **ridurre la complessità delle dipendenze**: è un buon consiglio per qualunque applicazione, ma in Lambda ha un impatto diretto sulle prestazioni.

### Il contenuto del pacchetto zip

| Linguaggio | Contenuto dello zip | Strumenti per le dipendenze |
|---|---|---|
| **Node.js, Python, Ruby** | Codice e dipendenze, anche binarie | npm, pip, RubyGems |
| **Java** | Classi compilate e JAR delle dipendenze, oppure un unico *uber JAR* | Maven, Gradle, plugin di Eclipse |
| **.NET** | DLL della funzione e delle dipendenze, oppure un'unica DLL | NuGet, file `.csproj`, Amazon.Lambda.Tools (`dotnet lambda`) |

Le dipendenze vanno collocate alla **radice** dell'archivio o nelle cartelle previste dal runtime: per Java le classi e le risorse alla radice e i JAR in `lib/`. Gli strumenti di build come il task di Gradle, il plugin Maven o lo strumento `dotnet lambda` producono già l'archivio nella struttura corretta.

### Distribuire con un'immagine di container

Il processo è questo:

1. si crea il progetto e si scrive il codice;
2. si parte da un'**immagine base fornita da AWS**, che include il **Lambda Runtime Interface Client**: il componente che comunica con il servizio Lambda e invoca il codice;
3. si carica l'immagine in **Amazon Elastic Container Registry (ECR)**;
4. si crea la funzione indicando l'immagine.

Le immagini di container possono arrivare fino a **10 GB**, ed è uno dei motivi principali per sceglierle: superare i limiti di dimensione dei pacchetti zip. L'altro motivo è usare un **runtime non supportato** da AWS. Per esempio, per del codice C++, per cui non esiste un runtime gestito, si può costruire un'immagine che lo contiene.

### I limiti del servizio

I limiti seguenti erano validi al momento della stesura del corso. AWS li aggiorna periodicamente, quindi conviene sempre verificare la documentazione.

| Risorsa | Limite |
|---|---|
| **Memoria** | Da 128 MB a 10.240 MB (10 GB). La memoria influisce sul prezzo |
| **Durata massima (timeout)** | 15 minuti |
| **Concorrenza** | Quota regionale dell'account (storicamente 1.000 per default, con capacità di burst iniziale variabile per Region), aumentabile su richiesta |
| **Payload dell'invocazione** | 6 MB per le invocazioni sincrone. 256 KB per le asincrone (valore del corso, verificare la documentazione per gli aggiornamenti) |
| **Pacchetto di distribuzione** | 50 MB compresso (upload diretto). 250 MB decompresso, layer inclusi |
| **Immagine di container** | 10 GB |
| **Spazio in `/tmp`** | Da 512 MB (predefinito) fino a 10 GB |

Alcune considerazioni pratiche. Assegnare 10 GB di memoria a ogni funzione è una tentazione, ma si paga anche quella: il dimensionamento è un esercizio di costo. Il limite dei **15 minuti** rende la programmazione in Lambda diversa dal solito, ma molte applicazioni seguono un modello a microservizi in cui ogni richiesta si completa in millisecondi. Per i **payload** troppo grandi la soluzione è passare un **riferimento**: si scrivono i dati in S3 e l'evento contiene solo la posizione dell'oggetto da elaborare. Per i **pacchetti** troppo grandi la soluzione è il container. È il caso tipico di librerie voluminose come l'SDK di SageMaker, che una volta decompresso supera i 250 MB.

## 8.10 Verifica dei concetti

**"AWS Lambda gestisce per te server, capacità e necessità di distribuzione."** *Vero.* Il termine "distribuzione" può trarre in inganno, perché la funzione va comunque distribuita la prima volta. Ma l'infrastruttura su cui gira, il suo scaling e la sostituzione delle versioni sono gestiti dal servizio.

**"La tua funzione Lambda ha bisogno di permessi di *invocazione* per accedere ad altre risorse AWS del tuo account."** *Falso.* I permessi di invocazione stabiliscono chi può invocare la funzione. Per accedere ad altre risorse servono i **permessi di esecuzione**, concessi dal ruolo di esecuzione.

**"Sviluppare e creare una funzione Lambda è possibile solo tramite la console AWS Lambda."** *Falso.* Per funzioni piccole in Node.js o Python la Console può bastare. Per Java e .NET l'editor della Console non è nemmeno disponibile. Si usano IDE, CLI, SAM e gli altri strumenti di sviluppo.

**"Se si abilitano i DynamoDB Streams su una tabella, si può associare l'ARN dello stream a una funzione Lambda."** *Vero.* Lo stream diventa una event source con modello **pull**: è Lambda a leggere gli eventi, e a autorizzarlo è il **ruolo di esecuzione**.

**"Ogni funzione Lambda viene eseguita in un ambiente isolato, con risorse e vista del file system proprie."** *Vero.* Ogni funzione ha il proprio ambiente di esecuzione, compresa la cartella `/tmp`. L'ambiente è effimero: se vi si memorizza una cache, il codice deve essere pronto a ripopolarla in qualunque momento.

**"Il modello di programmazione di AWS Lambda è stateless, ma può comunque accedere a dati con stato."** *Vero.* Lo stato, come un carrello della spesa, non va conservato nell'ambiente di esecuzione ma in un servizio esterno: una tabella DynamoDB, un altro database o una cache.
