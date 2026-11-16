# Capitolo 11
# Modern Application Architecture

## Introduzione alle Applicazioni Moderne

Le moderne applicazioni cloud-native sono progettate per essere scalabili, resilienti, osservabili e facilmente distribuibili. Questo approccio supera i limiti delle architetture tradizionali monolitiche.

## Architetture Monolitiche

Un'applicazione monolitica concentra tutte le funzionalità in un'unica unità distribuibile.

Sebbene inizialmente semplice da realizzare, la crescita del sistema porta spesso a difficoltà di manutenzione, rilasci complessi e limitazioni nella scalabilità.

## Evoluzione verso i Microservizi

I microservizi suddividono l'applicazione in componenti indipendenti, ciascuno responsabile di uno specifico dominio funzionale.

Questa separazione favorisce autonomia dei team, distribuzioni indipendenti e maggiore resilienza.

## Domain Driven Design

La suddivisione dei servizi dovrebbe seguire i domini di business piuttosto che la struttura tecnologica dell'applicazione.

Ogni microservizio diventa responsabile di una specifica capacità aziendale.

## DevOps e Continuous Delivery

Le applicazioni moderne adottano modelli DevOps basati su automazione continua, integration pipeline, deployment automatico e monitoraggio costante.

## Serverless Architecture

I servizi serverless consentono di concentrarsi sulla logica applicativa delegando l'infrastruttura alla piattaforma cloud.

AWS Lambda, DynamoDB, API Gateway e S3 costituiscono esempi significativi di questo paradigma.

## Comunicazione tra Servizi

I microservizi possono comunicare tramite API sincrone oppure attraverso modelli asincroni basati su eventi e code.

La scelta dipende dai requisiti di latenza, disponibilità e accoppiamento tra servizi.

## Benefici dei Microservizi

I principali vantaggi comprendono:

- distribuzione indipendente;
- scalabilità selettiva;
- isolamento dei guasti;
- flessibilità tecnologica;
- maggiore produttività dei team.

## Migrazione da Monolite a Microservizi

La migrazione deve essere incrementale.

Lo Strangler Pattern prevede l'estrazione progressiva delle funzionalità dal monolite fino alla sua completa sostituzione.

## Organizzazione dei Team

Le applicazioni moderne influenzano anche l'organizzazione aziendale. I team vengono spesso strutturati attorno ai servizi che sviluppano e mantengono.

## AWS Step Functions

Step Functions permette di orchestrare workflow distribuiti mediante state machine visuali.

Le attività possono essere eseguite in sequenza, in parallelo o attraverso percorsi condizionali.

## Gestione degli Errori

Le workflow moderne prevedono retry automatici, gestione delle eccezioni e percorsi alternativi in caso di errore.

## Osservabilità

CloudWatch, X-Ray e i sistemi di logging centralizzati permettono di monitorare il comportamento dell'intera architettura distribuita.

## Best Practice

Le applicazioni moderne dovrebbero essere debolmente accoppiate, orientate agli eventi, automatizzate e progettate per il cambiamento continuo.

## Conclusioni

La modern application architecture combina microservizi, DevOps, serverless, automazione e osservabilità per creare sistemi resilienti e capaci di evolvere rapidamente nel tempo.
