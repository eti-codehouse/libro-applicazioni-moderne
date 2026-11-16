# Capitolo 2
# Accesso Programmatico ai Servizi AWS e Strumenti di Sviluppo

## Introduzione

Una delle caratteristiche fondamentali di Amazon Web Services è la possibilità di interagire con ogni servizio attraverso API standardizzate. Ogni operazione eseguibile dalla console grafica può essere eseguita anche tramite chiamate programmatiche, consentendo agli sviluppatori di automatizzare infrastrutture, distribuire applicazioni e integrare servizi cloud direttamente nel proprio software.

## Le API come fondamento di AWS

AWS è costruita su un modello API-first. I servizi espongono endpoint che consentono di creare, leggere, modificare ed eliminare risorse mediante richieste HTTP sicure. La console AWS stessa utilizza queste API dietro le quinte.

Questo approccio consente la completa automazione dell'infrastruttura e rappresenta il fondamento dei moderni processi DevOps e Infrastructure as Code.

## Le modalità di accesso ai servizi AWS

Gli sviluppatori possono interagire con AWS attraverso tre modalità principali: la AWS Management Console, la AWS Command Line Interface (CLI) e gli AWS Software Development Kit (SDK).

La console è particolarmente utile nelle fasi esplorative e amministrative. La CLI permette di eseguire comandi automatizzabili tramite script. Gli SDK consentono invece alle applicazioni di comunicare direttamente con i servizi AWS utilizzando linguaggi come Python, Java, JavaScript, .NET e Go.

## AWS SDK

Gli SDK astraggono la complessità delle chiamate HTTP, della firma delle richieste e della gestione delle risposte. Lo sviluppatore può concentrarsi sulle logiche di business utilizzando librerie native del linguaggio scelto.

Ad esempio, utilizzando Boto3 per Python è possibile caricare un oggetto su Amazon S3 con poche righe di codice, senza gestire direttamente autenticazione e protocolli di basso livello.

## AWS CLI

La AWS CLI rappresenta uno strumento essenziale per amministratori e sviluppatori. Consente di eseguire operazioni da terminale, integrarle nelle pipeline CI/CD e automatizzare procedure ripetitive.

Comandi come la creazione di bucket S3, la distribuzione di stack CloudFormation o l'invocazione di funzioni Lambda possono essere incorporati in script e processi di deployment.

## Autenticazione delle richieste

Ogni richiesta inviata ai servizi AWS deve essere autenticata. A tale scopo viene utilizzato il meccanismo Signature Version 4, che garantisce integrità e autenticità delle chiamate API.

Le credenziali possono essere recuperate tramite utenti IAM, ruoli temporanei o profili configurati localmente. Le best practice raccomandano l'utilizzo di credenziali temporanee e del principio del minimo privilegio.

## Gestione delle operazioni asincrone

Molti servizi AWS eseguono operazioni che richiedono tempo. Per gestire tali situazioni gli SDK implementano meccanismi di polling e waiter che monitorano automaticamente lo stato delle risorse fino al completamento dell'operazione.

Questo approccio semplifica il codice applicativo e migliora l'affidabilità delle automazioni.

## Ambienti di sviluppo nel cloud

AWS mette a disposizione ambienti di sviluppo integrati che permettono di scrivere, testare e distribuire applicazioni direttamente nel cloud. Questi ambienti eliminano molti problemi legati alla configurazione locale e favoriscono la collaborazione tra team distribuiti.

## Logging e monitoraggio

Le applicazioni moderne devono essere osservabili. Amazon CloudWatch raccoglie log e metriche provenienti dal codice e dall'infrastruttura. AWS X-Ray consente invece di analizzare il percorso delle richieste in sistemi distribuiti.

La combinazione di questi strumenti permette di identificare rapidamente colli di bottiglia, errori e problemi prestazionali.

## Automazione del ciclo di sviluppo

L'automazione rappresenta uno dei pilastri dello sviluppo cloud-native. Build, test, deployment e monitoraggio possono essere integrati in pipeline automatiche che riducono gli errori manuali e accelerano il rilascio delle nuove funzionalità.

## Best Practice per gli sviluppatori AWS

Gli sviluppatori dovrebbero preferire l'automazione alle operazioni manuali, adottare Infrastructure as Code, utilizzare ruoli IAM anziché credenziali statiche, implementare logging fin dalle prime fasi del progetto e monitorare continuamente costi e prestazioni.

## Conclusioni

La conoscenza delle API AWS, degli SDK, della CLI e degli strumenti di osservabilità costituisce la base operativa per qualsiasi progetto cloud-native. Questi strumenti consentono di costruire applicazioni scalabili, automatizzate e facilmente manutenibili, preparando il terreno per i servizi avanzati che verranno trattati nei capitoli successivi.
