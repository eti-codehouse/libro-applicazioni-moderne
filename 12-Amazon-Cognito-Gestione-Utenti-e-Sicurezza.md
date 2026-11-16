# Capitolo 12
# Amazon Cognito, Gestione degli Utenti e Sicurezza delle Applicazioni

## Introduzione

La gestione delle identità rappresenta uno degli aspetti più complessi nello sviluppo di applicazioni moderne. Oltre a realizzare le funzionalità di business, gli sviluppatori devono gestire registrazione degli utenti, autenticazione, recupero password, multi-factor authentication, federazione delle identità e protezione delle API.

Amazon Cognito è il servizio AWS progettato per affrontare queste necessità fornendo una piattaforma completa per autenticazione, autorizzazione e gestione dell'identità degli utenti applicativi.

---

## Autenticazione e Autorizzazione

Sebbene vengano spesso utilizzati come sinonimi, autenticazione e autorizzazione rappresentano concetti differenti.

L'autenticazione risponde alla domanda: chi sei?

L'autorizzazione risponde invece alla domanda: cosa puoi fare?

Un utente può autenticarsi con successo mediante username, password e autenticazione multifattore, ma ciò non implica automaticamente la possibilità di accedere a tutte le funzionalità disponibili.

Nelle applicazioni enterprise questi due livelli devono essere progettati separatamente.

---

## Le sfide della gestione delle identità

Gestire utenti all'interno di un'applicazione richiede numerose funzionalità: registrazione, login, recupero password, conferma dell'identità, gestione dei profili, integrazione con provider esterni e controllo degli accessi.

Lo sviluppo manuale di tali componenti aumenta tempi, costi e rischi di sicurezza.

Amazon Cognito nasce per eliminare questa complessità fornendo servizi gestiti e scalabili.

---

## Cos'è Amazon Cognito

Amazon Cognito è un servizio di Customer Identity and Access Management completamente gestito.

Consente di archiviare identità utente, autenticare accessi, integrare provider federati e fornire credenziali temporanee per l'accesso alle risorse AWS.

Può essere utilizzato da applicazioni web, mobile e serverless.

---

## Hosted UI

Una delle funzionalità più interessanti di Cognito è la Hosted UI.

AWS può fornire direttamente le pagine di login, registrazione, conferma account e recupero password.

Questo approccio è particolarmente utile per startup, proof of concept e piccoli team che desiderano accelerare il rilascio delle applicazioni.

---

## User Pool

Il componente principale di Cognito è il User Pool.

Un User Pool rappresenta un archivio di identità che memorizza utenti, password, attributi e configurazioni di autenticazione.

Attraverso il User Pool è possibile gestire:

- registrazione;
- autenticazione;
- recupero password;
- verifica email;
- verifica numero telefonico;
- MFA;
- gruppi di utenti.

Il User Pool si occupa esclusivamente della gestione delle identità applicative.

---

## Configurazione della registrazione

Durante la creazione di un User Pool è possibile definire quali attributi sono obbligatori.

Gli attributi standard comprendono:

- username;
- email;
- nome;
- numero telefonico.

È inoltre possibile definire attributi personalizzati come avatar, nickname o classificazioni specifiche del dominio applicativo.

---

## Password Policy e MFA

La sicurezza delle identità dipende fortemente dalle politiche di autenticazione.

Cognito permette di definire requisiti minimi di lunghezza, complessità e scadenza delle password.

È inoltre possibile attivare il Multi-Factor Authentication obbligatorio oppure opzionale.

L'MFA rappresenta una delle principali difese contro il furto di credenziali.

---

## Verifica degli utenti

Una pratica comune consiste nel richiedere la verifica dell'indirizzo email o del numero telefonico.

La verifica garantisce che l'utente possa recuperare l'accesso in caso di smarrimento delle credenziali e riduce il numero di account fraudolenti.

---

## Gruppi e Ruoli Applicativi

Gli utenti possono essere organizzati in gruppi.

I gruppi permettono di rappresentare differenti categorie applicative come amministratori, moderatori, operatori o utenti finali.

Questa organizzazione semplifica l'implementazione delle autorizzazioni.

---

## Scope e Permessi

Cognito supporta il concetto di scope.

Gli scope consentono di definire quali funzionalità e quali risorse risultano accessibili a una determinata applicazione o categoria di utenti.

Questo approccio favorisce una gestione granulare degli accessi.

---

## Identity Pool

Accanto ai User Pool troviamo gli Identity Pool.

Mentre il User Pool autentica l'utente, l'Identity Pool consente di ottenere credenziali AWS temporanee.

L'obiettivo consiste nel permettere agli utenti dell'applicazione di interagire direttamente con determinati servizi AWS.

---

## Integrazione con AWS STS

Gli Identity Pool si integrano con AWS Security Token Service.

Quando un'identità viene riconosciuta, Cognito associa uno specifico ruolo IAM e richiede credenziali temporanee tramite STS.

L'applicazione riceve quindi credenziali a durata limitata che possono essere utilizzate per accedere alle API AWS.

---

## Federazione delle identità

Molte applicazioni consentono di accedere mediante provider esterni.

Amazon Cognito supporta la federazione con piattaforme quali Google, Facebook, Amazon e altri provider compatibili con OAuth, OpenID Connect e SAML.

Grazie alla federazione, gli utenti possono autenticarsi senza creare nuove credenziali specifiche per l'applicazione.

---

## Utenti autenticati e anonimi

Gli Identity Pool consentono di distinguere utenti autenticati e utenti anonimi.

Anche un utente non autenticato può ricevere credenziali temporanee fortemente limitate.

Questa funzionalità risulta utile per giochi, applicazioni pubbliche e sistemi che richiedono accesso preliminare senza registrazione.

---

## JSON Web Token (JWT)

Amazon Cognito utilizza il formato JWT per rappresentare le informazioni di autenticazione.

Un token JWT è composto da tre sezioni:

- Header;
- Payload;
- Signature.

L'Header descrive l'algoritmo utilizzato.

Il Payload contiene le informazioni sull'identità e sulle autorizzazioni.

La Signature garantisce integrità e autenticità del contenuto.

---

## Tipologie di Token

Durante l'autenticazione Cognito restituisce generalmente tre token differenti.

### ID Token

Contiene informazioni sull'identità dell'utente.

### Access Token

Definisce autorizzazioni e privilegi disponibili.

### Refresh Token

Permette di ottenere nuovi token senza richiedere una nuova autenticazione.

---

## Flusso di autenticazione

Il processo tipico prevede che l'utente effettui il login presso il User Pool.

Dopo la verifica delle credenziali, Cognito restituisce i token JWT.

L'applicazione utilizza tali token per accedere ai servizi backend e alle API protette.

---

## Protezione delle API con Cognito

Amazon API Gateway può utilizzare Cognito come Authorizer.

Quando una richiesta raggiunge l'endpoint API, API Gateway verifica il token JWT ricevuto.

Se il token risulta valido e autorizzato, la richiesta viene inoltrata ai servizi backend.

In caso contrario l'accesso viene negato.

---

## Cognito Authorizer e Lambda Authorizer

Le API AWS supportano differenti modelli di autorizzazione.

Il Cognito Authorizer delega completamente il controllo dell'identità a Cognito.

Il Lambda Authorizer consente invece di implementare logiche personalizzate mediante una funzione Lambda.

Questa seconda opzione risulta utile quando l'organizzazione possiede sistemi di autenticazione proprietari.

---

## Caso di studio: Polly Notes

Nell'applicazione Polly Notes gli utenti si autenticano attraverso un User Pool Cognito.

Dopo il login ricevono token JWT che vengono utilizzati per accedere alle API esposte tramite Amazon API Gateway.

API Gateway verifica il token e autorizza l'accesso alle funzioni Lambda responsabili della gestione delle note.

Questa architettura elimina la necessità di implementare manualmente meccanismi di autenticazione e autorizzazione.

---

## Best Practice

Le organizzazioni dovrebbero abilitare MFA quando possibile, verificare email e numeri telefonici, utilizzare token a breve durata, applicare il principio del minimo privilegio ai ruoli IAM associati agli Identity Pool e monitorare regolarmente le attività di autenticazione.

È inoltre consigliabile utilizzare provider federati quando ciò può semplificare l'esperienza utente e migliorare la sicurezza.

---

## Conclusioni

Amazon Cognito rappresenta una soluzione completa per la gestione delle identità nelle applicazioni cloud-native. Grazie a User Pool, Identity Pool, federazione delle identità, JWT e integrazione con API Gateway, consente di implementare rapidamente sistemi di autenticazione e autorizzazione robusti, sicuri e altamente scalabili.
