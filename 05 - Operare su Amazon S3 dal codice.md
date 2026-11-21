# Capitolo 5 – Operare su Amazon S3 dal codice

## 5.1 Obiettivi del capitolo

Dopo i concetti del capitolo precedente, questo capitolo lavora su S3 dal codice con gli SDK. Per i **bucket** si vedrà come crearli, verificarne l'esistenza, cancellarli e aggiornarne la configurazione. Per gli **oggetti** si vedrà come caricarli, aggiornarli, scaricarli e interrogarli. Si affronta poi il problema dei **file molto grandi** e dei **file molto numerosi**, che S3 risolve con API dedicate: caricamento multipart, paginazione e Batch Operations. Si vedrà come configurare un bucket per **ospitare un sito web statico** e come concedere un **accesso temporaneo** agli oggetti tramite i **presigned URL**. Il capitolo si chiude con un caso di studio che confronta due modi di leggere i dati da S3, sia per le prestazioni sia per i costi.

## 5.2 Lavorare con i bucket in Java

### Le operazioni sui bucket

Le operazioni più comuni sui bucket sono la **creazione**, l'**elenco** dei bucket esistenti, la **cancellazione** e l'**aggiornamento della configurazione**. Si può per esempio attivare la crittografia o il versioning, abilitare CORS o configurare il bucket per l'hosting di un sito statico.

Per creare un bucket servono due informazioni: un **nome** e una **Region**. Come già visto, il nome deve essere unico a livello globale, non solo nel proprio account ma su tutti gli account AWS. Un nome generico come `example` è certamente già preso, quindi serve qualcosa di più specifico.

Nella Console, quando si digita il nome di un bucket, si riceve subito un riscontro visivo se il nome è già in uso. Nel codice questo riscontro non c'è, e conviene verificarlo in modo esplicito prima della creazione. Il flusso tipico ha tre passi:

1. **Verificare se il bucket esiste** con l'API **HeadBucket**, che usa il metodo HTTP `HEAD`. È come una GET, ma restituisce solo le intestazioni, cioè i metadati, senza il contenuto.
2. **Creare il bucket** con **CreateBucket**.
3. **Verificare la creazione**, recuperando le informazioni sul bucket o elencando i bucket.

### Le dipendenze Maven

In un progetto Java le dipendenze si dichiarano nel `pom.xml`:

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>software.amazon.awssdk</groupId>
      <artifactId>bom</artifactId>
      <version>2.x.x</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>

<dependencies>
  <dependency>
    <groupId>software.amazon.awssdk</groupId>
    <artifactId>s3</artifactId>
  </dependency>
  <dependency>
    <groupId>software.amazon.awssdk</groupId>
    <artifactId>dynamodb</artifactId>
  </dependency>
  <dependency>
    <groupId>software.amazon.awssdk</groupId>
    <artifactId>dynamodb-enhanced</artifactId>
  </dependency>
</dependencies>
```

Il **groupId `software.amazon.awssdk`** identifica la **versione 2** dell'SDK for Java. Anche gli `import` del codice iniziano con `software.amazon.awssdk`. È un dettaglio utile per capire quale versione usa un esempio trovato online, perché la versione 1 (`com.amazonaws`) ha un'API diversa. Le dipendenze si includono **un servizio alla volta**: un progetto che usa solo S3 può eliminare le dipendenze di DynamoDB. Includere solo ciò che serve riduce la dimensione dell'artefatto finale.

### Esempio: verificare e creare un bucket

```java
public static void main(String[] args) {
    String bucketName = "morgan-example-dev";

    S3Client s3 = S3Client.builder()
            .region(Region.US_EAST_1)
            .credentialsProvider(ProfileCredentialsProvider.create())
            .build();

    if (!bucketExisting(s3, bucketName)) {
        createBucket(s3, bucketName);
    }
    s3.close();
}
```

Il client segue il **builder pattern**: dopo `S3Client.builder()`, ogni metodo successivo aggiunge un elemento di configurazione, qui la Region e il provider di credenziali. `ProfileCredentialsProvider.create()` usa il profilo predefinito contenuto nel file `~/.aws/credentials`. Una volta ottenuto il client, se ne possono chiamare i metodi, ed è utile consultare la documentazione della classe per conoscerli tutti.

Il metodo `bucketExisting` usa HeadBucket:

```java
static boolean bucketExisting(S3Client s3, String bucketName) {
    try {
        HeadBucketRequest request = HeadBucketRequest.builder()
                .bucket(bucketName)
                .build();
        HeadBucketResponse response = s3.headBucket(request);
        return response.sdkHttpResponse().statusCode() == 200;
    } catch (S3Exception e) {
        switch (e.statusCode()) {
            case 404 -> System.out.println("No such bucket existing.");
            case 400 -> System.out.println("Wrong Region for this bucket.");
            case 403 -> System.out.println("Bucket exists but access is forbidden.");
            default  -> System.out.println("Unexpected error: " + e.getMessage());
        }
        return false;
    }
}
```

I codici di risposta di HeadBucket hanno significati precisi, che il codice può gestire in modo diverso:

| Codice | Significato |
|---|---|
| **200** | Il bucket esiste ed è accessibile |
| **404** | Il bucket non esiste |
| **400** | La richiesta è indirizzata alla Region sbagliata |
| **403** | Il bucket esiste ma non si hanno i permessi per accedervi (spesso perché appartiene a un altro account) |

Il metodo `createBucket` crea il bucket e ne attende la disponibilità:

```java
static void createBucket(S3Client s3, String bucketName) {
    S3Waiter waiter = s3.waiter();

    CreateBucketRequest request = CreateBucketRequest.builder()
            .bucket(bucketName)
            .build();
    s3.createBucket(request);

    HeadBucketRequest wait = HeadBucketRequest.builder()
            .bucket(bucketName)
            .build();
    WaiterResponse<HeadBucketResponse> waiterResponse = waiter.waitUntilBucketExists(wait);
    waiterResponse.matched().response()
            .ifPresent(r -> System.out.println(bucketName + " is ready"));
}
```

La richiesta di creazione specifica solo il nome e accetta tutti i valori predefiniti, come quando nella Console si inserisce il nome e si clicca su *Create* senza modificare altro. La creazione richiede un po' di tempo: impercettibile per una persona, ma non per un programma. Per non dover scrivere a mano un ciclo di verifica e attesa, si usa un **`S3Waiter`**: `waitUntilBucketExists` ripete la HeadBucket finché il bucket non risulta disponibile.

Eseguendo il programma compaiono i messaggi "No such bucket existing" e "morgan-example-dev is ready". Il bucket appare nell'AWS Toolkit dell'IDE.

## 5.3 Lavorare con gli oggetti

### Il panorama delle operazioni

Sugli oggetti si eseguono operazioni di base, cioè caricare, scaricare, elencare, copiare, spostare, rinominare e cancellare, e operazioni massive. Si può per esempio copiare in S3 un gran numero di oggetti, **sincronizzare** una cartella locale con un prefisso di un bucket o eseguire **operazioni batch** su collezioni di oggetti.

### Caricare gli oggetti: PUT e multipart

Il caricamento usa il metodo HTTP **PUT**. Ci sono due strade:

- il **caricamento singolo**, con una sola richiesta. È possibile per oggetti fino a **5 GB**;
- il **caricamento multipart**, in cui l'oggetto viene diviso in più parti, ciascuna caricata separatamente, e S3 le riassembla alla fine. È **consigliato per oggetti oltre i 100 MB** e **obbligatorio oltre i 5 GB**.

Ogni singolo oggetto in S3 deve essere **inferiore a 5 TB**. Lo spazio complessivo di un bucket, invece, è di fatto illimitato.

S3 offre **consistenza forte read-after-write**. Dopo aver caricato un oggetto o una sua nuova versione, qualunque processo che elenchi il bucket o legga quell'oggetto ottiene **immediatamente** la versione più recente.

### Copiare, spostare e rinominare: CopyObject

L'API **CopyObject** ha diversi usi:

- **creare copie** di un oggetto;
- **rinominare** un oggetto: si copia con il nuovo nome e poi si cancella l'originale, perché non esiste un'operazione di "rename";
- **spostare** oggetti da un bucket a un altro;
- **aggiornare i metadati** di un oggetto, copiandolo su sé stesso con metadati nuovi.

### Il caricamento multipart passo per passo

Con i comandi di alto livello della CLI il multipart è automatico: `aws s3 cp` lo gestisce da solo. Con le API di basso livello (`aws s3api` o i metodi corrispondenti degli SDK) va orchestrato a mano:

1. Si divide il file in parti di dimensione inferiore a 5 GB.
2. Si avvia il caricamento con `create-multipart-upload`, che restituisce un **UploadId** valido per tutto il caricamento.
3. Si carica ogni parte con `upload-part`, conservando l'**ETag** restituito per ciascuna.
4. Facoltativamente si verificano le parti ricevute con `list-parts`.
5. Si prepara un file con gli ETag nell'ordine in cui le parti vanno ricomposte.
6. Si chiude il caricamento con `complete-multipart-upload`, chiedendo a S3 di ricostruire l'oggetto in quell'ordine.

```bash
aws s3api create-multipart-upload --bucket notes-bucket --key video.mp4
aws s3api upload-part --bucket notes-bucket --key video.mp4 \
    --part-number 1 --body video.part1 --upload-id <UploadId>
aws s3api list-parts --bucket notes-bucket --key video.mp4 --upload-id <UploadId>
aws s3api complete-multipart-upload --bucket notes-bucket --key video.mp4 \
    --upload-id <UploadId> --multipart-upload file://parts.json
```

Il multipart ha anche il vantaggio della resilienza: le parti si possono inviare in parallelo e, se una fallisce, si ritenta solo quella.

> **Box – Best practice: ripulire i multipart incompleti**
>
> Un caricamento multipart avviato e mai completato lascia in S3 delle parti "orfane", che non sono visibili come oggetti ma **si pagano**. Quando si gestisce il multipart a mano, conviene configurare sul bucket una **lifecycle rule** che elimini automaticamente i caricamenti multipart incompleti dopo un certo numero di giorni. Con i comandi di alto livello il problema è gestito automaticamente.

### Leggere gli oggetti: GET e HEAD

L'API **GetObject** usa il metodo `GET` e può scaricare l'oggetto **intero** oppure solo un **intervallo di byte** (*byte range*). Il caso tipico è lo streaming di un film su smartphone. Il client non scarica tutto il file in una volta: ne scarica un pezzo, lo riproduce e intanto chiede il pezzo successivo (*buffering*). Se l'utente guarda solo i primi dieci minuti, non si è sprecata la banda per scaricare l'intero film.

Il metodo `HEAD` e le API collegate restituiscono invece solo informazioni sull'oggetto:

- **HeadObject** restituisce i metadati dell'oggetto;
- **GetObjectAcl** restituisce l'**ACL**, cioè i permessi;
- **GetObjectTagging** restituisce i **tag**. Serve per esempio a contare quanti oggetti hanno un certo tag e decidere di conseguenza.

Esempio da riga di comando:

```bash
aws s3api head-object --bucket notes-bucket --key index.html
```

```json
{
    "LastModified": "2022-10-01T10:15:30+00:00",
    "ContentLength": 1024,
    "ETag": "\"6805f2cfc46c0f04559748bb039d69ae\"",
    "VersionId": "null",
    "ContentType": "text/html",
    "Metadata": {}
}
```

Il `ContentType` è `text/html`. Il `VersionId` vale `null`, segno che il versioning non è attivo sul bucket. Compaiono anche l'`ETag` e la data di ultima modifica.

### Interrogare un oggetto: S3 Select

**S3 Select** permette di interrogare il **contenuto di un singolo oggetto** con istruzioni **SQL**, del tipo `SELECT ... FROM ... WHERE ...`, e di ricevere solo i dati che soddisfano la query. Funziona su oggetti in formato CSV, JSON o Parquet.

Supponiamo di avere un oggetto JSON molto grande e di cercare una singola informazione al suo interno. Senza S3 Select bisogna scaricare l'intero oggetto e poi, nel codice, analizzarlo, scorrerlo e trovare la chiave e il valore cercati. Con S3 Select l'elaborazione avviene **lato S3**: si invia la query e il servizio restituisce solo i dati pertinenti. I vantaggi sono tre: **meno dati trasferiti** fuori dal bucket, quindi minori costi, **meno banda** occupata e **meno codice** da scrivere per l'analisi.

> **Box – Lo stato di S3 Select oggi**
>
> Dal luglio 2024 AWS non rende più disponibile S3 Select ai nuovi clienti: chi lo usava già può continuare a farlo. Per interrogare dati in S3 con SQL, AWS indica come alternative principali **Amazon Athena** e **S3 Object Lambda**, oppure il filtro lato client. Il concetto resta comunque importante, perché è un esempio chiaro di come spostare l'elaborazione vicino ai dati riduca trasferimenti e costi, e compare ancora nelle domande d'esame.

## 5.4 Lavorare con gli oggetti in Java

### Leggere i metadati e scaricare un oggetto

Nel bucket c'è un file `airports.csv` con l'elenco degli aeroporti di tutto il mondo. Il programma ne stampa i metadati e poi lo scarica. Il nome del bucket arriva da una **variabile d'ambiente** impostata nella configurazione di esecuzione dell'IDE, una pratica che evita di cablare nel codice i nomi delle risorse.

```java
public static void main(String[] args) {
    S3Client s3 = S3Client.builder()
            .region(Region.US_EAST_1)
            .build();
    String bucketName = System.getenv("BUCKET_NAME");
    String keyName = "airports.csv";

    printHeadObject(s3, bucketName, keyName);
    byte[] data = getObjectBytes(s3, buildObjectRequest(bucketName, keyName));
    writeBytesToFile(data, keyName);
    s3.close();
}

static void printHeadObject(S3Client s3, String bucket, String key) {
    HeadObjectRequest request = HeadObjectRequest.builder()
            .bucket(bucket)
            .key(key)
            .build();
    HeadObjectResponse response = s3.headObject(request);
    System.out.println(response.toString());
}

static GetObjectRequest buildObjectRequest(String bucket, String key) {
    return GetObjectRequest.builder()
            .key(key)
            .bucket(bucket)
            .build();
}

static byte[] getObjectBytes(S3Client s3, GetObjectRequest request) {
    ResponseBytes<GetObjectResponse> objectBytes = s3.getObjectAsBytes(request);
    return objectBytes.asByteArray();
}
```

Lo schema è sempre lo stesso: si **costruisce la richiesta** con un builder, la si **passa all'API** e si legge la **risposta**. `headObject` restituisce un `HeadObjectResponse`, il cui `toString()` stampa su una riga data di ultima modifica, dimensione, ETag e tipo di contenuto. `getObjectAsBytes` restituisce un `ResponseBytes<GetObjectResponse>`, da cui `asByteArray()` estrae il contenuto come array di byte, poi scritto su un file locale. All'esecuzione compaiono i metadati, il messaggio di download completato e, nella cartella del progetto, il file `airports.csv` con i dati degli aeroporti.

### S3 Select in Java

Lo stesso file serve per un esempio di S3 Select: estrarre **solo gli aeroporti degli Stati Uniti**. Nell'SDK for Java 2.x l'operazione `selectObjectContent` è disponibile **solo sul client asincrono** `S3AsyncClient`, perché i risultati arrivano come **flusso di eventi**.

```java
S3AsyncClient s3 = S3AsyncClient.builder().region(Region.US_EAST_1).build();

InputSerialization input = InputSerialization.builder()
        .csv(CSVInput.builder().fileHeaderInfo(FileHeaderInfo.USE).build())
        .build();

OutputSerialization output = OutputSerialization.builder()
        .csv(CSVOutput.builder().build())
        .build();

SelectObjectContentRequest request = SelectObjectContentRequest.builder()
        .bucket(bucketName)
        .key("airports.csv")
        .expressionType(ExpressionType.SQL)
        .expression("SELECT * FROM S3Object s WHERE s.\"iso_country\" LIKE 'US'")
        .inputSerialization(input)
        .outputSerialization(output)
        .build();

Handler handler = new Handler();
s3.selectObjectContent(request, handler).get();
```

La richiesta specifica bucket, chiave, tipo di espressione (`SQL`), la query stessa e due oggetti di **serializzazione**. L'**input serialization** dice a S3 in che formato è l'oggetto, qui CSV con riga di intestazione (`FileHeaderInfo.USE`), così le colonne si possono chiamare per nome. L'**output serialization** stabilisce il formato dei risultati, anch'esso CSV.

La parte più articolata è il **gestore della risposta**: una classe interna che implementa l'interfaccia `SelectObjectContentResponseHandler`.

```java
private static class Handler implements SelectObjectContentResponseHandler {
    private SelectObjectContentResponse response;
    private final List<SelectObjectContentEventStream> receivedEvents = new ArrayList<>();

    @Override
    public void responseReceived(SelectObjectContentResponse response) {
        this.response = response;
    }

    @Override
    public void onEventStream(SdkPublisher<SelectObjectContentEventStream> publisher) {
        publisher.subscribe(receivedEvents::add);
    }

    @Override
    public void exceptionOccurred(Throwable throwable) { /* gestione errori */ }

    @Override
    public void complete() { }
}
```

S3 restituisce in streaming le porzioni che soddisfano la query. Il metodo `onEventStream` si iscrive al publisher e aggiunge ogni evento ricevuto alla lista. La chiamata `.get()` sul `CompletableFuture` restituito da `selectObjectContent` attende il completamento, rendendo sincrona l'operazione ai fini della dimostrazione. Alla fine si filtrano gli eventi di tipo *Records* e se ne stampa il contenuto:

```java
handler.receivedEvents.stream()
        .filter(e -> e.sdkEventType() == SelectObjectContentEventStream.EventType.RECORDS)
        .map(e -> ((RecordsEvent) e).payload().asUtf8String())
        .forEach(System.out::println);
```

L'output contiene solo aeroporti statunitensi: S3 ha restituito il sottoinsieme richiesto senza trasferire il resto del file.

## 5.5 Concedere accesso temporaneo agli oggetti: i presigned URL

Gli oggetti S3 sono **privati per default**. Spesso però vanno condivisi con altre applicazioni o con terze parti, e renderli **pubblici**, cioè accessibili a chiunque su Internet, non è quasi mai la soluzione giusta.

Lo strumento adatto è il **presigned URL**: un URL che contiene in sé tutte le informazioni di autorizzazione necessarie per accedere a un oggetto specifico. Viene firmato con le credenziali di un'identità che ha accesso all'oggetto, e **chiunque possieda l'URL può usarlo**. Per questo va protetto come una credenziale. Ha una **scadenza configurabile**, da pochi minuti a qualche giorno a seconda del caso d'uso, e può autorizzare sia il **download** (GET) sia il **caricamento** (PUT) di un oggetto.

> **Box – Caso d'uso: dal server FTP ai presigned URL**
>
> Uno scenario aziendale frequente: un'organizzazione deve condividere report con clienti esterni. Il processo tradizionale prevede di estrarre il report dai sistemi interni, caricarlo su un server FTP condiviso e dare ai clienti l'accesso al server. A ogni aggiornamento del report bisogna ricaricarlo a mano. Ne risultano più passaggi manuali e più copie dello stesso documento.
>
> Con AWS il processo si semplifica. I report vivono in un bucket privato. Un'applicazione, per esempio su EC2, autentica il cliente e, su richiesta, **genera un presigned URL** per il report richiesto. Il cliente clicca il link e scarica il file. Alla scadenza dell'URL basta tornare nell'applicazione per ottenerne uno nuovo, che punterà alla versione più recente. Non serve un server FTP, non ci sono copie duplicate e non c'è alcun intervento manuale.

### Esempio: generare un presigned URL in Java

```java
S3Presigner presigner = S3Presigner.builder()
        .region(Region.US_EAST_1)
        .build();
String bucketName = System.getenv("BUCKET_NAME");

GetObjectRequest getObjectRequest = GetObjectRequest.builder()
        .bucket(bucketName)
        .key("airports.csv")
        .build();

GetObjectPresignRequest presignRequest = GetObjectPresignRequest.builder()
        .signatureDuration(Duration.ofMinutes(60))
        .getObjectRequest(getObjectRequest)
        .build();

PresignedGetObjectRequest presigned = presigner.presignGetObject(presignRequest);
System.out.println("Presigned URL: " + presigned.url());
```

Il punto di partenza è un client specifico, **`S3Presigner`**. Si costruisce una normale `GetObjectRequest` e la si "avvolge" in una `GetObjectPresignRequest`, che aggiunge la **durata della firma**, qui 60 minuti. `presignGetObject` restituisce l'URL firmato. Generare l'URL non richiede alcuna chiamata di rete a S3: la firma viene calcolata localmente con le credenziali del client. Cliccando sull'URL stampato, il browser scarica `airports.csv`.

## 5.6 Operazioni massive (bulk) in Java

Finora si è operato su un oggetto alla volta. Ma se occorre caricare migliaia di file, o aggiungere un tag a mille oggetti che ne sono privi?

### Copia e sincronizzazione dalla CLI

I comandi di alto livello coprono i casi più semplici e gestiscono automaticamente il multipart:

```bash
# Copiare un file locale in S3
aws s3 cp report.csv s3://notes-bucket/reports/

# Sincronizzare due bucket, escludendo un prefisso
aws s3 sync s3://notes-bucket s3://notes-bucket-backup --exclude "tmp/*"

# Sincronizzare una cartella locale con un bucket
aws s3 sync ./site s3://notes-bucket
```

### Iterare con ListBuckets e ListObjects

Per operazioni diverse dalla copia, come aggiungere tag, modificare metadati o ACL o elaborare gli oggetti, si possono usare le API **ListBuckets** e **ListObjects** per iterare sulle risorse e applicare a ciascuna la logica desiderata.

```java
ListBucketsResponse buckets = s3.listBuckets();
for (Bucket bucket : buckets.buckets()) {
    System.out.println(bucket.name());
}
```

Per gli oggetti, l'SDK for Java 2.x offre un **paginator**:

```java
ListObjectsV2Request request = ListObjectsV2Request.builder()
        .bucket(bucketName)
        .maxKeys(2)
        .build();

ListObjectsV2Iterable pages = s3.listObjectsV2Paginator(request);
pages.stream()
     .flatMap(page -> page.contents().stream())
     .forEach(object -> System.out.println(object.key()));
```

`maxKeys(2)` limita a due il numero di chiavi **per richiesta**: un valore volutamente basso, per rendere visibile la paginazione. Il paginator esegue automaticamente le chiamate successive finché non ci sono più oggetti. Con il logging dell'SDK attivo lo si vede chiaramente. Parte una prima GET con `max-keys=2` che restituisce `airports.csv` e un'immagine. Segue una seconda GET verso lo stesso host, che restituisce altre due chiavi, poi un'altra ancora, fino all'esaurimento dell'elenco. Al posto della stampa, in un'operazione bulk reale si applicherebbe a ogni oggetto l'operazione desiderata, come aggiornare un tag o modificare l'ACL.

Senza paginator la paginazione va gestita manualmente. Ogni risposta contiene un **continuation token**, da passare alla richiesta successiva per ottenere la pagina seguente, finché la risposta non indica che i risultati sono finiti.

### S3 Batch Operations

Per lavorare su grandi collezioni di oggetti esiste una soluzione gestita: **S3 Batch Operations**, utilizzabile anche dalla Console. Si scelgono gli oggetti, si seleziona l'operazione e si segue l'avanzamento di un vero e proprio **job batch**.

Per indicare su quali oggetti lavorare serve un **manifest**. Lo si può ottenere da un report di **Amazon S3 Inventory**, che elenca gli oggetti del bucket ed è filtrabile per prefisso, se gli oggetti sono organizzati con prefissi coerenti. Le operazioni disponibili includono:

- modificare i **metadati** degli oggetti;
- **copiare** oggetti tra bucket;
- **ripristinare** oggetti da S3 Glacier;
- modificare il **controllo degli accessi**;
- **sostituire i tag**;
- **invocare una funzione Lambda** su ogni oggetto.

L'ultima opzione è particolarmente potente. Se un bucket contiene migliaia di video da transcodificare, prima bisognava scrivere codice che li elencasse con ListObjects e avviasse l'elaborazione. Con Batch Operations basta indicare la funzione Lambda da eseguire sulla collezione, avviare il job e controllarne l'esito dalla Console.

## 5.7 Ospitare un sito web statico

Un **sito web statico** è composto da file HTML, CSS e JavaScript che il browser scarica e visualizza così come sono. "Statico" non significa però privo di contenuti dinamici: il codice JavaScript può chiamare API, come quelle di Polly Notes su API Gateway, e mostrare nella pagina informazioni aggiornate.

Tradizionalmente servire questi file richiedeva un **web server**. S3 elimina questa necessità: un bucket può ospitare e servire direttamente un sito statico, senza server da installare o gestire.

```bash
aws s3 website s3://notes-bucket \
    --index-document index.html \
    --error-document error.html
```

Servono due informazioni. Il **documento indice** è la pagina principale, quella mostrata quando si visita il sito, di solito `index.html`. Il **documento di errore** è la pagina mostrata in caso di errore, per esempio `error.html`. Entrambi devono esistere come oggetti nel bucket. S3 rende poi disponibile il website endpoint:

```
http://notes-bucket.s3-website-us-west-2.amazonaws.com
```

Visitandolo si ottiene `index.html`. Perché il sito sia leggibile da chiunque servono anche una bucket policy che consenta `s3:GetObject` a tutti e la disattivazione del *Block Public Access*, come visto nel Capitolo 4.

### CORS

Quando si lavora con un sito statico o con risorse condivise può servire **CORS** (*Cross-Origin Resource Sharing*). Per motivi di sicurezza, i browser impediscono a una pagina caricata da un dominio di accedere a risorse di un **dominio diverso**, a meno che quest'ultimo non lo consenta esplicitamente. CORS è il meccanismo con cui un server dichiara quali origini esterne possono accedere alle sue risorse.

I casi tipici sono due. Il primo è una pagina su `example.com` che accede a oggetti in un bucket con un dominio diverso, per esempio un **web font** condiviso pubblicamente. Il secondo è codice JavaScript che effettua richieste verso il bucket da un altro dominio. In S3 la configurazione CORS si esprime come documento XML in cui si indicano le **origini consentite** e i **metodi HTTP consentiti**:

```xml
<CORSConfiguration>
  <CORSRule>
    <AllowedOrigin>*</AllowedOrigin>
    <AllowedMethod>GET</AllowedMethod>
  </CORSRule>
</CORSConfiguration>
```

Questa regola permette a qualunque dominio di effettuare richieste GET verso il bucket. Nella Console attuale la stessa configurazione si scrive in JSON.

## 5.8 Caso di studio: S3 Select contro GetObject

Una piccola sfida di programmazione aiuta a ragionare sulle API di S3. In un bucket sono memorizzate le **opere di Shakespeare**, una per oggetto. Data una parola, bisogna contare quante volte compare in tutte le opere.

L'approccio più ovvio è scaricare ogni oggetto e cercare la parola nel codice. Ma S3 offre anche **S3 Select**, che si presta allo scopo anche se non è il suo caso d'uso più evidente: basta una query con una clausola `WHERE` che selezioni le righe contenenti la parola cercata.

Il programma, scritto in Python, implementa entrambe le strade:

```python
def get_string_count(s3, bucket, search, operation):
    count = 0
    search = search.upper()
    objects = s3.list_objects_v2(Bucket=bucket, Prefix="shakespeare/")
    for obj in objects["Contents"]:
        if operation == "getobject":
            body = s3.get_object(Bucket=bucket, Key=obj["Key"])["Body"].read()
            count += body.decode("utf-8").upper().count(search)
        else:
            response = s3.select_object_content(
                Bucket=bucket,
                Key=obj["Key"],
                ExpressionType="SQL",
                Expression=f"SELECT s._1 FROM S3Object s WHERE UPPER(s._1) LIKE '%{search}%'",
                InputSerialization={"CSV": {"FileHeaderInfo": "NONE"}},
                OutputSerialization={"CSV": {}},
            )
            for event in response["Payload"]:
                if "Records" in event:
                    count += event["Records"]["Payload"].decode("utf-8").upper().count(search)
    return count
```

Prima si elencano tutti gli oggetti sotto il prefisso `shakespeare/`. Con **GetObject** si scarica l'intero testo di ogni opera, lo si converte in maiuscolo e si contano le occorrenze. Con **S3 Select** si tratta ogni file come un CSV a una sola colonna, `_1` per convenzione quando le colonne non hanno nome, e si chiede a S3 di restituire solo le righe che contengono la parola. Le occorrenze si contano poi su quelle righe. C'è un accorgimento: virgole e virgolette sono state eliminate dai testi, così che ogni riga venga letta come un unico campo CSV.

Cercando la parola *queen* entrambe le implementazioni trovano **1.129 occorrenze**. I tempi sono simili. Nella prima esecuzione GetObject impiega 3,6 secondi e S3 Select 3,18. In una seconda esecuzione i valori sono 1,99 e 2,4 secondi. Le prestazioni, in questo scenario, sono quindi paragonabili.

La differenza decisiva sta nei **costi**. S3 addebita il **trasferimento dei dati in uscita** dal bucket (*data transfer out*). Con GetObject si paga il trasferimento di **ogni opera per intero**, anche se interessano poche righe. Con S3 Select si trasferiscono solo le righe che contengono la parola, cioè una frazione minima dei dati. A parità di tempi di esecuzione, S3 Select è quindi la scelta più conveniente. Sarebbe interessante ripetere il test con file più grandi o su una connessione più lenta, dove il vantaggio di trasferire meno dati diventerebbe evidente anche nei tempi.

> **Box – Ottimizzare i costi di S3**
>
> Il caso di studio illustra un principio generale: in S3 non si paga solo lo spazio occupato, ma anche le **richieste** e il **trasferimento dei dati in uscita**. Filtrare i dati vicino a dove risiedono, con S3 Select, Athena o le letture a intervalli di byte, è una delle leve principali per ridurre la spesa.

## 5.9 Verifica dei concetti

**"Conviene valutare il caricamento multipart per oggetti più grandi di 100 MB."** *Vero.* È la soglia raccomandata da AWS, e i comandi di alto livello della CLI usano il multipart automaticamente: un file da 10 MB era già stato diviso in parti. Con le API di basso livello bisogna invece dividere il file e gestire UploadId ed ETag a mano.

**"Alcuni servizi hanno API che richiedono la paginazione per accedere a tutti i dati restituiti."** *Vero.* Un esempio è l'elenco dei tipi di istanza EC2: senza un client che paginasse, l'istanza cercata semplicemente non compariva nei risultati. Senza paginator integrato bisogna usare il continuation token fino a esaurire i risultati.

**"Con un presigned URL si possono condividere oggetti S3 specifici con un accesso limitato nel tempo."** *Vero.* Un caso tipico è una pagina di download di un'applicazione. L'URL viene firmato con un'identità dell'applicazione autorizzata all'oggetto, non con l'identità dell'utente, e ha una durata breve, così da non poter essere passato ad altri e riutilizzato a piacere.

**"S3 Select, con query in stile SQL, permette di selezionare e scaricare gli *oggetti* che soddisfano determinati criteri."** *Falso.* S3 Select lavora **dentro un singolo oggetto** e ne restituisce righe o record. Non permette di interrogare più oggetti insieme.

**"Per default le notifiche di eventi di S3 vengono inviate in risposta ad azioni come PUT, POST, COPY o DELETE."** *Falso.* Le notifiche si possono configurare per rispondere a queste azioni, ma non sono attive per default.

**"Un web server usa CORS per consentire o negare il caricamento di risorse dello *stesso* dominio da cui si accede al sito."** *Falso.* CORS, *Cross-Origin Resource Sharing*, riguarda per definizione l'accesso da **domini diversi**.
