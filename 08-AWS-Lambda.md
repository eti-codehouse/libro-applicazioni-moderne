# Capitolo 8
# AWS Lambda e il Paradigma Function-as-a-Service

## Introduzione

L'evoluzione delle architetture cloud ha portato alla nascita del paradigma serverless, un modello in cui gli sviluppatori possono concentrarsi esclusivamente sulla logica applicativa delegando alla piattaforma la gestione dell'infrastruttura sottostante. In questo contesto, AWS Lambda rappresenta uno dei servizi più importanti dell'ecosistema AWS.

Lambda consente di eseguire codice in risposta a eventi senza dover amministrare server, installare sistemi operativi o pianificare capacità computazionale. L'infrastruttura viene gestita interamente da AWS e il modello di pricing è basato sul consumo effettivo delle risorse.

---

## I servizi di calcolo in AWS

AWS mette a disposizione differenti modelli di esecuzione delle applicazioni.

Amazon EC2 fornisce macchine virtuali complete sulle quali l'organizzazione mantiene il controllo del sistema operativo e dell'ambiente di esecuzione.

Amazon ECS ed Amazon EKS introducono il paradigma dei container, consentendo di distribuire applicazioni containerizzate su infrastrutture gestite.

AWS Lambda si colloca a un livello di astrazione superiore. Lo sviluppatore non gestisce host, cluster o sistemi operativi ma fornisce esclusivamente il codice da eseguire.

### Confronto dei modelli di esecuzione

EC2 astrae l'hardware.

ECS ed EKS astraggono il sistema operativo attraverso i container.

Lambda astrae direttamente il runtime applicativo.

Questo consente di ridurre drasticamente la complessità operativa e accelerare il rilascio delle applicazioni.

---

## Cos'è AWS Lambda

AWS Lambda è una piattaforma Function-as-a-Service che esegue codice in risposta a eventi.

Lo sviluppatore crea una funzione, definisce il runtime desiderato e distribuisce il pacchetto applicativo. Da quel momento AWS si occupa del provisioning, della disponibilità, della sicurezza infrastrutturale e della scalabilità automatica.

L'applicazione viene eseguita soltanto quando necessario e il costo è proporzionale alle richieste ricevute.

---

## Runtime supportati

Lambda supporta numerosi linguaggi di programmazione.

Tra i principali troviamo:

- Python
- Java
- .NET
- Node.js
- Ruby
- Go

È inoltre possibile implementare runtime personalizzati oppure utilizzare immagini container per linguaggi non supportati nativamente.

---

## Anatomia di una funzione Lambda

Ogni funzione Lambda è composta da diversi elementi fondamentali.

Il primo è il codice applicativo contenente la logica di business.

Il secondo è il runtime responsabile dell'esecuzione.

Sono inoltre presenti configurazioni relative a memoria, timeout, concorrenza, variabili di ambiente, autorizzazioni e trigger.

La componente centrale della funzione è l'handler, ovvero il metodo che Lambda invoca quando viene ricevuto un evento.

---

## Come funziona una funzione Lambda

Quando viene ricevuto un evento, AWS Lambda crea o riutilizza un ambiente di esecuzione.

L'evento viene passato all'handler della funzione insieme alle informazioni di contesto.

Il codice può elaborare i dati ricevuti, interagire con altri servizi AWS, chiamare sistemi esterni e restituire una risposta.

Una volta completata l'elaborazione, l'ambiente può essere riutilizzato per richieste successive oppure terminato.

---

## Event Source e Trigger

Uno dei principali punti di forza di Lambda consiste nella profonda integrazione con l'ecosistema AWS.

Le funzioni possono essere attivate da numerose sorgenti evento.

Tra le più comuni troviamo:

- Amazon S3
- Amazon API Gateway
- DynamoDB Streams
- Amazon Kinesis
- Amazon SNS
- Amazon SQS
- AWS CodeCommit
- AWS CloudFormation

Questo approccio consente di costruire architetture completamente event-driven.

---

## Esempio: Elaborazione automatica delle immagini

Si consideri un'applicazione che permette agli utenti di caricare immagini.

Quando un file viene inserito in un bucket Amazon S3, il servizio genera un evento che attiva automaticamente una funzione Lambda.

La funzione recupera l'immagine originale, genera una miniatura e la salva in un secondo bucket.

L'intero processo avviene senza server dedicati e scala automaticamente in funzione del numero di immagini caricate.

---

## Modelli di invocazione

AWS Lambda supporta tre modelli principali di esecuzione.

### Invocazione sincrona

Nel modello sincrono il chiamante attende la risposta della funzione.

Il caso più comune è l'integrazione con Amazon API Gateway.

L'utente invia una richiesta HTTP e attende che la funzione produca il risultato.

### Invocazione asincrona

Nel modello asincrono il chiamante delega l'esecuzione alla piattaforma senza attendere il completamento.

Lambda accetta l'evento, lo inserisce internamente in coda e lo esegue successivamente.

Questo approccio è utilizzato da servizi come Amazon S3 e Amazon SNS.

### Poll-Based Invocation

Nel modello poll-based è Lambda a interrogare periodicamente una sorgente dati.

Questo meccanismo viene utilizzato con:

- DynamoDB Streams
- Amazon Kinesis
- Amazon MSK (Kafka)

---

## Retry, Destinazioni e Dead Letter Queue

Le invocazioni asincrone permettono di configurare meccanismi di affidabilità avanzati.

Se una funzione fallisce, Lambda può riprovare automaticamente l'elaborazione.

È inoltre possibile configurare destinazioni per gli eventi completati o falliti.

Quando un evento continua a generare errori può essere inviato a una Dead Letter Queue tramite Amazon SQS per analisi successive.

---

## Il ciclo di vita di una funzione Lambda

Ogni esecuzione segue un ciclo di vita ben definito.

La piattaforma scarica il codice, carica eventuali layer, inizializza il runtime e quindi esegue l'handler.

L'inizializzazione avviene una sola volta per ogni ambiente di esecuzione.

Le invocazioni successive possono riutilizzare l'ambiente già presente evitando costi di inizializzazione aggiuntivi.

---

## Cold Start e Warm Start

Quando Lambda crea un nuovo ambiente di esecuzione si verifica un fenomeno denominato Cold Start.

Durante questa fase vengono caricati codice, dipendenze e runtime.

Le richieste successive possono invece utilizzare ambienti già attivi, beneficiando dei Warm Start.

I Warm Start riducono significativamente la latenza percepita dagli utenti.

---

## Ridurre i Cold Start

AWS mette a disposizione diverse strategie per mitigare i Cold Start.

Una possibilità consiste nell'utilizzare Amazon EventBridge per eseguire periodicamente la funzione mantenendo attivi gli ambienti.

Una seconda opzione è Provisioned Concurrency, che mantiene un numero predefinito di ambienti pronti per l'esecuzione.

La scelta dipende dai requisiti di latenza e dall'analisi costi-benefici.

---

## Concorrenza e Scalabilità

Una delle caratteristiche più apprezzate di Lambda è la scalabilità automatica.

La piattaforma crea nuovi ambienti di esecuzione in risposta all'aumento del traffico.

Gli sviluppatori non devono progettare cluster, bilanciatori o meccanismi di scaling.

La gestione della concorrenza è integrata direttamente nel servizio.

---

## Permessi e Sicurezza

Le autorizzazioni in Lambda si dividono in due categorie.

### Invocation Permissions

Determinano chi può eseguire la funzione.

Possono essere utenti, ruoli IAM oppure altri servizi AWS.

### Execution Permissions

Determinano quali azioni può eseguire la funzione durante l'esecuzione.

Tali autorizzazioni sono definite tramite l'Execution Role associato alla funzione.

---

## Execution Role

L'Execution Role rappresenta uno degli elementi più importanti della sicurezza Lambda.

Questo ruolo viene utilizzato per autorizzare ogni interazione della funzione con altri servizi AWS.

Ad esempio:

- lettura da DynamoDB;
- scrittura su Amazon S3;
- pubblicazione su Amazon SNS;
- accesso ai log CloudWatch.

L'applicazione del principio del minimo privilegio è fondamentale anche nelle architetture serverless.

---

## Logging e Monitoraggio

Tutti i log generati dalle funzioni vengono inviati automaticamente ad Amazon CloudWatch Logs.

Questo permette di analizzare errori, verificare l'esecuzione delle richieste e raccogliere informazioni diagnostiche.

Un errore molto comune consiste nel dimenticare le autorizzazioni CloudWatch all'interno dell'Execution Role, impedendo la registrazione dei log.

---

## Tracciamento Distribuito con AWS X-Ray

AWS X-Ray consente di seguire una richiesta lungo l'intera catena applicativa.

In applicazioni distribuite composte da API Gateway, Lambda, DynamoDB e altri servizi, X-Ray permette di identificare rapidamente il punto in cui si concentra la latenza.

Questa funzionalità è particolarmente importante nelle architetture a microservizi.

---

## Sviluppo Locale e Tooling

Le funzioni Lambda possono essere sviluppate utilizzando qualsiasi ambiente professionale.

Tra gli strumenti più diffusi troviamo:

- Visual Studio Code con AWS Toolkit;
- AWS Cloud9;
- AWS CLI;
- AWS SDK;
- AWS SAM.

Gli strumenti consentono di testare localmente il codice prima della distribuzione in ambiente cloud.

---

## Programmazione Stateless

Lambda adotta un modello di esecuzione stateless.

L'ambiente può essere terminato in qualsiasi momento e non deve essere utilizzato per memorizzare informazioni permanenti.

Lo stato applicativo deve essere conservato in servizi esterni quali:

- Amazon DynamoDB;
- Amazon RDS;
- Amazon S3;
- sistemi di caching dedicati.

---

## Event Object e Context Object

Ogni funzione riceve normalmente due parametri fondamentali.

L'Event Object contiene le informazioni associate all'evento che ha generato l'invocazione.

Il Context Object fornisce informazioni sull'ambiente di esecuzione, sulla memoria disponibile, sul timeout residuo e sui dettagli della funzione.

La comprensione di questi oggetti è fondamentale per la realizzazione di funzioni robuste e diagnostibili.

---

## Lambda Layers

I Layer consentono di separare librerie e dipendenze condivise dal codice applicativo.

Una singola libreria può essere riutilizzata da molteplici funzioni Lambda riducendo duplicazioni e semplificando la manutenzione.

AWS mette inoltre a disposizione layer pubblici contenenti componenti software frequentemente utilizzati.

---

## Packaging e Deployment

Le funzioni Lambda possono essere distribuite in due modalità principali.

### Deployment ZIP

Il codice e le dipendenze vengono compressi in un archivio ZIP e caricati nel servizio.

Questa modalità è particolarmente adatta per funzioni leggere.

### Deployment con Container

È possibile utilizzare immagini Docker archiviate in Amazon ECR.

Questo approccio consente di raggiungere dimensioni significativamente superiori e supportare runtime personalizzati.

---

## Versioni e Alias

Lambda supporta il versionamento delle funzioni.

Ogni versione rappresenta uno snapshot immutabile del codice distribuito.

Gli alias permettono di associare nomi logici come Dev, Test e Prod a specifiche versioni.

Questa capacità facilita il rilascio controllato delle nuove funzionalità.

---

## Strategie di test

Le funzioni Lambda possono essere testate a diversi livelli.

I test unitari verificano la correttezza del codice isolando le dipendenze esterne.

I test di integrazione consentono di validare l'interazione con i servizi AWS.

Le invocazioni manuali tramite console o AWS CLI permettono infine di verificare il comportamento operativo dopo il deployment.

---

## AWS Serverless Application Model (SAM)

AWS SAM semplifica il ciclo di vita delle applicazioni serverless.

Consente di definire funzioni, API e infrastruttura mediante template dichiarativi.

Offre strumenti per sviluppo locale, test, packaging e deployment automatico.

SAM rappresenta oggi uno degli strumenti più importanti nello sviluppo professionale basato su Lambda.

---

## Limiti Operativi di Lambda

Tra i principali limiti del servizio troviamo:

- durata massima di esecuzione pari a 15 minuti;
- memoria configurabile fino a diversi gigabyte;
- limiti sulle dimensioni dei pacchetti ZIP;
- limiti sui payload sincroni e asincroni;
- limiti di concorrenza configurabili.

La conoscenza di questi vincoli è essenziale per la progettazione corretta delle applicazioni.

---

## Best Practice

Lo sviluppo efficace di funzioni Lambda dovrebbe seguire alcune linee guida fondamentali.

È opportuno mantenere il codice leggero, minimizzare le dipendenze, sfruttare i layer per le librerie condivise, archiviare lo stato all'esterno della funzione e monitorare costantemente prestazioni e costi.

L'adozione di SAM, CloudWatch e X-Ray contribuisce ulteriormente alla qualità complessiva delle soluzioni serverless.

---

## Conclusioni

AWS Lambda rappresenta uno dei pilastri dello sviluppo cloud-native moderno. Grazie al modello Function-as-a-Service, alla scalabilità automatica e alla profonda integrazione con i servizi AWS, consente di costruire applicazioni altamente elastiche, affidabili ed economicamente efficienti.

La comprensione dei modelli di invocazione, del ciclo di vita delle funzioni, delle autorizzazioni e delle strategie di deployment costituisce la base per progettare sistemi serverless professionali in grado di sostenere carichi enterprise e scenari mission-critical.
