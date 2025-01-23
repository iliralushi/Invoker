**File System**
È una gerarchia di **file** e **directory** memorizzati in un dispositivo di storage.

**File**
È una **sequenza di byte** memorizzata su un dispositivo di storage. I byte contenuti sono **arbitrari** ed è identificabile univocamente con un nome. Contiene:
- Il file control block.
- Un insieme di blocchi di dati.

**File Control Block**
È una struttura dati memorizzata su disco che contiene proprietà del file importanti.
- Proprietario, date notevoli (creazione, modifica), dimensione, permessi di accesso, puntatori a blocchi di dati, solo sul **primo** blocco o su **tutti**.

**Recupero dei Metadati**
In GNU/Linux la chiamata di sistema `stat()` ritorna una struttura dati con all'interno i **metadati** di un file. I parametri passati sono due:
- Il nome del file.
- Il puntatore ad una struttura dati dove verranno inseriti i metadati.

**Tipo di File**
I file vengono classificati in **tipi**. Viene quindi assegnato un **programma di default** per ogni tipologia di file.
- **Esempio**: File multimediale -> Lettore audio/video.

**Estensione**
Permette di riconoscere il **tipo del file**.
- `<nome_file> <separatore> <estensione>.`
- `<ciao>      <.>          <txt>.`

**Associazione Diretta**
Meccanismo usato dall'SO che legge l'estensione di un file e gli associa un **programma tipo**. Il nome dell'applicazione tipo è **cablato nell'SO** tramite comando `lesspipe.`

**Lista di Estensioni**
- Ciascuna applicazione gestisce una **lista di estensioni**. Diverse applicazioni possono gestire diverse estensioni ma un solo programma può essere assegnato **di default**.
- Quando un file viene aperto normalmente l'SO carica l'applicazione di default. Quando viene usato **apri con** si possono vedere tutti i programmi compatibili con l'estensione.

**Magic Numbers**
Sono una sequenza di byte posizionati in **offset strategico**. Servono per riconoscere il tipo di file. Il path `/usr/share/file/misc/magic` contiene il **magic file** dove son contenuti tutti i magic numbers.
- **Esempio**: Un programma ELF (eseguibile) ha nel byte 2, 3, 4 i caratteri 'E' 'L' 'F' e per verificare si può usare il comando `less (/bin/ls).`

**Comando File**
`file` scandisce il file alla ricerca del magic number. Si ferma quando ne trova uno corretto.

**Loader**
In GNU/Linux un file eseguibile è associato ad un programma detto loader (**caricatore**).
- Carica in memoria le **librerie** necessarie all'esecuzione ed il programma stesso.
- Il loader si trova in `/lib/ld-linux.so.2`

**Script**
Il loader controlla se all'inizio del file c'è la **she-bang** (**#!bin/bash**) che specifica l'interprete dello script. Esso quindi carica l'interprete con le varie librerie e lo esegue prendendo come argomento il nome dello script.

**Organizzazione File**
Due aspetti per un file:
- **Logico**: I file vengono accessi per **unità logiche**.
- **Fisico**: Unità logiche vengono immagazzinate nei **blocchi fisici** (**settori**) del disco.

**Settore**
Sono i blocchi fisici del disco, è grande **512 byte** negli hard disk.
- Nei files il record logico è più piccolo. Il gestore del file system cerca di immagazzinare più record logici nei settori.

**Organizzazione Logica**
Un file può essere organizzato logicamente in tre modi:
- Flusso di byte, record logici, indice.

**Flusso di Byte**
Il file è visto come una sequenza di **record logici** grandi uno. L'applicazione legge flussi di byte. È suo compito dare un significato al flusso di byte.

**Record Logici**
Il file è visto come una sequenza di record logici con `dimensione > 1.` L'applicazione legge e scrive un numero **variabile** di record logici.
- Possono essere linee o strutture dati.

**Indice**
Il file contiene una sequenza di record logici di lunghezza **fissa**. Ciascun record contiene un indice di posizione **fissa**.
- Con una binary search sull'indice troviamo l'elemento `i` in `O(log(n)).`

**Indice II**
In caso di inserzioni/cancellazioni dei record frequenti l'ordinazione del file diventa costosa.
- **Doppio**: Si ha un file indice con la coppia `<chiave, record logico>` ed un **file dati** con il contenuto dei record.
- **Gerarchico**: Si ha vari file indice. Un file indice **secondario** punterà al file dati.

**Metodi Accesso**
Un file può essere acceduto in tre modi:
- Sequenzialmente, direttamente, ad indice.

**Accesso Sequenziale**
Il file viene accesso **sequenzialmente**, quindi un record dopo l'altro (**nastro**).
- **Esempio**: Copie di file, caricamento di un programma.

**Accesso Diretto**
Il file viene accesso a singoli **blocchi fisici** (**disco**) scelto **direttamente** (**casualmente**).
- **Esempio**: Deframmentazione del disco.

**Accesso ad Indici**
Il file viene accesso **per indici**, **casualmente** (**base di dati**).
- **Esempio**: DBMS.

**Relazione Organizzazione -> Accesso**
- Organizzazione a Flussi di Byte -> Accesso Sequenziale.
- Organizzazione per Record Logici -> Accesso Diretto.
- Organizzazione per Indici -> Accesso ad Indici.

Non è sempre così, per esempio la copia di un file contenente DBMS avviene tramite lettura e scrittura **a blocchi**. È il tipo di applicazione che determina il tipo di accesso.

**End of File**
Con l'accesso **sequenziale** e **diretto** si può eccedere l'EOF. L'SO deve catturare e segnalare questo caso all'utente. Può farlo con due strategie:
- Ritorno di un valore opportuno all'operazione di lettura.
- Uso di un'operazione che controlla se la lettura è arrivata ad EOF.

**File Aperti**
Ciascun PCB di un processo ha il campo **tabella file aperti**. Contiene una struttura dati che si trova **all'interno del kernel** che descrive i file aperti da un processo.
- In Linux la `task_struct` ha un campo `struct files_struct* files.`

**I/O Diretto e Bufferizzato**
- **Diretto**: Le operazioni vengono svolte direttamente dal kernel singolarmente.
- **Bufferizzato**: Le operazioni sono gestite da un **buffer** intermediario. La lettura/scrittura viene messa in stallo finchè il buffer si riempie.

**Buffering in GNU/Linux**
- **Applicativo**: Fa uso delle funzioni di libreria del C. Le operazioni di I/O possono lavorare con un buffer **più grande di un carattere**. Viene usata una chiamata di sistema.
- **Kernel**: Memorizza la quantità di letture svolte e raggruppa le operazioni di scrittura.

![[Buffer-Linux.png]]

**Read Bufferizzata**

![[Read-Linux.png]]

**Buffer Applicativo**
- **No Buffer**: Il buffer ha dimensione nulla.
- **Singola Riga**: Considerato pieno quando **raggiunge un newline** oppure raggiunge la massima capienza.
- **Completo**: Pieno quando raggiunge la massima capienza.

**Flushing Buffer**
L'operazione di flushing è la scrittura del buffer su disco. È possibile forzarla.
- **Sync**: Flusha il buffer **del kernel**.
- **Flush**: Flusha il buffer **applicativo**.

**File Descriptor**
In caso di I/O **diretto** un file aperto viene identificato da un **int** chiamato **file descriptor** che viene usato come indice all'elemento relativo della **tabella dei file aperti**.
- Si ottiene tramite valore di ritorno della funzione `open().`

**FILE** *
In caso di I/O **bufferizzato** un file aperto viene chiamato **stream** ed è identificato da un puntatore ad una `struct _IO_FILE(FILE *)`
- La struct contiene il **file descriptor** che identifica l'elemento esatto sulla **tabella dei file aperti**, un puntatore al **buffer** di default, informazioni di stato dello stream.
- Si ottiene tramite valore di ritorno della funzione `fopen().`
  
![[Filedescriptor-File*.png]]

**Funzioni I/O Diretto**

![[Funzioni-IO-Diretto.png]]
![[Funzioni-IO2-Diretto.png]]

**Funzioni I/O Bufferizzato**

![[Funzioni-IO-Buffer.png]]
![[Funzioni-IO2-Buffer.png]]
![[Funzioni-IO3-Buffer.png]]

**Directory**
È un **file** contenente la coppia `(nome, puntatore).`
- **Nome**: Nome ad alto livello per contenitore di informazioni.
- **Puntatore**: Punta ad una struttura dati contenente i metadati di un file (FCB) oppure un'altra directory.

**Strutture di Directory**
Una directory può essere rappresentata in diversi modi.
- A singolo livello, a due livelli, ad albero, **a grafo**.

**Difetti Primi Schemi**
- **Singolo Livello**: Scomodo per i sistemi multiutente.
- **Doppio Livello**: Isolamento completo tra utenti.
- **Albero**: Scomodo condividere file con altri utenti. Impossibile creare **links**.

**Directory a Grafo Aciclico**
- **Vertici**: Directory o file.
- **Vertice Radice**: Directory **radice** del file system `(/ o C:\).`
- **Archi**: Relazione tra directory e file.

![[Directory-Grafo-Aciclico.png]]

**Hard Link**
Elementi delle directory possono ***puntare*** al contenuto dello stesso file.
- **Esempio**: In Debian i programmi `sudo` e `sudoedit` sono hard links. Due elementi di directory puntano allo **stesso file**. Il contatore hard link è due. Il comando di verifica è `ls -l /usr/bin/sudo*.`
- **Rationale**: Il programma si comporta in maniera diversa rispetto al nome `(ARGV[0]).`  Si comportano come programmi **indipendenti**.

**Limitazione Grafo Aciclico**
- Non è possibile introdurre un ciclo nel grafo.
- Non è possibile creare un **hard link** ad **un file** contenuto in un altro file system.

**Directory Grafo Ciclico**
- **Vertici**: Directory o file.
- **Vertice Radice**: Directory **radice** del file system `(/ o C:\).`
- **Archi**: Relazione tra directory e file.

![[Directory-Grafo-Ciclico.png]]

**Soft Link**
Sono in grado di **concludere cicli**. Il contenuto ed i metadati del file sono **unici**.
- **Esempio**: In Debian il comando eseguibile `xzcat` è soft link al file `xz.` La dimensione del link simbolico è due. Gli hard link sono posti ad **uno**. Due file; uno col contenuto originale ed uno col percorso. Verificabile con `ls -l /usr/bin/xz{cat,}.`
- **Rationale**: Gli stessi dell'hard link.

**Soft Link vs Hard Link**
- I soft link possono essere creati tra vari file system. Se il file non è disponibile viene colorato di rosso.
- I soft link possono essere creati verso **directory superiori**.

**Gestione Cicli Infiniti**
- **Scenario**: Il comando `find` mostra i contenuti di un sottoalbero delle directory con un **soft link** alla directory di partenza. Si ha un ciclo infinito, viene gestito dalle apps.
- L'immagine mostra il comando `find /home/andreoli -name andreoli -follow.`

![[Ciclo-Infinito-Hardlink.png]]

**Cancellazione File**
L'hard link di un file **è un riferimento**. Si ha un contatore di hard links per un file.
- Quando si cancella un hard link il contatore viene decrementato di uno. Se il contatore è zero **si stacca il contenuto del file puntato** dal suo FCB.

**Osservazione**
Il contenuto del file non viene **fisicamente cancellato** ma staccato dal file system. Può venir recuperato tramite programmi.
- Un file aperto rimane accessibile fino alla chiusura del **descrittore.**

**Gestione Links**

![[Funzioni-Link.png]]

**Implementazione Directory I**
È una **lista semplice** di strutture dati. Ciascuna lista contiene:
- Un **nome file/directory** ed un **puntatore ad FCB**.
- L'implementazione è semplice ma l'efficienza è molto bassa (scansione lineare).

**Implementazione Directory II**
Un'altra implementazione delle directory è fatta con le **hash tables** e le **liste semplici**.
- **Lista**: Contiene la coppia `<nome, puntatore FCB>.`
- **Hash Table**: La **chiave** è un **hash** del nome file. Il **valore** è un puntatore all'elemento della lista corrispondente.

**Implementazione Directory III**
La versione attuale fa uso di un **albero binario**.
- L'albero contiene hash dei nomi dei file **ordinati alfanumericamente**.
- I nodi **foglia** contengono puntatori ai FCB.

**Accesso Directory**
Avviene tramite un **descrittore di directory**, simile a quello dello stream nell'I/O bufferizzato.
- Rappresentato dalla `struct __dirstream.`
- Si ottiene un puntatore al descrittore tramite la funzione `opendir().`

**Elementi Directory**
In GNU/Linux ogni elemento di una directory è identificato dalla `struct dirent.`
- Contiene il nome dell'elemento, tipo di file, puntatore FCB associato al file.
- Struct definita nel file `$LINUX/include/dirent.h.`

**Operazioni su Directory**
La directory viene gestita come uno stream di **record logici** di tipo `struct dentry.`
- Riposizionamento dello stream, recupero della sua posizione, lettura della prossima `struct dentry.`

**Gestione Directory**

![[Funzioni-Directory.png]]![[Funzioni-Directory2.png]]

**Aggancio Nuovo File System**
