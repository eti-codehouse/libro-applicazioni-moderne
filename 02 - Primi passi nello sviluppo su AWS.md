# Capitolo 2 – Primi passi nello sviluppo su AWS

## 2.1 Obiettivi del capitolo

Uno sviluppatore incontra AWS in due ruoli diversi. A volte AWS è l'**infrastruttura che ospita** l'applicazione: le macchine virtuali, i container o le funzioni su cui gira il codice. Altre volte è l'applicazione stessa a **dialogare con le risorse AWS**: salvare un file in un bucket, scrivere un record in una tabella, avviare un'istanza. In entrambi i casi l'interazione avviene sempre nello stesso modo, cioè con chiamate alle API dei servizi.

Questo capitolo spiega che cosa succede quando un client parla con AWS, dal livello più basso, la singola richiesta HTTP, fino agli strumenti che nascondono questa complessità: la AWS CLI e gli AWS SDK. Presenta poi i principali **pattern di programmazione** degli SDK, come le chiamate sincrone e asincrone, i *waiter*, la gestione degli errori e la paginazione, con un esempio in Java che crea una tabella DynamoDB e attende che sia pronta. Si passa quindi agli strumenti di lavoro quotidiano, gli IDE con l'AWS Toolkit e l'ambiente cloud AWS Cloud9. Il capitolo si chiude con due servizi di intelligenza artificiale generativa rivolti agli sviluppatori: Amazon Bedrock e Amazon Q Developer.

## 2.2 Accedere ai servizi AWS in modo programmatico

### Il modello richiesta/risposta

Ogni interazione con AWS è, alla base, uno scambio **richiesta/risposta HTTP** verso gli endpoint REST dei servizi. Conviene esaminarlo a questo livello prima di vedere come gli SDK lo automatizzano, perché saperlo leggere è di grande aiuto nel debug.

Supponiamo che un'applicazione client voglia avviare un'istanza EC2. La richiesta che parte contiene diversi elementi. Viaggia su **HTTPS**, quindi gode delle garanzie di **TLS**: nessuno può leggerla né modificarla durante il transito. Include un'intestazione di autenticazione **Signature Version 4 (SigV4)**, costruita a partire da una **access key IAM**. Un'identità IAM, approfondita nel Capitolo 3, possiede un identificativo (*access key ID*) e un segreto (*secret access key*). Solo chi conosce il segreto può costruire la firma, e AWS la verifica all'arrivo della richiesta.

Quando la richiesta raggiunge l'API REST di AWS, l'intestazione di autorizzazione ha due funzioni. Serve a stabilire che il chiamante è davvero chi dichiara di essere e serve a determinare **su quale account** operare. Se chiedo di descrivere le istanze, è la firma a dire ad AWS in quale account cercarle. La richiesta viene poi inoltrata al servizio interessato con un'istruzione precisa: avvia delle istanze, carica un file in un bucket S3, aggiorna un item in DynamoDB. Il servizio esegue l'azione e restituisce una risposta, per esempio "item aggiornato correttamente" oppure "istanze avviate".

### Il formato sul filo

Una richiesta per creare un bucket S3 chiamato `pollynotes` ha questo aspetto:

```http
PUT / HTTP/1.1
Host: pollynotes.s3.amazonaws.com
Authorization: AWS4-HMAC-SHA256 Credential=AKIA.../s3/aws4_request, SignedHeaders=..., Signature=...
x-amz-date: 20221001T120000Z
```

Il **metodo HTTP** è definito dallo standard HTTP, che prevede anche GET, POST, PATCH, HEAD e altri, e i servizi AWS lo usano secondo la sua semantica. In questo caso PUT significa "voglio creare qualcosa". L'**host** contiene il nome del bucket. L'intestazione **Authorization** contiene la firma SigV4.

La risposta riporta uno **status HTTP** standard: 200 OK significa che i parametri erano validi e l'azione è stata completata. Include poi due intestazioni preziose, l'**Amazon ID** (`x-amz-id-2`) e l'**Amazon Request ID** (`x-amz-request-id`). Se si contatta il supporto AWS, è proprio questi identificativi che verranno richiesti per diagnosticare il problema. Seguono la data e, dopo le intestazioni, il **payload** con le informazioni sulla risorsa creata.

Gli status code seguono le classi dello standard HTTP:

| Classe | Significato | Esempio in AWS |
|---|---|---|
| **1xx** | Informativo | `100 Continue` |
| **2xx** | Successo | `200 OK` alla creazione di un bucket |
| **3xx** | Redirect | Un bucket appena creato può rispondere con un redirect verso il dominio specifico della sua Region |
| **4xx** | Errore del client | `404 Not Found`: la risorsa richiesta non esiste o i parametri sono errati |
| **5xx** | Errore del server | Qualcosa è andato storto lato servizio; con AWS accade raramente |

Una risposta di DynamoDB a un'operazione di *Scan* contiene, oltre al RequestId, un **checksum CRC** per verificare l'integrità, il `Content-Type` (JSON), il `Content-Length` e il payload con gli item trovati.

### Osservare il traffico reale

Per vedere tutto questo in pratica basta eseguire un comando qualsiasi della CLI mentre un proxy di intercettazione (*sniffing proxy*) registra il traffico:

```bash
aws ec2 describe-regions
```

Il proxy mostra una richiesta **POST** verso l'endpoint di EC2. Contiene l'intestazione di autorizzazione completa, che ha una scadenza e non può essere riutilizzata né rigiocata, e nel corpo l'azione `DescribeRegions`. La risposta contiene il `RequestId`, un'intestazione di sicurezza del trasporto (*Strict-Transport-Security*), il tipo e la lunghezza del contenuto e un corpo XML con la descrizione di tutte le Region disponibili.

### Tre client, le stesse API

Tre diversi strumenti parlano con le API di AWS:

- la **AWS Management Console**, l'interfaccia grafica nel browser;
- la **AWS CLI**, per lavorare da riga di comando o automatizzare con script;
- gli **AWS SDK**, per chiamare i servizi dal codice applicativo, per esempio per creare un record in DynamoDB o avviare e fermare istanze EC2.

Questi tre strumenti non sono indipendenti. La Console è costruita sopra gli SDK. La CLI è scritta in Python e usa l'SDK per Python, **Boto**. Tutti usano **esattamente le stesse API**. Ciò che si può fare in uno lo si può fare negli altri, e gli errori e i permessi si comportano allo stesso modo.

> **Box – Le API nell'architettura Polly Notes**
>
> Nell'applicazione di riferimento le API sono ovunque. L'utente si autentica con **Cognito** e scarica il front-end da un bucket **S3**, che grazie alle sue policy funziona da web server. Il front-end chiama un'API ospitata in **API Gateway**. Dietro API Gateway ci sono le funzioni **Lambda**, che usano l'**SDK** per parlare con **DynamoDB**, anche per scorrere i risultati pagina per pagina. **X-Ray** sorveglia il tutto: una singola azione dell'utente può toccare più servizi, e con l'SDK X-Ray le chiamate vengono ricomposte in un'unica *trace*.

### Che cosa fanno gli SDK per noi

Nella pratica nessuno costruisce a mano firme e richieste HTTP. Questo codice è già scritto negli **AWS SDK**, disponibili per moltissimi linguaggi: Python (Boto3), Java, .NET, C++, JavaScript, Go e altri ancora. È difficile trovare un linguaggio diffuso che non abbia un SDK. Gli SDK offrono quattro servizi fondamentali:

- **Language binding**: si lavora con metodi e oggetti idiomatici del proprio linguaggio, non con richieste HTTP.
- **Firma delle richieste**: l'SDK costruisce automaticamente l'intestazione SigV4.
- **Resilienza integrata**: gli errori transitori vengono gestiti con tentativi automatici (*retry*). Per esempio, se si supera la capacità di lettura provisionata di una tabella DynamoDB, l'SDK ripete la richiesta senza che il codice debba occuparsene.
- **Supporto alla paginazione**: molte operazioni non si esauriscono in una singola risposta a causa del volume dei dati, come l'elenco dei tipi di istanza EC2 o degli oggetti di un bucket. L'SDK nasconde in gran parte le richieste multiple necessarie.

## 2.3 L'AWS Command Line Interface (AWS CLI)

La **AWS CLI** è il modo di interagire con AWS da riga di comando. La Console va benissimo per esplorare. Quando un'operazione va ripetuta, però, conviene automatizzarla con uno script.

Supponiamo di dover creare un bucket in ogni Region AWS. Dalla Console bisognerebbe creare il bucket, cambiare Region, crearne un altro, cambiare di nuovo Region, e così via: un lavoro monotono e soggetto a errori. Con la CLI basta una riga per ogni bucket, con la Region passata come parametro. Lo script si scrive una volta e si riesegue quando serve.

La CLI si può eseguire in molti ambienti:

- **in locale**, come qualunque applicazione da riga di comando, su Linux, Windows o macOS;
- **su un'istanza EC2**, collegandosi via SSH;
- in **AWS Cloud9**, l'IDE nel browser che include un terminale;
- in **AWS CloudShell**, una shell Bash nel browser già autenticata con le credenziali della Console;
- tramite **AWS Systems Manager Session Manager**, un'alternativa a SSH per amministrare da remoto le istanze EC2.

### Esempio: primi comandi su S3

```bash
# Creare un bucket (mb = make bucket). Il nome deve essere unico a livello globale.
aws s3 mb s3://russs-banana

# Creare un file vuoto e caricarlo nel bucket
touch myfile.txt
aws s3 cp myfile.txt s3://russs-banana

# Elencare il contenuto del bucket
aws s3 ls s3://russs-banana
```

La sintassi `s3://nome-bucket` identifica un bucket nei parametri dei comandi S3. Per scoprire i comandi disponibili si usa l'aiuto integrato. Digitando `aws s3` senza altro, la CLI ricorda che servono un comando e un sottocomando. Con `aws s3 help` si apre una pagina simile a una *man page* di Unix, con l'elenco dei comandi (`sync`, `mb`, `ls`, `cp`…) e numerosi esempi d'uso.

## 2.4 Gli AWS SDK e i pattern di programmazione in Java

### Chiamate sincrone e asincrone

Le interazioni con AWS si dividono in due famiglie. Le chiamate **sincrone** (bloccanti) restituiscono il risultato nella risposta stessa. Se chiedo a DynamoDB quali record contiene una tabella, il servizio elabora la richiesta per qualche millisecondo e risponde con i dati.

Altre operazioni sono **asincrone** (non bloccanti). Qui "asincrono" va inteso dal punto di vista di AWS, non del codice locale. Se chiedo di creare una tabella DynamoDB, la risposta arriva subito ma la tabella **non è ancora pronta**: la risposta dice in sostanza "ricevuto, la sto creando", con lo stato `CREATING`. Come si fa a sapere quando la tabella è utilizzabile?

### Polling e waiter

La prima strategia è il **polling**: si interroga periodicamente il servizio finché lo stato non cambia.

```bash
aws dynamodb describe-table --table-name Notes --query "Table.TableStatus"
```

Il parametro `--query` usa **JMESPath**, un linguaggio di interrogazione per JSON. In questo caso entra nella risposta, seleziona l'oggetto `Table` e ne estrae la chiave `TableStatus`. Se lo stato non è ancora `ACTIVE`, si attende e si ripete la chiamata.

Questo ciclo di attesa e verifica è così comune che la CLI e molti SDK lo offrono già pronto sotto forma di **waiter**:

```bash
aws dynamodb wait table-exists --table-name Notes
```

Il traffico di rete è identico a quello del polling manuale. Il waiter automatizza solo il ciclo "verifica, attendi un intervallo configurabile, verifica di nuovo".

Un altro servizio tipicamente asincrono è **AWS CloudFormation**, che crea l'infrastruttura descritta in un file YAML o JSON. La creazione di uno *stack* richiede tempo, quindi il servizio risponde subito "sto creando lo stack". Mentre un aggiornamento è in corso si può persino inviare un'altra chiamata per annullarlo (`cancel-update-stack`). È un'altra interazione asincrona: si avvia un processo lungo e lo si governa con chiamate successive.

### Esempio: un waiter con l'SDK for Java

```java
DynamoDbClient ddb = DynamoDbClient.create();
DynamoDbWaiter waiter = ddb.waiter();

WaiterResponse<DescribeTableResponse> waiterResponse =
    waiter.waitUntilTableExists(r -> r.tableName("Employees"));

waiterResponse.matched().response().ifPresent(System.out::println);
System.out.println("The table has been created");
```

Il codice crea prima il client DynamoDB e poi un **waiter**. Il metodo `waitUntilTableExists` riceve una *lambda* Java che configura la richiesta, qui il nome della tabella `Employees`. Il risultato espone il metodo `matched()`, che contiene o una risposta o un'eccezione. Dalla risposta, `ifPresent` riceve a sua volta una funzione, qui `System.out::println`, a cui passa l'oggetto `DescribeTableResponse`. In Java una lambda si può pensare come un puntatore a funzione: il waiter chiamerà la funzione indicata passandole il risultato.

Eseguendo il codice nel debugger si vede il waiter entrare in pausa, interrogare il servizio, attendere e riprovare. Quando la tabella risulta pronta, la `DescribeTableResponse` viene stampata. Tutto il codice di attesa e verifica che altrimenti andrebbe scritto a mano è incapsulato in una sola chiamata.

### Errori ed eccezioni

Gli errori capitano, e l'SDK li rende gestibili. A livello di protocollo il servizio risponde con status 4xx o 5xx. Gli SDK per Java e .NET li trasformano in **eccezioni** da intercettare con `try/catch`. L'SDK per Python solleva eccezioni di **botocore**, la libreria su cui si basa Boto3.

### Metriche, log e trace

Gli SDK aiutano anche a capire come l'applicazione interagisce con AWS:

- **Metriche integrate**: alcuni SDK possono inviare ad **Amazon CloudWatch** metriche sul proprio funzionamento, come il numero di errori, il numero di retry, le richieste API e la loro durata.
- **Logging**: gli SDK si integrano con i framework di logging più diffusi. Attivando un livello di dettaglio maggiore si vede il contenuto delle richieste e il tipo di risposta ricevuta. È uno strumento prezioso quando si insegue un errore difficile e si vuole sapere esattamente quale API è stata chiamata.
- **CloudWatch Logs**: i log applicativi inviati a CloudWatch Logs vengono **tolti dalla macchina di calcolo**. Se l'istanza EC2 viene terminata o spenta, i log restano conservati in un luogo permanente.
- **AWS X-Ray**: un'azione dell'applicazione può coinvolgere più servizi, e X-Ray ricompone queste chiamate in una sola *trace* mostrando quanto tempo ha richiesto ogni passaggio. L'argomento è trattato nel Capitolo 13.

## 2.5 Gli ambienti di sviluppo integrati (IDE) e l'AWS Toolkit

AWS mette a disposizione un **AWS Toolkit** per i principali IDE: Visual Studio Code, Visual Studio, Eclipse e gli IDE JetBrains (IntelliJ IDEA, PyCharm e altri). Il Toolkit porta AWS dentro l'editor:

- permette di **navigare le risorse AWS**, come bucket S3 e funzioni Lambda, senza aprire la Console;
- aggiunge funzionalità per **creare, fare il debug e distribuire** le applicazioni;
- si integra con **AWS SAM** (Capitolo 12): si possono creare applicazioni SAM e farne il debug in locale. Il Toolkit simula l'ambiente di esecuzione di Lambda e permette di avviare il debugger ed entrare passo passo nel codice che girerà nella funzione.

**AWS Cloud9** è invece un IDE interamente **nel cloud**, che si usa da browser per scrivere, eseguire e fare il debug del codice. Ha l'aspetto di un IDE tradizionale e aggiunge funzionalità di **programmazione collaborativa in tempo reale**: un collega può vedere e modificare lo stesso codice, cosa molto utile per fare il debug in coppia durante una chiamata. Anche Cloud9 offre l'integrazione con SAM per creare applicazioni serverless e farne il debug.

> **Box – Lo stato di Cloud9 e CodeWhisperer oggi**
>
> Da luglio 2024 AWS Cloud9 non è più disponibile per i nuovi clienti: chi lo usava già può continuare a farlo, ma per i nuovi progetti AWS consiglia alternative come un IDE locale con l'AWS Toolkit, AWS CloudShell o ambienti basati su VS Code. Allo stesso modo **Amazon CodeWhisperer**, citato nel Toolkit, è confluito in **Amazon Q Developer** (paragrafo 2.7.2). I concetti mostrati in questo capitolo restano validi, ma all'esame e nei progetti reali conviene sapere quali strumenti sono attuali.

## 2.6 Laboratorio: AWS Cloud9 e AWS CLI

Questo laboratorio ripercorre la creazione e l'uso di un ambiente Cloud9.

**Creazione dell'ambiente.** Dalla Console di Cloud9 si crea un nuovo ambiente, per esempio `russ-environment`, e si scelgono le opzioni principali:

- **calcolo**: si può far creare a Cloud9 una nuova istanza EC2 oppure collegarsi a un server esistente;
- **tipo di istanza**: per esempio `m5.large`;
- **piattaforma**: Amazon Linux 2 oppure Ubuntu;
- **timeout di inattività**: dopo un periodo senza attività l'istanza viene fermata, così non si paga la macchina virtuale quando non la si usa;
- **connessione**: Cloud9 deve poter eseguire comandi sull'istanza, e le opzioni sono **AWS Systems Manager** (predefinita) oppure **SSH**.

Mentre l'ambiente si crea, nella Console EC2 compare una nuova istanza `m5.large`. È lì che avviene il calcolo: Cloud9 ospita l'interfaccia dell'IDE, ma il codice e il terminale girano su quell'istanza.

**Personalizzazione.** Aperto l'IDE, le impostazioni permettono di configurare l'editor, l'esecuzione e il debug, i formattatori di codice, il supporto ai linguaggi (PHP, Python…), lo stop automatico dell'istanza, l'integrazione con Git e l'AWS Toolkit, e il tema grafico.

**Uso della CLI e di Python.** Il terminale integrato esegue la CLI come qualunque altro terminale. Si può quindi creare un piccolo script `main.py`:

```python
import boto3

ec2 = boto3.client('ec2')
response = ec2.describe_instances()
print(response)
```

Alla prima esecuzione compare l'errore "No module named boto3": la dipendenza non è installata. Si risolve dal terminale con:

```bash
pip3 install boto3
```

Alla seconda esecuzione lo script stampa la risposta. Impostando un **breakpoint** e attivando il debugger si ottengono le funzioni attese da un IDE: esecuzione passo passo, *call stack* e variabili locali.

**Controllo di versione.** Cloud9 integra Git sia da riga di comando sia da interfaccia:

```bash
git init
ls -la                      # compare la cartella .c9 con le impostazioni di Cloud9
echo ".c9" >> .gitignore    # le impostazioni dell'IDE non vanno nel repository
git status
git commit -m "initial commit"
```

Se nome utente ed email di Git non sono configurati, Git mostra un avviso al momento del commit.

**AWS Toolkit.** La scheda AWS dell'IDE mostra l'infrastruttura dell'account. Espandendo S3 si trovano il bucket creato in precedenza e il file caricato, ed è possibile caricare altri file con il tasto destro. Ci sono anche le integrazioni con CDK e con l'assistente al codice. Nella sezione Lambda compare l'integrazione con SAM per progettare applicazioni serverless destinate a Lambda e API Gateway.

## 2.7 L'intelligenza artificiale generativa per lo sviluppatore: Amazon Bedrock e Amazon Q Developer

### 2.7.1 Amazon Bedrock

L'**intelligenza artificiale generativa** (*generative AI*) è oggi uno dei temi centrali del settore tecnologico. Chi lavora nell'IT deve conoscerla anche quando non incide ancora sul lavoro di tutti i giorni. Milioni di persone la usano già per interagire con assistenti virtuali, rilevare frodi, gestire chatbot di assistenza clienti e generare codice durante lo sviluppo software.

L'esempio più comune è la **chat**. Si fornisce al modello un input in linguaggio naturale, il *prompt*, per esempio: "Fingi di essere il mio insegnante e spiegami che cos'è l'AI generativa". Il modello risponde generando un testo che spiega l'argomento con il tono richiesto. L'AI generativa è infatti un tipo di intelligenza artificiale capace di **creare nuovi contenuti** su richiesta: conversazioni, storie, immagini, video, musica. Questi contenuti si basano sui dati con cui il modello è stato addestrato.

Due termini ricorrono spesso:

- Un **Large Language Model (LLM)** è un'applicazione dell'AI generativa progettata per compiti legati al **linguaggio naturale**. Viene addestrato su enormi insiemi di dati e sa comprendere e produrre testo che simula il linguaggio umano. È adatto a scrivere, riassumere, tradurre e compiti simili.
- Un **Foundation Model (FM)** è un modello di machine learning di grandi dimensioni, **pre-addestrato per scopi generali**. È una base pronta all'uso, ma per diventare esperto in un dominio specifico va di solito adattato con dati propri tramite *fine-tuning*.

**Amazon Bedrock** è un servizio **fully managed** che dà accesso, tramite **API**, a un'ampia scelta di foundation model ad alte prestazioni di aziende leader del settore, tra cui Amazon, Anthropic, Cohere e Meta. I modelli differiscono per capacità e specializzazioni, ed è importante scegliere quello adatto al proprio caso d'uso. Bedrock rende facile **sperimentare e confrontare** i modelli con poche chiamate API o pochi clic, senza dover creare un modello proprio e senza vincolarsi in anticipo a una scelta tecnologica. Tra i modelli disponibili c'è la famiglia **Amazon Titan**, foundation model general purpose sviluppati da AWS.

I foundation model si possono **personalizzare** in vari modi:

- **Continued pre-training**: si prosegue l'addestramento del modello con nuovi dati **non etichettati**, per esempio documenti del proprio dominio.
- **Fine-tuning**: si migliorano le prestazioni del modello su compiti specifici fornendo un dataset di **esempi etichettati**.
- **Knowledge Base**: si caricano fonti di dati aziendali, creando un archivio di informazioni usato per arricchire i prompt o le risposte del modello.

Le Knowledge Base si basano sulla tecnica **RAG (Retrieval Augmented Generation)**: prima di interrogare il modello si recuperano dalle fonti aziendali i dati pertinenti e li si aggiungono al prompt. Così le risposte possono fare riferimento a informazioni proprietarie e risultano più accurate. La RAG aiuta anche a ridurre le **allucinazioni**, cioè i casi in cui il modello produce informazioni inesatte o inventate.

Dietro le quinte i dati sono conservati in un **vector database**, ottimizzato per gestire **vettori**, cioè rappresentazioni numeriche dell'informazione chiamate *embedding*. Convertendo i testi in vettori, la Knowledge Base può trovare rapidamente le informazioni rilevanti con **ricerche per similarità** e passarle al modello.

Bedrock è **serverless**: si inizia subito, si personalizzano i modelli in modo sicuro con i propri dati e li si integra nelle applicazioni senza gestire infrastruttura. Per prendere confidenza ci sono i **Playground** della Console. Nel *chat playground* si sceglie un modello, per esempio **Amazon Titan Text G1**, e si avvia una conversazione. Un prompt come "Scrivi un breve script Python che legga i file locali di una cartella e li carichi su Amazon S3" produce una risposta con codice di esempio da valutare e riutilizzare. Si può poi iterare con il chatbot, cambiare la configurazione del modello o affinare il prompt (*prompt engineering*) fino a ottenere il risultato desiderato. Esistono anche playground dedicati al testo e alle immagini.

Quando si passa all'applicazione vera, integrare l'AI generativa è relativamente semplice. Non si addestrano modelli da zero: si chiamano le **API di Bedrock** tramite gli **AWS SDK** con i parametri corretti, eventualmente dopo aver personalizzato il modello.

### 2.7.2 Amazon Q Developer

**Amazon Q Developer** è un **assistente alla programmazione** basato sull'AI generativa, che affianca lo sviluppatore in tutto il processo di sviluppo. Può conversare sul codice, proporre **suggerimenti inline**, generare codice nuovo, **scansionare il codice** alla ricerca di vulnerabilità e suggerire miglioramenti come aggiornamenti di linguaggio, correzioni di bug e ottimizzazioni. È uno strumento general purpose, ma è stato ottimizzato per i servizi AWS e ne è un vero esperto.

Si usa tramite l'**estensione Amazon Q** nell'IDE, accedendo con un **AWS Builder ID**. Dietro le quinte è alimentato da **Amazon Bedrock**. Le modalità di interazione principali sono due.

**La chat.** Nell'IDE si può dialogare in linguaggio naturale con Amazon Q per chiedere suggerimenti o miglioramenti al codice, risposte a domande generali sullo sviluppo software, chiarimenti sulle best practice AWS e aiuto per debug, test e ottimizzazione. La chat **conserva il contesto della sessione**, quindi si possono fare domande di approfondimento o richiamare risposte precedenti. È particolarmente utile quando non si ricorda una sintassi comune o non si conosce l'API di un servizio AWS: invece di uscire dall'IDE per cercare sul web, si chiede direttamente ad Amazon Q e si resta concentrati sul lavoro. Un'altra funzione molto apprezzata è la **spiegazione del codice**: si evidenzia un blocco e si chiede ad Amazon Q di spiegarlo, e l'assistente lo commenta riga per riga. È utile davanti a codice che "non funziona e non si sa perché", ma anche a codice che "funziona e non si sa perché".

**I suggerimenti inline.** Mentre si scrive, Amazon Q propone in tempo reale suggerimenti che vanno da una singola riga a intere funzioni. Basta scrivere un commento in linguaggio naturale che descrive il compito, per esempio:

```java
// upload a file to Amazon S3 with server side encryption
```

Amazon Q propone uno o più frammenti di codice che svolgono il compito, e lo sviluppatore può accettarli o scorrere le alternative. I suggerimenti si basano su LLM addestrati su miliardi di righe di codice, sia di Amazon sia open source. Come contesto usano gli snippet, i commenti, la posizione del cursore e i file aperti nell'IDE. Si possono anche **personalizzare**, perché rispettino le librerie interne, il codice esistente, le tecniche proprietarie e lo stile del team.

Amazon Q Developer offre infine due funzioni aggiuntive:

- **Security scan**: individua vulnerabilità difficili da trovare e suggerisce come correggerle, seguendo le best practice indicate per esempio dall'**OWASP** (Open Worldwide Application Security Project) o quelle per l'uso corretto delle librerie crittografiche.
- **Riga di comando**: fornisce completamenti in stile IDE per centinaia di CLI diffuse, come Git, npm, Docker e la stessa AWS CLI.

> **Box – Best practice: usare l'AI come collega, non come oracolo**
>
> Gli assistenti generativi accelerano molto lo sviluppo, ma il codice suggerito va sempre letto, capito e testato come quello scritto da un collega. Conviene in particolare verificare che i permessi IAM richiesti siano minimi (Capitolo 3), che gli errori siano gestiti e che le chiamate paginate leggano davvero tutti i risultati. Lo scan di sicurezza è un aiuto in più, non sostituisce la revisione del codice.
