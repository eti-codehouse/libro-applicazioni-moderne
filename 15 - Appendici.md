# Appendici

## Appendice A – Glossario dei termini AWS

Per ogni termine è indicato, tra parentesi, il capitolo in cui è trattato in modo approfondito.

**Access key** – Coppia di credenziali statiche (*access key ID* e *secret access key*) associata a un utente IAM, usata per firmare le chiamate programmatiche tramite CLI e SDK (3).

**Access point (S3)** – Punto di accesso con nome, ARN, dominio e policy propri, creato per un bucket. Semplifica la gestione dei permessi quando molti principal usano lo stesso bucket (4).

**ACL (Access Control List)** – Meccanismo di permessi di S3 a livello di bucket e di oggetto, precedente alle bucket policy. Oggi è consigliato disattivarlo (4).

**Activity (Step Functions)** – Task eseguito da un worker esterno, per esempio su EC2, che interroga Step Functions, esegue il lavoro e ne restituisce il risultato (10).

**Adaptive capacity** – Funzionalità di DynamoDB che ridistribuisce automaticamente la capacità verso le partizioni più sollecitate (6).

**Alias (Lambda)** – Puntatore con nome a una versione di una funzione Lambda, usato per gestire ambienti come *dev* e *prod* e per i deploy graduali (8, 12).

**Annotation (X-Ray)** – Coppia chiave-valore indicizzata, aggiunta a un segment o subsegment, utilizzabile per filtrare le trace (13).

**API Gateway** – Servizio per creare, pubblicare, mantenere, monitorare e proteggere API REST, HTTP e WebSocket (9).

**App client (Cognito)** – Configurazione che rappresenta un'applicazione presso un user pool e ne definisce i flussi di autenticazione consentiti (11).

**Application Insights (CloudWatch)** – Funzionalità che raccomanda e configura automaticamente metriche, log e allarmi per un'applicazione e ne rileva i problemi (13).

**ARN (Amazon Resource Name)** – Identificativo univoco di una risorsa AWS, nella forma `arn:aws:<servizio>:<region>:<account>:<risorsa>` (3, 4).

**ASL (Amazon States Language)** – Linguaggio JSON con cui si definiscono le state machine di Step Functions (10).

**Authorizer** – Componente di API Gateway che verifica l'identità del chiamante. Può essere di tipo IAM, Cognito, JWT o Lambda (9, 11).

**Authorization code grant** – Flusso OAuth 2.0 in cui l'utente riceve un codice anziché i token, che l'applicazione scambia poi in modo sicuro. È il flusso raccomandato (11).

**Batch Operations (S3)** – Servizio per eseguire un'operazione, come copia, tagging, restore o invocazione Lambda, su grandi collezioni di oggetti descritte da un manifest (5).

**Blue/green deployment** – Strategia di rilascio in cui la nuova versione viene installata su nuove risorse (*green*) e il traffico viene spostato da quelle vecchie (*blue*). Facilita il rollback (12).

**Bucket** – Contenitore regionale di oggetti S3, con un nome unico a livello globale (4).

**Bucket policy** – Resource-based policy JSON attaccata a un bucket S3 (3, 4).

**Burst capacity** – Capacità di DynamoDB non utilizzata negli ultimi cinque minuti, spendibile per assorbire picchi di traffico (6).

**Canary release / deployment** – Rilascio graduale in cui una piccola percentuale di traffico viene inviata alla nuova versione prima del passaggio completo (9, 12).

**CloudFormation** – Servizio di Infrastructure as Code che crea e aggiorna risorse a partire da template JSON o YAML (12).

**CloudShell** – Shell nel browser, già autenticata con le credenziali della Console, con la AWS CLI preinstallata (2, 7).

**CloudWatch** – Servizio di monitoraggio per metriche, log, allarmi e dashboard (13).

**CloudWatch agent** – Software installato sulle istanze per inviare log e metriche aggiuntive a CloudWatch (13).

**Cold start** – Ritardo della prima invocazione servita da un nuovo ambiente di esecuzione Lambda, dovuto al download del codice e all'inizializzazione (8).

**Concurrency (Lambda)** – Numero di ambienti di esecuzione attivi contemporaneamente per una funzione (8).

**Condition expression** – Condizione che una scrittura DynamoDB deve soddisfare per essere eseguita. È la base dell'optimistic locking (7).

**Continuation token** – Valore restituito da un'API paginata da passare alla richiesta successiva per ottenere la pagina seguente (5).

**CORS (Cross-Origin Resource Sharing)** – Meccanismo con cui un server autorizza pagine di altri domini ad accedere alle proprie risorse (5, 9).

**DAX (DynamoDB Accelerator)** – Cache in memoria compatibile con l'API di DynamoDB, con latenze di microsecondi per le letture eventualmente consistenti (7).

**Dead-letter queue (DLQ)** – Coda in cui finiscono gli eventi che una funzione non è riuscita a elaborare dopo i retry (8).

**Delete marker** – Marcatore creato da una cancellazione semplice in un bucket con versioning. Rimuovendolo, l'oggetto torna visibile (4).

**Destination (Lambda)** – Servizio (SQS, SNS, Lambda, EventBridge) a cui viene inviato l'esito di un'invocazione asincrona (8).

**Dimension (CloudWatch)** – Coppia nome/valore che fa parte dell'identità di una metrica (13).

**DynamoDB** – Database NoSQL chiave-valore e documentale, serverless, con latenze di millisecondi a una cifra (6, 7).

**DynamoDB Local** – Versione di DynamoDB eseguibile in locale, anche in Docker, per sviluppo e test (6).

**DynamoDB Streams** – Flusso ordinato delle modifiche agli item di una tabella, consumabile da Lambda (8).

**Edge-optimized endpoint** – Tipo di endpoint di API Gateway che instrada le richieste attraverso le edge location di CloudFront (9).

**EFS (Elastic File System)** – File system NFS gestito, montabile contemporaneamente da molte istanze in più Availability Zone (4, 14).

**Enhanced Client (DynamoDB)** – Interfaccia di alto livello dell'SDK for Java 2.x che mappa classi annotate sugli item DynamoDB (7).

**ETag** – Identificativo restituito da S3 per un oggetto o per una parte di un caricamento multipart (4, 5).

**Event notification (S3)** – Notifica di un evento su un bucket, come creazione o cancellazione di oggetti, verso Lambda, SQS o SNS (4).

**Event source mapping** – Configurazione con cui Lambda legge da uno stream o da una coda (Kinesis, DynamoDB Streams, SQS, MSK) secondo il modello a polling (8).

**Execution role** – Ruolo IAM che una funzione Lambda assume per eseguire azioni su altri servizi (8).

**Foundation Model (FM)** – Modello di machine learning di grandi dimensioni, pre-addestrato per scopi generali e adattabile con fine-tuning (2).

**GSI (Global Secondary Index)** – Indice DynamoDB con partition key e sort key qualsiasi, creabile in ogni momento, con capacità propria e aggiornamento asincrono (6).

**Handler** – Funzione d'ingresso di una funzione Lambda, invocata a ogni esecuzione con i parametri `event` e `context` (8).

**HeadBucket / HeadObject** – API S3 che restituiscono solo i metadati di un bucket o di un oggetto (5).

**Hosted UI (Cognito)** – Pagine di login, registrazione e recupero password ospitate da AWS (11).

**Hot partition** – Partizione DynamoDB che riceve una quota sproporzionata del traffico, diventando un collo di bottiglia (6, 7).

**IAM (Identity and Access Management)** – Servizio che controlla l'autenticazione e l'autorizzazione per l'accesso alle risorse AWS (3).

**Identity-based policy** – Policy attaccata a un utente, a un gruppo o a un ruolo IAM. Non contiene l'elemento `Principal` (3).

**Identity pool (Cognito)** – Componente che scambia token di identity provider con credenziali AWS temporanee, assumendo ruoli IAM (11).

**Implicit deny** – Regola di IAM per cui tutto ciò che non è esplicitamente consentito è negato (3).

**Infrastructure as Code (IaC)** – Pratica di descrivere l'infrastruttura in file di codice versionati (12).

**Instance profile** – Contenitore che associa un ruolo IAM a un'istanza EC2 (3).

**JMESPath** – Linguaggio di interrogazione per JSON usato dal parametro `--query` della CLI (2).

**JWT (JSON Web Token)** – Token composto da header, payload e signature, usato da Cognito (11).

**Knowledge Base (Bedrock)** – Archivio di dati aziendali usato per arricchire prompt e risposte dei modelli tramite RAG (2).

**Lambda** – Servizio di calcolo serverless che esegue codice in risposta a eventi o invocazioni (8).

**Lambda authorizer** – Funzione Lambda che implementa una logica di autorizzazione personalizzata per API Gateway, di tipo TOKEN o REQUEST (11).

**LastEvaluatedKey** – Chiave restituita da Query e Scan di DynamoDB quando i risultati proseguono nella pagina successiva (7).

**Layer (Lambda)** – Archivio di librerie o file condiviso tra più funzioni Lambda (8).

**Lifecycle rule (S3)** – Regola che sposta, archivia o elimina automaticamente gli oggetti, o i multipart incompleti, dopo un certo tempo (5).

**LLM (Large Language Model)** – Modello di AI generativa specializzato nel linguaggio naturale (2).

**LSI (Local Secondary Index)** – Indice DynamoDB con la stessa partition key della tabella e una sort key diversa. Si crea solo insieme alla tabella (6).

**Mapping template** – Script VTL che trasforma richieste e risposte in API Gateway (9).

**Metadata (X-Ray)** – Coppia chiave-valore non indicizzata, aggiunta a un segment per fornire contesto (13).

**Metric filter** – Filtro che trasforma le occorrenze di un pattern nei log di CloudWatch in una metrica (13).

**Mock integration** – Integrazione di API Gateway che restituisce risposte simulate senza chiamare un backend (9).

**Model (API Gateway)** – JSON Schema che descrive la forma di una richiesta o di una risposta e ne permette la validazione (9).

**Multipart upload** – Caricamento di un oggetto S3 in più parti, consigliato oltre i 100 MB e obbligatorio oltre i 5 GB (4, 5).

**Named profile** – Insieme di credenziali e impostazioni con un nome, nei file `credentials` e `config` della CLI (3).

**On-demand (capacity mode)** – Modalità di DynamoDB in cui si paga per richiesta senza provisionare capacità. È adatta al traffico imprevedibile (7).

**OpenAPI** – Standard per descrivere le API in YAML o JSON, supportato da API Gateway per import ed export (9).

**Optimistic locking** – Tecnica che usa un attributo di versione e una condition expression per evitare sovrascritture concorrenti (7).

**Paginator** – Componente dell'SDK che esegue automaticamente le chiamate successive di un'API paginata (5).

**PartiQL** – Linguaggio in stile SQL per interrogare DynamoDB (6).

**Partition key** – Attributo della chiave primaria DynamoDB che determina la partizione in cui è memorizzato un item (6).

**Permissions boundary** – Limite massimo di permessi che un principal IAM può avere, indipendentemente dalle sue policy (3).

**Presigned URL** – URL firmato che concede un accesso temporaneo a un oggetto S3 privato, per GET o PUT (5).

**Principal** – Entità che effettua una richiesta ad AWS: un utente, un ruolo, un servizio o un utente federato (3).

**Provisioned (capacity mode)** – Modalità di DynamoDB in cui si definiscono RCU e WCU. È adatta al traffico prevedibile (6, 7).

**Provisioned concurrency** – Numero di ambienti Lambda mantenuti pre-inizializzati per eliminare i cold start (8).

**Proxy integration** – Integrazione di API Gateway che passa la richiesta invariata al backend (9).

**RAG (Retrieval Augmented Generation)** – Tecnica che arricchisce il prompt con dati recuperati da fonti aziendali per migliorare l'accuratezza delle risposte (2).

**RCU (Read Capacity Unit)** – Una lettura fortemente consistente al secondo, o due eventualmente consistenti, di un item fino a 4 KB (6).

**Refresh token** – Token Cognito a lunga durata usato per ottenere nuovi ID e access token (11).

**Resource-based policy** – Policy attaccata a una risorsa, come una bucket policy o una resource policy Lambda. Contiene l'elemento `Principal` (3).

**ResultPath** – Campo ASL che stabilisce dove inserire il risultato di uno stato nel documento in transito (10).

**Role (IAM)** – Identità senza credenziali statiche che si assume per ottenere credenziali temporanee (3).

**S3 (Simple Storage Service)** – Servizio di storage a oggetti con durabilità di 11 nove (4, 5).

**S3 Select** – Funzionalità per interrogare con SQL il contenuto di un singolo oggetto S3 (5).

**SAM (Serverless Application Model)** – Framework open source per definire, testare in locale e distribuire applicazioni serverless (12).

**Scan** – Operazione DynamoDB che legge tutti gli item di una tabella o di un indice (6, 7).

**Segment / Subsegment (X-Ray)** – Unità di una trace che descrive il lavoro di un servizio (segment) e i suoi dettagli e le chiamate a valle (subsegment) (13).

**Service map (X-Ray)** – Mappa visiva dei componenti di un'applicazione con il loro stato di salute e la latenza (13).

**Service role** – Ruolo IAM che un servizio AWS assume per agire su altri servizi (3).

**SigV4 (Signature Version 4)** – Protocollo di firma delle richieste AWS che autentica il chiamante e protegge dagli attacchi di replay (2, 3).

**Sort key** – Attributo facoltativo della chiave primaria DynamoDB che ordina gli item all'interno di una partizione (6).

**Stage (API Gateway)** – Istanza distribuita di un'API, come *dev* o *prod*, con un proprio URL e una propria configurazione (9).

**Stage variable** – Variabile nome-valore associata a uno stage di API Gateway (9).

**State machine** – Workflow di Step Functions composto da stati e transizioni (10).

**Step Functions** – Servizio di orchestrazione di workflow serverless (10).

**Strangler pattern** – Modello di migrazione che sostituisce gradualmente le funzionalità di un monolite con microservizi (10).

**STS (Security Token Service)** – Servizio che rilascia credenziali temporanee, per esempio tramite `AssumeRole` (3).

**Throttling** – Limitazione delle richieste oltre una soglia, da parte di API Gateway, Lambda o DynamoDB (8, 9).

**Trace (X-Ray)** – Registrazione completa del percorso di una richiesta attraverso l'applicazione (13).

**TTL (Time to Live)** – Durata di validità di un dato in cache (9).

**User pool (Cognito)** – Directory di utenti che gestisce registrazione, login e rilascio dei token JWT (11).

**Versioning (S3)** – Conservazione della cronologia delle versioni di ogni oggetto di un bucket (4).

**VTL (Velocity Template Language)** – Linguaggio dei mapping template di API Gateway (9).

**Waiter** – Funzionalità di SDK e CLI che attende che una risorsa raggiunga uno stato desiderato, ripetendo automaticamente le verifiche (2, 5, 7).

**WCU (Write Capacity Unit)** – Una scrittura al secondo di un item fino a 1 KB (6).

**WebSocket API** – API di API Gateway con connessione bidirezionale persistente (9).

**X-Ray** – Servizio di tracing distribuito (13).

---

## Appendice B – Comandi CLI e classi SDK citati nel libro

### B.1 Comandi AWS CLI

**Configurazione e identità**

| Comando | Scopo | Capitolo |
|---|---|---|
| `aws configure [--profile nome]` | Configurare credenziali, Region e formato di output | 3 |
| `aws sts get-caller-identity` | Verificare quale identità sta usando la CLI | 3 |
| `aws <comando> help` | Consultare l'aiuto integrato | 2 |
| `--query "..."` | Filtrare l'output con JMESPath | 2 |
| `--generate-cli-skeleton` / `--cli-input-json file://...` | Generare e usare un file di input JSON | 7 |

**Amazon S3**

| Comando | Scopo | Capitolo |
|---|---|---|
| `aws s3 mb s3://bucket` | Creare un bucket | 2 |
| `aws s3 cp file s3://bucket/key` | Copiare file, con multipart automatico | 2, 4, 5 |
| `aws s3 ls s3://bucket` | Elencare il contenuto | 2 |
| `aws s3 sync src dst [--exclude ...]` | Sincronizzare cartelle e bucket | 4, 5 |
| `aws s3 website s3://bucket --index-document ... --error-document ...` | Configurare l'hosting statico | 5 |
| `aws s3api head-object --bucket ... --key ...` | Leggere i metadati di un oggetto | 5 |
| `aws s3api create-multipart-upload` / `upload-part` / `list-parts` / `complete-multipart-upload` | Gestire manualmente un multipart upload | 5 |

**Amazon DynamoDB**

| Comando | Scopo | Capitolo |
|---|---|---|
| `aws dynamodb describe-table --table-name ...` | Descrivere una tabella e il suo stato | 2 |
| `aws dynamodb wait table-exists --table-name ...` | Attendere la creazione di una tabella | 2 |
| `aws dynamodb put-item --table-name ... --item ...` | Inserire o sostituire un item | 7 |
| `aws dynamodb batch-write-item --request-items ...` | Inserire o cancellare fino a 25 item | 7 |
| `aws dynamodb get-item --table-name ... --key ...` | Leggere un item | 7 |
| `aws dynamodb query --key-condition-expression ... --expression-attribute-values ...` | Interrogare per chiave | 7 |
| `aws dynamodb scan --filter-expression ...` | Scansionare la tabella | 7 |
| `aws dynamodb update-item --update-expression ... [--condition-expression ...]` | Aggiornare un item, anche in modo condizionale | 7 |
| `aws dynamodb delete-item --table-name ... --key ...` | Cancellare un item | 7 |

**AWS Lambda e Amazon CloudWatch**

| Comando | Scopo | Capitolo |
|---|---|---|
| `aws lambda create-function --function-name ... --runtime ... --zip-file fileb://... --handler ... --role ...` | Creare una funzione | 8 |
| `aws lambda invoke --function-name ... [--invocation-type Event\|RequestResponse\|DryRun] [--payload ...] out.json` | Invocare una funzione | 8 |
| `aws lambda update-function-configuration --function-name ... --environment ...` | Modificare la configurazione | 8 |
| `aws logs tail /aws/lambda/nome --follow` | Seguire i log in tempo reale | 8 |
| `aws cloudwatch put-metric-data` | Pubblicare una metrica personalizzata | 13 |
| `aws cloudwatch put-metric-alarm` | Creare un allarme | 13 |

**Amazon Cognito**

| Comando | Scopo | Capitolo |
|---|---|---|
| `aws cognito-idp sign-up` | Registrare un utente | 11 |
| `aws cognito-idp admin-confirm-sign-up` | Confermare un utente | 11 |
| `aws cognito-idp admin-initiate-auth --auth-flow ADMIN_USER_PASSWORD_AUTH` | Autenticarsi e ottenere i token | 11 |

**AWS SAM CLI**

| Comando | Scopo | Capitolo |
|---|---|---|
| `sam init` | Inizializzare un progetto | 12 |
| `sam build [--use-container]` | Compilare l'applicazione | 12 |
| `sam local invoke [--event file]` | Eseguire una funzione in locale | 12 |
| `sam local start-api` | Avviare un'API locale (porta 3000) | 12 |
| `sam local generate-event <servizio> <evento>` | Generare eventi di esempio | 12 |
| `sam deploy [--guided]` | Distribuire con CloudFormation | 12 |

### B.2 Classi e metodi dell'SDK for Java 2.x

| Classe o metodo | Scopo | Capitolo |
|---|---|---|
| `S3Client.builder()...build()` | Client S3 sincrono | 4, 5 |
| `S3AsyncClient` | Client S3 asincrono, necessario per `selectObjectContent` | 5 |
| `ProfileCredentialsProvider.create()` | Credenziali dal profilo locale | 4, 5 |
| `CreateBucketRequest`, `HeadBucketRequest` | Creare e verificare un bucket | 4, 5 |
| `S3Waiter.waitUntilBucketExists()` | Attendere la creazione di un bucket | 5 |
| `HeadObjectRequest`, `GetObjectRequest`, `getObjectAsBytes()` | Leggere metadati e contenuto | 5 |
| `SelectObjectContentRequest`, `SelectObjectContentResponseHandler` | S3 Select | 5 |
| `S3Presigner`, `GetObjectPresignRequest`, `presignGetObject()` | Generare presigned URL | 5 |
| `listBuckets()`, `listObjectsV2Paginator()` | Elencare bucket e oggetti con paginazione | 5 |
| `DynamoDbClient` | Client DynamoDB | 2, 7 |
| `DynamoDbWaiter.waitUntilTableExists()` | Attendere la creazione di una tabella | 2, 7 |
| `CreateTableRequest`, `AttributeDefinition`, `KeySchemaElement`, `ProvisionedThroughput` | Creare una tabella | 7 |
| `UpdateTableRequest`, `DescribeTableRequest`, `DeleteTableRequest`, `ListTablesRequest` | Gestire le tabelle | 7 |
| `DynamoDbEnhancedClient`, `TableSchema.fromBean()`, `DynamoDbTable<T>` | Interfaccia a oggetti | 7 |
| `@DynamoDbBean`, `@DynamoDbPartitionKey`, `@DynamoDbSortKey`, `@DynamoDbAttribute` | Annotazioni di mapping | 7 |
| `Key`, `QueryConditional`, `Expression`, `QueryEnhancedRequest`, `PageIterable` | Query con l'Enhanced Client | 7 |
| `RequestHandler<I, O>`, `Context` | Handler di una funzione Lambda | 8, 12 |
| `TracingInterceptor` | Strumentazione X-Ray di un client SDK | 13 |

### B.3 Altri SDK e linguaggi

| Elemento | Linguaggio | Scopo | Capitolo |
|---|---|---|---|
| `boto3.client(...)`, `boto3.resource(...)` | Python | Client e risorse AWS | 2, 6 |
| `Key(...)`, `Attr(...)` | Python (boto3) | Condizioni di query e filtri DynamoDB | 6 |
| `select_object_content()` | Python (boto3) | S3 Select | 5 |
| `Moto` | Python | Mock dei servizi AWS negli unit test | 8 |
| `xray_recorder`, `patch_all()` | Python | Strumentazione X-Ray | 13 |
| `AWSSDKHandler.RegisterXRayForAllServices()` | .NET | Tracciare tutte le chiamate AWS | 13 |
| `TraceableSqlCommand` | .NET | Tracciare le query SQL | 13 |
| `$input.path`, `$input.params`, `$input.json`, `$input.body`, `$context`, `$stageVariables` | VTL | Mapping template di API Gateway | 9 |

---

## Appendice C – Note di aggiornamento

Il corso da cui nasce il libro è stato registrato in un momento preciso, e diversi servizi e limiti sono cambiati da allora. La tabella riassume le differenze più rilevanti segnalate nei capitoli. AWS aggiorna frequentemente servizi e quote, quindi prima di un esame o di una decisione progettuale conviene sempre verificare la documentazione ufficiale.

| Argomento | Nel corso | Situazione aggiornata | Capitolo |
|---|---|---|---|
| **AWS Cloud9** | IDE nel browser raccomandato | Dal luglio 2024 non è più disponibile per i nuovi clienti. Le alternative sono un IDE locale con AWS Toolkit, CloudShell o ambienti basati su VS Code | 2, 12 |
| **Amazon CodeWhisperer** | Assistente al codice nell'AWS Toolkit | È confluito in **Amazon Q Developer** | 2 |
| **AWS CodeCommit** | Repository Git di riferimento | Nel 2024 l'accesso per i nuovi clienti è stato limitato. Spesso si usano GitHub, GitLab o Bitbucket con **AWS CodeConnections**. Verificare lo stato attuale | 12 |
| **S3 Select** | Strumento consigliato per filtrare i dati lato S3 | Dal luglio 2024 non è più disponibile per i nuovi clienti. Le alternative sono **Amazon Athena**, **S3 Object Lambda** o il filtro lato client | 5 |
| **Disponibilità di S3** | "Cinque nove" | S3 Standard è progettato per il **99,99%** di disponibilità. La durabilità resta di 11 nove | 4 |
| **Path-style URL S3** | Non trattato esplicitamente | Non supportato per i bucket creati dopo il 30 settembre 2020. Usare il formato virtual-hosted | 4, 14 |
| **ACL di S3** | Disattivabili | Per i nuovi bucket sono **disattivate per default** (*Object Ownership: Bucket owner enforced*) | 4 |
| **Resource-based policy di DynamoDB** | Non disponibili | Dal 2024 DynamoDB supporta le resource-based policy su tabelle e stream, utili anche per l'accesso cross-account | 3, 14 |
| **Concorrenza Lambda** | "Da 500 a 3.000 per Region" | È una **quota regionale dell'account**, storicamente di 1.000 per default, aumentabile su richiesta, con regole di scaling aggiornate nel 2023 | 8 |
| **`/tmp` di Lambda** | "Fino a 10 GB" | Da **512 MB** (predefinito) fino a **10 GB**, configurabile | 8 |
| **Payload asincrono Lambda** | 256 KB | Il limite è stato rivisto da AWS. Verificare il valore corrente nella documentazione delle quote Lambda | 8 |
| **Cold start Java** | EventBridge e provisioned concurrency | È disponibile anche **Lambda SnapStart** per Java (e successivamente per altri runtime) | 8 |
| **Dimensioni CloudWatch** | Fino a 10 per metrica | Fino a **30** per metrica | 13 |
| **X-Ray SDK** | Strumento di strumentazione di riferimento | Gli SDK sono in **manutenzione**. AWS raccomanda **OpenTelemetry** tramite AWS Distro for OpenTelemetry (ADOT) e l'agente CloudWatch | 13 |
| **Console di API Gateway** | Interfaccia con il menu *Actions* | La Console è stata ridisegnata: le stesse funzionalità (risorse, metodi, deploy, stage) si trovano in pulsanti e schede diversi | 9 |
| **Console di Lambda** | Editor classico | L'editor è stato aggiornato con un'esperienza basata su VS Code. I concetti di test event, deploy e configurazione restano invariati | 8 |
| **Modelli di Bedrock** | Amazon Titan Text G1 nel playground | Il catalogo di modelli si è molto ampliato, con nuove famiglie Amazon Nova e modelli più recenti di Anthropic, Meta e altri. I concetti (playground, Knowledge Base, RAG, fine-tuning) restano validi | 2 |

> **Box – Come restare aggiornati**
>
> Le fonti più affidabili per verificare le informazioni sono tre:
>
> - la **documentazione ufficiale** di ogni servizio, in particolare le pagine *Quotas* e *What's new*;
> - la pagina **AWS What's New** (aws.amazon.com/new), che annuncia ogni novità;
> - la **guida all'esame** (*Exam Guide*) della certificazione AWS Certified Developer – Associate, che elenca i domini e i servizi effettivamente coperti dalla versione corrente dell'esame.
