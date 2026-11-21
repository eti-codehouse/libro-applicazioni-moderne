# Sviluppare su AWS
## Manuale pratico per lo sviluppatore cloud: dalle API al deploy serverless
### Con percorso di preparazione alla certificazione AWS Certified Developer – Associate

---

## Struttura del libro

Il libro è composto da **14 capitoli**, uno per ogni file della cartella `src`, raggruppati in **7 parti** tematiche. Ogni capitolo contiene un paragrafo per ogni sezione della trascrizione originale. I knowledge check diventano paragrafi di autoverifica commentati.

Un filo conduttore attraversa tutto il volume: l'applicazione di esempio **Polly Notes**. È un'app web serverless per prendere note e ascoltarle in audio, costruita con Cognito, API Gateway, Lambda, DynamoDB, S3 e Polly, e osservata con CloudWatch e X-Ray.

| Parte | Capitoli | File sorgente |
|---|---|---|
| I – Le fondamenta dello sviluppo su AWS | 1, 2, 3 | 0, 1, 2 |
| II – Lo storage a oggetti con Amazon S3 | 4, 5 | 3, 4 |
| III – I dati applicativi con Amazon DynamoDB | 6, 7 | 5, 6 |
| IV – Logica applicativa e API | 8, 9 | 7, 8 |
| V – Architetture moderne e identità degli utenti | 10, 11 | 9, 10 |
| VI – Rilascio e osservabilità | 12, 13 | 11, 12 |
| VII – Verifica finale | 14 | 13 |

---

## Indice completo

### Parte I – Le fondamenta dello sviluppo su AWS

**Capitolo 1 – Sviluppare su AWS: panoramica del percorso** *(src: 0. Developing on AWS)*
- 1.1 Il percorso di formazione e i servizi trattati
- 1.2 L'applicazione di riferimento: Polly Notes

**Capitolo 2 – Primi passi nello sviluppo su AWS** *(src: 1. Getting Started with Development on AWS)*
- 2.1 Obiettivi del capitolo
- 2.2 Accedere ai servizi AWS in modo programmatico
- 2.3 L'AWS Command Line Interface (AWS CLI)
- 2.4 Gli AWS SDK e i pattern di programmazione in Java
- 2.5 Gli ambienti di sviluppo integrati (IDE) e l'AWS Toolkit
- 2.6 Laboratorio: AWS Cloud9 e AWS CLI
- 2.7 L'intelligenza artificiale generativa per lo sviluppatore: Amazon Bedrock e Amazon Q Developer
  - 2.7.1 Amazon Bedrock
  - 2.7.2 Amazon Q Developer

**Capitolo 3 – Gestire i permessi** *(src: 2. Getting Started with Permissions)*
- 3.1 Obiettivi del capitolo
- 3.2 AWS Identity and Access Management (IAM)
- 3.3 Laboratorio: applicare il principio del privilegio minimo
- 3.4 Configurare le credenziali nell'ambiente di sviluppo
- 3.5 Laboratorio: l'ordine di ricerca delle credenziali nell'AWS SDK for Java
- 3.6 Verifica dei concetti

### Parte II – Lo storage a oggetti con Amazon S3

**Capitolo 4 – Introduzione allo storage** *(src: 3. Getting Started with Storage)*
- 4.1 Obiettivi del capitolo
- 4.2 Le opzioni di storage su AWS e i fondamenti di Amazon S3
- 4.3 Gli endpoint del servizio Amazon S3
- 4.4 Interagire con Amazon S3 tramite AWS CLI
- 4.5 Interagire con Amazon S3 tramite AWS SDK for Java
- 4.6 Verifica dei concetti

**Capitolo 5 – Operare su Amazon S3 dal codice** *(src: 4. Processing your Storage Operations)*
- 5.1 Obiettivi del capitolo
- 5.2 Lavorare con i bucket in Java
- 5.3 Lavorare con gli oggetti
- 5.4 Lavorare con gli oggetti in Java
- 5.5 Concedere accesso temporaneo agli oggetti: i presigned URL
- 5.6 Operazioni massive (bulk) in Java
- 5.7 Ospitare un sito web statico
- 5.8 Caso di studio: S3 Select contro GetObject
- 5.9 Verifica dei concetti

### Parte III – I dati applicativi con Amazon DynamoDB

**Capitolo 6 – Introduzione ai database** *(src: 5. Getting Started with Databases)*
- 6.1 Obiettivi del capitolo
- 6.2 Le opzioni di database su AWS
- 6.3 I concetti chiave di Amazon DynamoDB
- 6.4 Laboratorio: Amazon DynamoDB in azione
- 6.5 Le modalità di accesso a DynamoDB per lo sviluppatore
- 6.6 Laboratorio: NoSQL Workbench
- 6.7 Programmare con Amazon DynamoDB
- 6.8 Richieste e risposte
- 6.9 Verifica dei concetti

**Capitolo 7 – Operare su DynamoDB dal codice** *(src: 6. Processing your Database Operations)*
- 7.1 Obiettivi del capitolo
- 7.2 Progettare le tabelle
- 7.3 Lavorare con le tabelle in Java
- 7.4 Caricare e leggere gli item
- 7.5 Aggiornare e cancellare gli item
- 7.6 Laboratorio: gestire gli item con l'AWS CLI
- 7.7 Le interfacce di alto livello in Java: il DynamoDB Enhanced Client
- 7.8 Il caching per DynamoDB
- 7.9 Verifica dei concetti

### Parte IV – Logica applicativa e API

**Capitolo 8 – La logica applicativa con AWS Lambda** *(src: 7. Processing your Application Logic)*
- 8.1 Obiettivi del capitolo
- 8.2 I servizi di calcolo su AWS
- 8.3 Come funziona AWS Lambda
- 8.4 I modelli di invocazione di Lambda
- 8.5 I permessi di una funzione Lambda
- 8.6 Sviluppare funzioni Lambda
- 8.7 Laboratorio: sviluppare una funzione in Java
- 8.8 Testare le funzioni Lambda
- 8.9 Distribuire le funzioni Lambda
- 8.10 Verifica dei concetti

**Capitolo 9 – Gestire le API con Amazon API Gateway** *(src: 8. Managing the APIs)*
- 9.1 Obiettivi del capitolo
- 9.2 Che cos'è Amazon API Gateway
- 9.3 Lavorare con API Gateway
- 9.4 L'elaborazione di richieste e risposte
- 9.5 La progettazione delle API come codice
- 9.6 Distribuire le API
- 9.7 Caso di studio: endpoint Regional o Edge-optimized?
- 9.8 Verifica dei concetti

### Parte V – Architetture moderne e identità degli utenti

**Capitolo 10 – Costruire un'applicazione moderna** *(src: 9. Building a Modern Application)*
- 10.1 Obiettivi del capitolo
- 10.2 Le applicazioni moderne: dal monolite ai microservizi
- 10.3 Il modello operativo serverless
- 10.4 L'applicazione Polly Notes come architettura serverless
- 10.5 L'orchestrazione con AWS Step Functions
- 10.6 Laboratorio: orchestrare funzioni Lambda con Step Functions
- 10.7 Verifica dei concetti

**Capitolo 11 – Gestire l'accesso degli utenti dell'applicazione** *(src: 10. Granting Access to your Application Users)*
- 11.1 Obiettivi del capitolo
- 11.2 La complessità di autenticazione e autorizzazione
- 11.3 Amazon Cognito
- 11.4 Gestire l'accesso degli utenti
- 11.5 Concedere l'accesso con i User Pool
- 11.6 Concedere l'accesso con gli Identity Pool
- 11.7 Proteggere l'accesso alle API
- 11.8 Laboratorio: Amazon Cognito con una HTTP API
- 11.9 Verifica dei concetti

### Parte VI – Rilascio e osservabilità

**Capitolo 12 – Distribuire l'applicazione** *(src: 11. Deploying your Application)*
- 12.1 Obiettivi del capitolo
- 12.2 Pianificazione e pratiche di deployment
- 12.3 AWS Serverless Application Model (AWS SAM) con Java
- 12.4 Distribuire con AWS SAM
- 12.5 Le strategie di deployment
- 12.6 Verifica dei concetti

**Capitolo 13 – Osservare l'applicazione** *(src: 12. Observing your Application)*
- 13.1 Obiettivi del capitolo
- 13.2 L'observability
- 13.3 Osservare un'applicazione con Amazon CloudWatch
- 13.4 Il tracing con AWS X-Ray
- 13.5 Abilitare il tracing nell'applicazione
- 13.6 Laboratorio: AWS X-Ray
- 13.7 Verifica dei concetti

### Parte VII – Verifica finale

**Capitolo 14 – Test di autovalutazione finale** *(src: 13. Post Assestement)*
- 14.1 Come usare questo capitolo
- 14.2 Le venti domande commentate (raggruppate per area: SDK e CLI, IAM e Cognito, S3, DynamoDB, Lambda e API Gateway, SAM e Step Functions, CloudWatch)
- 14.3 Mappa delle domande verso i capitoli di riferimento

### Appendici
- A. Glossario dei termini AWS
- B. Comandi CLI e classi SDK citati nel libro
- C. Note di aggiornamento: servizi e limiti cambiati dopo la registrazione del corso

---

## Elenco degli argomenti individuati

**Cap. 1** – Panoramica dei servizi del percorso (Console, CLI, SDK, IAM, S3, DynamoDB, Lambda, API Gateway, Cognito, Step Functions, SAM, CloudWatch, X-Ray). Architettura di Polly Notes: Cognito → API Gateway → Lambda (una per operazione CRUD/search/dictate) → DynamoDB, Polly → MP3 su S3, sito statico su S3.

**Cap. 2** – Richieste HTTPS verso gli endpoint REST, TLS, firma SigV4, codici di stato 1xx–5xx, RequestId. Console, CLI e SDK sulle stesse API. Cosa fa l'SDK: binding, firma, retry, paginazione. CLI: dove eseguirla (locale, CloudShell, Session Manager), `aws s3 mb/cp/ls`, help. Chiamate sincrone e asincrone, polling con `describe-table` e JMESPath, waiter (CLI e `DynamoDbWaiter`), eccezioni, metriche e logging dell'SDK. AWS Toolkit per gli IDE, Cloud9 (ambiente EC2, Git, debugger, boto3). GenAI: LLM e foundation model, Bedrock (playground, fine-tuning, continued pre-training, RAG, knowledge base, vector DB), Amazon Q Developer (chat, suggerimenti inline, security scan, CLI).

**Cap. 3** – IAM: user, group, role, policy, principal, action, resource. Policy identity-based e resource-based, SCP, permissions boundary, credenziali temporanee, federazione, cross-account, service role. Logica di valutazione (explicit deny > explicit allow > implicit deny). Best practice: policy sui gruppi e least privilege. Laboratorio: scoprire le API con Log4j/`X-Amz-Target` e scrivere una policy minima. `aws configure`, named profile, file config e credentials, ordine di ricerca delle credenziali, SigV4 e replay attack, STS AssumeRole, credenziali gestite da Cloud9, default credentials provider chain in Java.

**Cap. 4** – Block storage (EBS), file storage (EFS, FSx) e object storage (S3), classi di storage, durabilità e disponibilità, casi d'uso. Oggetti, key, metadata, bucket regionali con nome globale, prefix e delimiter, event notification, versioning e delete marker, tag, IAM policy, bucket policy e ACL, Access Point. Endpoint standard, dual-stack, access point e website. `aws s3` e `aws s3api`, multipart automatico osservato via proxy. `S3Client` con builder e `CreateBucketRequest`.

**Cap. 5** – HeadBucket e codici 200/404/400/403, CreateBucket, `S3Waiter`, dipendenze Maven dell'SDK v2. Upload singolo e multipart (soglie 100 MB / 5 GB / 5 TB), lifecycle per i multipart incompleti, consistenza read-after-write, CopyObject, GetObject con range, HeadObject, S3 Select. Java: `headObject`, `getObjectAsBytes`, `selectObjectContent` con `S3AsyncClient`. Presigned URL con `S3Presigner`. `cp`/`sync`, `listObjectsV2Paginator`, continuation token, S3 Batch Operations e Inventory. Sito statico (`aws s3 website`), CORS. Caso di studio Shakespeare: costi e tempi di S3 Select.

**Cap. 6** – Database relazionali, key-value, graph e in-memory. Confronto SQL/NoSQL: schema, query, scalabilità, ACID, CAP, consistenza. Partition key e sort key, item e attributi, tipi di dato, RCU/WCU, scan e query, LSI e GSI, partizioni e adaptive capacity. Laboratorio Python con la tabella CityPopulation e `ReturnConsumedCapacity`. Console, PartiQL, DynamoDB Local in Docker, NoSQL Workbench. Interfacce low-level, document e object persistence. Wire protocol (`X-Amz-Target`, descrittori di tipo), errori 4xx/5xx.

**Cap. 7** – Control plane e data plane. Progettazione guidata dagli access pattern, cardinalità della partition key, sort key gerarchiche, capacità provisioned e on-demand. Java: `listTables`, `createTable` con waiter, `updateTable` con polling, `deleteTable`. PutItem, BatchWriteItem (25 item, UnprocessedItems), import da S3, GetItem, Query (key condition), paginazione e LastEvaluatedKey, Scan e parallel scan. UpdateItem, conditional write, optimistic locking, DeleteItem. Laboratorio in CloudShell (`--generate-cli-skeleton`, `--cli-input-json`). Enhanced Client: `@DynamoDbBean`, `TableSchema`, `QueryConditional`. Caching con ElastiCache e DAX (solo letture eventually consistent).

**Cap. 8** – EC2, ECS/Fargate, EKS e Lambda a confronto (astrazione, packaging, prezzo, scaling). Event source e integrazioni di Lambda. Anatomia di una funzione: handler, runtime, memoria e CPU, timeout, layer. Invocazione sincrona, asincrona (retry, destination, DLQ) e poll-based. Ciclo di vita dell'ambiente, cold start, provisioned concurrency. Invocation permission e execution role (push e pull). Modello di programmazione stateless, `/tmp`, event e context, monitoring e X-Ray. Laboratorio Java con Gradle e `create-function`/`invoke`/`update-function-configuration`. Test con console, CLI, mock (Moto) e SAM. Version e alias, classi di errore. Packaging zip o container, limiti del servizio.

**Cap. 9** – API Gateway: REST, HTTP e WebSocket API e loro funzioni (stage, API key, throttling, VTL, SDK, mock, cache). Flusso della richiesta, resource e method, endpoint Regional/Edge/Private, integrazione proxy, non-proxy e mock. Method request/response, integration request/response, model e validator. Laboratorio di integrazione diretta con DynamoDB PutItem (ruolo IAM, model JSON Schema, mapping template VTL, test). OpenAPI/Swagger, import ed export. Stage, invoke URL, stage variable, canary release, log e tracing, test con Postman. Caso di studio sulla latenza tra regioni in .NET.

**Cap. 10** – Ciclo DevOps. Monolite e microservizi: caratteristiche, benefici, principi di design, comunicazione sincrona e asincrona. Decomposizione e strangler pattern. Modello operativo tradizionale e serverless, catalogo dei servizi serverless. Polly Notes come architettura completamente serverless. Step Functions: tipi di stato (sequenza, choice, parallel, wait, retry, catch), Amazon States Language, Workflow Studio, InputPath/Parameters/ResultPath/OutputPath, integrazioni di servizio e SDK. Laboratorio: il quiz delle capitali come state machine.

**Cap. 11** – Autenticazione e autorizzazione, OAuth, SAML, OIDC, JWT. Cognito: User Pool (sign-up/sign-in, hosted UI, MFA, password policy, attributi, gruppi, scope) e Identity Pool (credenziali temporanee, federazione, accesso guest). Struttura del JWT (header, payload, signature), ID, access e refresh token. Authorizer Cognito/JWT e Lambda authorizer (TOKEN). Laboratorio: user pool, `aws cognito-idp sign-up`/`admin-initiate-auth`, authorizer JWT su HTTP API, test con curl.

**Cap. 12** – Rischi del deployment tradizionale, cultura e pratiche DevOps (CI/CD, IaC, monitoring). Strumenti AWS: CodeCommit, CodeBuild, CodeArtifact, CodeDeploy, CodePipeline, Config, CloudFormation. SAM: template, risorse `AWS::Serverless::*`, `sam init/build/deploy`, `sam local invoke/start-api/generate-event`, build in container. Laboratorio Java (StarsFunction). Strategie di deployment: canary, linear e all-at-once per Lambda; in-place e blue/green per EC2.

**Cap. 13** – Monitoring e observability, catena dati → informazione → conoscenza → insight → azione, i tre pilastri (log, metriche, trace). CloudWatch: metriche, dimensioni, statistiche, dashboard, allarmi (soglia, periodo, SNS, Auto Scaling), metriche custom, metric filter, CloudWatch agent, Application Insights. X-Ray: daemon, segment e subsegment, trace, service map e colori, annotation e metadata. Strumentazione SDK in .NET, Java e Python, laboratorio con Lambda e API Gateway.

**Cap. 14** – Venti domande d'esame commentate, con le risposte del materiale originale verificate e corrette dove necessario (es. `$input.params`, retry di Lambda, EFS, last writer wins).

---

## Box di approfondimento previsti (selezione)

- *SigV4 e attacchi di replay* (cap. 2, 3)
- *Cloud9 e CodeWhisperer oggi* (cap. 2)
- *Disponibilità e durabilità di S3: i numeri corretti* (cap. 4)
- *Path-style e virtual-hosted-style* (cap. 4, 14)
- *Calcolare RCU e WCU* (cap. 6)
- *Single-table design* (cap. 7)
- *Retry di Lambda per modello di invocazione* (cap. 8, 14)
- *`$input.params` vs `$input.path` in VTL* (cap. 9, 14)
- *Last writer wins nelle Global Tables* (cap. 14)
- *Annotation e metadata in X-Ray* (cap. 13)
