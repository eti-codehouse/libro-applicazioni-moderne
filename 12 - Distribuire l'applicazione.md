# Parte VI – Rilascio e osservabilità

# Capitolo 12 – Distribuire l'applicazione

## 12.1 Obiettivi del capitolo

Un'applicazione ben scritta va ancora portata in produzione, e poi aggiornata decine o centinaia di volte nel corso della sua vita. Questo capitolo parte dai **rischi delle pratiche tradizionali** di sviluppo e rilascio del software, problemi che chiunque abbia lavorato qualche anno nel settore ha vissuto in prima persona. Vediamo poi come la cultura **DevOps** li affronta, con un forte accento sull'**automazione**: applicare alle operazioni le lezioni apprese nel mondo dello sviluppo, così da automatizzare gran parte delle attività operative.

Lo strumento principale del capitolo è **AWS Serverless Application Model (AWS SAM)**. Nel Capitolo 8 una funzione Lambda era stata impacchettata e distribuita a mano. SAM automatizza queste attività per molti servizi: distribuisce funzioni Lambda, crea API in API Gateway e permette di **testare in locale**, sul proprio computer, prima di distribuire. Il capitolo si chiude con le diverse **strategie di deployment** disponibili.

## 12.2 Pianificazione e pratiche di deployment

### I rischi del rilascio tradizionale

I rischi tipici del rilascio tradizionale hanno un denominatore comune: i **processi manuali**.

**Packaging incoerente.** Quando si prepara a mano il pacchetto da distribuire, comprimendo i file e consegnandoli a chi farà il deploy, la probabilità di dimenticare qualcosa è alta.

**Deployment incoerenti.** Quando si distribuisce un aggiornamento su una flotta di server, può succedere di ritrovarsi con versioni diverse del software su istanze diverse. Prima o poi questa incoerenza crea problemi.

**Aggiornamenti difficili.** Gli aggiornamenti non dovrebbero fare paura. Quando emerge una nuova vulnerabilità di sicurezza bisogna poter applicare la patch rapidamente. Tempi di aggiornamento brevi migliorano direttamente la sicurezza, e l'automazione li rende possibili.

**Nessun piano di rollback.** Molti moduli di richiesta di deploy chiedono "qual è il piano di rollback?". La risposta scritta è spesso "ripristinare il database e tornare alla versione precedente del codice". Nella pratica, non di rado si continua a ritentare il deploy finché non funziona, e il rollback non viene mai eseguito davvero. Pianificare in anticipo, e automatizzare, rende i deploy molto meno rischiosi.

**Errore umano.** Ovunque intervengano le mani di una persona, qualcosa può andare storto. Un lungo documento di istruzioni (*runbook*) può dire: fai il backup del database, ferma il database, disattiva i backup automatici perché non interferiscano con l'aggiornamento. Basta dimenticare un passo, per esempio i backup automatici, che partono proprio nel mezzo dell'aggiornamento, per causare un disastro. Se il passo fosse incluso in un processo automatico, verrebbe eseguito a ogni deploy senza eccezioni.

**Approvazioni manuali.** Dopo il deploy si chiede a un collega di "fare un giro" nell'applicazione per verificare che tutto vada bene, con un *smoke test* manuale. Deve per forza farlo una persona? Spesso test automatici possono dare la stessa fiducia, in modo più rapido e ripetibile, prima di proseguire con il deploy sugli altri host.

### La cultura DevOps e Agile

Una cultura DevOps e Agile si fonda su alcuni principi.

**Concentrarsi sui bisogni del cliente.** È facile farsi prendere dalla tecnologia. In un progetto si era deciso di supportare ebraico, arabo e giapponese, e qualcuno ha chiesto: "abbiamo clienti che ne hanno bisogno?". La risposta era no. Prima di sviluppare una funzionalità bisogna verificare che sia davvero un requisito, partendo da una *user story* che descriva esattamente ciò che serve al cliente.

**Collaborare intensamente.** Le lezioni apprese da un singolo vanno condivise con il team. Chi ha automatizzato il proprio deploy spiega agli altri come farlo.

**Sperimentare e imparare continuamente.** Nel cloud è più facile: quando esce una nuova funzionalità AWS si può accendere l'infrastruttura necessaria, fare esperimenti, inviarle una percentuale del traffico e, se i risultati sono buoni, adottarla, magari cambiando persino il tipo di persistenza. Le decisioni architetturali si possono prendere rapidamente.

**Includere la sicurezza ovunque.** In passato la sicurezza era spesso un'aggiunta finale. Con l'automazione si individuano i problemi in anticipo. Per esempio, quando si aggiungono nuove dipendenze a un'applicazione, si possono scansionare automaticamente alla ricerca di **vulnerabilità note**. Prima si scopre un problema, migliore è la postura di sicurezza.

**Automatizzare, automatizzare, automatizzare.** Una regola pratica: se si fa una cosa a mano per la seconda volta, è il momento di scrivere uno script, o di cercare uno strumento che la faccia già. Più si automatizza, più cresce la fiducia nei rilasci.

**Sviluppare a piccoli passi.** Un piano che prevede "la versione 2 uscirà tra nove mesi" rischia di mancare il bersaglio: alla consegna i requisiti potrebbero essere cambiati, o la versione potrebbe non soddisfarli. È meglio aggiungere le funzionalità gradualmente, farle provare a qualcuno del business in un ambiente di staging per verificare che si stiano risolvendo i problemi giusti, e poi distribuirle in produzione.

**Rilasciare spesso, con l'automazione.** I deploy non devono essere un evento temuto. La domanda giusta è: quanto rapidamente può arrivare ai clienti una nuova funzionalità?

**Migliorare continuamente.** Si monitora la latenza e si cercano margini di miglioramento: c'è sempre qualcosa da affinare, come una query sul database. Più metriche si raccolgono, più si può essere scientifici. Si è introdotta una modifica pensando che migliorasse le prestazioni? Si controlla se è davvero così.

### Le pratiche DevOps

**Continuous Integration (CI).** A ogni commit e push sul repository, un server di integrazione continua esegue automaticamente il *linting* e gli **unit test** e fornisce un riscontro immediato. Se il codice "rompe la build" ostacola il lavoro di tutto il team, e bisogna saperlo il prima possibile.

**Continuous Delivery e Continuous Deployment (CD).** Dopo il commit, il codice viene impacchettato e distribuito automaticamente. Si può iniziare con il deploy automatico in **staging**. L'idea di farlo anche in produzione può spaventare, ma ci si può arrivare: quando si ha piena fiducia nelle modifiche, il merge di un feature branch nel branch principale porta automaticamente la modifica davanti ai clienti.

**Microservizi.** Come visto nel Capitolo 10, sono una scelta architetturale molto diffusa nei team DevOps per i benefici che offre all'organizzazione del lavoro.

**Infrastructure as Code (IaC).** Anche l'infrastruttura che ospita e supporta il codice si può descrivere come codice. **AWS CloudFormation** ne è un esempio: descrive elementi come il VPC e i security group dell'architettura. Mettendo il template sotto controllo di versione si ha la **storia completa** dell'architettura e si può tornare a una versione precedente in qualunque momento.

**Monitoring e logging.** Sul lato operativo, si spediscono i log fuori dalle istanze e si costruiscono dashboard. Metriche interessanti possono essere il numero di login recenti, il numero di login falliti o la presenza di variazioni improvvise. Su queste metriche si impostano allarmi, così che qualcuno indaghi. Bisogna accorgersi degli errori **prima che li segnalino gli utenti**: se c'è un picco improvviso di errori 500, il sistema deve rilevarlo da solo e allertare qualcuno.

**Comunicazione e collaborazione.** Una dashboard ben costruita permette a chiunque in azienda di vedere lo stato attuale dell'applicazione ed è un ottimo strumento di comunicazione tra team.

### Gli strumenti AWS per il DevOps

| Strumento | Ruolo |
|---|---|
| **AWS CodeCommit** | Repository Git gestiti in cui fare push del codice |
| **AWS CodeBuild** | Compila, testa e impacchetta le applicazioni: è il motore della CI |
| **AWS CodeArtifact** | Repository privato e autenticato di pacchetti software, compatibile con Maven, pip (Python), npm e NuGet (.NET) |
| **AWS CodeDeploy** | Automatizza i deploy. Su EC2 un **agente** installato sulle istanze esegue i passi descritti in un file manifest (`appspec.yml`). Distribuisce anche su Lambda ed ECS |
| **AWS CodePipeline** | Orchestra le fasi (*stage*) della pipeline di rilascio: al commit preleva il codice dal repository, lo fa compilare a CodeBuild e lo fa distribuire a CodeDeploy o CloudFormation |
| **AWS Cloud9** | IDE nel browser |
| **AWS Config** | Monitora la configurazione dell'infrastruttura e verifica il rispetto di regole, per esempio che tutte le istanze EC2 abbiano determinati tag |
| **AWS CloudFormation** | Infrastructure as Code in JSON o YAML, da conservare in un repository |
| **Amazon CloudWatch** | Metriche dei servizi AWS e metriche personalizzate, log, *metric filter* che trasformano i log in metriche, allarmi e dashboard |
| **AWS X-Ray** | Tracing distribuito: mostra dove si accumula la latenza lungo il percorso di una richiesta |

Con CloudFormation, per esempio, se bisogna aprire una porta in un security group, non lo si fa dalla Console: si modifica il template nel repository e un processo automatico aggiorna lo stack nell'ambiente. Si ottiene così una traccia completa di ogni modifica all'infrastruttura.

AWS CodeDeploy, per la parte EC2, è uno strumento operativo, ma uno sviluppatore ha tutto l'interesse a sapere come le operazioni gestiscono le sue applicazioni.

> **Box – Lo stato di CodeCommit e Cloud9**
>
> Nel 2024 AWS ha limitato l'accesso di nuovi clienti ad alcuni servizi, tra cui **AWS Cloud9** e **AWS CodeCommit**. Molti team usano oggi repository Git esterni, come GitHub, GitLab o Bitbucket, collegati a CodePipeline tramite **AWS CodeConnections**. Conviene verificare nella documentazione lo stato attuale di questi servizi. I concetti di CI/CD e il ruolo di CodeBuild, CodeDeploy e CodePipeline restano comunque invariati e sono materia d'esame.

## 12.3 AWS Serverless Application Model (AWS SAM) con Java

### Che cos'è SAM

**AWS SAM** è un **framework open source** per costruire e distribuire applicazioni serverless. Ha due componenti.

La **specifica del template SAM** somiglia molto a un template CloudFormation, ma offre **tipi di risorsa serverless** dedicati e molto più concisi, come `AWS::Serverless::Function`, `AWS::Serverless::Api`, `AWS::Serverless::HttpApi` e `AWS::Serverless::SimpleTable`. Un template SAM è un'**estensione** di CloudFormation, identificata dalla riga `Transform: AWS::Serverless-2016-10-31`. Al momento del deploy viene trasformato in un normale template CloudFormation.

La **SAM CLI** automatizza molte attività dello sviluppatore: impacchetta le funzioni Lambda, crea l'infrastruttura e **simula in locale** Lambda e API Gateway, così da lavorare sul proprio computer prima di distribuire su AWS.

Il flusso è semplice: si scrive il template SAM, si esegue `sam build` e poi `sam deploy`. La CLI comunica con CloudFormation, che crea le risorse. La SAM CLI si usa quindi per **creare, compilare, testare e distribuire** le applicazioni, automatizzando tutto il packaging visto nei capitoli precedenti.

### Laboratorio: un'applicazione Java con SAM

L'applicazione di esempio è un'API che restituisce il numero di "stelle" che il repository dell'AWS SDK for Java ha su GitHub. Il template è il seguente:

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Transform: AWS::Serverless-2016-10-31
Description: Java SAM app - GitHub stars

Globals:
  Function:
    Timeout: 20
    MemorySize: 512

Resources:
  StarsApi:
    Type: AWS::Serverless::Api
    Properties:
      StageName: Prod
      Cors:
        AllowOrigin: "'*'"

  StarsFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: StarsFunction
      Handler: stars.App::handleRequest
      Runtime: java17
      Events:
        Stars:
          Type: Api
          Properties:
            RestApiId: !Ref StarsApi
            Path: /stars
            Method: get

Outputs:
  StarsApiUrl:
    Description: URL dell'endpoint API
    Value: !Sub "https://${StarsApi}.execute-api.${AWS::Region}.amazonaws.com/Prod/stars"
  StarsFunctionArn:
    Description: ARN della funzione Lambda
    Value: !GetAtt StarsFunction.Arn
```

Il template dichiara un'**API serverless** con le intestazioni **CORS** abilitate, perché l'API verrà chiamata da un front-end web su un altro dominio. Dichiara poi una **funzione Lambda** il cui codice si trova nella cartella `StarsFunction`: SAM la compilerà, la impacchetterà e la distribuirà. L'evento di tipo `Api` collega la funzione alla route `GET /stars`. La sezione **Outputs** restituisce, al termine del deploy, l'URL dell'API e l'ARN della funzione.

Il codice Java usa una libreria per GitHub per leggere il numero di stelle del repository, costruisce una stringa JSON e restituisce una risposta con il **body** e lo **statusCode** 200. In caso di errore restituisce un codice d'errore.

```java
public class App implements RequestHandler<APIGatewayProxyRequestEvent, APIGatewayProxyResponseEvent> {
    public APIGatewayProxyResponseEvent handleRequest(APIGatewayProxyRequestEvent input, Context context) {
        try {
            GitHub github = GitHub.connectAnonymously();
            int stars = github.getRepository("aws/aws-sdk-java-v2").getStargazersCount();
            String output = String.format("{ \"stars\": %d }", stars);
            return new APIGatewayProxyResponseEvent()
                    .withStatusCode(200)
                    .withHeaders(Map.of("Access-Control-Allow-Origin", "*"))
                    .withBody(output);
        } catch (IOException e) {
            return new APIGatewayProxyResponseEvent().withStatusCode(500).withBody("{}");
        }
    }
}
```

**Build e deploy.** Si compila con `sam build` e si distribuisce con la modalità guidata:

```bash
sam build
sam deploy --guided
```

La procedura guidata pone alcune domande:

- il **nome dello stack**: `java-sam-app`;
- la **Region**: `us-west-2`;
- se **confermare le modifiche** prima di applicarle, cioè mostrare il *change set*;
- se consentire a SAM di **creare i ruoli IAM** necessari: sì;
- se **disattivare il rollback**: no;
- un avviso: *"StarsFunction may not have authorization defined, Is this okay?"*. SAM segnala che l'API non è autenticata, e per l'esempio lo si accetta;
- se **salvare le risposte** in un file di configurazione, `samconfig.toml`, così che i deploy successivi richiedano solo `sam deploy`, e quale **profilo AWS** usare.

SAM carica il pacchetto in un **bucket S3** e passa il template a **CloudFormation**, che crea la funzione Lambda, l'API REST in API Gateway e il ruolo IAM. Sono le stesse operazioni fatte a mano nei capitoli precedenti, ora riassunte in un solo comando. Il flusso di lavoro dello sviluppatore diventa: modificare, compilare, verificare, distribuire.

**Verifica.** Al termine, l'output mostra l'URL dell'API. Chiamandolo con `curl` si ottiene il numero di stelle. C'è anche un piccolo front-end web, il motivo per cui si è abilitato CORS. Si inserisce l'URL dell'API nel codice del front-end, lo si copia in un bucket S3 configurato come sito statico, e ricaricando la pagina nel browser compare il numero di stelle.

**Sviluppo locale.** SAM permette di lavorare senza distribuire nulla:

```bash
# Invocare la funzione in un ambiente Lambda simulato
sam local invoke StarsFunction

# Avviare un API Gateway locale sulla porta 3000
sam local start-api
curl http://localhost:3000/stars
```

Con `sam local invoke` la funzione viene eseguita interamente sul computer locale. Con `sam local start-api` si avvia un'API locale sulla porta 3000, e la chiamata a `localhost:3000/stars` restituisce la risposta della funzione eseguita in locale. Il codice generato da SAM come punto di partenza include anche gli **unit test**, così si può lavorare in cicli rapidi di codice, test e debug prima di distribuire.

## 12.4 Distribuire con AWS SAM

### Prerequisiti

Per usare SAM servono tre cose:

- le **credenziali AWS**, cioè un profilo configurato. La procedura guidata chiede quale usare;
- **Docker**, facoltativo ma consigliato. Serve per due scopi. Il primo è la **build in container** (`sam build --use-container`), che simula l'ambiente Amazon Linux di Lambda ed è indispensabile quando ci sono **dipendenze binarie**, che devono essere compilate per quell'ambiente. Il secondo è la **simulazione locale** di Lambda e API Gateway;
- l'installazione della **SAM CLI**, che è semplice.

### Il ciclo di vita di un progetto SAM

| Comando | Funzione |
|---|---|
| `sam init` | **Inizializza** un nuovo progetto a partire da un *Quick Start template* (Java, .NET, Python, Node.js, Go…) |
| `sam build` | **Compila** l'applicazione e ne prepara gli artefatti |
| `sam local invoke` | Esegue una funzione in un **container Docker** che simula Lambda |
| `sam local start-api` | Avvia un **API Gateway locale** |
| `sam local generate-event` | Genera **eventi JSON di esempio** che simulano i payload di altri servizi AWS: un file caricato in S3, una richiesta da API Gateway, un messaggio SNS e molti altri |
| `sam deploy` | **Impacchetta** il codice, lo carica in S3 e passa il template a **CloudFormation**, che crea o aggiorna lo **stack** |
| `sam sync` | Sincronizza rapidamente le modifiche con l'ambiente cloud durante lo sviluppo |
| `sam logs` | Recupera i log delle funzioni da CloudWatch |

Il progetto del laboratorio era stato creato con `sam init`, scegliendo un Quick Start template, e poi adattato per interrogare GitHub. Il ciclo completo è quindi: `sam init`, una volta; poi `sam build` e `sam local invoke` o `start-api` per i test locali, senza che il codice abbia ancora lasciato il computer; infine `sam deploy`, che crea lo stack con l'API Gateway, la funzione Lambda e il **ruolo IAM** di cui la funzione ha bisogno.

```bash
# Generare un evento S3 di esempio e usarlo per invocare la funzione in locale
sam local generate-event s3 put --bucket notes-bucket --key photo.jpg > event.json
sam local invoke ThumbnailFunction --event event.json
```

## 12.5 Le strategie di deployment

### Le strategie per AWS Lambda

**AWS CodeDeploy** offre tre strategie per spostare il traffico da una versione all'altra di una funzione Lambda, tramite un alias.

La strategia **Canary** sposta il traffico in **due passi**. Prima una piccola percentuale, per esempio il **10%**, viene inviata alla nuova versione. Dopo un intervallo configurabile, per esempio 5, 10 o 15 minuti, il **restante 90%** passa anch'esso alla nuova versione. L'idea è eseguire la nuova funzione con un **campione del traffico di produzione** e verificarne le metriche prima di spostare tutto.

La strategia **Linear** sposta il traffico in **incrementi uguali** a intervalli regolari: 10%, poi 20%, poi 30%, e così via, per esempio ogni minuto o ogni tre minuti. Ci sono configurazioni predefinite e se ne possono creare di personalizzate.

La strategia **All-at-once** sposta **tutto il traffico** sulla nuova versione non appena è pronta.

Con SAM queste strategie si configurano direttamente nel template:

```yaml
StarsFunction:
  Type: AWS::Serverless::Function
  Properties:
    # ...
    AutoPublishAlias: live
    DeploymentPreference:
      Type: Canary10Percent5Minutes   # oppure Linear10PercentEvery1Minute, AllAtOnce
      Alarms:
        - !Ref StarsErrorsAlarm
```

`AutoPublishAlias` pubblica una nuova versione a ogni deploy e aggiorna l'alias `live`. `DeploymentPreference` stabilisce come CodeDeploy sposta il traffico sull'alias. Gli **allarmi** CloudWatch associati provocano un **rollback automatico** se, durante lo spostamento, la nuova versione genera errori.

### Le strategie per EC2: deployment in-place

Le applicazioni non vivono solo in Lambda. CodeDeploy distribuisce anche su **istanze EC2**, in due modalità. La prima è il **deployment in-place**: si usano le istanze esistenti e l'applicazione viene aggiornata direttamente su di esse. L'agente CodeDeploy su ogni istanza esegue gli script configurati nel manifest. Le opzioni principali sono tre:

- **All at once**: tutte le istanze vengono aggiornate più o meno contemporaneamente. È il metodo più rapido, ma l'applicazione può essere indisponibile durante l'aggiornamento;
- **One at a time**: gli script vengono eseguiti su un'istanza, si attende il completamento e si passa alla successiva. È il metodo più lento ma più prudente;
- **Half at a time**: metà delle istanze alla volta.

Il numero o la percentuale di istanze da aggiornare insieme sono **configurabili**. La scelta dipende dai **requisiti di disponibilità** e dalla tolleranza dell'applicazione ad avere **più versioni attive contemporaneamente** durante il deploy.

### Le strategie per EC2: deployment blue/green

La seconda modalità è il **deployment blue/green**, che coinvolge un **Application Load Balancer**. L'applicazione attuale gira sulle istanze **blue**, collegate al load balancer. Per il deploy vengono avviate **nuove istanze**, le **green**, su cui si installa la nuova versione. Anche qui ci sono due modalità.

Con **All at once**, l'applicazione viene installata su tutte le istanze green più o meno contemporaneamente. Poi tutte le green vengono collegate al load balancer e infine le blue vengono scollegate.

Con **One at a time**, l'applicazione viene installata su un'istanza green, che viene collegata al load balancer. Poi si passa alla successiva, e così via. Solo alla fine vengono scollegate tutte le istanze blue.

Il blue/green offre due vantaggi importanti.

Il primo è il **rollback**. Le istanze blue possono restare disponibili per un certo periodo. Se la nuova versione ha problemi, basta reindirizzare il traffico verso di esse.

Il secondo sono le **istanze immutabili**. Su ogni istanza green viene installata una sola applicazione, una sola volta. Quando l'istanza non serve più, viene eliminata, e non viene mai aggiornata. Si sa quindi esattamente che cosa contiene ogni istanza, e non restano a lungo in esercizio macchine con una storia di modifiche accumulate. È un vantaggio anche per la **sicurezza**.

| Strategia | Dove | Traffico o istanze | Vantaggi | Svantaggi |
|---|---|---|---|---|
| **Canary** | Lambda, ECS | 2 passi (es. 10% → 100%) | Prova la nuova versione su un campione reale | Due versioni attive per un breve periodo |
| **Linear** | Lambda, ECS | Incrementi uguali e regolari | Esposizione graduale e controllata | Rilascio più lungo |
| **All-at-once** | Lambda, ECS, EC2 | Tutto subito | Velocità | Nessuna rete di sicurezza graduale |
| **In-place** | EC2, on-premises | Istanze esistenti aggiornate a gruppi | Nessuna nuova infrastruttura | Rollback più lento, possibile riduzione di capacità |
| **Blue/green** | EC2, ECS, Lambda | Nuove istanze, poi spostamento del traffico | Rollback rapido, istanze immutabili | Richiede capacità doppia durante il deploy |

## 12.6 Verifica dei concetti

**"L'errore umano è uno dei rischi dei deployment tradizionali."** *Vero.* Chiunque abbia eseguito deploy manuali lo sa per esperienza.

**"L'Infrastructure as Code è una pratica DevOps importante."** *Vero.* Risponde direttamente al rischio dell'errore umano. Le modifiche all'infrastruttura si fanno nel template sotto controllo di versione, e si può sempre tornare a una configurazione precedente. Con le configurazioni fatte a mano nella Console, invece, capita di perdere una configurazione funzionante senza poterla ricostruire.

**"AWS CodePipeline può orchestrare AWS CodeCommit, AWS CodeBuild e AWS CodeDeploy."** *Vero.* CodePipeline definisce la pipeline: il commit avvia la fase di build, che a sua volta porta al deploy.

**"I template AWS SAM sono un'estensione dei template AWS CloudFormation."** *Vero.* I template CloudFormation si possono scrivere direttamente, ma SAM ne semplifica la scrittura per le risorse serverless. Al deploy il template SAM viene convertito in un template CloudFormation.

**"Si può usare AWS SAM per costruire un'applicazione serverless solo se è scritta in Node.js, Java, Python o Go."** *Falso.* SAM supporta tutti i runtime di Lambda, tra cui .NET e Ruby, oltre ai runtime personalizzati e alle immagini di container.

**"Il comando `sam build` inizializza un nuovo progetto AWS SAM con i parametri richiesti."** *Falso.* Il progetto si inizializza con **`sam init`**. `sam build` compila e prepara gli artefatti di un progetto esistente.
