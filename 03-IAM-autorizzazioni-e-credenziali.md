# Capitolo 3
# IAM, Autorizzazioni e Gestione delle Credenziali AWS

## Introduzione alla sicurezza nel cloud

La sicurezza rappresenta uno degli aspetti più importanti nello sviluppo di applicazioni cloud-native. In un ambiente distribuito, composto da decine o centinaia di servizi che interagiscono tra loro, diventa essenziale stabilire con precisione chi può accedere alle risorse, quali operazioni può eseguire e in quali condizioni.

Amazon Web Services affronta questo problema attraverso Identity and Access Management (IAM), il servizio che costituisce il fondamento dell'intero modello di sicurezza della piattaforma. Comprendere IAM significa comprendere il modo in cui AWS controlla l'accesso a qualunque risorsa: bucket S3, tabelle DynamoDB, funzioni Lambda, API Gateway, database, strumenti di monitoraggio e servizi di rete.

Per uno sviluppatore, la conoscenza di IAM non è una competenza accessoria ma una parte integrante della progettazione applicativa. Un'applicazione scalabile ma non adeguatamente protetta rappresenta infatti un rischio operativo e di business.

---

## Il modello di responsabilità condivisa

Prima di analizzare IAM è necessario comprendere il modello di responsabilità condivisa adottato da AWS.

AWS è responsabile della sicurezza dell'infrastruttura cloud, dei data center, dell'hardware, della rete fisica e dei servizi gestiti. Il cliente rimane invece responsabile della configurazione delle autorizzazioni, della protezione dei dati, della gestione delle identità e dell'utilizzo corretto delle risorse.

Questo principio implica che una configurazione errata delle autorizzazioni può esporre informazioni sensibili anche se l'infrastruttura AWS è perfettamente sicura.

---

## Identità e principal

In AWS qualsiasi entità che richiede l'accesso a una risorsa viene definita Principal.

Un principal può essere un utente umano, un'applicazione, una funzione Lambda, un servizio AWS oppure un'entità proveniente da un account differente.

Ogni richiesta inviata alla piattaforma viene valutata identificando innanzitutto il principal che la sta effettuando. Solo successivamente vengono analizzate le autorizzazioni associate.

Questo modello permette di applicare controlli estremamente granulari e di tracciare con precisione le attività svolte da ciascuna identità.

---

## IAM Users

Un IAM User rappresenta un'identità associata a una persona o a un'applicazione.

Gli utenti possono autenticarsi tramite password per l'accesso alla console oppure mediante Access Key e Secret Access Key per l'accesso programmatico.

In passato era comune creare utenti IAM per ogni sviluppatore. Oggi le best practice raccomandano l'utilizzo di sistemi federati e credenziali temporanee, limitando gli utenti statici ai soli casi realmente necessari.

Un'organizzazione matura dovrebbe considerare gli utenti IAM come un'eccezione e non come la modalità principale di gestione delle identità.

---

## IAM Groups

I gruppi consentono di assegnare autorizzazioni a insiemi di utenti.

Invece di configurare ogni singolo utente, è possibile creare gruppi come Sviluppatori, Amministratori, Operatori o Auditor e applicare le policy direttamente al gruppo.

Questo approccio riduce la complessità amministrativa e garantisce maggiore coerenza nella gestione delle autorizzazioni.

---

## IAM Roles

I ruoli rappresentano uno dei concetti più importanti dell'intero ecosistema AWS.

A differenza degli utenti, un ruolo non possiede credenziali permanenti. Viene invece assunto temporaneamente da utenti, applicazioni o servizi AWS.

Una funzione Lambda può assumere un ruolo per leggere dati da DynamoDB. Un'istanza EC2 può assumere un ruolo per accedere a un bucket S3. Un utente federato può assumere un ruolo amministrativo per svolgere specifiche attività operative.

L'utilizzo dei ruoli elimina la necessità di memorizzare credenziali statiche nel codice applicativo, migliorando significativamente la sicurezza.

### Approfondimento: perché i ruoli sono preferibili alle access key

Le credenziali permanenti possono essere smarrite, condivise impropriamente o esposte accidentalmente in repository Git.

I ruoli generano invece credenziali temporanee, con durata limitata e possibilità di revoca immediata. Per questo motivo AWS raccomanda sistematicamente il loro utilizzo.

---

## Le policy IAM

Le autorizzazioni vengono espresse attraverso documenti JSON chiamati Policy.

Una policy descrive azioni consentite o negate su specifiche risorse.

Ogni policy contiene tipicamente:

- Effect
- Action
- Resource
- Condition

L'elemento Effect determina se l'operazione è consentita o negata. Action identifica le API autorizzate. Resource specifica le risorse coinvolte. Condition introduce regole aggiuntive basate sul contesto.

Le policy rappresentano il linguaggio con cui viene definita l'intera strategia di sicurezza AWS.

---

## Identity-based e Resource-based Policy

AWS utilizza due categorie fondamentali di policy.

Le Identity-based Policy vengono associate a utenti, gruppi o ruoli.

Le Resource-based Policy vengono invece applicate direttamente a una risorsa, come un bucket Amazon S3.

La combinazione di questi due modelli consente una gestione estremamente flessibile degli accessi, specialmente negli scenari multi-account.

---

## Il principio del minimo privilegio

Una delle pratiche più importanti nella sicurezza cloud consiste nel concedere esclusivamente i privilegi necessari.

Se una funzione Lambda deve soltanto leggere una tabella DynamoDB, non dovrebbe possedere autorizzazioni di scrittura o cancellazione.

Se uno sviluppatore deve amministrare API Gateway non dovrebbe automaticamente ottenere accesso completo agli ambienti di produzione.

L'applicazione sistematica del principio del minimo privilegio riduce la superficie d'attacco e limita gli effetti di eventuali compromissioni.

---

## Credenziali temporanee e AWS STS

AWS Security Token Service consente di generare credenziali temporanee utilizzabili per un periodo limitato.

Questo approccio è oggi considerato uno standard per la maggior parte delle architetture moderne.

Le credenziali temporanee risultano particolarmente utili in scenari che coinvolgono applicazioni serverless, federazione delle identità e accesso cross-account.

---

## Accesso cross-account

Le grandi organizzazioni utilizzano frequentemente più account AWS.

IAM permette di creare relazioni di trust che consentono a utenti o servizi di un account di assumere ruoli presenti in un altro account.

Questo modello facilita la separazione tra ambienti di sviluppo, test e produzione, migliorando il controllo amministrativo e la governance.

---

## Permission Boundaries

Le Permission Boundaries introducono un ulteriore livello di controllo.

Anche quando una policy concede determinate autorizzazioni, una boundary può limitarne la portata massima.

Questa funzionalità viene spesso utilizzata nelle grandi organizzazioni per impedire escalation involontarie dei privilegi.

---

## Audit e tracciabilità

La sicurezza non si limita alla concessione delle autorizzazioni. È altrettanto importante tracciare le attività svolte.

Le organizzazioni devono essere in grado di rispondere a domande quali:

- Chi ha modificato una policy?
- Chi ha eliminato una risorsa?
- Chi ha effettuato un accesso amministrativo?

La tracciabilità costituisce uno dei pilastri della sicurezza operativa e della conformità normativa.

---

## Best practice per gli sviluppatori

Gli sviluppatori dovrebbero evitare l'utilizzo dell'account root per le attività quotidiane, preferire i ruoli IAM alle credenziali statiche, limitare i privilegi concessi alle applicazioni, eseguire revisioni periodiche delle policy e conservare le credenziali esclusivamente in sistemi approvati.

Le autorizzazioni dovrebbero essere considerate parte integrante del codice applicativo e gestite tramite Infrastructure as Code.

---

## Caso di studio: sicurezza dell'applicazione Polly Notes

Nell'architettura Polly Notes, una funzione Lambda necessita di accedere alle tabelle DynamoDB contenenti le note degli utenti.

La soluzione corretta consiste nell'associare alla funzione un ruolo con permessi limitati alle specifiche operazioni richieste.

Analogamente, gli utenti autenticati tramite Cognito devono ricevere esclusivamente le autorizzazioni necessarie per accedere alle proprie informazioni applicative.

Questo esempio dimostra come il controllo granulare degli accessi sia fondamentale per costruire applicazioni sicure e scalabili.

---

## Conclusioni

IAM rappresenta il cuore del modello di sicurezza AWS. Utenti, gruppi, ruoli, policy e credenziali temporanee consentono di implementare controlli estremamente granulari e di proteggere efficacemente le risorse cloud.

La comprensione approfondita di questi concetti costituisce un prerequisito fondamentale per lo sviluppo di applicazioni cloud-native sicure, resilienti e conformi alle migliori pratiche del settore.
