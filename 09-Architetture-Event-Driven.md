# Capitolo 9
# Amazon API Gateway e la Gestione delle API Cloud-Native

## Introduzione

Nelle architetture moderne, le API costituiscono il punto di accesso principale ai servizi applicativi. Client web, applicazioni mobili, microservizi e sistemi esterni necessitano di un'interfaccia stabile, sicura e documentata attraverso cui accedere alle funzionalità di business.

Amazon API Gateway è il servizio AWS progettato per creare, pubblicare, proteggere, monitorare e gestire API su larga scala. Il servizio rappresenta spesso la porta di ingresso delle applicazioni serverless e dei sistemi a microservizi.

---

## Il ruolo delle API nelle architetture moderne

Un'API può essere vista come un contratto tra consumatori e servizi backend.

Tale contratto definisce:

- operazioni disponibili;
- formato delle richieste;
- struttura delle risposte;
- modalità di autenticazione;
- gestione degli errori.

Questo approccio consente di separare il frontend dalla logica applicativa, favorendo evoluzione indipendente e scalabilità.

---

## Cos'è Amazon API Gateway

Amazon API Gateway consente di esporre servizi backend attraverso endpoint HTTPS gestiti.

I backend possono essere implementati tramite AWS Lambda, Amazon DynamoDB, applicazioni ospitate su Amazon EC2 oppure servizi HTTP di terze parti.

API Gateway gestisce autenticazione, autorizzazione, throttling, monitoraggio, trasformazione dei dati e pubblicazione degli endpoint.

---

## Tipologie di API supportate

API Gateway supporta tre modelli principali.

### REST API

Le REST API offrono il maggior numero di funzionalità avanzate.

Supportano modelli, mapping template, validazione, caching, versionamento e trasformazioni dei dati.

### HTTP API

Le HTTP API sono una versione più leggera e ottimizzata.

Richiedono una configurazione più semplice, offrono costi inferiori e latenze ridotte.

Sono particolarmente adatte all'integrazione diretta con Lambda.

### WebSocket API

Le WebSocket API mantengono connessioni persistenti tra client e backend.

Sono utilizzate per notifiche in tempo reale, chat, gaming online e dashboard interattive.

---

## API Gateway come Front Door delle applicazioni

Nelle architetture serverless API Gateway rappresenta il punto di ingresso unico verso il backend.

Gli utenti interagiscono esclusivamente con l'endpoint pubblico.

Le richieste vengono quindi instradate verso servizi interni come Lambda o DynamoDB.

Questo modello migliora sicurezza, controllo e manutenibilità.

---

## Risorse e Metodi

Le API REST vengono organizzate mediante Resources e Methods.

Una Resource rappresenta un'entità logica dell'applicazione, ad esempio:

- /notes
- /users
- /orders

I Methods definiscono invece le operazioni HTTP disponibili:

- GET
- POST
- PUT
- DELETE
- PATCH

La combinazione tra resource e metodo identifica un endpoint specifico.

---

## Endpoint Regional, Edge e Private

API Gateway mette a disposizione differenti modalità di pubblicazione.

Gli endpoint Regional operano esclusivamente nella regione AWS selezionata.

Gli endpoint Edge-Optimized sfruttano la rete globale CloudFront per ridurre la latenza degli utenti distribuiti geograficamente.

Le Private API sono invece accessibili soltanto all'interno di reti private e VPC autorizzati.

---

## Integrazioni con i Servizi AWS

Uno dei principali punti di forza di API Gateway consiste nella possibilità di integrarsi direttamente con altri servizi AWS.

L'integrazione più comune coinvolge AWS Lambda.

Tuttavia è possibile interagire direttamente con Amazon DynamoDB senza introdurre componenti intermedi.

In questo scenario API Gateway riceve la richiesta dell'utente, costruisce la chiamata richiesta dal servizio backend e restituisce il risultato al client.

---

## Tipologie di Integrazione

### Proxy Integration

Nella modalità proxy il payload viene inoltrato senza trasformazioni.

Lambda riceve direttamente il contenuto della richiesta e si occupa della logica di validazione e trasformazione.

### Non-Proxy Integration

Nelle integrazioni non proxy API Gateway controlla completamente la trasformazione delle richieste e delle risposte.

Questo approccio permette di adattare formati differenti tra client e backend.

### Mock Integration

Le Mock Integration consentono di simulare un backend inesistente.

Sono particolarmente utili durante lo sviluppo API-First e nelle attività di test.

---

## Il ciclo di elaborazione di una richiesta

Una richiesta REST passa attraverso diverse fasi.

Inizialmente API Gateway valida la richiesta.

Successivamente applica eventuali trasformazioni.

La richiesta viene quindi inoltrata al backend.

La risposta ritorna ad API Gateway che può applicare ulteriori trasformazioni prima di restituirla al client.

Questo modello consente di isolare frontend e backend semplificando l'evoluzione architetturale.

---

## Modelli e Validazione

Le REST API supportano modelli basati su JSON Schema.

Un modello definisce la struttura che una richiesta deve rispettare.

Ad esempio è possibile richiedere obbligatoriamente:

- UserId
- NoteId
- Note

Qualunque richiesta che non soddisfa il modello viene rifiutata prima di raggiungere il backend.

Questa tecnica riduce errori e migliora la qualità dei dati elaborati.

---

## Mapping Template e Velocity Template Language

API Gateway utilizza Velocity Template Language (VTL) per trasformare richieste e risposte.

I template consentono di convertire il payload fornito dal client nel formato richiesto dal backend.

Quando si utilizza DynamoDB come integrazione diretta, VTL permette di trasformare una semplice richiesta JSON nei parametri richiesti dall'operazione PutItem.

---

## Sicurezza e Autorizzazione

API Gateway supporta differenti strategie di controllo degli accessi.

Le API possono essere pubbliche oppure protette mediante AWS IAM, token di autenticazione o sistemi federati.

Le autorizzazioni consentono di determinare quali utenti, applicazioni o servizi possano utilizzare un determinato endpoint.

---

## API Keys, Metering e Throttling

Le API Key permettono di identificare i consumatori delle API.

Attraverso queste chiavi è possibile applicare limiti di consumo, controllare il traffico ed effettuare attività di metering.

Il throttling protegge il backend da utilizzi eccessivi e contribuisce a mantenere stabile il livello di servizio.

---

## Caching delle Risposte

API Gateway integra funzionalità di response caching.

Quando più utenti richiedono frequentemente gli stessi dati, il servizio può rispondere direttamente dalla cache senza coinvolgere il backend.

I benefici principali sono:

- riduzione della latenza;
- diminuzione dei costi;
- alleggerimento del carico sui servizi applicativi.

---

## API Design as Code

La gestione moderna delle API segue il principio Design as Code.

API Gateway supporta gli standard OpenAPI e Swagger.

L'intera definizione dell'API può essere descritta mediante file YAML o JSON versionabili all'interno di repository Git.

Questo approccio favorisce collaborazione, automazione e governance.

---

## SDK Generation

A partire dalle definizioni API è possibile generare SDK client per diversi linguaggi.

Gli sviluppatori che consumano le API possono utilizzare librerie già predisposte riducendo il tempo necessario all'integrazione.

---

## Deployment e Stages

Una volta completata la progettazione, l'API viene distribuita mediante Stage.

Ogni stage rappresenta un ambiente indipendente.

Gli esempi più comuni sono:

- Development
- Test
- QA
- Production

Ogni stage dispone di un endpoint dedicato.

---

## Stage Variables

Le Stage Variables sono coppie chiave-valore associate allo stage.

Possono essere utilizzate per selezionare differenti alias Lambda, risorse backend o parametri applicativi.

Questo meccanismo permette di utilizzare la stessa definizione API in ambienti differenti senza modificare il codice.

---

## Canary Release

API Gateway supporta strategie di rilascio progressive.

Con una Canary Release è possibile inviare una percentuale limitata di traffico verso una nuova versione dell'API.

Monitorando metriche e comportamento degli utenti è possibile verificare la stabilità della nuova release prima del rilascio completo.

Questo approccio riduce significativamente il rischio operativo.

---

## Testing delle API

Le API possono essere testate attraverso differenti strumenti:

- console API Gateway;
- AWS CLI;
- AWS SDK;
- Postman.

I test consentono di verificare validazione, autorizzazioni, mapping template e comportamento del backend.

---

## Caso di Studio: Polly Notes

Nell'applicazione Polly Notes, API Gateway espone endpoint dedicati alla gestione delle note.

Ogni operazione viene instradata verso una funzione Lambda differente.

L'API costituisce il punto di ingresso unico per tutte le richieste provenienti dal frontend dell'applicazione.

Questo modello rappresenta un esempio concreto di microservizi serverless basati su AWS.

---

## Best Practice

È consigliabile adottare API versionate, utilizzare modelli di validazione, documentare le interfacce tramite OpenAPI, abilitare logging e monitoraggio, applicare throttling adeguato e utilizzare canary release per l'introduzione di modifiche significative.

---

## Conclusioni

Amazon API Gateway rappresenta uno dei servizi fondamentali dello sviluppo cloud-native su AWS. Grazie alla capacità di esporre, proteggere, monitorare e trasformare API, costituisce il punto di connessione tra utenti, applicazioni e servizi backend, consentendo la realizzazione di architetture moderne, scalabili e orientate ai microservizi.
