# Parte I – Le fondamenta dello sviluppo su AWS

# Capitolo 1 – Sviluppare su AWS: panoramica del percorso

## 1.1 Il percorso di formazione e i servizi trattati

Chi si avvicina allo sviluppo di software su Amazon Web Services si trova davanti a un catalogo di servizi molto ampio. Ciascun servizio risolve un problema specifico: dove memorizzare i dati, dove eseguire il codice, come esporre le API, come autenticare gli utenti, come rilasciare e monitorare l'applicazione. Questo libro non vuole passare in rassegna l'intero catalogo. Si concentra sul gruppo di servizi che uno sviluppatore usa più spesso per costruire su AWS architetture **altamente disponibili** e **ottimizzate**, e li presenta nell'ordine in cui servono durante la vita reale di un'applicazione.

Il punto di partenza sono gli strumenti con cui si dialoga con AWS. La **AWS Management Console** è l'interfaccia web, utile per esplorare e configurare. La **AWS Command Line Interface (CLI)** permette di automatizzare le operazioni con script. Gli **AWS Software Development Kit (SDK)** portano le API di AWS dentro il codice applicativo, nel linguaggio di programmazione scelto. Questi tre strumenti sono modi diversi di fare la stessa cosa: inviare richieste alle API dei servizi AWS. Capire questo legame è il primo passo per lavorare con consapevolezza.

Subito dopo viene la **sicurezza**. Ogni interazione con AWS deve essere autenticata e autorizzata, e lo sviluppatore deve saper scrivere **policy IAM** (AWS Identity and Access Management) che concedano esattamente i permessi necessari, né più né meno. La sicurezza è trattata all'inizio del percorso proprio perché condiziona tutto il resto: un'applicazione che non riesce a chiamare un servizio per un permesso mancante, o che ne ha troppi, è uno dei problemi più comuni nello sviluppo cloud.

Quasi ogni applicazione deve conservare dei dati, e il libro approfondisce due servizi complementari. **Amazon DynamoDB** è un database NoSQL di tipo chiave-valore, adatto a memorizzare i dati strutturati dell'applicazione con latenze molto basse e a qualunque scala. **Amazon Simple Storage Service (Amazon S3)** è il servizio di storage a oggetti, adatto a file di qualsiasi tipo e dimensione. Offre funzionalità come il **versioning**, l'**archiviazione** a lungo termine e un **controllo degli accessi** granulare.

Il codice deve poi essere eseguito da qualche parte. Il percorso dedica ampio spazio ad **AWS Lambda**, il servizio di calcolo **serverless**: lo sviluppatore fornisce solo il codice, mentre i server su cui gira sono gestiti e mantenuti interamente da AWS.

Con queste basi si passa alle **architetture moderne**. **Amazon API Gateway** permette di pubblicare le API di un microservizio. **Amazon Cognito** gestisce l'identità dei clienti dell'applicazione: registrazione, login, token. **AWS Step Functions** orchestra i workflow delle applicazioni distribuite. Sono tutti servizi serverless, e insieme mostrano come si possa costruire un'applicazione completa senza gestire neppure un server.

Un'applicazione moderna deve anche essere rilasciata in modo ripetibile e automatico. Per questo viene introdotto **AWS Serverless Application Model (AWS SAM)**, un framework che automatizza la creazione e l'aggiornamento di tutte le risorse di un'applicazione serverless a partire da un unico template.

L'ultima tappa è la vita in produzione: il **monitoraggio** e l'**observability**. I log dell'applicazione vengono inviati ad **Amazon CloudWatch Logs**, le metriche di applicazione e infrastruttura vengono raccolte in **CloudWatch Metrics**, e **AWS X-Ray** ricostruisce le *trace*, cioè il percorso completo di una richiesta attraverso i componenti di un'applicazione distribuita.

> **Box – La mappa del percorso**
>
> Il libro segue il ciclo di vita di un'applicazione. Dopo aver visto come *dialogare* con AWS (Console, CLI, SDK) e come farlo in modo *sicuro* (IAM), si impara dove *memorizzare i dati* (S3 e DynamoDB) e dove *eseguire il codice* (Lambda). Poi si vede come *esporre* le funzionalità (API Gateway), come *proteggerle* per gli utenti finali (Cognito) e come *orchestrarle* (Step Functions). Infine si impara a *rilasciare* l'applicazione (SAM e le pratiche DevOps) e a *osservarla* in produzione (CloudWatch e X-Ray). Ogni capitolo aggiunge un tassello alla stessa applicazione di esempio, descritta nel paragrafo seguente.

## 1.2 L'applicazione di riferimento: Polly Notes

Per evitare che i servizi restino concetti astratti, il percorso usa un'architettura di riferimento che torna in quasi tutti i capitoli: l'applicazione **Polly Notes**. È un sito web per prendere appunti, semplice nelle funzionalità ma completo nella struttura. Per questo è un ottimo banco di prova per vedere come i diversi servizi AWS collaborano.

Dal punto di vista dell'utente il funzionamento è immediato. Si accede al sito autenticandosi tramite **Amazon Cognito**, il servizio che gestisce le identità degli utenti. Una volta autenticati si possono creare note di testo libero. Le operazioni disponibili sono quelle classiche di gestione dei dati: **creare** una nota, **leggerla**, **aggiornarla**, **cancellarla** e **cercare** le note di un determinato utente tramite il suo identificativo. A queste si aggiunge una funzionalità caratteristica che dà il nome all'applicazione, la **dettatura**: a partire dal testo di una nota, l'applicazione genera un file audio usando **Amazon Polly**, il servizio AWS di sintesi vocale (*text-to-speech*).

Il flusso tecnico di una richiesta attraversa diversi componenti:

1. L'utente si autentica con **Amazon Cognito**.
2. Il browser invia le richieste ad **Amazon API Gateway**, che fa da porta d'ingresso pubblica davanti a un insieme di funzioni Lambda.
3. Ogni operazione (creazione, lettura, aggiornamento, cancellazione, ricerca, dettatura) è implementata da una specifica funzione **AWS Lambda**: un ambiente di calcolo serverless che contiene il codice dell'operazione.
4. Le funzioni Lambda leggono e scrivono su una tabella **Amazon DynamoDB**, che conserva tutte le note.
5. La funzione di dettatura invoca **Amazon Polly**, che produce un file MP3. Il file viene salvato in un **bucket Amazon S3** dedicato.
6. Il sito web stesso, cioè le pagine HTML, i fogli di stile e il codice JavaScript che il browser scarica, è ospitato in un **secondo bucket S3**, configurato per servire contenuti statici.

Due servizi lavorano trasversalmente su tutta l'architettura. **Amazon CloudWatch** raccoglie log e metriche. **AWS X-Ray** traccia le richieste da un componente all'altro. Insieme danno all'applicazione l'observability necessaria per capire in ogni momento se funziona correttamente e, in caso contrario, dove si trova il problema. Infine, l'intera architettura viene distribuita con **AWS SAM**, che permette di creare e aggiornare tutte le risorse in modo semplice, coerente e ripetibile.

```
            ┌──────────────┐
  Utente ──▶│   Cognito    │  autenticazione
            └──────┬───────┘
                   ▼
  Browser ◀── S3 (sito web statico)
     │
     ▼
┌─────────────┐     ┌──────────────────────────────┐     ┌──────────┐
│ API Gateway │────▶│ Lambda: create / list / update│────▶│ DynamoDB │
│             │     │ delete / search / dictate     │     │ (note)   │
└─────────────┘     └──────────────┬───────────────┘     └──────────┘
                                   │ dictate
                                   ▼
                              Amazon Polly ──▶ S3 (file MP3)

  Trasversali: CloudWatch (log e metriche) · X-Ray (trace) · SAM (deploy)
```

> **Box – Perché un'architettura completamente serverless?**
>
> In Polly Notes non c'è nessun server da gestire: nessuna istanza EC2 da avviare, aggiornare o dimensionare. Ogni componente è un servizio gestito e lo sviluppatore lavora solo con le sue API e la sua configurazione. Questo approccio verrà approfondito nel Capitolo 10. Fin d'ora conviene notarne i vantaggi pratici. Lo scaling è automatico. Si paga in base all'uso effettivo e non per server inattivi. L'alta disponibilità è già integrata nei servizi. E soprattutto lo sviluppatore può concentrarsi sulla logica dell'applicazione invece che sull'infrastruttura.

Nei capitoli successivi ogni servizio di questo diagramma verrà ripreso e approfondito: come si scrive il codice che salva una nota su DynamoDB, come si configura l'API che la espone, come si protegge con Cognito, come si rilascia con SAM e come si osserva con X-Ray. Conviene quindi tenere a mente questa architettura: è il filo conduttore dell'intero libro.
