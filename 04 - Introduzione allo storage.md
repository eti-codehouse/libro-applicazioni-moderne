# Parte II – Lo storage a oggetti con Amazon S3

# Capitolo 4 – Introduzione allo storage

## 4.1 Obiettivi del capitolo

Quasi tutte le applicazioni devono conservare dei dati: immagini caricate dagli utenti, documenti, file audio, pagine web, backup. Questo capitolo introduce **Amazon Simple Storage Service (Amazon S3)**, il servizio di **storage a oggetti** di AWS. Un oggetto, in prima approssimazione, è semplicemente una sequenza di byte a cui si associa un nome.

Dopo un confronto con le altre forme di storage offerte da AWS, il capitolo presenta i concetti fondamentali di S3: oggetti, chiavi, bucket, prefissi, versioning, notifiche di eventi e tag. Descrive poi le opzioni per **mettere in sicurezza i dati**, in modo che solo chi è autorizzato possa leggerli, modificarli o cancellarli. Si passa quindi a come ci si **connette** al servizio, cioè agli endpoint e alle operazioni di rete che l'SDK rende trasparenti, e a come si lavora con S3 dalla **CLI** e dall'**SDK for Java**, esaminando gli oggetti di richiesta e di risposta sia attraverso l'SDK sia direttamente sul filo.

## 4.2 Le opzioni di storage su AWS e i fondamenti di Amazon S3

### Tre famiglie di storage

AWS offre tre grandi famiglie di storage, ciascuna adatta a esigenze diverse.

Il **block storage** è rappresentato da **Amazon Elastic Block Store (EBS)**: sono i dischi virtuali collegati alle macchine virtuali EC2. I volumi EBS esistono in diversi tipi pensati per requisiti differenti. I volumi *General Purpose* coprono la maggior parte dei casi. I *Provisioned IOPS* garantiscono le massime prestazioni quando servono dischi molto veloci. I *Throughput Optimized* sono adatti a carichi sequenziali di grandi dimensioni. I *Cold HDD* servono per dati di archivio letti di rado, dove le prestazioni non sono la priorità.

Il **file storage** permette di collegare un **file system condiviso** alle macchine virtuali, proprio come un file server. **Amazon Elastic File System (EFS)** espone una condivisione **NFS** che più istanze possono montare contemporaneamente, ed esiste anche una classe *Infrequent Access* per i file usati di rado. La famiglia **Amazon FSx** offre file system gestiti per esigenze specifiche: **FSx for Windows File Server**, che parla il protocollo **SMB** tipico degli ambienti Windows, e **FSx for Lustre**, un file system ad alte prestazioni per il calcolo intensivo.

L'**object storage** è probabilmente il concetto più nuovo per chi viene dallo sviluppo tradizionale. Si memorizza una sequenza di byte, per esempio un'immagine JPEG, e la si associa a una **chiave**. In modo semplificato, è un archivio **chiave-valore**: con una chiave si salvano i byte e con la stessa chiave li si recupera. S3 offre molte funzionalità in più rispetto a questo schema di base, oltre a diverse **classi di storage** in base ai requisiti:

- **S3 Standard** ha il livello di ridondanza standard ed è la scelta predefinita;
- le classi **S3 Glacier** sono pensate per l'archiviazione a lungo termine;
- **S3 One Zone-Infrequent Access** memorizza i dati in una sola Availability Zone, con meno ridondanza e a un costo inferiore, ed è adatta a dati facilmente ricreabili.

### Le caratteristiche di S3

Per descrivere S3 si usa spesso una serie di "-ità".

La **disponibilità** (*availability*) indica se il servizio è raggiungibile e operativo quando serve. La **durabilità** (*durability*) indica quanto è improbabile perdere un dato una volta memorizzato. S3 è progettato per una durabilità di **undici nove**, cioè 99,999999999%. Si può interpretare come una probabilità: i dati sono replicati su così tanti dispositivi che la probabilità che si guastino tutti contemporaneamente è infinitesimale.

> **Box – I numeri corretti di S3**
>
> È facile confondere disponibilità e durabilità. La **durabilità** di S3 Standard è progettata per il 99,999999999% (11 nove): è una misura della probabilità di *perdere* un oggetto. La **disponibilità** di S3 Standard è progettata per il **99,99%** (quattro nove), con un accordo sul livello di servizio (SLA) del 99,9%: è una misura della probabilità che il servizio *risponda*. Altre classi hanno disponibilità inferiori, per esempio One Zone-IA è progettata per il 99,5%. All'esame la distinzione è importante.

La **sicurezza** è essenziale. Se un'applicazione memorizza documenti legali in S3, bisogna essere certi che solo i principal autorizzati possano leggerli o modificarli. Le policy che lo garantiscono sono descritte più avanti in questo paragrafo.

La **gestibilità** (*manageability*) riguarda le operazioni di amministrazione. Per esempio, i **tag** sui bucket aiutano a capire chi possiede un bucket, quanto costa all'organizzazione e se occorre riaddebitarne il costo a un altro reparto (*chargeback*).

Le **prestazioni** di S3 sono elevate, con tempi di risposta molto rapidi, e conviene verificarle con test propri. La **scalabilità** è collegata alle prestazioni e riguarda come queste si mantengono quando il carico cresce. S3 supporta operazioni in parallelo che permettono di aumentare la scalabilità delle applicazioni.

### A che cosa serve S3

Gli usi tipici di S3 sono molti:

- **Contenuti dell'applicazione**: immagini, documenti, media.
- **Siti web statici**: con una policy adeguata, un bucket può fungere direttamente da web server (Capitolo 5).
- **Backup e archiviazione**: grazie alla durabilità, è un luogo ideale per i backup. Sapere che le proprie foto personali sono replicate su più dispositivi in più data center permette di dormire sonni tranquilli.
- **Big data e analytics**: molti strumenti di analisi conoscono S3, che può fare da base per un **data lake**. Per esempio Apache Hive su **Amazon EMR** può leggere e scrivere dati direttamente su S3.
- **Disaster recovery**: strettamente legato al backup. Quando un disastro colpisce un'applicazione, i dati non devono andare persi, e S3 è un buon posto in cui conservarli.

### Oggetti, chiavi e bucket

Gli elementi con cui si interagisce in S3 sono pochi e chiari.

Un **oggetto** è composto da tre parti. La **chiave** (*key*) è il nome univoco con cui l'oggetto viene memorizzato e recuperato, per esempio `dog.jpeg`. I **dati** sono i byte veri e propri: un'immagine, un documento legale, qualunque contenuto. I **metadati** sono informazioni associate all'oggetto, come intestazioni personalizzate da restituire insieme all'oggetto o attributi a fini di gestione e contabilità.

Un **bucket** è la **risorsa regionale** che contiene gli oggetti. La prima operazione da fare è crearne uno, e alla creazione si sceglie la **Region** in cui vivrà, per esempio Singapore o Sydney. La scelta conta: di solito si mette il bucket **vicino alla capacità di calcolo** che lo usa. Se l'applicazione gira a Sydney, anche il bucket starà a Sydney. Pur essendo regionale, il bucket ha un nome che deve essere **unico a livello globale**, su tutto S3 e su tutti gli account. Il motivo è che dalla combinazione di bucket e chiave si costruisce un **URL**, e quell'URL deve essere univoco:

```
https://notes-bucket.s3.us-west-2.amazonaws.com/files/index.html
        └─ bucket ─┘    └─ Region ─┘               └── chiave ──┘
```

All'interno di un bucket la struttura è **piatta**: non esistono cartelle vere. Una chiave è semplicemente una stringa, e `files/index.html` per S3 è un nome come un altro, che contiene una barra.

### Prefissi e delimitatori

Anche senza cartelle reali si può lavorare con S3 *come se* avesse una gerarchia, grazie ai parametri dell'operazione **List**. Consideriamo un bucket `notes-bucket` con queste chiavi:

```
readMe.txt
dev/awsservice/summary.txt
dev/awsservice/DynamoDB/notes.txt
dev/awsservice/Polly/voices.txt
```

Il parametro **`prefix`** filtra le chiavi che iniziano con una certa stringa. Con `prefix=dev` vengono restituite tutte le chiavi tranne `readMe.txt`, che non inizia con `dev`.

Il parametro **`delimiter`** "taglia" le chiavi in corrispondenza del delimitatore indicato. Usando `/` si ottiene un comportamento simile a quello di un file system. Con `prefix=dev/awsservice/` e `delimiter=/`, la risposta si ferma al delimitatore successivo. Restituisce l'oggetto `summary.txt` e i due "prefissi comuni" `DynamoDB/` e `Polly/`, che appaiono come sottocartelle. È proprio così che la Console S3 e molti strumenti mostrano una navigazione a cartelle.

### Le notifiche di eventi

Un bucket può reagire automaticamente agli eventi che lo riguardano tramite le **event notification**. Quando un oggetto viene creato, rimosso, ripristinato o replicato, S3 può:

- **invocare una funzione AWS Lambda**, cioè un frammento di codice eseguito in modo serverless (Capitolo 8);
- **scrivere un messaggio in una coda Amazon SQS**, da cui altri processi possono leggere;
- **pubblicare su un topic Amazon SNS**, che inoltra la notifica ai propri sottoscrittori, per esempio via email.

Un esempio classico: un'applicazione carica fotografie in un bucket e per ognuna serve una **miniatura**. Invece di scrivere codice che sorveglia il bucket o di decidere all'interno dell'applicazione quando creare le miniature, si configura una notifica. A ogni nuova foto viene invocata una funzione Lambda che genera la miniatura e la salva in un altro bucket.

### Il versioning

Abilitando il **versioning** su un bucket, S3 conserva la **cronologia** di ogni oggetto. Se si carica `photo.gif` per la terza volta con la stessa chiave, le due versioni precedenti non vengono sovrascritte ma restano disponibili. Ogni versione ha un **version ID** univoco, restituito dall'API, con cui si può "tornare indietro nel tempo" e recuperare le versioni precedenti.

Il versioning protegge da due rischi. Il primo è la **corruzione dei dati**: se per errore si carica un file vuoto al posto di quello buono, si recupera la versione precedente. Il secondo sono le **cancellazioni accidentali**. In un bucket con versioning, una semplice cancellazione con la sola chiave non elimina nulla: crea un **delete marker** in cima alla cronologia. L'oggetto smette di comparire negli elenchi, a meno di non chiedere esplicitamente tutte le versioni. Per annullare la cancellazione basta rimuovere il delete marker. Se invece si invia una cancellazione **specificando un version ID**, per esempio `121212`, quella specifica versione viene **eliminata definitivamente**.

### I tag degli oggetti

Come molte risorse AWS, anche gli oggetti S3 possono avere dei **tag**, coppie chiave-valore usate per molti scopi:

- **Organizzazione**: per progetto o per classificazione, per esempio `classification=confidential`, così da poter raggruppare e cercare gli oggetti riservati.
- **Allocazione dei costi**: i tag confluiscono nei report di costo. Si può sapere, per esempio, quanto costano i documenti riservati.
- **Gestione del ciclo di vita**: un processo automatico può annotare in un tag lo stato di avanzamento di un oggetto.
- **Controllo degli accessi**: le policy possono fare riferimento ai tag. Per esempio si può negare all'utente Russ l'accesso a tutti gli oggetti con tag `classification=confidential`.
- **Supporto e monitoraggio**: un tag può contenere un attributo utile ai team operativi.

### Il controllo degli accessi in S3

L'accesso ai dati in S3 si governa con diversi strumenti. Le **policy IAM** (identity-based) sono attaccate a utenti, gruppi e ruoli, come visto nel Capitolo 3. Le **bucket policy** (resource-based) sono attaccate al bucket: si scrivono nello stesso linguaggio JSON di IAM e dichiarano quale principal può fare che cosa nel bucket. Ogni bucket ha **una sola** bucket policy.

Esistono poi le **ACL** (*Access Control List*) di oggetto e di bucket. È una tecnologia più vecchia, nata prima delle bucket policy e di IAM, e somiglia ai permessi di un file system: questa identità può leggere, scrivere e così via. Oggi è possibile, ed è consigliato, **disattivare le ACL** su un bucket e gestire tutto con le policy.

> **Box – Tutto è privato per default**
>
> In S3 tutti i bucket e tutti gli oggetti sono **privati per default**. Rendere pubblico un contenuto richiede un'azione esplicita: disattivare il *Block Public Access* e scrivere una policy che lo consenta. È un'impostazione voluta, perché i casi in cui serve davvero un accesso pubblico sono pochi.

### Esempio: una bucket policy per l'accesso pubblico a un oggetto

Supponiamo di avere, in un bucket nella Region `us-west-2` (Oregon), due oggetti: `index.html` e `myfile.txt`. Vogliamo rendere `index.html` accessibile a un utente **non autenticato**, cioè a chiunque apra l'URL nel browser senza firma né intestazione di autorizzazione. La bucket policy, nella scheda *Permissions*, è la seguente:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::notes-bucket/index.html"
    }
  ]
}
```

`"Principal": "*"` rappresenta chiunque, compreso l'utente anonimo. L'azione consentita è solo `s3:GetObject`. La risorsa è indicata con un **ARN** (*Amazon Resource Name*), il formato con cui tutti i servizi AWS identificano univocamente le proprie risorse. Di solito un ARN contiene anche il numero di account e la Region. Per S3 non servono, perché il nome del bucket è già unico a livello globale e il servizio sa a quale account e a quale Region appartiene. L'ARN di un oggetto è formato da nome del bucket, una barra e chiave.

Nella Console, selezionando `index.html`, compare l'**Object URL**, costruito con il nome del bucket e la chiave: è lo stesso URL che l'SDK costruisce dietro le quinte. Aprendolo nel browser, la pagina viene visualizzata: l'utente anonimo può leggere l'oggetto. Aprendo `myfile.txt` si otterrebbe invece *Access Denied*, perché la policy non lo riguarda.

### Gli S3 Access Point

Un **access point** è un punto di accesso con un nome proprio, creato per un bucket. Un bucket può avere **più access point**, ciascuno con un **ARN** e un **dominio** propri. Il vantaggio principale riguarda la scrittura delle policy. Se lo stesso bucket è usato dal supporto, dagli sviluppatori e dal QA, invece di gestire un'unica bucket policy sempre più complessa si creano tre access point, ciascuno con la propria policy dedicata. È una soluzione molto usata nei data lake e negli scenari big data, dove molti principal diversi accedono agli stessi dati.

## 4.3 Gli endpoint del servizio Amazon S3

Comunicare con S3 significa comunicare con un **endpoint**. Nella maggior parte dei casi è l'SDK o la CLI a costruirlo, ma è utile saperlo riconoscere. A volte bisogna configurarlo esplicitamente, e durante il debug lo si ritrova nei log del client. Le tipologie principali sono quattro.

L'**endpoint standard** è quello già visto nell'esempio precedente:

```
https://bucketname.s3.region.amazonaws.com/key
```

L'**endpoint dual-stack** serve a chi ha requisiti IPv6. Lo si abilita nella configurazione dell'SDK e usa un dominio diverso:

```
https://bucketname.s3.dualstack.region.amazonaws.com/key
```

Interrogando questo dominio con uno strumento come `dig`, si vede che restituisce sia indirizzi IPv4 sia indirizzi IPv6.

Gli **endpoint degli access point** sono unici per ciascun access point: un bucket con tre access point ha tre URL diversi.

Il **website endpoint** viene creato quando si abilita l'hosting di un sito statico sul bucket (Capitolo 5). Serve automaticamente un documento indice (`index.html`) e una pagina di errore, con un URL che contiene il nome del bucket, `s3-website` e la Region:

```
http://bucketname.s3-website-us-west-2.amazonaws.com
http://bucketname.s3-website.eu-south-1.amazonaws.com
```

A seconda della Region il separatore prima della Region è un trattino o un punto, quindi conviene verificare nella documentazione. Abilitare l'hosting web **non sostituisce** l'endpoint standard: ne **aggiunge** un altro.

> **Box – Virtual-hosted-style e path-style**
>
> Gli URL visti finora sono in stile **virtual-hosted**: il nome del bucket fa parte del dominio (`bucket.s3.region.amazonaws.com/key`). Esiste anche lo stile **path-style**, in cui il bucket compare nel percorso (`s3.region.amazonaws.com/bucket/key`). AWS ha deprecato il path-style, che non è supportato per i bucket creati dopo il 30 settembre 2020. Per i bucket recenti il formato corretto è quindi virtual-hosted.

## 4.4 Interagire con Amazon S3 tramite AWS CLI

La CLI offre due livelli di comandi per S3.

I **comandi di basso livello**, `aws s3api`, hanno una corrispondenza **uno a uno** con le API di S3. Tutte le operazioni del servizio sono disponibili, con un controllo molto fine: qualunque cosa si possa fare in S3 si può fare con `s3api`.

I **comandi di alto livello**, `aws s3`, sono un'astrazione sopra quelli di basso livello e automatizzano operazioni composte da più chiamate API. Due sono particolarmente utili:

- `aws s3 cp` copia un file. Se il file supera una certa soglia, passa automaticamente al **caricamento multipart**: avvia il caricamento, invia le singole parti e lo completa, operazioni che con `s3api` andrebbero eseguite una per una.
- `aws s3 sync` **sincronizza** una cartella locale con un bucket o due bucket tra loro. Verifica da sé quali oggetti vanno copiati.

### Esempio: osservare un caricamento multipart

```bash
aws s3 cp 10meg.dat s3://russs-banana
```

Il comando copia nel bucket un file di 10 MB e mostra a video la velocità di trasferimento. Un errore frequente è dimenticare `s3` dopo `aws`: `aws cp` non esiste, perché `cp` è un sottocomando di `aws s3`.

Intercettando il traffico con un proxy, si scopre che questo singolo comando ha generato diverse chiamate:

1. una **POST** che avvia il caricamento multipart, a cui il servizio risponde con un `InitiateMultipartUploadResult` contenente l'identificativo del caricamento;
2. due **PUT**, una per ciascuna parte del file. Ogni risposta contiene un **ETag**, l'identificativo con cui il server riconosce in modo univoco la parte ricevuta;
3. una **POST** finale, `CompleteMultipartUpload`, che elenca gli ETag delle parti e chiede a S3 di ricomporle in un unico oggetto.

È un esempio concreto del valore dei comandi di alto livello: una riga di comando nasconde un'intera sequenza di API.

## 4.5 Interagire con Amazon S3 tramite AWS SDK for Java

Per usare S3 dal codice applicativo con l'SDK si seguono quattro passi:

1. **Configurare le impostazioni di S3 per l'SDK**, nel file di configurazione oppure direttamente nel codice.
2. **Definire le dipendenze**: nel mondo Java si usano Maven o Gradle per includere le librerie dell'AWS SDK.
3. **Creare un client S3**, la classe che rappresenta il servizio e con cui si effettuano le chiamate.
4. **Eseguire le operazioni**, per esempio creare o elencare i bucket.

### Esempio: creare un bucket in Java

```java
Region region = Region.US_WEST_2;

S3Client s3 = S3Client.builder()
        .region(region)
        .credentialsProvider(ProfileCredentialsProvider.create())
        .build();

CreateBucketRequest bucketRequest = CreateBucketRequest.builder()
        .bucket("russ-123-notes-bucket-java")
        .build();

s3.createBucket(bucketRequest);
```

L'SDK for Java 2.x fa largo uso del **builder pattern**. Il client si costruisce con `S3Client.builder()`, impostando la Region e un **credentials provider**: qui `ProfileCredentialsProvider` legge le credenziali dal profilo configurato nei file locali. La chiamata `build()` restituisce il client pronto. Anche la richiesta si costruisce con un builder: `CreateBucketRequest.builder()` espone metodi per impostare i parametri, qui solo il nome del bucket, e `build()` restituisce l'oggetto richiesta popolato. Infine `s3.createBucket(bucketRequest)` invia la richiesta.

Eseguendo il codice nel debugger con un breakpoint sulla chiamata `createBucket`, si vede che l'SDK si occupa di tutto. Costruisce l'endpoint, firma la richiesta, la invia, interpreta la risposta del servizio e, se qualcosa va storto, la trasforma in un'eccezione. Al termine della chiamata il bucket esiste.

## 4.6 Verifica dei concetti

**"In S3 i dati sono memorizzati nei bucket come oggetti, e un oggetto può essere qualsiasi tipo di file: testo, video, immagini o altri formati binari."** *Vero.* Per S3 un oggetto è solo una sequenza di byte e non ci sono restrizioni sul tipo di file.

**"Un bucket S3 è creato a livello globale e non dipende da una Region."** *Falso.* Alla creazione di un bucket si sceglie sempre la Region in cui collocarlo, quindi il bucket è una risorsa regionale. A dover essere unico a livello globale è il **nome**. Sapere dove sono memorizzati i dati è fondamentale anche per i requisiti di conformità normativa.

**"Gli AWS SDK definiscono API di basso livello per Amazon S3, mappate sulle operazioni REST sottostanti."** *Vero.* Le API di basso livello degli SDK, come i comandi `s3api` della CLI, corrispondono direttamente alle operazioni REST del servizio.

**"Abilitare l'hosting di un sito web su un bucket ne cambia l'endpoint."** *Falso.* L'endpoint originale non cambia: viene **aggiunto** un website endpoint.

**"Tutti gli oggetti e i bucket sono privati per default."** *Vero.* Per rendere pubblico un contenuto bisogna intervenire esplicitamente, e i casi d'uso che lo richiedono sono limitati.

**"Le ACL di Amazon S3, configurate tramite IAM, concedono accesso completo a livello di oggetto o di bucket."** *Falso.* Le ACL si configurano nei permessi di S3, non in IAM. Sono una tecnologia precedente a IAM, ed è per questo che vivono nel servizio S3.
