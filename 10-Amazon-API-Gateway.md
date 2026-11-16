# Capitolo 10
# Amazon API Gateway: Progettazione e Pubblicazione delle API

## Introduzione

Le API rappresentano il collegamento fondamentale tra frontend, applicazioni mobili, microservizi e sistemi esterni. In un'architettura cloud-native, l'API non è semplicemente un'interfaccia tecnica ma un contratto di business che definisce come i consumatori possono accedere alle funzionalità dell'applicazione.

Amazon API Gateway costituisce il servizio AWS dedicato alla pubblicazione, protezione, monitoraggio e gestione delle API.

## API First Design

La progettazione moderna delle API adotta un approccio API First. Prima di implementare il backend, viene definito il contratto dell'interfaccia, documentando endpoint, parametri, payload e risposte.

Questo approccio favorisce collaborazione tra team frontend e backend, accelerando lo sviluppo e riducendo le ambiguità progettuali.

## Tipologie di API

Amazon API Gateway supporta REST API, HTTP API e WebSocket API.

Le REST API offrono il maggior numero di funzionalità avanzate. Le HTTP API privilegiano semplicità, prestazioni e costi ridotti. Le WebSocket API consentono comunicazioni bidirezionali in tempo reale.

## Risorse e Metodi

Le API vengono organizzate mediante risorse e metodi HTTP.

Una risorsa rappresenta un'entità applicativa come utenti, note o ordini. I metodi HTTP definiscono invece le operazioni disponibili quali GET, POST, PUT e DELETE.

## Integrazione con AWS Lambda

L'integrazione più comune collega API Gateway a funzioni Lambda. In questo modello le richieste vengono trasformate in eventi che attivano logiche serverless scalabili e indipendenti.

## Integrazione Diretta con DynamoDB

API Gateway può interagire direttamente con DynamoDB senza componenti intermedi. Questo approccio riduce latenza, complessità e costi operativi.

## Request Validation

La validazione delle richieste consente di intercettare dati errati prima che raggiungano il backend.

Attraverso JSON Schema è possibile definire attributi obbligatori, formati accettati e vincoli di struttura.

## Mapping Template e VTL

Velocity Template Language permette di trasformare richieste e risposte adattando i formati tra client e backend.

Questa capacità rende possibile il disaccoppiamento completo tra consumatori e servizi interni.

## Sicurezza delle API

Le API possono essere protette mediante IAM, JWT, Amazon Cognito, Lambda Authorizer e meccanismi personalizzati di autenticazione e autorizzazione.

## API Keys e Throttling

Le API Key consentono di identificare i consumatori e applicare quote di utilizzo. Il throttling protegge le applicazioni da accessi eccessivi o comportamenti anomali.

## Deployment e Stage

API Gateway supporta ambienti distinti quali Development, Test e Production.

Gli stage consentono di distribuire nuove versioni senza impattare gli ambienti esistenti.

## OpenAPI e API as Code

Le definizioni OpenAPI permettono di descrivere le API mediante file versionabili, favorendo automazione, governance e Continuous Delivery.

## Canary Release

Le Canary Release consentono di distribuire modifiche a una piccola percentuale di utenti prima del rollout completo.

## Monitoraggio

L'integrazione con CloudWatch consente di raccogliere log, errori, metriche e informazioni prestazionali sulle API.

## Best Practice

Le API devono essere versionate, documentate, validate e monitorate. Inoltre è consigliabile applicare il principio del minimo privilegio e separare gli ambienti di sviluppo da quelli di produzione.

## Conclusioni

Amazon API Gateway rappresenta il punto di ingresso ideale per applicazioni serverless e microservizi, offrendo sicurezza, scalabilità e governance centralizzata.
