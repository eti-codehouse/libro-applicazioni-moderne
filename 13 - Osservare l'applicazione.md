# Capitolo 13 – Osservare l'applicazione

## 13.1 Obiettivi del capitolo

Un'applicazione in produzione va **osservata**: bisogna sapere in ogni momento se funziona, e quando qualcosa va storto bisogna capire perché e dove. Questo capitolo distingue anzitutto tra **monitoring** e **observability** e spiega perché l'observability è indispensabile nello sviluppo moderno e quali sono i suoi **componenti chiave**.

Si vedrà poi come **Amazon CloudWatch** aiuta a raggiungere l'observability, con metriche, log e allarmi, e come **CloudWatch Application Insights** semplifica il monitoraggio delle applicazioni. Infine si mostrerà come usare **AWS X-Ray** per fare il debug delle applicazioni distribuite, seguendo ogni richiesta attraverso i componenti che attraversa.

## 13.2 L'observability

### Monitoring e observability

L'**observability** è la capacità di **osservare, comprendere e usare i dati** per conoscere lo stato delle proprie applicazioni. Viene spesso confusa con il **monitoring**, ma i due concetti sono diversi.

Il **monitoring** è l'atto di **raccogliere i dati** da cui ricavare informazioni operative. Aiuta a scoprire **che** c'è un problema. Può limitarsi alla raccolta di metriche di prestazione come l'utilizzo della CPU.

L'**observability** aiuta a scoprire **perché** si verifica un problema operativo. Va oltre le metriche di prestazione: raccoglie **metriche e log da tutte le risorse AWS, le applicazioni e i servizi**. Il monitoring resta, e si continuano a raccogliere tutte le metriche consuete, ma con i dati giusti, presi dai log e dalle applicazioni, si possono **analizzare** le informazioni e ottenere *insight* che permettono di **agire** e di **automatizzare la risposta** ai cambiamenti operativi.

In pratica, l'observability significa poter guardare una dashboard, o scendere nel dettaglio dei dati prodotti dalle applicazioni, e capire **in generale lo stato dell'applicazione** senza doversi collegare a un server per leggerne i log. Le domande a cui deve rispondere sono di questo tipo:

- l'applicazione è in buono stato o qualcosa è rotto?
- gli utenti ricevono molti errori 500? Gli errori 500 sono in aumento?
- c'è un problema di memoria sul database?
- le latenze sono aumentate? Per esempio di una certa percentuale nelle ultime due ore?
- e se la latenza è aumentata, **dove** è aumentata?

L'ultima domanda è la più importante, perché è quella che indirizza la diagnosi.

### Perché serve un piano di observability

L'observability offre **visibilità** sull'applicazione, permette il **troubleshooting in tempo reale** e migliora la **customer experience**. Oggi, inoltre, **le prestazioni equivalgono a fatturato**: se qualcosa è lento, per l'utente è come se fosse rotto. Gli utenti tollerano raramente la lentezza. Davanti a una pagina che non risponde, cliccano più volte il pulsante o ricaricano la pagina, e di certo non aspettano dieci secondi una risposta. Un'applicazione lenta o malfunzionante fa calare il fatturato nel tempo.

Serve quindi un **piano**: come si raccoglieranno metriche e dati? Come si verificherà che le prestazioni e l'esperienza dei clienti siano buone? Come si osserverà lo stato dell'applicazione senza doverci scavare dentro ogni volta?

### Dai dati all'azione

Un piano di observability trasforma progressivamente i dati in azioni, attraverso cinque livelli:

1. **Dati**: le misurazioni grezze raccolte. Da soli dicono poco.
2. **Informazione**: i dati messi nel **contesto**.
3. **Conoscenza**: la comprensione dello **stato del sistema**.
4. **Insight**: allargando lo sguardo, la capacità di capire **che cosa non va** nell'applicazione.
5. **Azione**: l'intervento che risolve il problema.

Tutti questi passaggi si possono **automatizzare**. Ci saranno sempre persone che controllano le dashboard, soprattutto durante il troubleshooting, ma molte azioni possono avvenire in modo automatico. L'obiettivo è progettare l'applicazione in modo che le informazioni ne escano e arrivino in un luogo in cui siano **utilizzabili per agire**.

### I tre pilastri dell'observability

L'observability poggia su **tre pilastri**.

I **log** sono la **registrazione degli eventi** di un'applicazione: log applicativi, log di accesso, e i diversi tipi di log prodotti dai servizi AWS e dalle applicazioni. Vanno portati tutti in un **luogo centralizzato**, così da non doversi mai collegare ai server per raccoglierli.

Le **metriche** sono **dati numerici** relativi alle prestazioni e al comportamento complessivo del sistema. Un esempio semplice è l'utilizzo della CPU. Un esempio più significativo è la **profondità di una coda SQS**, cioè il numero di messaggi in attesa: se la coda si allunga molto, l'elaborazione non tiene il passo e gli eventi non vengono consumati abbastanza in fretta.

Le **trace** mostrano **dove sono i colli di bottiglia** nell'applicazione. Un'applicazione è una serie di componenti interconnessi, e un rallentamento visibile nell'interfaccia utente può essere causato da un servizio molto in profondità nel sistema. Invece di controllare ogni servizio uno per uno, si vuole aprire una **mappa** del sistema e vedere visivamente quali servizi causano il rallentamento, con i tempi di richiesta e risposta di ciascun componente.

| Pilastro | Che cosa rappresenta | Servizio AWS |
|---|---|---|
| **Log** | Eventi registrati da applicazioni e servizi | **Amazon CloudWatch Logs** |
| **Metriche** | Valori numerici nel tempo su prestazioni e comportamento | **Amazon CloudWatch Metrics** |
| **Trace** | Il percorso di una richiesta attraverso i componenti, con le relative latenze | **AWS X-Ray** |

### I servizi AWS per i tre pilastri

**CloudWatch Logs** raccoglie i log delle applicazioni. Per un'applicazione in **Lambda** è quasi una semplice casella da spuntare: la funzione invia i propri log a CloudWatch Logs. Avere un luogo centralizzato è essenziale, perché con Lambda tutto è **effimero**. Se un'invocazione ha avuto un problema, i suoi log non restano da nessuna parte nell'ambiente Lambda, e vanno quindi inviati a un sistema esterno.

Anche le istanze **EC2** vanno trattate come **stateless**, allo stesso modo. Invece di entrare nel server per leggere i log, si installa il **CloudWatch agent**, che li spedisce a CloudWatch Logs. Così restano tutti in un unico posto, anche se il server viene terminato o riavviato.

**CloudWatch Metrics** raccoglie le metriche: la maggior parte dei servizi AWS vi invia metriche **per default**, e vi si possono pubblicare **metriche personalizzate** per tutto ciò che non viene raccolto automaticamente.

**AWS X-Ray** è il servizio di **tracing**: costruisce la mappa dei componenti e mostra da dove arriva la latenza.

## 13.3 Osservare un'applicazione con Amazon CloudWatch

### Metriche e log

**Amazon CloudWatch** monitora le risorse AWS. Molti servizi vi pubblicano metriche **automaticamente**: un'istanza EC2, appena avviata, inizia a inviare metriche a intervalli regolari. In CloudWatch si creano poi **dashboard** che offrono una visibilità visiva e in tempo reale sulle risorse.

"In tempo reale" va precisato, perché la **granularità** è configurabile. Per EC2 il *basic monitoring* invia metriche ogni 5 minuti, mentre il *detailed monitoring* lo fa ogni minuto. Le metriche personalizzate possono essere ad **alta risoluzione**, fino a un secondo.

Sotto l'ombrello di CloudWatch ci sono due cose distinte: **CloudWatch Metrics**, su cui si costruiscono le dashboard, e **CloudWatch Logs**. La differenza pratica è importante. Molte metriche arrivano automaticamente. I **log** delle applicazioni su EC2 **non** vengono invece inviati a CloudWatch Logs per default: bisogna configurarlo nel progetto oppure installare il **CloudWatch agent**.

### Come funziona CloudWatch

Il flusso dei dati in CloudWatch è il seguente.

**Le sorgenti.** Le **risorse AWS** pubblicano le proprie metriche. A queste si aggiungono i **dati personalizzati**, perché le metriche predefinite non sempre sono le più significative per un caso d'uso specifico. Le metriche personalizzate si pubblicano con l'**SDK**, con la **CLI** (`aws cloudwatch put-metric-data`) o con altri strumenti.

**Le statistiche.** Le metriche, raccolte nel tempo, diventano **statistiche**, come media, minimo, massimo, somma e percentili, calcolate su un **periodo**. Si visualizzano in **dashboard** composte da **widget**, ciascuno dei quali rappresenta una o più metriche nel tempo, e su di esse si prendono decisioni.

**La consultazione.** Le statistiche si consultano nella Management Console o si inviano ad altri consumatori.

**Gli allarmi.** Gli **allarmi** di CloudWatch tengono d'occhio una metrica e reagiscono quando questa supera una **soglia** per un certo tempo. Per esempio, se l'utilizzo della CPU supera il 70% per cinque minuti, l'allarme può inviare un'email a un amministratore tramite **Amazon SNS** oppure avviare un'azione di **Auto Scaling** per aggiungere istanze EC2 al gruppo e far fronte al picco di domanda.

È un sistema integrato: le risorse pubblicano su CloudWatch, e gli allarmi di CloudWatch avviano azioni su altri servizi.

### I componenti chiave di CloudWatch

Le **metriche** sono serie temporali di **data point**.

Le **dimensioni** (*dimensions*) sono come **categorie** delle metriche. Sono coppie **nome/valore** che fanno parte dell'**identità** di una metrica, come `InstanceId=i-0abc…` o `TableName=Notes`. Il corso indica fino a 10 dimensioni per metrica, mentre oggi CloudWatch ne consente fino a 30.

Gli **allarmi** sorvegliano **una singola metrica** per un periodo di tempo ed eseguono **azioni** in base al suo valore rispetto a una soglia.

I **log** sono pubblicati da servizi AWS e applicazioni e permettono di conservare in un unico luogo la registrazione delle attività.

I **metric filter** sono una funzionalità particolarmente utile: se nei log c'è un evento specifico da cercare, si crea un filtro che lo **trasforma in una metrica**. Per esempio, "se questo messaggio compare nei log X volte in un certo periodo, voglio essere avvisato". Sulla metrica così ottenuta si può poi impostare un allarme.

### Esempio: una metrica di DynamoDB

Quando si crea un allarme, la Console mostra le metriche disponibili raggruppate **per servizio**. Selezionando **DynamoDB** compaiono le metriche del servizio, divise per **dimensioni**. Per esempio, la combinazione delle dimensioni `TableName = Notes` e `GlobalSecondaryIndexName = User-NoteId-Index` individua l'indice secondario globale della tabella delle note. La metrica scelta è **`ConsumedReadCapacityUnits`**, la capacità di lettura consumata dall'indice. Su questa metrica si può creare un allarme e agire in base al suo andamento, per esempio aumentando la capacità provisionata.

### Il funzionamento di un allarme nel tempo

Un allarme non scatta ogni volta che una metrica supera la soglia. Consideriamo una metrica che fluttua nel tempo, con una soglia definita. La metrica sale e supera la soglia, ma per far scattare l'allarme il valore deve **restare oltre la soglia per un certo periodo**. I brevi picchi sono normali, e non si vuole reagire a ciascuno di essi. Si vuole reagire solo a un **evento prolungato**. Trascorso quel periodo, l'azione viene invocata, per esempio aggiungendo istanze.

Supponiamo che poi il valore scenda e risalga di nuovo sopra la soglia, ma solo per un momento, e torni subito giù. Non è un evento prolungato ma un breve picco, e non viene eseguita alcuna nuova azione.

Valutare la metrica su un periodo evita il cosiddetto **thrashing**: aggiungere un gruppo di istanze all'Auto Scaling group, poi toglierle, poi riaggiungerle, poi toglierle ancora. L'approccio è più misurato. Tecnicamente un allarme si configura con il **periodo** di aggregazione, il numero di **periodi di valutazione** e il numero di **data point** che devono superare la soglia (*M out of N*). Un allarme si trova sempre in uno di tre stati: `OK`, `ALARM` o `INSUFFICIENT_DATA`.

### Portare i log in CloudWatch Logs

Per le applicazioni su EC2 ci sono due strade.

La prima è il **CloudWatch agent**. Lo si installa sull'istanza e lo si configura indicando dove l'applicazione scrive i propri log. L'agent li legge e li pubblica su CloudWatch Logs.

La seconda sono i **plug-in di logging**. Per alcune librerie di logging esistono plug-in, o *appender*, che inviano automaticamente i log a CloudWatch Logs senza bisogno dell'agent: basta configurarli nell'applicazione.

### Configurare l'applicazione per CloudWatch

CloudWatch è disponibile tramite gli **AWS SDK** e la **CLI**, e l'applicazione può interagirvi in molti modi. Per esempio, da un'istanza EC2 con il CloudWatch agent installato si possono eseguire comandi della CLI per creare allarmi (`put-metric-alarm`) o pubblicare **metriche personalizzate** (`put-metric-data`) che CloudWatch non raccoglie per default:

```bash
# Pubblicare una metrica personalizzata: note create nell'ultimo minuto
aws cloudwatch put-metric-data \
    --namespace "PollyNotes" \
    --metric-name NotesCreated \
    --dimensions Environment=prod \
    --value 42 \
    --unit Count

# Creare un allarme sull'utilizzo della CPU di un'istanza
aws cloudwatch put-metric-alarm \
    --alarm-name high-cpu \
    --namespace AWS/EC2 \
    --metric-name CPUUtilization \
    --dimensions Name=InstanceId,Value=i-0123456789abcdef0 \
    --statistic Average \
    --period 300 \
    --evaluation-periods 1 \
    --threshold 70 \
    --comparison-operator GreaterThanThreshold \
    --alarm-actions arn:aws:sns:us-east-1:123456789012:ops-alerts
```

### CloudWatch Application Insights

**CloudWatch Application Insights** è una funzionalità di CloudWatch che offre una visione più approfondita delle applicazioni **senza dover configurare tutto manualmente**.

Supponiamo di aver distribuito un'applicazione con **CloudFormation**: uno stack con un Application Load Balancer, alcune istanze EC2 e un database. In Application Insights si registra un **resource group** basato su quello stack, e tutte le risorse create dal template vengono incluse nel monitoraggio. Si indica poi dove si trovano i log dell'applicazione, e Application Insights li raccoglie. Il passo successivo è il **rilevamento intelligente dei problemi**. Application Insights conosce in generale che cosa può andare storto nei diversi tipi di applicazione, come .NET, SQL Server o Java, e il tipo si sceglie in fase di configurazione. Di conseguenza **raccomanda e configura** automaticamente le metriche, i log e gli allarmi più adatti. Analizza poi i dati in modo continuo. Se rileva un'anomalia, per esempio un aumento delle risposte **HTTP 500** dal load balancer, invia un avviso o mostra nella Console che c'è un problema per quell'applicazione, correlando i segnali dei diversi componenti.

È quindi un ottimo modo per avere visibilità su log e metriche senza dover decidere da soli quali metriche osservare e quali allarmi impostare.

## 13.4 Il tracing con AWS X-Ray

### Che cos'è AWS X-Ray

L'ultimo pilastro è il **tracing**, e il servizio AWS dedicato è **AWS X-Ray**. X-Ray permette di implementare il tracing per le **applicazioni distribuite** su AWS. Raccoglie i dati sulle **richieste** e offre strumenti per visualizzarli nella Console:

- una **rappresentazione visiva**, cioè una **mappa** dei componenti, con il loro stato di salute e la **latenza** tra di essi;
- strumenti di **filtro** per trovare rapidamente una trace specifica e scendere nel dettaglio;
- il **tracciamento delle richieste** attraverso l'applicazione. Si vede che cosa accade dal momento in cui il client invia la richiesta, il percorso che compie, la latenza di ogni passaggio e le informazioni sulla richiesta in ciascun componente.

### Come funziona X-Ray

L'architettura di X-Ray si basa su tre elementi.

Il primo è l'**X-Ray SDK**, incluso nel codice dell'applicazione. Registra le chiamate verso i servizi AWS e le chiamate personalizzate.

Il secondo è l'**X-Ray daemon**, un processo che gira sul nodo in cui gira il codice: in un ambiente di esecuzione Lambda, dove è già integrato, in un container o su un'istanza EC2. Raccoglie i dati prodotti dall'SDK, inviati via UDP, e li pubblica sull'**X-Ray API**.

Il terzo è il **servizio X-Ray**, che riceve i dati, li elabora e li rende consultabili nella **Console**.

Ogni volta che l'applicazione chiama un servizio AWS, o quando si definisce una chiamata di tracing personalizzata, l'SDK crea un **segment** e lo invia a X-Ray, insieme ai **subsegment**, cioè dati aggiuntivi su quelle chiamate e sulle chiamate a valle. Il daemon raccoglie tutto e lo invia al servizio. Nella Console si visualizza infine la mappa dei servizi.

### La service map

Per Polly Notes la mappa mostra il **client**, poi l'**API** su API Gateway, poi le diverse **funzioni Lambda** (list, update, dictate e così via), e infine la tabella **DynamoDB** di backend e il servizio **Amazon Polly**. È la cosiddetta **service map**, consultabile nella Console.

### I concetti chiave: trace, segment, subsegment

La **trace** è la registrazione **completa** di una richiesta, dall'utente finale fino all'ultimo componente dell'applicazione. È identificata da un *trace ID*, propagato tra i servizi nell'intestazione `X-Amzn-Trace-Id`.

Una trace è suddivisa in **segment**. Ogni segment contiene le informazioni sul lavoro svolto da un servizio per quella richiesta.

Ogni segment si può scomporre in **subsegment**, che offrono **dettagli di temporizzazione più granulari** e informazioni sulle **chiamate a valle** che l'applicazione ha effettuato per soddisfare la richiesta originale.

L'organizzazione è quindi **gerarchica**: trace, poi segment, poi subsegment. Un esempio con Lambda la rende chiara. Il **segment** è "è stata invocata una funzione Lambda". Al suo interno ci sono diversi **subsegment**. Il primo è l'**inizializzazione**: l'ambiente di esecuzione è stato avviato, il codice scaricato e il runtime inizializzato. È la fase del *cold start* vista nel Capitolo 8. Il secondo è l'**invocazione**, cioè l'esecuzione dell'handler. Il terzo, a un livello ancora più profondo, è la **chiamata a DynamoDB** fatta dalla funzione.

Più si scende, più il dettaglio aumenta. Configurando l'applicazione perché X-Ray catturi le **chiamate API AWS a valle**, ogni chiamata a DynamoDB, S3 o altri servizi compare come subsegment.

### Annotation e metadata

Oltre ai dati raccolti automaticamente, si possono aggiungere informazioni personalizzate a segment e subsegment, di due tipi.

Le **annotation** sono coppie chiave-valore **indicizzate**, e quindi **utilizzabili nei filtri**. Per esempio, un'annotation `UserId = student` permette di filtrare nella Console solo le trace relative a quello studente. In produzione le trace sono moltissime, e le annotation sono il modo per arrivare a quelle che interessano.

I **metadata** sono coppie chiave-valore **non indicizzate**, e quindi **non filtrabili**. Servono ad aggiungere informazioni extra su una richiesta, utili per il troubleshooting o il debug una volta trovata la trace che interessa. Un esempio è il `request_id` o un oggetto con i parametri della richiesta.

> **Box – Annotation o metadata?**
>
> La regola da ricordare è semplice: **le annotation sono ricercabili, i metadata no**. Si usano le annotation per i valori su cui si vorrà filtrare, come ID utente, tipo di operazione o ambiente. Si usano i metadata per i dati di contesto da consultare una volta trovata la trace, anche oggetti complessi.

### Leggere una trace

Nella Console, la mappa di una **singola trace** mostra il percorso di quella richiesta: il client ha chiamato l'API, l'API ha invocato la funzione Lambda *list*, che ha interrogato DynamoDB. Sotto la mappa la **timeline** mostra la **durata di ogni passaggio**. Se un client lamenta latenza e le sue trace lo confermano, si vede esattamente quale parte rallenta. Se per esempio la tabella DynamoDB subisce **throttling**, la latenza si concentrerà nel subsegment della chiamata a DynamoDB. È proprio questo il valore del tracing: capire **che cosa** accade nell'applicazione e **dove**.

### La service map aggregata e i suoi colori

La service map **aggregata** mostra tutte le trace insieme: il client, l'API e le diverse funzioni Lambda. I nodi hanno **colori diversi** che ne indicano lo stato di salute:

| Colore | Significato |
|---|---|
| **Verde** | Richieste completate con successo |
| **Giallo** | **Errori del client** (4xx) |
| **Rosso** | **Fault del server** (5xx) |
| **Viola** | Errori di **throttling** (429) |

Confrontando la service map con il diagramma architetturale di Polly Notes (utente, Cognito, API Gateway, funzioni Lambda, tabella di backend) si nota una forte somiglianza. La service map è quindi anche un ottimo modo per **rappresentare visivamente l'architettura**, vista dall'alto dal punto di vista del percorso delle richieste.

## 13.5 Abilitare il tracing nell'applicazione

### La strumentazione con gli SDK

Per usare X-Ray bisogna **strumentare** l'applicazione con l'X-Ray SDK. La procedura cambia un po' a seconda del linguaggio.

In **.NET** si aggiunge il pacchetto dell'X-Ray SDK come dipendenza e poi si scrive:

```csharp
// Traccia tutte le chiamate a tutti i servizi AWS
AWSSDKHandler.RegisterXRayForAllServices();

// Oppure solo per un servizio specifico
AWSSDKHandler.RegisterXRay<IAmazonDynamoDB>();
```

`RegisterXRayForAllServices` cattura il tracing di **ogni chiamata API AWS**. Se non è il comportamento desiderato, `RegisterXRay<T>` lo limita a un servizio specifico, qui DynamoDB tramite l'interfaccia del suo client.

In **Java** si includono le dipendenze dell'X-Ray SDK nel `pom.xml` o nel file Gradle. Basta includere il modulo di strumentazione automatica perché vengano tracciate tutte le chiamate dell'AWS SDK. In alternativa si strumenta un client specifico passando un *tracing interceptor* al builder:

```java
DynamoDbClient ddb = DynamoDbClient.builder()
        .overrideConfiguration(ClientOverrideConfiguration.builder()
                .addExecutionInterceptor(new TracingInterceptor())
                .build())
        .build();
```

In **Python** si installa l'X-Ray SDK nel progetto. La classe **`xray_recorder`** fornisce un recorder globale, e la funzione `patch_all()` strumenta automaticamente le librerie supportate, come boto3 e requests, registrando tutte le chiamate API AWS a valle:

```python
from aws_xray_sdk.core import xray_recorder, patch_all

patch_all()   # traccia boto3, requests e altre librerie supportate

@xray_recorder.capture("list_notes")
def list_notes(user_id):
    xray_recorder.put_annotation("UserId", user_id)
    xray_recorder.put_metadata("request", {"user_id": user_id})
    return table.query(KeyConditionExpression=Key("UserId").eq(user_id))
```

### Un esempio completo in .NET

Una volta chiamato `AWSSDKHandler.RegisterXRayForAllServices()`, tutte le chiamate successive ai servizi AWS (DynamoDB, S3, Parameter Store…) vengono catturate. Lo stesso esempio mostra un secondo caso d'uso: tracciare non una chiamata AWS ma una **query SQL** verso un database relazionale. Per questo si usa **`TraceableSqlCommand`** al posto del normale `SqlCommand`:

```csharp
using (var connection = new SqlConnection(connectionString))
using (var command = new TraceableSqlCommand("SELECT * FROM Notes WHERE UserId = @id", connection))
{
    command.Parameters.AddWithValue("@id", userId);
    connection.Open();
    using var reader = command.ExecuteReader();
    // ...
}
```

La query compare così nella trace come subsegment, con la sua durata.

L'X-Ray SDK serve anche ad aggiungere **annotation** e **metadata** e a creare **subsegment personalizzati** per misurare porzioni specifiche del codice. È uno strumento molto configurabile, e il suo uso si può estendere ben oltre la strumentazione automatica.

> **Box – X-Ray e OpenTelemetry**
>
> AWS sta spostando la strumentazione verso lo standard aperto **OpenTelemetry**, tramite la **AWS Distro for OpenTelemetry (ADOT)** e l'agente CloudWatch, e gli X-Ray SDK sono entrati in fase di manutenzione. I concetti restano gli stessi (trace, segment o span, annotation, service map), e le trace continuano a essere visualizzate in X-Ray e in CloudWatch. Per i nuovi progetti conviene valutare OpenTelemetry, ma per l'esame i concetti e le API di X-Ray restano il riferimento.

## 13.6 Laboratorio: AWS X-Ray

Il laboratorio usa una semplice funzione Lambda *hello-world* esposta tramite API Gateway, con il tracing X-Ray abilitato.

**Abilitare il tracing su Lambda.** Nella Console Lambda si seleziona la funzione, si apre la scheda *Configuration* e si sceglie *Monitoring and operations tools*. Si vede che **log e metriche sono abilitati per default**, mentre il **tracing non lo è**. Attivando l'*Active tracing*, Lambda invierà le trace a X-Ray e si potrà generare la service map.

**Abilitare il tracing su API Gateway.** Il progetto è stato generato con **SAM** a partire dal template standard *hello-world*. Durante la creazione SAM ha chiesto se abilitare il tracing, e la risposta è stata sì, così ha configurato tutto automaticamente. Nel template questo corrisponde a:

```yaml
Globals:
  Function:
    Tracing: Active
  Api:
    TracingEnabled: true
```

Nella Console di API Gateway, in *Stages → Prod → Logs and tracing*, si verifica che il **tracing X-Ray** sia attivo.

**Generare traffico.** Si inviano alcune richieste all'*invoke URL* dello stage, per esempio con **Postman**.

**La service map.** Nella Console di X-Ray compare la **service map** con tre nodi: il **client**, cioè le richieste inviate con Postman, l'**API Gateway** con lo stage `prod`, e la **funzione Lambda**.

**La panoramica delle trace.** Nella sezione *Traces* la panoramica riporta un **tempo di risposta medio di 402 ms**, con **4 richieste OK**, **0 in throttling**, **0 errori** e **0 fault**. Già da qui si capisce ad alto livello come si comporta il servizio.

**Il dettaglio di una trace.** Selezionando la prima trace della lista, X-Ray calcola la mappa di quella singola richiesta. Il client ha chiamato API Gateway, che in questa trace ha impiegato **29 ms** in totale e ha invocato la funzione Lambda. La funzione ha effettivamente eseguito il codice per **4 ms**: restituisce semplicemente un saluto, e non fa molto altro. Scorrendo verso il basso si vedono i **subsegment**:

- **Initialization**: 12 ms, il cold start;
- **Invocation**: 3 ms, l'esecuzione dell'handler;
- **Overhead**: 0,2 ms.

Scendendo nel dettaglio di segment e subsegment si ottiene una rappresentazione visiva molto chiara di ciò che accade nell'applicazione, preziosa per il troubleshooting e il debug quando si presentano problemi.

## 13.7 Verifica dei concetti

**"Usa CloudWatch Logs per centralizzare i log di tutti i tuoi sistemi, applicazioni e servizi AWS."** *Vero.* Lambda invia i log da sola, e sulle istanze EC2 si installa un agent che li spedisce. Avere tutto in un luogo centralizzato significa sapere sempre dove cercare le informazioni sul sistema.

**"I tre pilastri dell'observability sono log, trace e disponibilità."** *Falso.* La parola fuori posto è *disponibilità*: i tre pilastri sono **log, metriche e trace**.

**"Una trace mostra un solo subsegment generato da una richiesta."** *Falso.* Una trace di una funzione Lambda mostra per esempio l'inizializzazione, l'invocazione e la chiamata a DynamoDB, tutti subsegment del segment più grande. La struttura è gerarchica: trace, segment, subsegment.

**"Le trace sono una rappresentazione numerica dei dati usata per analizzare le prestazioni e il comportamento complessivo di un sistema."** *Falso.* Questa è la definizione di una **metrica**, per esempio "quanti login falliti nell'ultima ora?". Una trace è una gerarchia di segment che descrive il percorso di una richiesta.

**"Gli allarmi sorvegliano una singola metrica per un periodo di tempo specificato ed eseguono un'azione specificata in base al valore della metrica rispetto a una soglia nel tempo."** *Vero.* È il caso dell'Auto Scaling: si associa un allarme a una metrica dell'istanza, come l'utilizzo della CPU, e quando supera per esempio il 60% per il periodo stabilito si esegue l'azione di scale-up.

**"I subsegment forniscono informazioni di temporizzazione più granulari e dettagli sulle chiamate a valle che l'applicazione ha effettuato per soddisfare la richiesta originale."** *Vero.* Un esempio è la funzione Lambda che chiama DynamoDB. In Python, inoltre, l'SDK X-Ray può strumentare la libreria *requests*, e ogni chiamata HTTP fatta con essa compare come subsegment.
