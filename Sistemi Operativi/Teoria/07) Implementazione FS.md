### Preparazione Dispositivo

**Formattazione a Basso Livello**
Procedura con la quale si prepara un dispositivo al **primo uso**. 
- HDD viene marcato in **settori**, l'unità più piccola di spazio.
- Solitamente si svolge per creare un nuovo file system.

**Formato di Settore**
- **Preambolo**: Marcatura che segna l'inizio di un nuovo settore.
- **Error Correcting Code (ECC)**: Codice usato per la correzione automatica di errori.
- **Intersector Gap**: Separazione tra settori.

**Partizionamento**
Divisione del disco **statica** in più zone, chiamate **partizioni**. Tipicamente contengono:
- File system, area di **swap**, spazio non utilizzato.

![[Partizione-Disco.png]]

**Vantaggi**
- Possibilità di installare più SO.
- La corruzione del file system di una partizione non ha effetto sulle altre.
- Possibilità di controllare più file system in parallelo.
- Implementazione della partizione di swap.

**Svantaggi**
- È difficile modificare le partizioni **una volta create**. Ormai, è un concetto obsoleto.

**Volume Fisico**
**Partizione** di un disco rigido usabile per i volumi.
- Il contenuto è organizzato in **blocchi fisici**.

![[Volume-Fisico.png]]

**Gruppo di Volumi**
Insieme dei blocchi fisici offerti da un gruppo di volumi fisici. I blocchi fisici vengono utilizzati per creare **volumi logici**.

**Volume Logico**
Insieme di **blocchi logici** pescati dal gruppo di volumi. Risolve il problema del partitioning, vengono usate come **partizioni** e non sono rigide.
- Modificabili, estendibili, criptabili.

**Superblocco**
**Primo** blocco del file system. Contiene le informazioni su se stesso.
- Tipo, dimensione, stato, posizione di altre strutture...
- Ogni file system definisce il suo superblocco. Col formato **EXT4** si usa la struttura dati `struct ext4_super_block` definita in `$LINUX/fs/ext4/ext4.h.`

**Formattazione ad Alto Livello**
Procedura con la quale un file system viene inizializzato su partizione o volume logico.
- **Scrive il superblocco**.
- Prepara spazio per gli FCB e l'elenco dei blocchi liberi.
- Prepara una sequenza di blocchi liberi.

**Layout di EXT3**

![[EXT3.png]]

### Allocazione Blocchi

**Scenario**
Diversi processi vogliono scrivere su disco **contemporaneamente**. L'SO deve allocare i blocchi del file system ai diversi processi durante le operazioni di scrittura.
- Come usare lo spazio in maniera efficace?
- Come garantire un accesso veloce ai file?

**Frammentazione Interna**
- Se il blocco è grande ed il processo scrive file piccoli c'è il rischio che il blocco venga usato in minima parte. Abbiamo uno **spreco** di spazio.
- Se il blocco è piccolo ed il proecsso scrive file grandi allora può richiedere anche molti blocchi. Verranno occupati in **ordine sparso**.

**Frammentazione Esterna**
Un file grande non può essere memorizzato. Esistono blocchi sufficienti per poterlo memorizzare, ma non esiste alcuna sequenza **contigua** di blocchi.

**Assegnazione dei Blocchi**
- Assegnazione contigua, concatenata, indicizzata.

**Assegnazione Contigua**
Il file occupa una posizione **contigua** di blocchi. L'ordinamento dei blocchi fisici è dato dagli **indirizzi** associati ai blocchi fisici.
- Primo blocco del file ha indirizzo più basso, il secondo è l'indirizzo successivo etc...
- **Esempio**: File con `n` blocchi, memorizzazione parte dal blocco `b`. Vengono occupati i blocchi `b, b+1, b+2, ... , b+n-1.`

![[Assegnazione-Contigua.png]]

**Vantaggi**
- La lettura sequenziale di un file è ottimale. La testina del disco si sposta al più una volta, i blocchi sono **contigui**.
- Se un file è grande può essere memorizzato su **blocchi contigui**. Gli spostamenti della testina sono minimi.

**Svantaggio - Frammentazione Esterna**
In caso di creazioni e cancellazioni frequenti si avrà **frammentazione esterna**, quindi si alterneranno blocchi occupati a blocchi vuoti.
- Risolvibile con **deframmentazione**.

**Svantaggio - Stima Dimensione**
È necessario **stimare la dimensione del file**. Non si riesce a trovare una sequenza di blocchi contigui altrimenti. Un file rischia di avere una dimensione errata se la stima è sbagliata.
- A volte è **impossibile**. Questo succede con programmi che crescono lentamente nel tempo.

**Estensioni**
Tentativo di risolvere la frammentazione esterna. Ad un file viene assegnata una porzione di blocchi contigui. Alla fine della scrittura **viene assegnata un'altra porzione di blocchi**. Il file system segna l'inizio e la lunghezza dell'estensione.

**Problemi**
È molto difficile scegliere la lunghezza dell'estensione.
- **Troppo Piccola**: Con troppe creazioni/cancellazioni si ripresenta **il problema**.
- **Troppo Grande**: L'estensione rischia di non essere utilizzata. Problema della frammentazione interna.

**Assegnazione Concatenata**
Un file è composto da una **lista concatenata** di blocchi.
- L'elemento della directory contiene un puntatore al primo blocco del file.
- Il blocco `i-mo` contiene un puntatore al blocco `i-mo+1.`
- L'ultimo blocco contiene un puntatore ad un blocco fittizio `(-1).`

![[Assegnazione-Concatenata.png]]

**Vantaggi**
- Il problema della frammentazione esterna è risolto. I blocchi possono essere occupati "casualmente." Inoltre, non è più richiesta la stima della dimensione del file.

**Svantaggi**
- L'accesso sequenziale è efficiente finchè i blocchi occupati sono in zone sparse del disco. La testina si muove troppo accumulando diversi **ms di ritardo**.
- L'accesso diretto ad un blocco non è efficiente. Essendo una **lista** dobbiamo scandirla dal primo blocco fino ad arrivare a quello desiderato.
- L'affidabilità **peggiora** rispetto all'assegnazione contigua. Se un puntatore si corrompe perdiamo il contenuto del file.
- Parte dello spazio su disco va sprecata in puntatori. Ogni blocco vale `(512 - 4) byte.`

**Cluster**
Sono un gruppo contiguo di blocchi. Risolvono il problema dello spazio, vengono usati meno puntatori. Il file system **alloca clusters**.
- Rischio di frammentazione interna.

**File Allocation Table**
Variante dell'assegnazione concatenata. All'inizio di ogni partizione viene dedicato spazio per contenere una **tabella di allocazione del file** (**FAT**).
- Ciascun blocco è identificato da un **numero intero univoco** detto **indice**. La FAT è acceduta attraverso indici.

**FAT - Assegnazione Concatenata**
L'indice contenuto in un elemento della FAT punta al prossimo blocco del file. L'elemento che corrisponde all'ultimo blocco del file contiene un valore reale. Ciascun elemento della directory contiene l'indice del primo blocco in FAT.
- **Implementazione Linux**: `$LINUX/fs/fat/fatent.c`

![[FAT.png]]

**Svantaggi**
- Un accesso in **lettura** al file comporta un accesso alla FAT per ogni blocco.
- Un accesso in **scrittura** comporta un accesso alla FAT per ogni **blocco scritto** e una scansione alla FAT per trovare un blocco libero. Eccessivo movimento della testina dato che la FAT è localizzata ad **inizio disco**.

**FAT Cache**
Per ridurre il movimento del disco la FAT è mantenuta in **memoria centrale**. La cache è implementata come una lista collegata di `struct fat_cache.` Ciascuno di questi elementi mappa un gruppo di cluster nel file. Questi corrispondono ai cluster contenuti su disco.
- **File**: `$LINUX/fs/fat/cache.c`

**Assegnazione Indicizzata**
Ad un file è assegnato un **blocco indice**. Contiene una sequenza di indirizzi di blocchi su disco appartenenti a file. Usato da UNIX.
- L'elemento della directory contiene un puntatore al blocco indice.
- Non appena creato un file il blocco indice è pieno di puntatori nulli.
- Il puntatore `i-mo` contiene l'indirizzo `i-mo` del blocco `i-mo` del file.

![[Assegnazione-Indicizzata.png]]

**Vantaggi**
- Il problema della frammentazione esterna è risolto. Un blocco del file può essere memorizzato in qualsiasi parte del disco.
- La frammentazione interna è sensibilmente ridotta.
- L'accesso al blocco avviene in tempo costante. Basta accedere all'`i-mo` elemento del vettore di puntatori.

**Svantaggi**
- La grandezza del file è limitata. Il blocco indice può contenere puntatori limitati. Per salvare un file più grande serve usare più blocchi indice.
- La dimensione del blocco indice è critica. Se **troppo piccola** corriamo il rischio di non poter rappresentare il file, se **troppo grande** possiamo avere frammentazione interna.

**Blocchi Indice Multipli**
- Schema concatenato, indice a più livelli, schema combinato.

**Schema Concatenato**
Il blocco indice ha una **intestazione** che contiene il nome del file ed i primi 100 puntatori del disco. L'ultimo elemento del blocco indice è **NULL** nel caso di un file piccolo, altrimenti ha un puntatore al **prossimo** blocco indice.

**Indice a Più Livelli**
Si usa un blocco indice di primo livello. 
- Gli **elementi** del blocco indice di primo livello puntano a **blocchi indice** di secondo livello.
- Gli **elementi** del blocco indice di secondo livello puntano ai **blocchi del file**.

Per accedere ad un blocco, il sistema operativo:
- Usa l'indice del primo per trovare il blocco indice del secondo.
- Usa l'indice del secondo per trovare il blocco di dati.

**Dimensione Massima File**
Disco fittizio.
- **Dimensione Blocco**: `4096 byte.`
- **Dimensione Puntatore**: `4 byte.`

Con due livelli di indici abbiamo `(4096/4)^2 = 1.048.576 blocchi di dati.`
- La dimensione massima di un file è di `4GB.`

**Schema Combinato**
A ciascun file è assegnato un blocco **descrittore** noto come ***inode***. È l'FCB del file.
Sono contenuti **15 puntatori a blocchi**:
- **Puntatori 1-12**: Puntatori diretti (**blocchi diretti**).
- **Puntatore 13**: Puntatore a puntatore di blocchi (**blocco indiretto singolo**).
- **Puntatore 14**: Puntatore a puntatore a puntatore di blocchi (**blocco indiretto doppio**).
- **Puntatore 15**: Puntatore a puntatore a puntatore a puntatore di blocchi (**blocco indiretto triplo**).

![[Inode.png]]

**Vantaggi**
- Per file piccoli l'accesso è di `O(1)` tramite blocchi diretti.
- Per file più grandi l'accesso è di `O(log(n)),` `n` dipende dal numero di blocchi del file.
- La dimensione massima di un file è aumentata notevolmente.

**Puntatore Inode EXT3**
È definito nella struttura struct `ext3_inode` nel file `$LINUX/fs/ext3/ext3.h.`
- Il campo `i_block` contiene l'array di puntatori.
- La funzione `ext3_block_to_path()` riceve il numero di un blocco e ritorna il percorso che seguono i puntatori. Definita nel file `$LINUX/fs/ext3/inode.c.`

### Gestione Spazio Libero

**Problema**
Seppure il disco contiene moltissimi blocchi essi sono **limitati**.
- Dobbiamo tracciare i **blocchi liberi**.

**Elenco Blocchi Liberi**
- Vettore di bit, lista concatenata, raggruppamento, conteggio.

**Vettore di Bit**
Ciascun blocco del disco è un gigantesco vettore di bit.
- Bit settato a 0 se il blocco è allocato.
- Bit settato a 1 se il blocco è libero.

**Definizioni**
- **Costante BIT_PER_PAROLA**: Dimensione di una parola (32/64 bit).
- **Parametro i**: Indice del bit `i-mo.`
- **Vettore [n]**: Vettore di `n` parole di lunghezza `BIT_PER_PAROLA.`
- **Bit**: Il valore del bit `i-mo.`

**Operazioni Vettore di Bit**
- **Offset Parola**: `i / BIT_PER_PAROLA.`
- **Offset Bit**: `i % BIT_PER_PAROLA.`
- **Parola Con Bit**: `*(vettore + offset parola).`
- **Bit-i**: `parola_con_bit & (2^offset_bit).`

![[Calcolo-Bit.png]]

**Individuazione Efficiente**
I processori moderni hanno istruzioni assembly che trovano in un clock il **primo** bit messo ad `1` in una parola di `16, 32, 64bit. Sono le istruzioni bsf, bsr.` 
- Si scandisce il vettore di bit una parola per volta. Se la parola vale zero si passa alla successiva, se vale uno viene eseguito il codice macchina per trovare il primo bit.

**Bitmap in EXT3**
Nei metadati in EXT3 viene allocato un blocco dove viene salvata la bitmap dei blocchi.
- Il blocco è puntato da un camp della struttura dati ext3_group_desc definita nel file `$LINUX/fs/ext3/ext3.h.`
- Anche gli **inode** hanno questa bitmap definita in `bg_inode_bitmap.`

**Gestione Bitmap**
La gestione della bitmap è affidata ad alcune funzioni di `$LINUX/fs/ext3/ext3.h.`
- `ext3_set_bit():` Imposta ad 1 un bit.
- `ext3_clear_bit():` Imposta a 0 un bit.
- `ext3_test_bit():` Ritorna il valore di un bit.
- `ext3_find_next_zero_bit():` Trova l'indice del prossimo bit settato a 0.
- Funzioni usate anche dagli allocatori di blocchi ed inode.

**Lista Concatenata**
I blocchi liberi sono tutti collegati tra di loro tramite puntatori posti al loro stesso interno.
- Il puntatore al primo blocco libero viene memorizzato in una posizione **speciale** e viene mantenuto in **memoria centrale**.

![[Blocchi-Liberi-Lista.png]]

**Vantaggi**
- Soluzione più semplice rispetto al vettore di bit.

**Svantaggi**
- Molto meno efficiente rispetto al vettore di bit. La scansione della lista di blocchi ha costo lineare `O(n).`

**Raggruppamento**
Riprende il modello di **lista concatenata**.
- Il primo blocco contiene una lista di `n` blocchi. `N-1` blocchi sono **liberi**, il blocco `n` invece contiene un'altra lista di `n` blocchi **liberi**.
- La scansione di blocchi è più efficiente; si navigano i blocchi contenenti la lista.

**Conteggio**
Approccio alternativo alla memorizzazione dei blocchi.
- Si mantiene una tabella `<indirizzo blocco, conteggio>` dove il conteggio corrisponde al numero di blocchi liberi **contigui**.

### Considerazioni - Efficienza

**Efficienza**
Capacità di eseguire una task con il minimo sforzo. Nel contesto dei file system si intende:
- Uso **compatto** dello spazio su disco.
- Rappresentazione di file molto grandi col minimo uso di spazio dedicato ai **metadati**.

**Piazzamento Inode Disco**
Sono tutti raggruppati vicini al **superblocco**. L'accesso sequenziale al disco porta in memoria un numero grande di inode.
- La lettura **ricorsiva** delle directory è veloce.

**Uso Cluster Dimensione Diversa**
Si usano cluster diversi a seconda della situazione.
- Memorizzazione di file piccoli usano cluster piccoli.
- File di dimensione crescente usano cluster di dimensione crescente.

**Metadati File I**
In particolare, i **timestamp di accesso** al file sono critici. Ad ogni accesso si ha:
- Una lettura e scrittura del blocco FCB, modifica metadati.

**Metadati File II**
Se un file è acceduto frequentemente è molto inefficiente come sistema. Abbiamo costantemente una lettura ed una scrittura nel FCB.
- **Soluzione**: Copia del FCB in **memoria centrale**, si aggiorna **periodicamente** l'FCB su disco.

**Dimensione Puntatori**
La dimensione massima del file è determinata dai puntatori. Gli SO introducono puntatori più grandi rispetto a quelli elencati. Un buon trade-off sono i puntatori a `48bit.`
- `16bit:` Dimensione massima file è di `64Kb.`
- `32bit:` Dimensione massima file è di `4GB.`

**Strutture Statiche vs Dinamiche**
- **Statiche**: Prestazione migliore rispetto ad allocazione dinamica ma si può avere uno spreco di memoria notevole oppure problemi se la memoria allocata è troppo poca.
- **Dinamiche**: Prestazione peggiore perchè la memoria viene gestita a runtime. Efficienza della memoria perfetta.

### Considerazioni - Prestazioni

**Prestazione**
Indica l'eseguimento delle task al massimo delle capacità. Nel contesto del file system:
- Aumento della velocità delle operazioni di I/O.
- Applicazione della **gerarchia di memoria**.

**Scritture Sincrone e Asincrone**
- **Sincrone**: Avvengono nell'ordine in cui il file system le riceve. La scrittura avviene più velocemente possibile, non si usa alcun buffer.
- **Asincrone**: Viene usato un buffer intermedio, si ritorna direttamente al processo chiamante. È compito dell'SO svuotare il buffer su disco. L'aggiornamento dei metadati è comunque sincrono.

**Scritture Asincrone Pro/Contro**
- **Pro**: L'SO può decidere un momento efficiente per svuotare il buffer su disco. La `write()` diventa velocissima, ritorna al processo chiamante dopo aver scritto su buffer.
- **Contro**: Se la macchina va in crash si perdono i dati salvati su buffer. Se i metadati sono già stati aggiornati si ha una inconsistenza dei dati.

**Assegnazione Ritardata**
Si usa il meccanismo di scrittura asincrona. Con l'assegnazione ritardata si aspetta l'ultimo momento possibile prima di svuotare i buffer su blocchi liberi.
- L'SO decide di scrivere i blocchi su disco. L'utente comanda un flush dei buffer.

**Pro/Contro**
- **Pro**: I metadati possono essere aggiornati in modalità sincrona. La probabilità di scrivere file contigui è più alta.
- **Contro**: Se la macchina va in crash vengono persi i dati salvati.

**Lettura Anticipata**
È il prelievo di un certo numero di blocchi contigui successivi all'ultimo blocco appena letto. In una lettura sequenziale, saranno quei blocchi i prossimi letti dall'applicazione.
- Miglioramento delle prestazioni in lettura.

**Gerarchia di Memoria**
Per migliorare la performance del file system è necessario spostare le **strutture dati** più utilizzate nei dispositivi di memoria più veloci. Tutte le strutture dati sono salvate su disco, le più utilizzate vengono salvate in **memoria centrale**.
- Disco -> RAM -> Cache CPU -> Registri.

### Considerazioni - Sicurezza

**Sicurezza**
Nel contesto dei file system indica mantenere il sistema in uno stato consistente.
- Controllo di consistenza del file system, journaling.

**Controllo di Consistenza**
In Linux viene svolto da `fsck.ext4.`
- **Automatico**: Al boot. Svolto col file system posto in sola lettura. Controlla l'integrità dei blocchi e la consistenza dei metadati.
- **Manualmente**: Da terminale. Svolto col file system smontato.

**Fsck Operazioni**
- **Controllo**: Superblocco, struttura delle directory, conteggio degli hard links, blocchi con checksum corrotto.
- **Riparazione**: Metadati, struttura delle directory, marcatura di blocchi corrotti come inusabili.

**Limiti del Controllo**
- Non tutti i blocchi sono riparabili. 
- Richiede molto tempo.

**Transazione**
L'insieme delle operazioni di scrittura come metadati e modifica del contenuto di un file.
- Modifica di timestamps/contenuto, scrittura di blocchi.
- Una transazione deve eseguire del tutto oppure niente.

**Journal**
File speciale gestito come buffer circolare. Sorta di diario per il file system, annota:
- Ogni inizio di transizione, ogni operazione della transazione.
- Una volta annotati l'SO considera la scrittura del journal effettuata. Non viene subito eseguita la scrittura sul disco.

**Sincronizzazione del Journal**


**Recupero Crash**
1) In caso di crash il journal contiene informazioni su alcune transazioni.
2) Le operazioni non marcate come complete non vengono scritte sul disco.
3) L'SO se ne accorge. Esegue tutte le operazioni non marcate come complete, nell'ordine in cui sono state scritte nel log.

**Vantaggi**
- Aumenta notevolmente la robustezza del file system. L'unico punto debole è un crash del journal stesso.
- I metadati vengono scritti in modo asincrono sul journal.
