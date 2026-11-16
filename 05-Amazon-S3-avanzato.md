# Capitolo 5
# Amazon S3 Avanzato: Elaborazione, Automazione e Gestione Operativa degli Oggetti

## Introduzione

Dopo aver compreso i fondamenti di Amazon S3, è necessario approfondire le attività operative che caratterizzano il lavoro quotidiano degli sviluppatori. Nella maggior parte dei progetti reali, il semplice caricamento di file rappresenta solo una piccola parte delle funzionalità richieste. Le applicazioni moderne devono creare bucket, verificare la disponibilità delle risorse, trasferire grandi quantità di dati, elaborare oggetti, automatizzare attività massive e distribuire contenuti web.

Questo capitolo analizza le principali operazioni avanzate di Amazon S3 e le tecniche utilizzate per realizzare applicazioni cloud-native efficienti e scalabili.

---

## Gestione dei bucket mediante API

Amazon S3 espone API che consentono di amministrare completamente bucket e oggetti tramite codice. Questo approccio permette di automatizzare attività che, in ambienti enterprise, risulterebbero impraticabili se eseguite manualmente tramite console.

Le operazioni più comuni comprendono la creazione di bucket, l'eliminazione di bucket non più necessari, la modifica delle configurazioni di sicurezza, l'attivazione del versioning, la gestione della crittografia e la configurazione dell'hosting statico.

Uno degli aspetti più importanti riguarda la scelta del nome del bucket. A differenza di molte altre risorse AWS, il nome di un bucket deve essere globalmente univoco. Ciò significa che due account AWS differenti non possono possedere bucket con lo stesso nome.

---

## Verificare l'esistenza di un bucket con HeadBucket

Prima di creare un bucket è buona pratica verificarne l'esistenza.

A questo scopo viene utilizzata l'API HeadBucket. Tale operazione utilizza il metodo HTTP HEAD e restituisce soltanto i metadati associati alla risorsa senza trasferire contenuti aggiuntivi.

L'analisi del codice di risposta permette di comprendere immediatamente lo stato della risorsa:

- 200 indica che il bucket esiste ed è raggiungibile.
- 404 indica che il bucket non esiste.
- 403 indica un problema di autorizzazioni.
- 400 può indicare un accesso alla regione errata.

Questa tecnica viene frequentemente utilizzata nelle procedure di provisioning automatico e nelle pipeline DevOps.

---

## Attendere la disponibilità delle risorse con i Waiter

La creazione di una risorsa cloud non è sempre istantanea.

Per questa ragione gli SDK AWS includono i cosiddetti Waiter. Un Waiter esegue automaticamente richieste di verifica fino a quando una determinata condizione non viene soddisfatta.

Nel caso di Amazon S3, un Waiter può attendere che un bucket diventi disponibile prima di consentire l'esecuzione delle operazioni successive.

L'utilizzo dei Waiter elimina la necessità di implementare manualmente cicli di polling e riduce la complessità del codice applicativo.

---

## Operazioni fondamentali sugli oggetti

Le applicazioni interagiscono normalmente con gli oggetti attraverso una serie di operazioni standard.

Tra le attività più frequenti troviamo il caricamento dei file, il download dei contenuti, la lettura dei metadati, la cancellazione degli oggetti e la copia tra bucket differenti.

Dal punto di vista architetturale, queste operazioni costituiscono il nucleo della maggior parte dei sistemi documentali, delle piattaforme media e delle applicazioni serverless.

---

## Caricamento degli oggetti

L'upload rappresenta una delle operazioni più utilizzate di Amazon S3.

Quando le dimensioni dell'oggetto sono limitate, è possibile effettuare un caricamento diretto mediante una singola richiesta PUT. Questo approccio risulta semplice e immediato.

Tuttavia, quando le dimensioni aumentano, entrano in gioco considerazioni legate a prestazioni, resilienza e affidabilità del trasferimento.

Amazon S3 supporta oggetti fino a 5 terabyte, una soglia estremamente elevata che consente di ospitare contenuti multimediali, backup, dataset e archivi di grandi dimensioni.

---

## Multipart Upload

Per file particolarmente grandi è consigliabile utilizzare Multipart Upload.

Con questa tecnica il file viene suddiviso in parti indipendenti che possono essere trasferite separatamente. Una volta completato il caricamento, S3 ricompone automaticamente il contenuto originale.

I vantaggi sono numerosi.

Innanzitutto, in caso di errore non è necessario ritrasferire l'intero file. È sufficiente ritentare la parte fallita. Inoltre, le varie parti possono essere trasferite in parallelo, migliorando significativamente le prestazioni.

Il flusso operativo tipico prevede:

1. Creazione della sessione multipart upload.
2. Upload delle singole parti.
3. Conservazione degli ETag restituiti da ogni parte.
4. Invio della richiesta di completamento.
5. Ricostruzione dell'oggetto da parte di S3.

Nelle implementazioni moderne gli SDK e la AWS CLI gestiscono automaticamente questo processo, semplificando notevolmente il lavoro dello sviluppatore.

### Best Practice

Quando si utilizzano multipart upload personalizzati è opportuno configurare Lifecycle Rules per eliminare le parti incomplete che potrebbero generare costi inutili nel tempo.

---

## Strong Read-After-Write Consistency

Uno degli aspetti più importanti introdotti nelle versioni moderne di Amazon S3 è la consistenza forte.

Ciò significa che immediatamente dopo una scrittura, una modifica o una cancellazione, tutte le successive richieste di lettura visualizzeranno il dato aggiornato.

Questo comportamento semplifica notevolmente la progettazione delle applicazioni e riduce la necessità di meccanismi compensativi utilizzati in passato.

---

## Copiare, spostare e rinominare gli oggetti

Amazon S3 non possiede un'operazione nativa di rinomina.

Per rinominare un file è necessario copiare l'oggetto verso una nuova chiave e successivamente eliminare quello originale.

La stessa logica viene utilizzata per spostare file tra bucket differenti o modificare metadati associati agli oggetti.

Sebbene possa apparire controintuitivo per chi proviene da sistemi file tradizionali, questo comportamento è coerente con il modello object storage.

---

## Lettura dei dati e recupero dei metadati

Le API GET consentono di scaricare completamente un oggetto oppure solo una porzione specifica.

Questa funzionalità è particolarmente utile nello streaming audio e video.

Un'applicazione che riproduce un filmato non ha infatti bisogno di scaricare l'intero file prima di iniziarne la fruizione. Può richiedere intervalli di byte successivi riducendo consumi di banda e tempi di attesa.

Accanto al download completo esistono API dedicate al recupero dei metadati.

L'operazione HeadObject permette di ottenere informazioni quali:

- data ultima modifica;
- dimensione del file;
- tipo di contenuto;
- ETag;
- informazioni di versioning.

Questo approccio riduce il traffico di rete quando è necessario analizzare gli oggetti senza scaricarli integralmente.

---

## S3 Select

S3 Select rappresenta una delle funzionalità più interessanti del servizio.

Tradizionalmente, per interrogare un file CSV o JSON memorizzato in un bucket era necessario scaricare il documento completo e successivamente elaborarlo all'interno dell'applicazione.

Con S3 Select è possibile eseguire query SQL direttamente sull'oggetto.

L'elaborazione viene eseguita all'interno dell'infrastruttura AWS e soltanto i risultati rilevanti vengono restituiti al client.

Ad esempio, un file CSV contenente milioni di record può essere interrogato per recuperare esclusivamente le righe corrispondenti a uno specifico paese, stato o categoria.

### Vantaggi principali

- Minore trasferimento dati.
- Riduzione dei costi di rete.
- Migliori prestazioni.
- Codice applicativo più semplice.
- Riduzione dell'elaborazione lato client.

### Caso pratico

Nel caso di un archivio aeroportuale memorizzato in CSV, una query S3 Select può restituire esclusivamente gli aeroporti statunitensi senza trasferire il resto del dataset.

---

## URL Pre-Firmati (Presigned URLs)

Uno dei requisiti più comuni nelle applicazioni moderne consiste nel condividere file privati senza renderli pubblicamente accessibili.

Le Presigned URL risolvono questo problema.

Una URL pre-firmata incorpora temporaneamente le autorizzazioni necessarie per accedere a un oggetto specifico. Chiunque possieda tale URL può utilizzare l'accesso consentito fino alla sua scadenza.

La durata può essere configurata in funzione delle esigenze applicative.

### Scenario tipico

Un'applicazione genera dinamicamente una URL valida per sessanta minuti e la invia a un utente autenticato. L'utente può scaricare il documento senza possedere credenziali AWS e senza che l'oggetto venga reso pubblico.

Questo modello viene utilizzato frequentemente per:

- distribuzione di report;
- condivisione documentale;
- download software;
- accesso controllato a contenuti multimediali.

---

## Operazioni massive sugli oggetti

Quando il numero di oggetti cresce, le operazioni singole diventano inefficienti.

Amazon S3 fornisce diversi strumenti per la gestione di attività massive.

Tra i più utilizzati troviamo i comandi AWS CLI cp e sync.

Il comando cp consente di copiare file individuali.

Il comando sync permette invece di sincronizzare directory, prefissi o bucket completi mantenendo allineati i contenuti.

---

## Listing e Paginazione

Servizi come Amazon S3 limitano il numero di risultati restituiti da una singola richiesta.

Per questo motivo viene utilizzato il meccanismo di paginazione.

Gli SDK moderni mettono a disposizione paginator automatici che gestiscono la navigazione tra le varie pagine di risultati.

Gli sviluppatori possono così elaborare grandi volumi di dati senza preoccuparsi dei dettagli implementativi legati ai continuation token.

---

## S3 Batch Operations

Per attività particolarmente estese AWS mette a disposizione S3 Batch Operations.

Questa funzionalità consente di eseguire operazioni su milioni di oggetti attraverso un unico job gestito.

Tra le attività supportate:

- modifica dei metadati;
- sostituzione dei tag;
- variazione delle ACL;
- copia di oggetti;
- ripristino da Glacier;
- invocazione di funzioni Lambda.

Le Batch Operations rappresentano uno strumento essenziale per attività di governance, migrazione e manutenzione su larga scala.

---

## Hosting di Siti Statici

Uno degli utilizzi più noti di S3 consiste nell'hosting di siti web statici.

File HTML, CSS e JavaScript possono essere pubblicati direttamente all'interno di un bucket senza necessità di server web dedicati.

È sufficiente indicare:

- il documento principale (index.html);
- la pagina di errore (error.html).

Una volta attivata la funzionalità, AWS genera automaticamente l'endpoint pubblico del sito.

Questa soluzione è particolarmente diffusa nelle architetture serverless in cui il frontend viene ospitato su S3 mentre il backend è costituito da API Gateway e Lambda.

---

## CORS e integrazione tra domini

Quando un browser tenta di accedere a risorse appartenenti a domini differenti entrano in gioco le politiche di Cross-Origin Resource Sharing (CORS).

Amazon S3 consente di definire esplicitamente quali origini possono accedere alle risorse e quali metodi HTTP sono consentiti.

Una corretta configurazione CORS è fondamentale per applicazioni web moderne che interrogano API o recuperano contenuti ospitati su domini differenti.

---

## Ottimizzazione dei costi con S3 Select

Un esempio pratico mostra come S3 Select possa contribuire alla riduzione dei costi.

Se un'applicazione deve cercare una parola all'interno di una grande raccolta di documenti, scaricare ogni file completo potrebbe comportare costi elevati di trasferimento dati.

Utilizzando S3 Select è possibile trasferire esclusivamente le righe pertinenti, riducendo banda utilizzata, tempi di elaborazione e costi operativi.

In ambienti caratterizzati da dataset di grandi dimensioni, questo approccio può generare risparmi significativi.

---

## Best Practice Operative

Le organizzazioni dovrebbero adottare alcune pratiche fondamentali:

- utilizzare multipart upload per file di grandi dimensioni;
- eliminare upload incompleti mediante lifecycle policy;
- preferire presigned URL agli oggetti pubblici;
- utilizzare paginator per elaborazioni massive;
- sfruttare S3 Batch Operations per attività ripetitive;
- implementare correttamente le politiche CORS;
- utilizzare S3 Select quando è necessario interrogare grandi oggetti strutturati.

---

## Conclusioni

Amazon S3 non è soltanto un archivio di file. È una piattaforma completa per la gestione dei dati non strutturati, capace di supportare caricamento distribuito, elaborazione selettiva dei dati, accessi temporanei sicuri, operazioni massive e pubblicazione di contenuti web.

La padronanza di queste funzionalità rappresenta una competenza fondamentale per qualunque sviluppatore AWS che desideri progettare applicazioni cloud-native efficienti, sicure e scalabili.
