# Capitolo 11 – Gestire l'accesso degli utenti dell'applicazione

## 11.1 Obiettivi del capitolo

Nel Capitolo 3 IAM ha risolto il problema di chi può accedere alle **risorse AWS**. Un'applicazione come Polly Notes ha però un problema diverso: deve gestire i propri **utenti finali**, che si registrano, effettuano il login e possono vedere e modificare solo le proprie note. Questo capitolo è dedicato ad **Amazon Cognito**, il servizio AWS che si occupa di questo.

Si esplora il **processo di autenticazione** con Cognito e si impara a **gestire l'accesso degli utenti** e ad **autorizzare le API serverless**, con un laboratorio finale. Si vedono le **best practice** di implementazione e la struttura dei **JSON Web Token (JWT)**, il formato dei token che Cognito rilascia.

## 11.2 La complessità di autenticazione e autorizzazione

Conviene prima fissare i due concetti, già incontrati con IAM ma qui riferiti agli utenti dell'applicazione.

L'**autenticazione** verifica che un utente sia **chi dichiara di essere**. Per Polly Notes significa che gli utenti devono poter accedere con nome utente e password, eventualmente con l'autenticazione a più fattori (**MFA**), così da poterli identificare e verificare prima di concedere l'accesso.

L'**autorizzazione** verifica se un utente già autenticato **può eseguire l'azione** che sta tentando. In un'applicazione esistono di solito diversi insiemi di permessi, e tipi di utenti diversi possono fare cose diverse.

Implementare in proprio autenticazione e autorizzazione è impegnativo. Lo sviluppatore deve occuparsi di molte cose:

- **memorizzare le identità** degli utenti, tipicamente in un database, in modo sicuro;
- gestire la **registrazione** (*sign-up*) e il **recupero della password**;
- offrire eventualmente il **single sign-on** o la **federazione** con identità di terze parti, come Google o Amazon, tramite OAuth;
- scegliere e implementare correttamente i **protocolli e i formati** standard: **OAuth 2.0**, **SAML**, **OpenID Connect (OIDC)**, **JWT**, e gestirli nel codice.

Sono responsabilità complesse e delicate dal punto di vista della sicurezza. **Amazon Cognito** se ne fa carico, aiutando sia nell'**autenticazione** (chi sono gli utenti, come accedono, come si registrano, quali attributi li descrivono) sia nell'**autorizzazione** (che cosa possono fare una volta entrati). Lo fa con due componenti, i **user pool** e gli **identity pool**.

## 11.3 Amazon Cognito

### Che cosa offre Cognito

Amazon Cognito è prima di tutto un **identity store** gestito: un luogo in cui conservare nomi utente, password e dati personali degli utenti di applicazioni web e mobile. La memorizzazione e la sua sicurezza sono gestite da AWS. Molti sviluppatori non amano scrivere codice di autenticazione e gestire i dati degli utenti, e Cognito li solleva da questo compito.

Cognito offre anche una **Hosted UI**: le pagine di login, di registrazione, di recupero password e così via sono ospitate da AWS e si integrano nell'applicazione senza doverle scrivere. È particolarmente utile per i team piccoli e per le applicazioni proof of concept, quando non si vuole investire tempo in queste funzionalità.

Cognito permette inoltre di **controllare l'accesso alle risorse AWS**, una funzione utile in particolare con le **identità federate**. Infine è conforme a diversi **programmi di compliance**: conviene comunque verificare nella documentazione che soddisfi i requisiti specifici del proprio progetto.

### User pool e identity pool

I due componenti di Cognito rispondono a domande diverse.

Un **user pool** risponde alla domanda *"chi è l'utente?"*. Fornisce l'**autenticazione**: registrazione e login, memorizzazione degli utenti, Hosted UI. Serve per far accedere gli utenti all'**applicazione** e non ha necessariamente a che fare con le credenziali AWS.

Un **identity pool** risponde alla domanda *"l'utente deve poter chiamare direttamente le API AWS?"*. Se la risposta è sì, l'identity pool fornisce all'utente **credenziali AWS temporanee**. Dietro le quinte si integra con **IAM** e **AWS STS**. È utile anche con le identità federate: un utente che ha effettuato il login con Facebook o Google può ottenere credenziali AWS. Può persino fornire credenziali a **utenti non autenticati** (*guest*), con permessi limitati.

I due componenti si possono usare **separatamente** oppure **insieme**. Usarli insieme è un modo molto efficace di far accedere gli utenti all'applicazione e poi dare loro, quando serve, credenziali per eseguire operazioni su AWS.

### L'integrazione con API Gateway

Cognito si integra con **Amazon API Gateway**. Si registrano gli utenti in un user pool e si crea un **app client**, cioè la configurazione che rappresenta l'applicazione presso il user pool. Il user pool si imposta poi come **authorizer** dell'API: gli utenti effettuano il login con Cognito, i loro token vengono inviati ad API Gateway, e API Gateway, grazie alla relazione con Cognito, verifica che l'utente possa accedere alle risorse di backend.

In alternativa si possono usare i **Lambda authorizer**, che contengono codice di autorizzazione personalizzato (paragrafo 11.7).

## 11.4 Gestire l'accesso degli utenti

I paragrafi seguenti si concentrano sui **user pool**, che gestiscono sia la **registrazione** sia il **login**. Alla creazione di un user pool si definiscono processi e attributi degli utenti.

**Verifica dei contatti.** È molto comune voler verificare l'**email** o il **numero di telefono** forniti in fase di registrazione prima di attivare l'account. Se l'utente dovrà recuperare o cambiare la password, si sa che potrà ricevere il codice.

**MFA.** Si decide se l'autenticazione a più fattori è **obbligatoria** per tutti, **facoltativa** a scelta dell'utente, o disattivata, e con quali metodi: app di autenticazione TOTP o SMS.

**Password policy.** Si stabiliscono i requisiti delle password: lunghezza minima, per esempio otto caratteri, e presenza di maiuscole, numeri e caratteri speciali.

**Authentication flow.** Si stabiliscono i flussi di autenticazione consentiti, a seconda del caso d'uso: per esempio il protocollo SRP per le applicazioni client o i flussi amministrativi per i backend.

**Attributi.** Ogni utente ha degli attributi, e si decide quali sono **obbligatori** e quali facoltativi. Esistono **attributi standard**, come email, nome, numero di telefono e username, e **attributi personalizzati**, come un nickname o un'immagine del profilo, per adattare il user pool all'applicazione.

**Gruppi.** Gli utenti si possono organizzare in **gruppi** che rappresentano tipi diversi di utenti. In un'applicazione di blog ci sono i **lettori**, che leggono e commentano gli articoli, e gli **amministratori**, che moderano i commenti e pubblicano nuovi contenuti. Un utente può appartenere a **più gruppi**, e a un gruppo si può associare un **ruolo IAM**.

**Scope.** Gli *scope* OAuth stabiliscono il **livello di accesso alle risorse** concesso a un'applicazione. Si possono definire scope personalizzati per le proprie API.

## 11.5 Concedere l'accesso con i User Pool

### La struttura di un JSON Web Token

I token rilasciati da Cognito sono in formato **JSON Web Token (JWT)**. Un JWT è composto da **tre sezioni**, codificate in Base64URL e separate da un punto:

```
eyJraWQiOiJhYmMxMjMiLCJhbGciOiJSUzI1NiJ9 . eyJzdWIiOiI5ZjE...In0 . dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk
└────────────── header ──────────────┘   └── payload ──┘   └──────────────── signature ────────────────┘
```

L'**header** contiene il **key id** (`kid`), cioè l'identificativo della chiave usata per firmare, e l'**algoritmo** di firma (`alg`):

```json
{ "kid": "abc123...", "alg": "RS256" }
```

Il **payload** contiene le informazioni codificate sull'utente e sulla sessione, i cosiddetti **claim**:

```json
{
  "sub": "9f1c2a7e-...",
  "cognito:username": "morgan",
  "given_name": "Morgan",
  "email": "morgan@example.com",
  "auth_time": 1664620800,
  "exp": 1664624400,
  "iss": "https://cognito-idp.us-east-1.amazonaws.com/us-east-1_AbCdEf123",
  "token_use": "id"
}
```

La **signature** è calcolata a partire da header e payload con la chiave privata di Cognito. Chi riceve il token la verifica con la chiave pubblica corrispondente, che Cognito pubblica in un endpoint JWKS, e ottiene così la certezza che il token sia autentico e non sia stato alterato.

Il payload è solo **codificato**, non cifrato: chiunque possieda il token può leggerne il contenuto, per esempio incollandolo in un decodificatore JWT. Nei token non vanno quindi inseriti dati riservati. La firma garantisce l'**integrità**, non la riservatezza.

### Il flusso di autenticazione

Il flusso con un user pool si svolge in tre passi:

1. l'utente fornisce le proprie credenziali e si **autentica presso il user pool** tramite l'applicazione;
2. il user pool restituisce all'applicazione **tre token JWT**: **ID token**, **access token** e **refresh token**;
3. l'applicazione usa i token per **accedere alle risorse**, tipicamente le API di backend.

I tre token hanno ruoli diversi:

| Token | Contenuto | Uso |
|---|---|---|
| **ID token** | Informazioni sull'**identità** dell'utente: nome, email e altri attributi | Personalizzare l'applicazione, identificare l'utente |
| **Access token** | Informazioni su **che cosa l'utente può fare** (scope, gruppi), **senza** i dettagli dell'identità | Autorizzare le chiamate alle API |
| **Refresh token** | Credenziale a lunga durata | Ottenere nuovi ID e access token alla loro scadenza, senza ripetere il login |

> **Box – I grant OAuth 2.0 e la sicurezza dei token**
>
> Quando si usa la Hosted UI, Cognito supporta diversi **grant** OAuth 2.0.
>
> - Con l'**Authorization code grant** il user pool, dopo il login, non restituisce direttamente i token al browser ma un **codice di autorizzazione**. L'applicazione lo scambia con i token tramite una chiamata server-to-server o con PKCE. È il metodo **raccomandato** e quello da scegliere quando i token del user pool **non devono mai essere esposti direttamente all'utente finale**.
> - Con l'**Implicit grant** i token vengono restituiti direttamente nell'URL. È più semplice ma meno sicuro e oggi sconsigliato.
> - Il **Client credentials grant** è pensato per la comunicazione **machine-to-machine**, senza un utente coinvolto.

## 11.6 Concedere l'accesso con gli Identity Pool

Un **identity pool** funziona in modo diverso da un user pool. Nell'identity pool si definiscono dei **ruoli IAM** e li si associa agli **identity provider**: Facebook, Google, Login with Amazon, Apple, provider OIDC o SAML e lo stesso user pool di Cognito. Il flusso è questo:

1. l'utente si **autentica presso l'identity provider**, dimostrando così la propria identità;
2. l'applicazione invia a Cognito il **token di autenticazione** ricevuto dal provider;
3. l'identity pool riconosce il provider di provenienza e associa all'identità il **ruolo** configurato;
4. l'identity pool restituisce all'applicazione **credenziali AWS temporanee**, ottenute tramite STS, con i permessi definiti nel ruolo;
5. l'applicazione usa le credenziali per chiamare **direttamente le API AWS**, per esempio per caricare un file in S3 o scrivere in DynamoDB.

L'identity pool **non conserva alcuna informazione** sul login dell'utente, né username né password. Questi dati restano presso l'identity provider. È quindi un modo per distribuire credenziali AWS senza doversi occupare di memorizzare i dati degli utenti, come invece avviene con un user pool o con una soluzione fatta in casa.

> **Box – User pool e identity pool insieme: il flusso completo**
>
> Uno scenario d'esame tipico chiede di permettere a tutti gli utenti autenticati tramite un social provider, per esempio Twitter, di inviare messaggi a uno stream Kinesis. La soluzione combina i due componenti.
>
> 1. L'utente si autentica con il provider e l'applicazione invia il token OpenID al **user pool** di Cognito, configurato con quel provider federato.
> 2. Il user pool restituisce i propri **token JWT**.
> 3. L'applicazione invia il token del user pool all'**identity pool**.
> 4. L'identity pool assume il **ruolo IAM** associato agli utenti autenticati, che ha la policy che consente `kinesis:PutRecord`, e restituisce **credenziali temporanee**.
>
> L'ordine è sempre **user pool, poi identity pool, poi ruolo IAM**. Inviare il token direttamente a STS per "assumere un utente IAM" non è possibile: gli utenti IAM non si assumono, si assumono solo i ruoli.

## 11.7 Proteggere l'accesso alle API

L'API creata nel Capitolo 9 era pubblica. Per proteggerla con Cognito ci sono due lati da considerare.

Lo **sviluppatore** crea il **user pool**, ne definisce attributi e scope e crea l'**app client**. Poi, in **API Gateway**, sceglie il user pool come **authorizer** dell'API. Da quel momento chiunque voglia accedere all'API deve prima essersi autenticato con Cognito. Con le REST API si usa un *Cognito user pool authorizer*. Con le HTTP API, come nel laboratorio seguente, si usa un *JWT authorizer* configurato con l'issuer del user pool.

L'**utente finale** effettua il login nell'applicazione. L'applicazione può avere una propria pagina di login e usare le **API di Cognito** dietro le quinte, oppure affidarsi alla **Hosted UI**, che fornisce già la pagina. Dopo l'autenticazione l'applicazione riceve il **JWT** e lo inserisce nelle richieste ad API Gateway, di solito nell'intestazione `Authorization`. API Gateway verifica il token e, se è valido, inoltra la richiesta ai servizi di backend. API Gateway funziona quindi da **cancello** tra l'utente e le risorse di backend.

### I Lambda authorizer

Un'alternativa a Cognito è il **Lambda authorizer**, utile quando si dispone già di un proprio sistema di autenticazione. Il flusso è il seguente:

1. l'applicazione invia la richiesta ad API Gateway includendo un **bearer token** nell'intestazione di autorizzazione;
2. API Gateway invoca la **funzione Lambda authorizer**, che applica una logica personalizzata per verificare l'identità del chiamante, per esempio validando un JWT o un token OAuth emesso da un provider esterno;
3. la funzione restituisce un **principal** e una **policy IAM** che consente o nega l'invocazione dell'API;
4. API Gateway **valuta la policy** e permette o nega l'accesso. Il risultato può essere messo in cache per un tempo configurabile, evitando di invocare l'authorizer a ogni richiesta.

> **Box – TOKEN authorizer e REQUEST authorizer**
>
> I Lambda authorizer sono di due tipi.
>
> - Un **TOKEN authorizer** (*token-based*) riceve l'identità del chiamante in un **bearer token**, come un JWT o un token OAuth, contenuto in un'intestazione. È la scelta quando l'identità arriva sotto forma di token.
> - Un **REQUEST authorizer** (*request parameter-based*) riceve l'identità tramite una **combinazione di parametri** della richiesta: intestazioni, query string, stage variable e variabili di contesto.

## 11.8 Laboratorio: Amazon Cognito con una HTTP API

Il laboratorio protegge con Cognito una **HTTP API** già esistente, *Hello World*, con una route `/hello` collegata a una funzione Lambda che risponde "Hello from Lambda". Nella sezione *Authorization* della Console di API Gateway si vede che attualmente la route non ha alcun authorizer. Durante il laboratorio bisogna annotare diversi identificativi (ID del user pool, ID dell'app client, URL), e conviene tenerli in un file di testo da cui copiare.

### Creare il user pool

Nella Console di Cognito si sceglie *Create user pool* e si configurano diverse opzioni:

- **Opzioni di login**: si sceglie l'accesso tramite **username**, lasciando i requisiti predefiniti. La Console avverte che alcune opzioni di login **non si possono modificare dopo la creazione**: vanno decise in anticipo, perché per cambiarle bisognerebbe creare un nuovo user pool.
- **Password policy**: per comodità della dimostrazione si sceglie una policy personalizzata con lunghezza minima 6 e senza requisiti aggiuntivi. In un'applicazione reale si imposterebbe una policy adeguata.
- **MFA**: viene disattivata per la dimostrazione. Le alternative sono MFA **obbligatoria** o **facoltativa**, con app di autenticazione come Authy o Google Authenticator.
- **Account recovery**: si lascia attivo il recupero self-service. Con la Hosted UI questo significa che nella pagina di login comparirà il link per il recupero della password.
- **Self-registration**: resta abilitata. La **verifica** di email e telefono viene invece disattivata per la dimostrazione, ma in produzione andrebbe attivata.
- **Attributi obbligatori**: si lasciano quelli predefiniti.
- **Invio delle email**: si usa il dominio di Cognito. In alternativa si può configurare un **dominio personalizzato tramite Amazon SES**, così che le email agli utenti partano dal proprio dominio.
- **Nome e app client**: il pool si chiama `notes pool`. Si crea un **public client** chiamato `notes client`, **senza client secret**. Tra gli *authentication flows* si abilita **`ALLOW_ADMIN_USER_PASSWORD_AUTH`**, che permette di autenticarsi dalla CLI con le API amministrative.

Creato il pool, si annotano tre dati: lo **User pool ID**, l'**ARN** e, in *App integration → App client list*, il **Client ID** di `notes client`. Gli app client sono le risorse di autenticazione del user pool associate all'applicazione.

### Registrare e confermare un utente

I comandi si eseguono in **AWS CloudShell**:

```bash
aws cognito-idp sign-up \
    --client-id <client-id> \
    --username morgan \
    --password password \
    --user-attributes Name=email,Value=morgan@example.com
```

La risposta indica che l'utente **non è confermato**, dato che la verifica è stata disattivata. Per la dimostrazione lo si conferma manualmente con un'API amministrativa:

```bash
aws cognito-idp admin-confirm-sign-up \
    --user-pool-id <user-pool-id> \
    --username morgan
```

In un'applicazione reale l'utente riceverebbe invece un'email con un codice o un link di conferma.

### Creare l'authorizer

In API Gateway, nella sezione *Authorization* dell'API *Hello World*, si sceglie *Create and attach an authorizer* e si configurano questi valori:

- **Tipo**: **JWT**;
- **Nome**: `Cognito`;
- **Issuer URL**: `https://cognito-idp.us-east-1.amazonaws.com/<user-pool-id>`, dove l'ID del pool ha la forma `us-east-1_XXXXXXXXX`;
- **Audience**: il **Client ID** dell'app client.

Si crea poi uno stage `prod`, si distribuisce l'API e si annota l'**invoke URL**.

### Autenticarsi e chiamare l'API

Si effettua il login dalla CLI:

```bash
aws cognito-idp admin-initiate-auth \
    --user-pool-id <user-pool-id> \
    --client-id <client-id> \
    --auth-flow ADMIN_USER_PASSWORD_AUTH \
    --auth-parameters USERNAME=morgan,PASSWORD=password
```

La risposta contiene un `AuthenticationResult` con **`AccessToken`**, **`RefreshToken`** e **`IdToken`**. Si copia l'access token e si chiama l'API:

```bash
curl -H "Authorization: <access-token>" https://<api-id>.execute-api.us-east-1.amazonaws.com/prod/hello
# Hello from Lambda
```

La risposta è *"Hello from Lambda"*. Ripetendo la stessa chiamata **senza** l'intestazione `Authorization`, la risposta è:

```json
{"message":"Unauthorized"}
```

L'authorizer Cognito fa il suo lavoro: solo le richieste con un token valido raggiungono la funzione.

## 11.9 Verifica dei concetti

**"I user pool di Amazon Cognito scambiano i token di autenticazione con credenziali AWS."** *Falso.* Lo scambio tra token e credenziali AWS temporanee è compito degli **identity pool**. I user pool si occupano dell'autenticazione e rilasciano i token JWT.

**"User pool e identity pool possono essere usati insieme per soluzioni di autenticazione e autorizzazione."** *Vero.* Si ottiene un token dal user pool, lo si passa all'identity pool, che ha un ruolo associato, e lo si usa per ottenere credenziali temporanee.

**"Per definire i permessi dei membri di un gruppo si può assegnare un ruolo IAM a un gruppo di Amazon Cognito."** *Vero.* I gruppi di un user pool possono avere un ruolo IAM associato. Anche l'identity pool definisce ruoli, uno per gli utenti autenticati e uno per quelli non autenticati.

**"La sezione payload di un JSON Web Token contiene informazioni codificate sui claim."** *Vero.* Decodificando il token, per esempio con un decodificatore JWT, i claim sono visibili nel payload.

**"Con la federazione di terze parti gli sviluppatori devono usare gli identity pool."** *Falso.* La federazione con provider esterni, come *Login with Amazon*, si può configurare interamente nei **user pool**. Gli identity pool servono quando l'utente ha bisogno di **credenziali AWS**.

**"Gli identity pool di Amazon Cognito possono fornire credenziali AWS agli utenti non autenticati."** *Vero.* All'identity pool si può associare un ruolo per gli utenti non autenticati, ovviamente con privilegi minimi. Per esempio, un gioco può permettere a chiunque di salvare il proprio punteggio in una tabella DynamoDB senza dover effettuare il login.
