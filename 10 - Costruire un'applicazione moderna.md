# Parte V – Architetture moderne e identità degli utenti

# Capitolo 10 – Costruire un'applicazione moderna

## 10.1 Obiettivi del capitolo

Con S3, DynamoDB, Lambda e API Gateway si hanno tutti i mattoni di un'applicazione cloud. Questo capitolo fa un passo indietro e guarda l'**architettura** nel suo insieme. Vengono descritti i limiti delle architetture tradizionali, in particolare del **monolite**, e i vantaggi delle architetture a **microservizi**. La scelta architetturale è spesso già fatta quando uno sviluppatore entra in un progetto: può capitare di dover lavorare con un monolite esistente e di doverlo far evolvere verso i microservizi.

Il capitolo presenta poi gli **approcci di progettazione** dei microservizi e i benefici della scomposizione di un'applicazione in servizi indipendenti. Descrive i **passi per disaccoppiare un monolite**, cioè che cosa spostare per primo, ricordando che in alcuni casi un monolite resta una scelta giustificata. Dopo un approfondimento sul **modello operativo serverless**, si arriva all'orchestrazione delle funzioni Lambda con **AWS Step Functions**. Questo servizio permette di "disegnare" la sequenza di operazioni di un workflow e di decidere, passo dopo passo, quale strada prendere. Un laboratorio finale mostra un'applicazione reale costruita con Step Functions.

## 10.2 Le applicazioni moderne: dal monolite ai microservizi

### I pilastri dello sviluppo moderno

Lo sviluppo moderno di applicazioni nel cloud si basa su alcuni pattern ricorrenti.

Il primo sono i **microservizi**. L'applicazione viene scomposta in servizi indipendenti, con vantaggi per il **deployment**, per lo **scaling** e anche per l'**organizzazione dei team** di sviluppo.

Il secondo è la **delivery del software tramite DevOps**, che applica alle operazioni le pratiche di automazione nate nel mondo dello sviluppo. Il ciclo DevOps è un anello senza fine: *plan → code → build → test → release → deploy → operate → monitor*, e poi di nuovo *plan*. Supponiamo di dover sviluppare una nuova funzionalità. La si scrive, la si compila, la si testa, la si rilascia e la si distribuisce. Una volta in produzione la si gestisce e la si monitora per capire come si comporta. A quel punto si torna in cima al ciclo per pianificare il rilascio successivo. In un ambiente DevOps si lavora per **sprint brevi**, percorrendo continuamente questo ciclo con l'obiettivo di portare rapidamente funzionalità ai clienti. Il deploy in particolare va **automatizzato**: rilasciare un'applicazione non dovrebbe fare paura, ma ridursi a premere un pulsante per mettere in produzione una funzionalità scritta il giorno prima.

Il terzo è il **serverless**. Molti servizi AWS sono serverless, nel senso che lo sviluppatore **non gestisce l'infrastruttura sottostante**, che gli viene astratta. Con Lambda, per esempio, si consegna una funzione senza preoccuparsi della configurazione degli host che la eseguono.

### Monolite e microservizi

Quando si progetta un'applicazione, o quando si eredita un'applicazione legacy, si incontra di solito uno di due stili architetturali.

Il **monolite** è un'applicazione che **fa tutto**: un unico grande eseguibile, al cui interno i vari servizi comunicano con **chiamate di funzione** tra procedure. Non è progettato come applicazione distribuita.

I **microservizi** scompongono l'applicazione in servizi indipendenti. Ciò che prima era una chiamata di funzione diventa una **chiamata di rete**, e l'applicazione diventa **distribuita**.

Il monolite non è sempre una scelta sbagliata. Per alcune applicazioni è giustificato: dipende da quanto rapidamente si vuole arrivare in produzione e da quanto si prevede che l'applicazione crescerà. Però un monolite può diventare **ingestibile molto in fretta**, quando cresce oltre le dimensioni per cui aveva senso. Molti sviluppatori, ripensando ai progetti meno amati della propria carriera, li identificano con quelli in cui erano meno efficaci, quelli in cui portare una funzionalità in produzione era lento e faticoso.

### Le caratteristiche di un monolite

Un'applicazione monolitica ha alcune caratteristiche tipiche:

- **Fa tutto**: tutte le funzionalità stanno nello stesso eseguibile.
- **È fortemente accoppiata** (*tightly coupled*): i servizi interni si integrano tramite chiamate di procedura.
- **Conserva lo stato nell'istanza di runtime**: questo ne limita la scalabilità e crea un rischio. Se il runtime si guasta, lo stato va perso con lui. Se in memoria c'è il carrello di un cliente, non lo si vuole perdere. Lo stato andrebbe esternalizzato.
- **Usa un unico stack tecnologico**: un solo linguaggio di programmazione e un solo tipo di persistenza. A volte è una scelta difendibile, ma spesso si vorrebbe usare per ogni servizio la tecnologia più adatta.
- **Ha opzioni limitate per i dati**: un solo database per tutte le esigenze.
- **È organizzato per livelli tecnologici** anziché per funzionalità di business.
- **Ha un deploy complesso**: bisogna distribuire tutta l'applicazione in un colpo solo, con il rischio di rompere qualcosa che non c'entra. Chi ha corretto un bug in una pagina ed è "sicurissimo" di non aver toccato la ricerca, ha spesso dovuto ricredersi. Se le parti fossero isolate, si potrebbe essere certi della portata di ogni modifica.
- **Ha calendari di rilascio rigidi**: una correzione pronta può dover aspettare la settimana o il mese successivo perché un altro team ha a sua volta delle modifiche in sospeso. Servono coordinamento tra team e finestre di rilascio comuni, e tutto rallenta.
- **Comporta un forte carico operativo**: più cose contiene l'applicazione, più ogni attività operativa diventa complessa.

### Le caratteristiche dei microservizi

Scomponendo il monolite in microservizi si ottengono caratteristiche opposte:

- **Servizi funzionali minimi**: ogni servizio contiene solo ciò che gli serve.
- **Deployment separati**: se c'è un bug in un servizio, si distribuisce solo quel servizio, con fiducia.
- **Dati "fit for purpose"**: ogni servizio sceglie la persistenza più adatta. Un team usa DynamoDB, un altro, che ha bisogno di operazioni SQL, usa un database relazionale.
- **Organizzazione per capacità di business**: è anche il modo in cui si strutturano i team. Ogni servizio è affidato a un team con le competenze di business più adatte, e le persone possono comunque spostarsi tra i servizi.
- **Libertà di scelta tecnologica**: un servizio può essere scritto in Python, un altro in Java.
- **Modello operativo serverless e automatizzato**: per servizi composti da API Gateway e Lambda esistono ottimi strumenti, come SAM, per automatizzarne distribuzione ed esercizio.

### I benefici dei microservizi

I vantaggi concreti di un'architettura a microservizi sono numerosi.

**Agilità di sviluppo.** Un nuovo sviluppatore che arriva in un team con un'applicazione monolitica deve capirla quasi tutta prima di essere produttivo. Con i microservizi il perimetro da imparare è più piccolo: si parte da un servizio e, col tempo, se ne imparano altri. Si è produttivi fin dal primo giorno.

**Deployment rapidi e indipendenti.** Si distribuisce un solo servizio, con la ragionevole certezza di non rompere gli altri. Questo però impone di pensare al **versioning delle API**: altri servizi usano quell'API, e bisogna chiedersi se le modifiche restano **retrocompatibili**.

**Maggiore sicurezza.** Poiché il perimetro di ogni servizio è ridotto, si può analizzare la sicurezza servizio per servizio anziché su un unico blocco enorme.

**Organizzazione per capacità.** I team sono piccoli, focalizzati su un servizio, e liberi di spostarsi su altri.

**Maggiore disponibilità e resilienza.** Si può progettare l'applicazione in modo che il guasto di un microservizio non la blocchi tutta. Pensiamo a un servizio di streaming televisivo: se il motore di raccomandazioni si guasta, i film continuano a essere trasmessi e semplicemente non compaiono più i suggerimenti. È un approccio a **compartimenti stagni** (*bulkhead*), che impedisce a un guasto di propagarsi.

**Scaling indipendente.** Un monolite si scala tutto insieme. Con i microservizi, se è il servizio di autenticazione a essere sotto carico, si scala solo quello.

### I principi di progettazione

Nel progettare un'applicazione a microservizi conviene seguire alcuni principi.

**Favorire la sperimentazione con una cultura della responsabilità** (*ownership*). È un approccio Agile e DevOps: quando il perimetro è piccolo è più facile sperimentare, e si può persino riprogettare da zero un microservizio. Se AWS rilascia una nuova funzionalità interessante, la si può adottare in un servizio restando compatibili verso l'esterno, e l'aggiornamento esce rapidamente, magari con prestazioni migliori o costi minori.

**Componentizzare tramite interfacce ben definite.** Le API sono il contratto tra i servizi. Prima di modificare un'API bisogna sapere chi la usa e decidere se restare retrocompatibili, se è ammissibile rompere la compatibilità o se occorre aggiornare i client. I servizi sono indipendenti, ma comunicano tra loro, e le loro API devono restare compatibili.

**Applicare il domain-driven design.** Ogni microservizio ha una funzionalità circoscritta a un **dominio** preciso. Questo spinge a scrivere codice migliore: un codice che fa una cosa sola e la fa bene è di solito migliore di un grande monolite di codice interconnesso.

**Definire i punti di integrazione con gli altri domini.** Capita che un servizio nato per un problema risolva anche il problema di un altro team, che diventa così un nuovo cliente. Bisogna stabilire in che modo i servizi interagiscono, sapendo che ora si tratta di comunicazioni di rete in un'applicazione distribuita.

### La comunicazione tra microservizi

Ci sono due modalità di comunicazione fondamentali.

La **comunicazione sincrona** (*request/response*) è la più diretta. Ciò che prima era una chiamata di funzione per recuperare un cliente diventa una chiamata HTTP a un'API: *"servizio clienti, mi dai il cliente 99?"*. La risposta torna al chiamante, che nel frattempo attende.

La **comunicazione asincrona** (*event-driven*) si usa quando l'operazione richiesta non deve avvenire subito. Un esempio è l'email di benvenuto inviata a un nuovo utente dopo la registrazione. Non serve inviarla nell'istante stesso: può aspettare cinque minuti, o anche un giorno, e soprattutto non deve bloccare la registrazione. L'applicazione pubblica un **evento**, per esempio scrivendo un messaggio in una **coda**, e il servizio di invio email, che legge da quella coda, lo elabora quando può. L'utente non resta ad aspettare che il codice invii l'email, e l'applicazione è più reattiva.

### Disaccoppiare un monolite

Supponiamo di aver deciso di scomporre un monolite. Da dove si comincia? Le indicazioni sono quattro.

**Iniziare in piccolo.** Se l'approccio è nuovo per il team, conviene partire con un esperimento poco complesso, per esempio un servizio con poco carico o con poco codice. Lo si estrae, si aggiorna il monolite perché lo chiami e si raccolgono le **metriche**: le prestazioni sono buone? Si chiede anche un riscontro al team: gli strumenti di sviluppo scelti sono comodi? Così si costruisce fiducia, e poi si passa a servizi via via più impegnativi.

**Minimizzare le dipendenze verso il monolite.** Idealmente si sceglie un servizio con dipendenze **in una sola direzione**. Se il monolite deve leggere i dati dei clienti, si crea un servizio clienti e si aggiorna il monolite perché lo usi. Quando invece la dipendenza è bidirezionale, si può creare temporaneamente un servizio che rappresenta il monolite, una **facciata** (*facade*) davanti al monolite stesso.

**Separare presto le dipendenze complesse.** In ogni applicazione ci sono componenti da cui dipendono molti altri, e che a loro volta hanno molte dipendenze. In un'architettura distribuita queste dipendenze diventano **traffico di rete**, quindi bisogna progettare API che lo riducano al minimo. A volte serve recuperare un solo cliente, a volte un elenco, e l'API deve supportare entrambi i casi. Più dipendenze ci sono tra microservizi, più chiamate di rete si generano. Vanno quindi monitorate le dipendenze e la **latenza** introdotta da una richiesta che attraversa molti servizi.

**Disaccoppiare per prime le funzionalità che cambiano spesso.** Sono quelle che traggono il massimo beneficio dalla scalabilità e dal deploy indipendenti, quindi conviene ottenere subito questi benefici.

### Lo strangler pattern

Un modello consolidato per la migrazione è lo **strangler pattern**, il pattern "strangolatore". Il nome evoca una pianta rampicante che avvolge lentamente un albero fino a sostituirlo. Si procede **per fasi**: si estrae una funzionalità dal monolite, la si trasforma in microservizio e la si distribuisce, si instrada verso di essa il traffico relativo, poi si passa alla funzionalità successiva, finché il monolite non è stato interamente scomposto.

> **Box – Le "cicatrici di battaglia" del codice in produzione**
>
> Una lezione importante della migrazione: il codice che gira in produzione da anni porta con sé delle **cicatrici di battaglia**. Un bug scoperto cinque anni fa è stato corretto nel monolite, e la correzione è lì, magari senza documentazione. Quando si riscrive una funzionalità in un microservizio bisogna preservare tutta questa conoscenza accumulata, altrimenti si reintroducono problemi già risolti. Test di regressione accurati e un confronto attento con il comportamento del vecchio codice sono indispensabili.

### L'impatto sulla delivery del software

La differenza tra i due mondi si vede bene nel ciclo DevOps. Con il monolite, per correggere un bug nella pagina di ricerca bisogna compilare, testare, rilasciare e monitorare **l'intero monolite**. La portata della modifica è enorme e bisogna essere certi che non rompa altre funzionalità. Altri team potrebbero avere rilasci in sospeso: si può creare un branch dall'ultima versione distribuita, ma la complessità cresce, e si moltiplica per il numero di team che lavorano sullo stesso monolite.

Con i microservizi i team lavorano in modo **indipendente**. La correzione riguarda solo il servizio di ricerca, che viene compilato, testato, rilasciato e monitorato da solo. Il bug fix arriva in produzione molto più rapidamente.

## 10.3 Il modello operativo serverless

### Che cosa scompare dalle responsabilità dello sviluppatore

In un modello **tradizionale**, in cui si gestiscono direttamente gli host, le attività operative sono molte:

- **provisioning** degli host;
- **patching del sistema operativo**, un'attività continua, perché le vulnerabilità vengono scoperte di continuo;
- **installazione** della piattaforma applicativa e **deploy** delle applicazioni;
- configurazione dell'**auto scaling** e del **load balancing**;
- **monitoraggio e manutenzione** delle applicazioni e degli host.

Con un modello **serverless** molte di queste voci scompaiono: sono tutte quelle legate agli host. Con Lambda la piattaforma applicativa è già pronta nel runtime. Lo sviluppatore si limita a **costruire e distribuire l'applicazione**, concentrandosi sui problemi di business da risolvere, e non configura né auto scaling né load balancing. L'applicazione opera a un **livello di astrazione più alto**. Resta da **monitorare e mantenere** l'applicazione, ma i servizi serverless aiutano anche in questo: molti pubblicano automaticamente metriche in CloudWatch, da cui si costruiscono dashboard sulle prestazioni.

### I benefici del serverless

- **Nessun server da gestire**: i server esistono, ma li provisiona e li gestisce AWS.
- **Scalabilità automatica**: molti servizi serverless scalano da soli. Alcuni richiedono una certa messa a punto, ma molto meno di quanto servirebbe con host gestiti in proprio.
- **Raramente si pagano risorse inattive**: dipende dal servizio, ma con Lambda, per esempio, si paga il tempo di esecuzione, e quando nulla è in esecuzione non si paga.
- **Disponibilità e tolleranza ai guasti integrate**: con EC2 è lo sviluppatore a progettare l'alta disponibilità, distribuendo le istanze su più Availability Zone dietro un load balancer. Con Lambda se ne occupa il servizio.

### Il catalogo dei servizi serverless

AWS offre servizi serverless per ogni livello di un'applicazione:

| Area | Servizi |
|---|---|
| **Calcolo** | AWS Lambda; AWS Fargate, che esegue container in modo serverless integrandosi con ECS ed EKS |
| **API** | Amazon API Gateway; AWS AppSync, per API GraphQL gestite |
| **Messaggistica e flussi** | Amazon SQS (code); Amazon SNS (notifiche pub/sub); Amazon EventBridge (eventi); Amazon Kinesis (stream di dati) |
| **Storage e dati** | Amazon S3; Amazon DynamoDB |
| **Analisi** | Amazon Athena, per interrogare con SQL i dati in S3 |
| **Orchestrazione** | AWS Step Functions |

Prendiamo SQS come esempio. Si definisce una coda, si scrivono messaggi nella coda e altre applicazioni li consumano, senza installare né gestire alcun software di accodamento: se ne occupa il servizio. **Step Functions** permette di orchestrare i molti passi di un processo che coinvolge diversi servizi AWS, ed è l'argomento del paragrafo 10.5.

## 10.4 L'applicazione Polly Notes come architettura serverless

Rivedendo il diagramma di Polly Notes con gli occhi di questo capitolo, salta all'occhio un dettaglio: **non c'è nemmeno un server**. Tutto è serverless.

- **Amazon Cognito** gestisce l'autenticazione tramite configurazione, con al più un po' di codice Lambda per personalizzare il flusso (Capitolo 11).
- **Amazon S3** conserva i file audio e ospita il front-end web: è un servizio AWS che si occupa dello storage, senza host da gestire.
- **AWS Lambda** esegue le funzioni. I server sottostanti vanno aggiornati e il loro sistema operativo va mantenuto, ma se ne occupa AWS.
- **Amazon DynamoDB** memorizza gli item delle note. Si crea una tabella e si chiamano le API per inserire e leggere gli item, senza configurare alcun server.
- **Amazon Polly** riceve un testo e restituisce un file audio, senza server da gestire.

Tutti questi servizi operano a un **livello di astrazione più alto**: lo sviluppatore interagisce solo con le loro API e ottiene esattamente ciò che gli serve, senza occuparsi dell'hardware sottostante.

## 10.5 L'orchestrazione con AWS Step Functions

### I workflow nelle applicazioni

Le applicazioni avviano spesso **workflow** che coinvolgono molti sistemi, e talvolta anche persone. Un esempio familiare è un ordine su un sito di e-commerce. Il cliente aggiunge un prodotto al carrello e completa l'ordine, e da lì parte un processo articolato:

1. l'ordine viene creato;
2. la carta di credito viene addebitata;
3. il sistema di magazzino verifica e aggiorna l'inventario;
4. un operatore o un robot preleva l'articolo e lo inscatola;
5. sulla scatola viene applicata un'etichetta;
6. se la consegna è all'estero, il processo segue un **ramo diverso**.

Ci sono molti passi e diversi punti di diramazione. **AWS Step Functions** è il servizio che permette di progettare questi workflow e di coordinarne le attività e lo stato mentre il processo avanza.

### Un esempio di architettura

Consideriamo un'applicazione che invia una richiesta POST ad API Gateway. Come visto nel Capitolo 9, API Gateway può passare una richiesta direttamente a un servizio AWS, e **Step Functions è uno di questi**: la richiesta avvia un'**esecuzione** del workflow.

A ogni esecuzione è associato un **documento JSON** che viene passato da uno stato all'altro. Alcuni compiti possono avvenire **in parallelo**. Per esempio, una funzione Lambda esegue una logica personalizzata mentre un'altra rileva il **sentiment** di un testo chiamando **Amazon Comprehend**. Il workflow stabilisce di eseguirle contemporaneamente, attendere che entrambe terminino e poi passare allo stato successivo.

Ogni stato può integrarsi con un servizio AWS. Per esempio si può configurare, **senza scrivere codice**, che al termine delle attività parallele venga inserito un item in una tabella DynamoDB. Più avanti il workflow può dover prendere una **decisione**, l'equivalente di un `if` o di uno `switch`: in base alla risposta, la transazione era valida o no?, si passa a stati diversi. Molto spesso gli stati sono implementati con funzioni Lambda.

### I tipi di flusso

Un workflow di Step Functions, cioè una **state machine**, supporta diversi tipi di flusso:

- **Sequenziale**: terminato uno stato si passa al successivo. Verificata la carta di credito, si prosegue.
- **Diramazione** (*branching*, stato `Choice`): in base al risultato di uno stato si sceglie lo stato successivo. Si parla di *macchina a stati* proprio perché il workflow definisce le **transizioni** tra uno stato e l'altro.
- **Retry**: in caso di errore lo stato viene ritentato, con un numero di tentativi e intervalli configurabili.
- **Gestione degli errori** (*catch*): se uno stato fallisce, per esempio una funzione Lambda incontra un problema transazionale aggiornando un record, il workflow passa a uno stato dedicato, come l'invio di un'email a un operatore perché verifichi l'ordine.
- **Attesa** (stato `Wait`): il workflow si ferma per un tempo stabilito. È utile, per esempio, quando bisogna aspettare che un processo batch esterno abbia completato il suo lavoro, anche se un'integrazione migliore renderebbe spesso superflua l'attesa.
- **Parallelo** (stato `Parallel`): più rami vengono eseguiti contemporaneamente.

### I benefici di Step Functions

Il vantaggio più importante è che **le funzioni Lambda non devono più occuparsi del workflow**. Ogni funzione può essere completamente indipendente dalla sequenza in cui è inserita, perché la sequenza la gestisce Step Functions. Gli altri benefici sono:

- **Coordinare i componenti di applicazioni distribuite con un workflow visuale**: inventario, spedizioni, pagamenti non stanno sullo stesso host, alcuni sono lenti e altri veloci.
- **Gestire i fallimenti**, con retry configurabili. Un servizio sotto carico può rifiutare temporaneamente le richieste, e si ritenta finché non va a buon fine.
- **Migliorare le prestazioni** eseguendo in parallelo ciò che è indipendente.
- **Integrare i servizi AWS** direttamente come passi del workflow.
- **Garantire l'osservabilità**. Il documento che attraversa gli stati viene **registrato a ogni passo**. Se qualcosa va storto, nella Console si vede fino a dove è arrivato il workflow e com'era il documento in ogni momento, e si risale con precisione al punto del problema. Anche questo è un compito che le funzioni non devono più svolgere.

### L'Amazon States Language

I workflow si definiscono con l'**Amazon States Language (ASL)**, un formato JSON che descrive il grafo degli stati. A partire dal JSON la Console genera una **visualizzazione** del grafo: quali stati portano a quali, quali dipendono da altri, quali sono le alternative di una scelta. Per workflow di grandi dimensioni il diagramma è di fatto una **documentazione automatica**: alla domanda "come funziona il nostro processo di spedizione?" si risponde con un'immagine.

Per progettare il JSON c'è anche **Workflow Studio**, un editor visuale. Durante l'esecuzione, nel diagramma della Console i riquadri degli stati **cambiano colore** man mano che il workflow avanza. Per ogni stato si vedono quando una funzione è stata pianificata, il documento che le è stato passato e la risposta ricevuta. È uno strumento molto comodo per la diagnosi, perché permette di saltare a qualunque punto del workflow e vedere esattamente quali erano i parametri.

### Elaborare input e output

Ogni stato può filtrare e trasformare il documento che riceve e quello che restituisce, tramite una serie di campi applicati in sequenza:

1. **`InputPath`** seleziona la parte del documento in ingresso che interessa allo stato;
2. **`Parameters`** costruisce i parametri da passare al task, per esempio l'input della funzione Lambda;
3. **`ResultSelector`** sceglie i campi del risultato da tenere, se serve;
4. **`ResultPath`** stabilisce **dove inserire il risultato** del task nel documento originale: per esempio `"$.result"` lo aggiunge in un nuovo campo, mentre `null` scarta il risultato e passa avanti l'input intatto;
5. **`OutputPath`** seleziona la parte del documento risultante che diventa l'**output dello stato** e prosegue verso lo stato successivo.

```json
"CalculateScores": {
  "Type": "Task",
  "Resource": "arn:aws:states:::lambda:invoke",
  "InputPath": "$.game",
  "Parameters": {
    "FunctionName": "scores-function",
    "Payload.$": "$"
  },
  "ResultSelector": { "scores.$": "$.Payload" },
  "ResultPath": "$.game.lastScores",
  "OutputPath": "$",
  "Next": "IsGameOver"
}
```

### Le integrazioni di servizio

Step Functions supporta molti servizi. Le funzioni Lambda sono le più usate, ma si possono anche eseguire task su **ECS**, inviare messaggi con **SNS**, scrivere in **DynamoDB**, lanciare job di **SageMaker** e molto altro: molti servizi hanno tipi di task già pronti. Esiste inoltre l'**integrazione AWS SDK**, che permette di chiamare **qualunque API** di qualunque servizio AWS, controllando i parametri passati e ciò che si estrae dalla risposta.

> **Box – Activity e workflow Standard ed Express**
>
> Oltre ai servizi AWS, un task può essere eseguito da un **worker esterno** tramite le **Activity**. Il workflow si ferma su uno stato di tipo Activity e attende che un processo in esecuzione ovunque, per esempio su un'istanza EC2 o in un data center, chieda il lavoro (`GetActivityTask`), lo esegua e restituisca il risultato (`SendTaskSuccess`). È il modo classico per coinvolgere in un workflow codice che non gira su servizi serverless. Per chiamare in parallelo due servizi su EC2 si usa quindi uno stato **Parallel** con due rami, ciascuno con la propria Activity.
>
> Esistono poi due tipi di workflow. Gli **Standard** durano fino a un anno, garantiscono un'esecuzione *exactly-once* e conservano la cronologia completa. Gli **Express** durano al massimo cinque minuti, sono pensati per volumi elevatissimi di eventi e costano meno.

## 10.6 Laboratorio: orchestrare funzioni Lambda con Step Functions

L'applicazione di questo laboratorio è un **quiz sulle capitali** a cui più giocatori partecipano in tempo reale. Si avvia la partita e compaiono le domande: la capitale del Canada? Ottawa, e il punteggio nell'angolo sale di 10. La Bulgaria? Sofia. La Mongolia? Ulan Bator. L'Angola? Luanda. Un secondo giocatore, collegato alla stessa partita, vede le stesse domande e i punteggi aggiornati.

### La state machine

A far funzionare il gioco è una **state machine** di Step Functions. Ogni partita avvia una **nuova esecuzione**, e il documento iniziale contiene **tutte le domande e le risposte** della partita. Nella Console, con *Edit*, si vedono affiancati il codice ASL e il diagramma. Una versione semplificata della definizione è la seguente:

```json
{
  "StartAt": "Question",
  "States": {
    "Question": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789012:function:trivia-question",
      "ResultPath": null,
      "Next": "QuestionWait"
    },
    "QuestionWait": {
      "Type": "Wait",
      "SecondsPath": "$.waitSeconds",
      "Next": "Scores"
    },
    "Scores": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789012:function:trivia-scores",
      "Next": "IsGameOver"
    },
    "IsGameOver": {
      "Type": "Choice",
      "Choices": [
        { "Variable": "$.gameOver", "BooleanEquals": true, "Next": "GameOver" }
      ],
      "Default": "Question"
    },
    "GameOver": {
      "Type": "Succeed"
    }
  }
}
```

Gli stati sono cinque.

**Question** è un task che invoca una **funzione Lambda**: è la funzione che invia ai client dei giocatori la nuova domanda. Il suo `ResultPath` è `null`, cioè "passa allo stato successivo l'intero documento ricevuto, così com'è".

**QuestionWait** è uno stato `Wait`: il workflow si ferma qualche secondo per dare ai giocatori il tempo di rispondere. Invece di un valore fisso usa `SecondsPath`, cioè legge la durata dell'attesa da un campo del documento.

**Scores** è un'altra funzione Lambda. Legge le risposte dei giocatori, salvate in una tabella **DynamoDB**, calcola i punteggi e li notifica a tutti. Aggiorna inoltre nel documento la **posizione della domanda corrente**, un iteratore, e indica se la partita è finita.

**IsGameOver** è uno stato `Choice`: se `gameOver` è vero si passa a **GameOver**, altrimenti si torna a **Question**. Il diagramma mostra chiaramente il **ciclo**: domanda, attesa, punteggi, controllo, e di nuovo domanda, finché la partita non termina.

**GameOver** chiude l'esecuzione.

### La cronologia dell'esecuzione

Nella cronologia dell'esecuzione appena giocata si ripercorre tutto il workflow:

- all'evento *ExecutionStarted* si vede il documento iniziale con domande e risposte, il campo con i secondi di attesa e un blocco **Iterator** che tiene traccia della posizione nella lista delle domande;
- per ogni stato Lambda si vedono gli eventi *LambdaFunctionScheduled*, *LambdaFunctionStarted* e *LambdaFunctionSucceeded*, con l'output della funzione;
- all'uscita di ogni task si vede il documento che ha lasciato lo stato. Dopo **Scores**, per esempio, la posizione della domanda è stata incrementata, così lo stato **Question** successivo sa quale domanda porre;
- alla fine, l'output di **Scores** contiene l'indicazione di partita conclusa, lo stato **IsGameOver** si limita a prendere la decisione e l'esecuzione entra in **GameOver**.

La lezione del laboratorio è chiara: **le funzioni Lambda non sanno in quale punto del workflow si trovano né quale passo verrà dopo**. Lo stato del gioco e la sequenza delle operazioni sono interamente gestiti da Step Functions, e ogni funzione resta semplice, focalizzata e riutilizzabile.

## 10.7 Verifica dei concetti

**"Con un'architettura serverless i team si concentrano sull'applicazione, mentre il cloud provider gestisce l'infrastruttura."** *Vero.* L'infrastruttura sottostante è gestita da AWS, e i team possono dedicarsi a ottimizzare l'applicazione e a generare valore di business.

**"Lo sviluppo moderno di applicazioni è un approccio alla progettazione, costruzione e gestione delle applicazioni nel cloud."** *Vero.*

**"In Step Functions tutto il lavoro della state machine viene svolto da funzioni Lambda."** *Falso.* Le funzioni Lambda sono un'opzione comune, ma ci sono anche le integrazioni con molti altri servizi AWS, l'integrazione AWS SDK, che permette di chiamare qualunque API, e le Activity, che possono essere eseguite anche da servizi su EC2.

**"Le state machine di Step Functions supportano diramazioni, esecuzioni parallele, retry e gestione degli errori, e l'avvio di task."** *Vero.* Il quiz delle capitali ne mostra diversi: decisioni, attese, cicli e task.

**"I microservizi sono debolmente accoppiati, hanno funzionalità indipendenti e sono guidati da eventi o da API."** *Vero.* È l'opposto del monolite, fortemente accoppiato: i servizi indipendenti sono esposti tramite API o comunicano tramite eventi.

**"I workflow di Step Functions gestiscono parallelizzazione e integrazione dei servizi, ma fallimenti e retry vanno gestiti a parte."** *Falso.* Retry e gestione degli errori sono funzionalità integrate di Step Functions e si configurano direttamente nella definizione del workflow.
