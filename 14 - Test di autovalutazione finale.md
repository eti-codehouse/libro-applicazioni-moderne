# Parte VII – Verifica finale

# Capitolo 14 – Test di autovalutazione finale

## 14.1 Come usare questo capitolo

Questo capitolo raccoglie venti domande nello stile dell'esame **AWS Certified Developer – Associate**, che coprono tutti gli argomenti del libro. Per sfruttarlo al meglio conviene procedere in tre passaggi. Prima si risponde a tutte le domande senza guardare le soluzioni, annotando le risposte su un foglio. Poi si confrontano le risposte con le soluzioni commentate: per ogni domanda la spiegazione dice non solo quale opzione è corretta, ma **perché** le altre sono sbagliate. È questo ragionamento che serve all'esame, dove le opzioni errate sono spesso plausibili. Infine si usa la tabella del paragrafo 14.3 per tornare ai capitoli in cui si è sbagliato.

Le domande sono raggruppate per area tematica e mantengono la numerazione originale. In alcuni casi la soluzione fornita dal materiale del corso era incompleta o imprecisa. Qui è stata corretta, e la correzione è segnalata nel commento.

## 14.2 Le venti domande commentate

### Area 1 – Strumenti di sviluppo: SDK, CLI, IDE

**Domanda 1.** Devi implementare una routine di automazione per avviare la tua flotta di web server basati su EC2. Il processo applicherà un insieme di tag per identificare i web server nell'ambiente. Quali opzioni potresti usare per implementare questo processo di automazione? *(Seleziona DUE.)*

- A. Amazon DynamoDB
- B. AWS CLI
- C. AWS SDK in un linguaggio di programmazione
- D. AWS API

**Risposta: B e C.** Per automatizzare l'avvio e il tagging di istanze EC2 si scrive uno script con la **AWS CLI** (`aws ec2 run-instances --tag-specifications ...`) oppure un programma che usa un **AWS SDK** (per esempio `RunInstances` con boto3 o con l'SDK for Java). DynamoDB è un database e non c'entra con l'avvio delle istanze. Le API di AWS sono il livello su cui si appoggiano CLI e SDK, come visto nel Capitolo 2. In teoria si potrebbero chiamare direttamente, ma bisognerebbe costruire e firmare a mano le richieste HTTP con SigV4, ed è proprio ciò che CLI e SDK evitano. La risposta attesa sono quindi i due strumenti di automazione veri e propri.

**Domanda 11.** Devi sviluppare un'applicazione che sfrutta AWS, ma non vuoi installare e configurare un ambiente di sviluppo integrato locale. Quale servizio AWS potresti usare per sviluppare l'applicazione in un IDE basato sul cloud?

- A. AWS CloudFormation
- B. AWS CodeDeploy
- C. AWS Cloud9
- D. AWS CodeBuild

**Risposta: C.** **AWS Cloud9** è un IDE nel browser per scrivere, eseguire e fare il debug del codice (Capitolo 2). CloudFormation è Infrastructure as Code, CodeDeploy automatizza i deploy e CodeBuild compila e testa: nessuno dei tre è un IDE. Come ricordato nel Capitolo 2, Cloud9 non è più disponibile per i nuovi clienti dal 2024, ma la domanda resta valida per l'esame.

### Area 2 – Sicurezza: IAM e Amazon Cognito

**Domanda 18.** Hai distribuito un'applicazione su un'istanza Amazon EC2 nell'account A e hai associato all'istanza un ruolo, Role A. L'applicazione deve accedere a una tabella DynamoDB in un altro account, l'account B. Qual è il modo più semplice di gestire la situazione?

- A. Dare a Role A la possibilità di assumere un ruolo nell'account B che consente l'accesso alla tabella DynamoDB.
- B. Aggiungere alla tabella DynamoDB una resource policy che concede l'accesso a Role A.
- C. Creare nell'account B un ruolo con accesso alla tabella e associarlo all'istanza EC2.
- D. Aggiungere a Role A i permessi necessari per accedere alla tabella DynamoDB nell'account B.

**Risposta: A.** È lo schema classico dell'**accesso cross-account tramite ruoli** (Capitolo 3). Nell'account B si crea un ruolo con i permessi sulla tabella e una **trust relationship** verso Role A. Role A riceve il permesso `sts:AssumeRole` su quel ruolo. L'applicazione assume il ruolo con STS e usa le credenziali temporanee ottenute. L'opzione C non è possibile, perché un'istanza può avere solo un instance profile del **proprio** account. L'opzione D da sola non basta, perché i permessi di un'identità non valgono sulle risorse di un altro account se quell'account non li concede. L'opzione B è il modo in cui la domanda era pensata come distrattore. Va però notato che dal 2024 DynamoDB supporta le resource-based policy, quindi oggi B è tecnicamente realizzabile. Lo schema raccomandato e atteso dall'esame resta comunque l'assunzione di un ruolo.

**Domanda 2.** Ti è stato chiesto di configurare un Cognito User Pool. È essenziale che i token del user pool non vengano mai esposti direttamente all'utente finale. Quale tipo di grant OAuth 2.0 dovresti usare?

- A. Pseudo Admin grant
- B. Authorization code grant
- C. Implicit grant
- D. Client credentials grant

**Risposta: B.** Con l'**authorization code grant**, dopo l'autenticazione l'utente riceve un **codice di autorizzazione**, non i token. L'applicazione scambia il codice con i token in modo sicuro (Capitolo 11). L'implicit grant restituisce i token direttamente nell'URL del browser, cioè esattamente ciò che la domanda vuole evitare. Il client credentials grant è per la comunicazione machine-to-machine, senza utente. Il "Pseudo Admin grant" non esiste.

**Domanda 12.** Devi scrivere un Lambda authorizer per Amazon API Gateway. L'authorizer riceverà l'identità del chiamante in un bearer token. Quale tipo di Lambda authorizer dovresti creare?

- A. REQUEST authorizer
- B. Lambda authorizer basato sui parametri della richiesta
- C. I bearer token sono gestiti da un Cognito authorizer
- D. Lambda authorizer basato su token

**Risposta: D.** Un **TOKEN authorizer** (*token-based*) riceve l'identità del chiamante in un **bearer token**, come un JWT o un token OAuth (Capitolo 11). A e B sono lo stesso tipo, il REQUEST authorizer, che riceve l'identità da una combinazione di intestazioni, query string, stage variable e variabili di contesto. C è un distrattore: un Cognito authorizer non è un Lambda authorizer e accetta solo token emessi da Cognito.

**Domanda 13.** Devi consentire a tutti gli utenti autenticati tramite Twitter di inviare un messaggio a Kinesis. Quale dei seguenti scenari lo permette?

- A. Inviare il token OpenID a un Cognito Identity Pool, poi inviare il token JWT ricevuto dall'Identity Pool a un Cognito User Pool per assumere un ruolo IAM con la policy corretta.
- B. Inviare il token OpenID a STS per assumere un utente IAM associato alla policy corretta.
- C. Inviare il token OpenID a un Cognito User Pool, poi inviare il token JWT ricevuto dal User Pool a STS per assumere un ruolo IAM con la policy corretta.
- D. Inviare il token OpenID a un Cognito User Pool, poi inviare il token JWT ricevuto dal User Pool a un Cognito Identity Pool per assumere un ruolo IAM con la policy corretta.

**Risposta: D.** L'ordine corretto è **User Pool, poi Identity Pool, poi ruolo IAM** (Capitolo 11). Il user pool autentica l'utente federato e restituisce i propri JWT. L'identity pool scambia quel token con credenziali AWS temporanee assumendo il ruolo associato agli utenti autenticati, che contiene i permessi `kinesis:PutRecord`. A inverte l'ordine. B è impossibile, perché gli **utenti IAM non si assumono**: si assumono solo i ruoli. C salta l'identity pool, che è proprio il componente che rilascia le credenziali AWS.

**Domanda 20.** Stai progettando un'applicazione che permetterà agli utenti di autenticarsi con un identity provider esterno. È essenziale usare credenziali di sicurezza temporanee per accedere alle risorse di backend dell'applicazione in AWS. Quali risorse supportano la soluzione? *(Seleziona DUE.)*

- A. Creare un ruolo IAM per gli utenti autenticati.
- B. Creare un gruppo IAM per le identità autenticate.
- C. Creare un Amazon Cognito Identity Pool.
- D. Creare un Amazon Cognito User Pool.
- E. Creare un gruppo IAM per gli utenti autenticati.

**Risposta: A e C.** L'**identity pool** federa le identità esterne e rilascia **credenziali temporanee**. Per farlo assume un **ruolo IAM**, configurato per gli utenti autenticati (Capitolo 11). I gruppi IAM (B ed E) non possono essere assunti e non producono credenziali temporanee. Il user pool (D) autentica gli utenti e rilascia token JWT, ma non credenziali AWS. Può far parte di un'architettura più ampia, ma non è ciò che fornisce le credenziali temporanee richieste.

**Domanda 15.** Quali delle seguenti sono sezioni di un JSON Web Token? *(Seleziona TRE.)*

- A. Header
- B. Public key
- C. Payload
- D. Footer
- E. Signature
- F. Handshake

**Risposta: A, C, E.** Un JWT ha tre sezioni codificate in Base64URL e separate da punti: **header** (algoritmo e key id), **payload** (i claim) e **signature** (Capitolo 11). La chiave pubblica serve a verificare la firma ma non fa parte del token. "Footer" e "handshake" sono distrattori.

### Area 3 – Storage: Amazon S3

**Domanda 7.** Che cosa serve per rendere un intero bucket disponibile a tutto il mondo? *(Seleziona DUE.)*

- A. Impostare l'ACL del bucket su public read.
- B. Usare chiavi gestite da Amazon S3 per la crittografia lato server a livello di bucket.
- C. Disattivare "Block all public access" a livello di account e di bucket.
- D. Creare una configurazione CORS con AllowedOrigin "*" e AllowedMethod "GET".
- E. Creare una bucket policy con effect "Allow", action ["s3:GetObject"], Principal "*" e come Resource l'ARN del bucket.

**Risposta: C ed E.** Per rendere pubblico un bucket servono due cose (Capitoli 4 e 5). La prima è **disattivare il Block Public Access**, che altrimenti prevale su qualunque policy. La seconda è una **bucket policy** che consenta `s3:GetObject` a chiunque. C'è una precisazione importante: `s3:GetObject` è un'azione **sugli oggetti**, quindi la Resource corretta è `arn:aws:s3:::nome-bucket/*`, non l'ARN del solo bucket. L'ACL pubblica (A) è un meccanismo superato, e sui bucket recenti le ACL sono disattivate per default. La crittografia (B) non riguarda l'accesso. CORS (D) regola le richieste del browser da altri domini, non i permessi.

**Domanda 14.** Qual è il modo corretto di accedere all'oggetto `Preview.mp4` nel bucket S3 `awstc`, creato il 1° ottobre 2022 nella Region di Milano (eu-south-1)?

- A. `https://awstc.s3.amazonaws.com/Preview.mp4`
- B. `https://s3.awstc.amazonaws.com/Preview.mp4`
- C. `https://awstc.s3.eu-south-1.amazonaws.com/Preview.mp4`
- D. `https://s3.eu-south-1.amazonaws.com/awstc/Preview.mp4`

**Risposta: C.** Il formato è quello **virtual-hosted-style**: `https://<bucket>.s3.<region>.amazonaws.com/<key>` (Capitolo 4). La data di creazione non è un dettaglio casuale. Il formato **path-style** (D) non è supportato per i bucket creati dopo il 30 settembre 2020. A omette la Region, che per le Region attivate dopo il 2019, come Milano, è necessaria. B non è un formato valido.

**Domanda 9.** Vuoi condividere immagini tra istanze EC2 Linux in Availability Zone diverse, e l'applicazione può leggere file solo dallo storage locale. Quale opzione di storage AWS offre la soluzione più semplice da gestire?

- A. L'applicazione legge e scrive sull'instance storage di EC2 e i dati vengono replicati con uno strumento di replica.
- B. L'applicazione legge e scrive su Amazon EBS e i dati vengono replicati con uno strumento di replica.
- C. Si monta un bucket Amazon S3 sull'istanza tramite NFS e l'applicazione legge e scrive sul mount point.
- D. *(Opzione corretta assente nel materiale originale)* Si monta un file system **Amazon EFS** su tutte le istanze e l'applicazione legge e scrive sul mount point.

**Risposta: D, Amazon EFS.** Il materiale del corso riporta questa domanda in forma corrotta: un'opzione è duplicata e la soluzione indicata è la replica dell'instance storage. La risposta corretta in questa classica domanda d'esame è **Amazon EFS** (Capitolo 4). EFS è un file system **NFS** gestito, montabile contemporaneamente da molte istanze Linux in **più Availability Zone**, e l'applicazione lo vede come un normale percorso locale. Le soluzioni con replica (A e B) richiedono di gestire uno strumento di sincronizzazione e sono tutt'altro che "le più semplici". Inoltre l'instance storage è effimero e i volumi EBS sono legati a una singola AZ. S3 (C) non si monta nativamente via NFS, perché è uno storage a oggetti, non un file system.

### Area 4 – Database: Amazon DynamoDB

**Domanda 17.** Una DynamoDB Global Table è configurata tra tabelle nelle Region di N. Virginia e Sydney. ClientA, in N. Virginia, scrive sulla tabella di N. Virginia, mentre ClientB, a Sydney, scrive sulla tabella di Sydney. Entrambi aggiornano un item con la stessa primary key, e ClientA invia l'aggiornamento appena prima di ClientB. Quale affermazione descrive meglio ciò che accade?

- A. ClientA riceve un errore.
- B. Entrambi gli aggiornamenti vanno a buon fine e alla fine in entrambe le tabelle ci sarà l'aggiornamento di ClientA.
- C. ClientB riceve un errore.
- D. Entrambi gli aggiornamenti vanno a buon fine e alla fine in entrambe le tabelle ci sarà l'aggiornamento di ClientB.

**Risposta: D.** Le Global Tables sono **multi-active**: ogni replica accetta scritture locali senza errori. I conflitti sulla stessa chiave si risolvono con la regola **last writer wins**: prevale la scrittura con il timestamp più recente. ClientB ha scritto per ultimo, quindi, una volta completata la replica, entrambe le tabelle conterranno il suo aggiornamento. Il materiale originale si limitava a dire che "entrambi gli aggiornamenti vanno a buon fine", senza indicare quale prevale.

### Area 5 – Logica applicativa e API: Lambda e API Gateway

**Domanda 5.** Hai configurato Amazon API Gateway come event source della tua funzione Lambda. Per default, quanti retry tenterà Lambda per invocare la funzione?

- A. 1
- B. 2
- C. 4
- D. 3

**Risposta attesa dal materiale: B (2), con un'importante precisazione.** I **due retry** automatici valgono per le **invocazioni asincrone**, come quelle da S3, SNS ed EventBridge (Capitolo 8). API Gateway invoca invece Lambda in modo **sincrono**, e in quel caso **Lambda non esegue alcun retry**: l'errore torna ad API Gateway e quindi al client, che decide se riprovare. La domanda, così come è formulata, è quindi imprecisa. All'esame va letta con attenzione: se si parla di invocazione **asincrona**, la risposta è 2. Se si parla di invocazione **sincrona**, Lambda non ritenta.

**Domanda 16.** Stai testando la tua funzione Lambda con il comando `aws lambda invoke` della CLI. Una volta testata, la funzione verrà distribuita e invocata **in modo asincrono**. Quale comando dovresti usare per testarla?

- A. `aws lambda invoke --function-name my-function --invocation-type RequestResponse --payload '{"name": "test"}'`
- B. `aws lambda invoke --function-name my-function --invocation-type DryRun --payload '{"name": "test"}'`
- C. `aws lambda invoke --function-name my-function --invocation-type Event --payload '{"name": "test"}'`
- D. `aws lambda invoke --function-name my-function --payload '{"name": "test"}'`

**Risposta: C.** Il parametro `--invocation-type` stabilisce il modello di invocazione (Capitolo 8). **`Event`** corrisponde all'invocazione **asincrona**, quella che la funzione avrà in produzione. **`RequestResponse`** (A) è sincrona ed è anche il **valore predefinito**, quindi D, che non specifica il tipo, equivale ad A. **`DryRun`** (B) verifica solo che il chiamante abbia i permessi, senza eseguire la funzione. Il materiale originale si limitava a descrivere il parametro senza indicare l'opzione corretta.

**Domanda 4.** Devi scrivere un mapping template per interrogare una tabella Amazon DynamoDB. Il template deve leggere il valore del parametro della method request dal **path** e passarlo a DynamoDB. Quale espressione dovresti includere nel mapping template?

- A. `$input.json(x)`
- B. `$input.params(x)`
- C. `$input.body`
- D. `$input.path(x)`

**Risposta: B.** **`$input.params('x')`** restituisce il valore del **parametro della richiesta** `x`, cercandolo nel path, nella query string o negli header (Capitolo 9). È quindi l'espressione per leggere un parametro di path come `{userId}`. `$input.path(x)` (D) è il distrattore più insidioso: nonostante il nome, **non** legge il path dell'URL, ma estrae un valore dal **corpo** della richiesta tramite un'espressione JSONPath. `$input.json(x)` (A) restituisce una porzione del corpo come stringa JSON. `$input.body` (C) restituisce l'intero corpo. Il materiale originale non indicava l'opzione corretta.

**Domanda 8.** Ti è stato chiesto di progettare un'applicazione di chat serverless. La soluzione richiede una comunicazione bidirezionale tra client e servizio. Quale tipo di API dovresti distribuire?

- A. Edge-optimized API
- B. HTTP API
- C. WebSocket API
- D. REST API

**Risposta: C.** Le **WebSocket API** di API Gateway mantengono una connessione **bidirezionale persistente**: il client invia messaggi al servizio, e il servizio può inviarne al client in modo indipendente (Capitolo 9). REST e HTTP API seguono il modello richiesta/risposta, in cui è sempre il client a iniziare. "Edge-optimized" è un **tipo di endpoint**, non un tipo di API.

**Domanda 10.** Dopo una revisione del tuo microservizio, hai stabilito che l'API Gateway deve essere ottimizzato per migliorare la reattività. L'obiettivo è ridurre il numero di chiamate all'endpoint e migliorare la latenza delle richieste. Come puoi ottenerlo?

- A. Abilitare l'API caching su API Gateway.
- B. Distribuire l'API Gateway come gateway Regional.
- C. Abilitare l'API caching su API Gateway per le richieste GET e PUT.
- D. Distribuire l'API Gateway come gateway Edge-optimized.

**Risposta: A.** La **cache delle risposte** di API Gateway, configurata per stage con un TTL, serve le richieste ripetute senza raggiungere il backend. Riduce quindi le chiamate all'endpoint e migliora la latenza (Capitolo 9). C è sbagliata perché la cache riguarda le letture: per default API Gateway mette in cache solo le risposte dei metodi **GET**, e mettere in cache una PUT non avrebbe senso. Cambiare tipo di endpoint (B e D) può influire sulla latenza di rete, ma **non riduce** il numero di chiamate al backend.

### Area 6 – Applicazioni moderne e deploy: Step Functions e SAM

**Domanda 19.** Devi chiamare **contemporaneamente** le API in esecuzione su due istanze Amazon EC2. Come configureresti la state machine?

- A. Usare uno stato Parallel per avviare due Task simultanei.
- B. Usare uno stato Parallel per avviare due Activity simultanee.
- C. Usare uno stato Choice per avviare due Activity simultanee.
- D. Usare uno stato Choice per avviare due Task simultanei.

**Risposta: B.** Per eseguire due rami contemporaneamente serve uno stato **Parallel**. Lo stato **Choice** sceglie **un solo** ramo in base a una condizione, quindi C e D sono escluse. Tra A e B, la risposta attesa è **Activity**: le Activity sono il meccanismo con cui Step Functions affida il lavoro a un **worker esterno**, come il codice in esecuzione sulle istanze EC2, che interroga il servizio, esegue il compito e ne restituisce il risultato (Capitolo 10). Tecnicamente anche le Activity si dichiarano come stati di tipo Task che puntano all'ARN di un'Activity, ma nel linguaggio dell'esame "Activity" indica proprio il lavoro svolto da worker esterni come i server EC2.

**Domanda 6.** Pensi di usare SAM per distribuire un microservizio. Quali delle seguenti risorse AWS SAM potrebbero essere usate per costruirlo? *(Seleziona TRE.)*

- A. `AWS::Serverless::DynamodbTable`
- B. `AWS::Serverless::Api`
- C. `AWS::Serverless::Redshift`
- D. `AWS::Serverless::Function`
- E. `AWS::Serverless::Lambda`
- F. `AWS::Serverless::SimpleTable`

**Risposta: B, D, F.** I tipi di risorsa SAM corrispondono ai servizi in questo modo (Capitolo 12):

- `AWS::Serverless::Api` crea un'API in **Amazon API Gateway**;
- `AWS::Serverless::Function` crea una funzione **AWS Lambda**;
- `AWS::Serverless::SimpleTable` crea una tabella **Amazon DynamoDB** con chiave primaria semplice.

A, C ed E sono nomi inventati. In particolare la funzione si chiama `Function`, non `Lambda`, e la tabella `SimpleTable`, non `DynamodbTable`. Altri tipi SAM reali sono `HttpApi`, `LayerVersion`, `StateMachine` e `Application`.

### Area 7 – Osservabilità: Amazon CloudWatch

**Domanda 3.** Un cliente vuole usare CloudWatch per monitorare la propria applicazione e analizzare continuamente i dati alla ricerca di segnali di problemi. Quale opzione raccomanda e configura metriche e log per l'applicazione?

- A. CloudWatch Alarms
- B. Log Groups
- C. Amazon CloudWatch ServiceLens
- D. CloudWatch Application Insights

**Risposta: D.** **CloudWatch Application Insights** analizza le risorse dell'applicazione, **raccomanda e configura** automaticamente le metriche, i log e gli allarmi più adatti al tipo di applicazione, e poi analizza i dati in modo continuo per individuare i problemi (Capitolo 13). Gli allarmi (A) sorvegliano una metrica, ma vanno configurati a mano. I log group (B) sono semplici contenitori di log. ServiceLens (C) integra trace, metriche e log in un'unica vista, ma non raccomanda né configura il monitoraggio.

## 14.3 Mappa delle domande verso i capitoli di riferimento

| Domanda | Argomento | Risposta | Capitolo |
|---|---|---|---|
| 1 | Automazione con CLI e SDK | B, C | 2 |
| 2 | Grant OAuth 2.0 in Cognito | B | 11 |
| 3 | CloudWatch Application Insights | D | 13 |
| 4 | Mapping template VTL (`$input.params`) | B | 9 |
| 5 | Retry di Lambda | B *(solo per le invocazioni asincrone)* | 8 |
| 6 | Risorse SAM | B, D, F | 12 |
| 7 | Bucket S3 pubblico | C, E | 4, 5 |
| 8 | WebSocket API | C | 9 |
| 9 | Storage condiviso tra AZ | Amazon EFS | 4 |
| 10 | Caching di API Gateway | A | 9 |
| 11 | IDE nel cloud | C | 2 |
| 12 | Lambda authorizer TOKEN | D | 11 |
| 13 | Federazione: User Pool → Identity Pool | D | 11 |
| 14 | URL di un oggetto S3 | C | 4 |
| 15 | Struttura del JWT | A, C, E | 11 |
| 16 | `--invocation-type Event` | C | 8 |
| 17 | Global Tables, last writer wins | D | 6, 7 |
| 18 | Accesso cross-account con ruoli | A | 3 |
| 19 | Step Functions: Parallel e Activity | B | 10 |
| 20 | Credenziali temporanee per utenti federati | A, C | 11 |

> **Box – Come leggere le domande d'esame**
>
> Molti errori in queste domande non nascono da lacune di conoscenza ma da una lettura frettolosa. Alcune abitudini aiutano.
>
> - **Cercare le parole chiave**: "asincrono" (domanda 16), "mai esposti all'utente" (2), "più semplice da gestire" (9), "contemporaneamente" (19), "bearer token" (12). Spesso una sola parola determina la risposta.
> - **Diffidare dei nomi plausibili ma inesistenti**, come `AWS::Serverless::Lambda` o "Pseudo Admin grant".
> - **Distinguere i concetti simili**: invocazione e permessi di esecuzione, user pool e identity pool, `$input.path` e `$input.params`, tipo di API e tipo di endpoint, annotation e metadata.
> - **Ricordare le regole di default**: tutto è privato in S3, tutto è negato in IAM, le scritture DynamoDB non sono condizionali e le notifiche S3 non sono attive.
>
> Ogni volta che si sbaglia una domanda, conviene tornare al capitolo indicato nella tabella e rileggere il paragrafo corrispondente, non solo la soluzione.
