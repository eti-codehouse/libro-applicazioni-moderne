# Capitolo 3 – Gestire i permessi

## 3.1 Obiettivi del capitolo

Nel capitolo precedente si è visto che ogni interazione con AWS è una chiamata API firmata con delle credenziali. Il passo naturale successivo è capire **chi** può fare **che cosa**: è il compito di **AWS Identity and Access Management (IAM)**. Il capitolo ripercorre le funzionalità e i componenti di IAM e il vocabolario che ogni sviluppatore deve padroneggiare: user, group, role, policy, principal. Spiega poi come configurare le credenziali nell'ambiente di sviluppo e mostra in un laboratorio come scrivere permessi IAM corretti per un programma, ricavandoli dai log dell'SDK. Infine descrive come gli SDK cercano e usano le credenziali AWS, con particolare attenzione all'SDK for Java.

## 3.2 AWS Identity and Access Management (IAM)

### Che cosa gestisce IAM e che cosa no

Quando si parla di gestione delle identità si pensa alle credenziali, ma il perimetro di IAM va chiarito subito. IAM **controlla l'accesso alle risorse AWS**. **Non** gestisce le credenziali applicative, le API key dell'applicazione o il login degli utenti finali dell'app, che sono temi del Capitolo 11 su Amazon Cognito. Il suo campo è il controllo granulare di chi può eseguire quali chiamate API in un account AWS, e in AWS ogni operazione è una chiamata API.

IAM copre due aspetti distinti. L'**autenticazione** risponde alla domanda *"chi sei?"* e si basa su username e password o su un'altra forma di identità verificabile. L'**autorizzazione** risponde alla domanda *"che cosa puoi fare?"*. Una volta dimostrata l'identità, entra in gioco un insieme di permessi che stabilisce quali chiamate API si possono inviare a quali risorse.

### Il vocabolario di IAM

**User.** Un utente IAM rappresenta un'identità con **credenziali statiche**: username e password per la Console e, opzionalmente, una o più **access key**. Per interagire con AWS in modo programmatico, tramite SDK o CLI, servono un *access key ID* e una *secret access key*. Con questa coppia le richieste vengono firmate, dimostrando ad AWS l'identità del chiamante.

**User group.** È un raggruppamento logico di utenti, come amministratori, sviluppatori o DevOps. Ogni gruppo ha il proprio insieme di permessi e gli utenti che ne fanno parte li ereditano.

**Permission e policy.** Un permesso è uno *statement* in formato **JSON** che stabilisce quali chiamate API consentire (*Allow*) o negare (*Deny*). Un insieme di statement forma una **policy**. Le policy si salvano e si applicano ai gruppi, perché tutti i membri ne ereditino i permessi, agli utenti direttamente oppure ai ruoli.

**Role.** Un ruolo IAM fornisce **credenziali temporanee**. A differenza dell'utente non ha username e password statici: un ruolo va **assunto** in modo programmatico, e a quel punto si ottengono credenziali valide per un tempo limitato. I ruoli si usano in tre scenari tipici:

- un **utente** assume un ruolo per ottenere permessi che non gli servono tutti i giorni;
- un **servizio AWS** assume un ruolo (*service role*) per poter chiamare un altro servizio;
- un'**istanza EC2** riceve un ruolo, così le applicazioni che vi girano possono chiamare le API AWS in modo autenticato senza chiavi salvate sulla macchina.

### Principal, action, resource: come avviene la valutazione

Utenti, applicazioni, servizi AWS e ruoli sono tutti esempi di **principal**: l'entità che effettua la richiesta. Dopo l'autenticazione, la richiesta contiene tra l'altro l'**action**, cioè la specifica chiamata API che si vuole eseguire, e la **resource** su cui la si vuole eseguire. AWS verifica poi se il principal è autorizzato, confrontando la richiesta con le policy applicabili.

Esistono due grandi famiglie di policy:

- Le **identity-based policy** sono attaccate a un'identità: utente, gruppo o ruolo. La stessa policy può essere attaccata a molti principal diversi, in una relazione uno-a-molti.
- Le **resource-based policy** sono attaccate a una **risorsa** AWS. L'esempio più noto è la **bucket policy** di S3, che stabilisce quali principal possono interagire con il bucket e in che modo. Molte altre risorse AWS supportano policy di questo tipo.

Le due famiglie **lavorano insieme**: per stabilire se un'azione è permessa si valutano tutte le policy pertinenti. A queste si aggiungono altri tipi di policy, come le **Service Control Policy (SCP)** di AWS Organizations. Un caso d'uso molto comune delle resource-based policy è l'**accesso cross-account**: un bucket nell'account B può concedere l'accesso a un principal dell'account A direttamente nella propria policy. Al termine della valutazione la chiamata, per esempio un `CreateBucket` o un `DeleteBucket`, viene consentita, e la risorsa viene modificata, oppure negata.

### Esempio: una identity-based policy

La policy seguente è attaccata a un gruppo e quindi vale per tutti i suoi utenti:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::notes-bucket"
    },
    {
      "Effect": "Allow",
      "Action": "s3:*Object",
      "Resource": "arn:aws:s3:::notes-bucket/*"
    }
  ]
}
```

Il primo statement consente di elencare il contenuto del bucket `notes-bucket`. Il secondo usa il carattere jolly `*` due volte. Nell'azione, `s3:*Object` consente tutte le API il cui nome termina con *Object*, come `GetObject`, `PutObject` e `DeleteObject`. Nella risorsa, `notes-bucket/*` indica tutti gli oggetti contenuti nel bucket. Nella policy **non compare alcun principal**: il principal è implicito, ed è l'identità a cui la policy è attaccata.

### Esempio: una resource-based policy

Lo stesso bucket può avere a sua volta una bucket policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::notes-bucket",
        "arn:aws:s3:::notes-bucket/*"
      ],
      "Condition": {
        "NotIpAddress": { "aws:SourceIp": "203.0.113.0/24" }
      }
    }
  ]
}
```

Qui il **principal è esplicito**: in una resource-based policy bisogna dire *chi* può o non può interagire con la risorsa. Lo statement, a prima vista, nega a chiunque (`"Principal": "*"`) qualsiasi azione S3 sul bucket e sul suo contenuto. Il punto interessante è la **Condition**: il divieto si applica solo se la richiesta **non** proviene dall'intervallo di indirizzi IP indicato. In pratica il bucket è utilizzabile solo da una rete specifica. Le condizioni permettono di stabilire con precisione in quali circostanze una chiamata API è consentita o negata. Esistono moltissime chiavi di condizione, e la documentazione IAM è il riferimento da consultare: le policy possono sembrare complesse all'inizio, ma diventano chiare una volta compreso il significato delle parole chiave.

### Permissions boundary

Un **permissions boundary** definisce l'**insieme massimo di permessi** che un principal può avere, indipendentemente da quanto gli concedono le sue policy.

Consideriamo il gruppo *Developer*, a cui è attaccata la policy gestita **PowerUserAccess**. Martha fa parte del gruppo ed eredita un accesso molto ampio alle risorse dell'applicazione di cui è responsabile, compreso il bucket S3 che ospita il sito di Polly Notes. Arriva un nuovo collega, Mateo, che entra nello stesso gruppo. Per lui viene impostato un permissions boundary che consente soltanto `cloudwatch:*` su tutte le risorse. Il risultato è che Mateo accede a CloudWatch ma **non** a S3, pur essendo membro di un gruppo PowerUserAccess. I permessi effettivi sono infatti l'**intersezione** tra ciò che concedono le policy e ciò che consente il boundary. I permissions boundary sono quindi una protezione per stabilire il limite massimo di permessi per un utente, un gruppo o un ruolo.

### Gli utenti IAM non sono sempre necessari

Gli utenti IAM hanno credenziali statiche, e le credenziali statiche vanno **ruotate** periodicamente. Gestire e ruotare centinaia di access key è un onere e un rischio. Per questo spesso conviene affidarsi a **credenziali temporanee**, fornite dai **ruoli IAM** e rinnovate automaticamente.

I ruoli rendono possibili diversi scenari:

- **Federazione e single sign-on**: le identità di una directory aziendale possono accedere all'account AWS. Chi effettua il login sul portatile con le credenziali aziendali ha già dimostrato la propria identità alla directory. Grazie a una **relazione di trust** tra AWS e la directory, può accedere all'account con un single sign-on.
- **Applicazioni senza chiavi nel codice**: un'applicazione può usare un ruolo invece di access key incorporate nel sorgente.
- **Privilegi elevati temporanei**: un gruppo di sviluppatori (Mary, Diego, Mateo e Martha) non ha normalmente accesso a DynamoDB. Quando serve un'operazione di amministrazione del database, uno di loro assume temporaneamente un ruolo *database admin*, dalla Console o in modo programmatico.
- **Accesso cross-account**: gli sviluppatori lavorano in un account di sviluppo e non dovrebbero avere un accesso permanente all'account di produzione. Nell'account di produzione si crea un ruolo, legato da una relazione di trust all'account di sviluppo, che gli sviluppatori assumono solo quando devono intervenire.
- **Service role**: una funzione Lambda scrive in una tabella DynamoDB dello stesso account. Il fatto che le due risorse stiano nello stesso account **non** le autorizza a comunicare. Serve un **ruolo di servizio**, assegnato alla funzione, che le dia le credenziali temporanee per chiamare l'API di DynamoDB. In generale, quando un servizio AWS ne chiama un altro, dietro le quinte c'è quasi sempre un ruolo assunto.

### La logica di valutazione

Per default in IAM **tutto è negato**: è il cosiddetto **implicit deny**. Se nessuna policy consente esplicitamente un'azione, l'azione è negata. Una versione semplificata della logica di valutazione procede così:

1. Si esaminano **tutte** le policy applicabili insieme (identity-based, resource-based e le altre). Se c'è un **explicit deny**, la richiesta è negata: **un deny vince sempre su un allow**.
2. Se non ci sono deny espliciti, si cerca un **explicit allow**. Se c'è, la richiesta è consentita.
3. Se non c'è nemmeno un allow, si applica l'**implicit deny** e la richiesta è negata.

### Best practice

Due regole guidano un uso corretto di IAM.

La prima è **applicare le policy ai gruppi anziché ai singoli utenti**. Se un utente passa dal gruppo di sviluppo a quello di test, basta spostarlo e i suoi permessi cambiano di conseguenza, senza doverli rivedere a mano uno per uno.

La seconda è il **principio del privilegio minimo** (*principle of least privilege*): concedere a ogni principal solo i permessi di cui ha bisogno, e nulla di più. Se un lavoro richiede quattro chiamate API, la policy deve elencare quelle quattro. Un'azione come `s3:*` è molto permissiva, perché concede tutte le API di S3, e va evitata quando ne servono solo alcune.

> **Box – Il principal: dove compare e dove no**
>
> Una distinzione che ricorre spesso negli esami riguarda l'elemento `Principal`. Nelle **identity-based policy** non compare, perché il principal è l'identità a cui la policy è attaccata. Nelle **resource-based policy** è obbligatorio, perché la risorsa deve dichiarare chi può accedervi. Una bucket policy con `"Principal": "*"` concede o nega l'accesso a chiunque, anche agli utenti anonimi.

## 3.3 Laboratorio: applicare il principio del privilegio minimo

Dire che bisogna concedere solo i permessi necessari è facile. Più difficile è sapere **quali** permessi servono davvero a un programma. Questo laboratorio mostra un metodo pratico in tre passi: scoprire le API usate dal codice tramite il **logging dell'SDK**, scrivere una policy IAM che consenta solo quelle e verificarla eseguendo il codice con un profilo che ha soltanto quella policy.

Il codice di partenza è un'applicazione Java che lavora su DynamoDB tramite il **DynamoDB Enhanced Client** (Capitolo 7). Legge una nota, ne aggiunge una nuova e mostra le note preferite. Alla prima esecuzione si usa un profilo con permessi volutamente ampi, così il programma funziona. È proprio la situazione da correggere.

**Passo 1: scoprire le API.** Nel file di configurazione di **Log4j** si attiva il livello `DEBUG` per il logging delle richieste dell'SDK. Rieseguendo il programma, l'output si arricchisce di dettagli su ogni richiesta. Ogni richiesta DynamoDB contiene l'intestazione **`X-Amz-Target`**, che indica l'operazione invocata. Cercandola nei log emergono tre API: `Query`, `PutItem` e `GetItem`. Il concetto vale per qualunque linguaggio, non solo per Java.

**Passo 2: verificare il problema.** Si cambia il profilo nel codice con `profile2`, che corrisponde a un utente IAM privo di permessi, e si disattiva il logging dettagliato. All'esecuzione compare uno *stack trace*: la richiesta `Query` fallisce con status **400**, perché l'utente non è autorizzato.

**Passo 3: scrivere la policy.** Nella Console IAM si apre *Policies → Create policy* e si sceglie il servizio **DynamoDB**. Tra le azioni si selezionano solo `GetItem`, `PutItem` e `Query`. Per la risorsa si lascia l'opzione *Specific* invece di *All resources*, e si aggiunge l'**ARN della tabella**, copiato dalla Console DynamoDB (*General information → Additional info*). Alla policy si dà un nome descrittivo, come `dynamodb-put-get-query`. Il JSON risultante è:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:Query"
      ],
      "Resource": "arn:aws:dynamodb:us-east-1:123456789012:table/Notes"
    }
  ]
}
```

**Passo 4: attaccare la policy e verificare.** In *Users* si seleziona l'utente `profile2` e si usa *Add permissions → Attach existing policies directly* per attaccare la policy appena creata. Rieseguendo il programma, la lettura, l'inserimento e la query vanno tutti a buon fine. La policy contiene esattamente i permessi necessari: né più né meno.

> **Box – Un metodo riutilizzabile**
>
> Il procedimento "log dell'SDK, poi elenco delle API, poi policy minima" si applica a qualsiasi servizio. In alternativa o in aggiunta, AWS offre strumenti come **IAM Access Analyzer**, che può generare policy a partire dall'attività registrata in CloudTrail. L'obiettivo resta lo stesso: partire da ciò che il codice fa davvero, non da ciò che *potrebbe* fare.

## 3.4 Configurare le credenziali nell'ambiente di sviluppo

Uno sviluppatore lavora con AWS soprattutto da un IDE o dalla riga di comando. La Console resta utile per imparare e per operazioni semplici. Perché il codice possa autenticarsi mentre lo si sviluppa, l'ambiente deve avere a disposizione delle credenziali, e ci sono diversi modi per fornirle: un **file di credenziali** locale, le **variabili d'ambiente** o i parametri passati direttamente al client dell'SDK. In un ambiente di sviluppo basato su EC2, come AWS Cloud9, si possono invece usare credenziali temporanee tramite ruoli IAM oppure le credenziali gestite da Cloud9.

### `aws configure` e i named profile

Per impostare credenziali permanenti nell'ambiente locale si usa il comando:

```bash
aws configure
```

Il comando chiede quattro informazioni: l'*access key ID*, la *secret access key* (entrambe ottenute da IAM), la **Region predefinita** e il **formato di output**, cioè il formato in cui la CLI restituirà i risultati (JSON, testo, tabella). Queste impostazioni costituiscono il **profilo `default`**, che SDK e CLI usano quando non viene indicato altro.

In un contesto reale, però, è raro lavorare con un solo account AWS: in produzione se ne usano spesso molti. Per passare da un account all'altro o da un insieme di credenziali all'altro si usano i **named profile**:

```bash
aws configure --profile restricted
aws s3 ls --profile restricted
```

Ogni profilo conserva un diverso insieme di credenziali, e codice e CLI possono essere indirizzati al profilo desiderato.

L'**AWS Toolkit** degli IDE supporta i profili. In IntelliJ IDEA, o in qualunque IDE JetBrains, il pannello **AWS Explorer** permette di navigare le risorse. Espandendo S3 si vedono i bucket, e con il tasto destro si può copiare l'ARN, eliminare il bucket o visualizzarne il contenuto. Un menu a tendina permette di cambiare profilo. Passando dal profilo `default`, con privilegi amministrativi, a un profilo `restricted` con permessi ridotti, la sezione S3 mostra *Access denied*: i permessi del profilo sono rispettati anche dentro l'IDE.

### I file `credentials` e `config`

`aws configure` scrive due file nella cartella `~/.aws/`. Il file **`credentials`** contiene le chiavi. Il file **`config`** contiene la Region, il formato di output e, all'occorrenza, impostazioni specifiche per servizio o per SDK. Un esempio per S3:

```ini
[default]
region = eu-south-1
output = json
s3 =
  max_concurrent_requests = 20
  max_queue_size = 10000
  multipart_threshold = 64MB
```

### L'ordine di ricerca delle credenziali

Uno scenario tipico per chi inizia: si crea un bucket dalla Console, si scrive il codice per caricarvi un oggetto seguendo la documentazione, lo si esegue… e si ottiene *Access denied*. Dove cerca le credenziali il codice? Gli SDK le cercano in un **ordine di priorità** preciso. In forma semplificata:

1. **Nel codice o nei parametri della CLI.** Tecnicamente si possono passare le chiavi direttamente al costruttore del client. È **fortemente sconsigliato**: se il codice finisce in un repository come GitHub, le chiavi diventano visibili ad altri.
2. **Nelle variabili d'ambiente.** Sono utili per esempio quando si eseguono container in locale. In produzione, su EC2 o Lambda, di solito non si usano.
3. **Nel profilo di credenziali predefinito.** È il **metodo preferito per lo sviluppo locale**.
4. **Nell'instance profile EC2.** Una volta distribuita l'applicazione su EC2, si associa all'istanza un ruolo IAM tramite un **instance profile**, e l'applicazione ne assume automaticamente le credenziali.

Più una fonte è in alto nell'elenco, più ha priorità. Ogni SDK applica una variante leggermente diversa di questo schema, come si vedrà per Java nel paragrafo 3.5.

### La firma delle richieste con SigV4

Le credenziali servono a **firmare** le richieste con **Signature Version 4 (SigV4)**. La firma ha tre scopi:

- **verificare l'identità** del richiedente, perché solo chi possiede la chiave segreta può produrla;
- **proteggere i dati in transito**, perché qualsiasi modifica alla richiesta invalida la firma;
- **proteggere dagli attacchi di replay**, perché ogni richiesta firmata è valida solo per un intervallo di tempo molto breve, che varia da servizio a servizio, e chi la intercetta non può riutilizzarla più tardi.

La firma si trasmette nell'intestazione HTTP `Authorization` oppure come parametri della query string. È possibile costruirla manualmente seguendo la procedura descritta nella documentazione, ma **SDK e CLI firmano automaticamente ogni richiesta**. Salvo casi particolari, è la scelta raccomandata.

### Le credenziali temporanee e AWS STS

Un ruolo fornisce **credenziali temporanee**: hanno vita breve e, una volta scadute, non si possono riutilizzare. Gli SDK possono assumere un ruolo quando il codice gira su EC2 o Lambda, gli utenti possono assumere ruoli, e anche istanze e servizi AWS ottengono credenziali temporanee.

Dietro le quinte il meccanismo è gestito da **AWS Security Token Service (STS)**. Un'entità già autenticata in qualche modo chiede di assumere un ruolo, per esempio con `AssumeRole`. L'entità può essere un utente IAM o un'identità federata tramite *web identity federation*, e il ruolo può trovarsi anche in un altro account. Il ruolo ha una **trust relationship** che stabilisce chi può assumerlo: non chiunque può farlo. Se la richiesta è autorizzata, STS restituisce le credenziali temporanee, che vengono poi usate per firmare le chiamate API, per esempio verso S3.

### Le credenziali in AWS Cloud9

In Cloud9 la situazione è diversa dallo sviluppo locale. Per default Cloud9 usa le **AWS managed temporary credentials**: un file di credenziali sull'istanza che i processi di Cloud9 **ruotano automaticamente**. Ci sono due alternative. Si possono inserire credenziali permanenti con `aws configure`. Oppure, ed è l'approccio consigliato, si **disattivano le credenziali gestite** e si **associa un instance profile IAM** all'istanza EC2 su cui gira Cloud9, così l'ambiente opera con i permessi di un ruolo.

## 3.5 Laboratorio: l'ordine di ricerca delle credenziali nell'AWS SDK for Java

L'elenco del paragrafo precedente è una semplificazione. La documentazione dell'**AWS SDK for Java 2.x** descrive la cosiddetta *default credentials provider chain*, che cerca le credenziali in quest'ordine:

1. le **proprietà di sistema Java** (`aws.accessKeyId`, `aws.secretAccessKey`);
2. le **variabili d'ambiente** (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`);
3. un **web identity token** ottenuto da AWS STS;
4. i **file condivisi** `credentials` e `config`;
5. le credenziali del **container Amazon ECS**, recuperate da una classe dedicata quando il codice gira in un task ECS;
6. l'**instance profile** EC2, se il codice gira su un'istanza con un ruolo associato.

La sequenza è leggermente diversa da quella generica, e altri SDK, come Python o .NET, hanno a loro volta piccole differenze. Quando le credenziali "non sono quelle che ci si aspetta", conviene consultare la documentazione dell'SDK in uso.

> **Box – Debug delle credenziali**
>
> Un errore classico: si aggiorna il file `credentials`, ma la CLI continua a usare le vecchie chiavi. Spesso la causa è una variabile d'ambiente impostata tempo prima, che ha priorità sul file. Il comando `aws sts get-caller-identity` mostra immediatamente quale identità sta usando la CLI ed è il primo controllo da fare.

## 3.6 Verifica dei concetti

Le affermazioni seguenti riassumono i concetti chiave del capitolo.

**"I permissions boundary impostano i permessi *minimi* che una identity-based policy può concedere."** *Falso.* Un permissions boundary imposta i permessi **massimi**. Se una policy consente di terminare istanze ma il boundary non lo consente, l'azione è negata: i permessi effettivi sono l'intersezione tra policy e boundary.

**"I ruoli IAM delegano temporaneamente l'accesso a utenti o servizi che normalmente non hanno accesso alle risorse AWS."** *Vero.* È un uso comune anche nel lavoro quotidiano. Il profilo abituale sul portatile non ha privilegi distruttivi. Quando serve un'operazione delicata si passa a un profilo che punta a un ruolo con più permessi, e lo si abbandona subito dopo. È una barriera in più verso le credenziali di produzione.

**"Le identity-based policy concedono al principal specificato il permesso di eseguire azioni su una risorsa, a determinate condizioni."** *Falso.* Il principal **si specifica** nelle resource-based policy. Una identity-based policy è attaccata al principal e quindi non lo nomina. Lo si nomina in una bucket policy, per esempio con `"Principal": "*"` per l'accesso anonimo.

**"La AWS CLI supporta più named profile."** *Vero.* È così che si passa da un account o da un insieme di permessi all'altro.

**"Quando le credenziali temporanee scadono, non è necessario ruotarle o revocarle esplicitamente."** *Vero.* La scadenza è applicata da IAM e STS. Un ruolo su un'istanza EC2 viene rinnovato automaticamente, e l'SDK gestisce lo scambio. Chi assume un ruolo manualmente dovrà solo richiedere nuove credenziali alla scadenza.

**"I file config e credentials contengono impostazioni che possono essere definite anche nelle variabili d'ambiente del sistema operativo."** *Vero.* Le variabili d'ambiente hanno precedenza sui file, e dimenticarlo è una causa frequente di comportamenti inattesi della CLI.
