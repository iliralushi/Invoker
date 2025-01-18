**Scenario**
Il kernel deve assegnare una risorsa ad `n` entità che la vogliono accedere.
- **Problema**: La risorsa è **limitata**, le entità eccedono le richieste possibili per quella risorsa.

**Ruolo Scheduler**
Lo **scheduler** gestisce l'accesso alle risorse.
1) **Accodamento**: Le richieste vengono ricevute ed inserite nella coda.
2) **Algoritmo**: Pesca una richiesta dalla coda.
3) **Dispatcher**: Assegna la richiesta alla risorsa.

![[Modello.png]]

**Obiettivi Scheduler**
1) Aumentare la produttività e l'assegnazione della risorsa.
2) Assegnare la risorsa in modo equo tra i richiedenti.
3) Ridurre il tempo di attesa per una risorsa.

**Scheduling Prelazione**
- **Con Prelazione**: Lo scheduler interrompe l'uso della risorsa per favorire un'altro utente.
- **Senza Prelazione**: La risorsa continua a fruire lo stesso utente finchè essa **non termina la funzionalità** oppure **l'utente ha bisogno di un'altra risorsa**.

**Scheduling Senza Prelazione**
- **Scheduling Cooperativo**: L'utente può gestire la risorsa e darla ad un altro utente con una operazione chiamata **yield**.
- **Non Cooperativo**: Lo scheduler gestisce le risorse in automatico.

**Modello di Riferimento I**
Questo è uno scheduler senza prelazione, i processi non effettuano richieste ad altre risorse.

![[Scheduling-CPU.png]]

**Indice di Prestazione**
È un numero che rappresenta la **performance** di un componente hardware o software. Per uno scheduler della CPU l'indice di prestazione è misurabile attraverso:
1) Tempo di attesa del processo.
2) Produttività dello scheduler.
3) Utilizzo del processore.

**Numero Processi Coda**
È il numero di processi presenti nella coda in un istante di tempo.
- Misura lo stato di congestione del processore. Più **basso** è meglio è.

**Tempo Attesa Coda**
È l'intervallo di tempo impiegato dal processo che parte dall'entrata in coda fino **alla presa in carica del processore**.
- Misura l'attesa di un processo ad iniziare **l'elaborazione**. Più **basso** è meglio è.

**Tempo di Servizio**
È il tempo che il processore impiega ad eseguire il codice del processo.
- Misura il **tempo di elaborazione** del processo. Più **basso** è meglio è.

**Tempo di Completamento**
È la somma del **tempo di attesa in coda** e del **tempo di servizio**.
- Misura il tempo complessivo impiegato dal processo. Più **basso** è meglio è.

**Latenza**
È l'intervallo di tempo impiegato dal processo che parte dall'entrata in coda fino **al calcolo del primo byte** del codice del processo.
- Misura l'attesa impiegata dal processo ad eseguire codice e la congestione del processore. Più **bassa** è meglio è per i processi **interattivi**.

**Throughput**
È il numero di processi che lo scheduler termina in un lasso di tempo.
- Misura **la produttività** dello scheduler. Più **alto** è meglio è per i processi **non interattivi**.

**Utilizzazione CPU**
È il tempo in cui il processore è occupato ed esegue operazioni.
- Misura **la produttività** del processore. Più **alto** è meglio è.

**Indici da Considerare**

![[Indice.png]]

**Modello di Riferimento II**
Questo è uno scheduler con prelazione. In aggiunta abbiamo il ritorno in coda del processo se viene interrotto. Un processo può **rientrare più volte in coda e più volte ricevere servizio**.

![[Scheduling-CPUII.png]]

**Indici Riassuntivi**
Abbiamo bisogno di un **unico valore complessivo**.
- **Generalmente**: Abbiamo **almeno** una misura per ogni processo coinvolto.
- **Scheduler con Prelazione**: Abbiamo una misura per ogni processo coinvolto ed una misura per ogni processo rientrato in coda.

**Indici Riassuntivi II**
- **Valori Min/Max**: Performance migliore/peggiore avuta dal processo.
- **Valore Medio**: La performance media di tutti i processi, oppure la performance media di un singolo processo, considerando **ogni rientro in coda di attesa**.
- **Valore Complessivo**: Misura la performance totale del processo durante la sua esecuzione.

**Throughput vs Latenza**
Sono **parametri contrastanti** tra loro.
- Uno scheduler ad alto throughput **passa ogni processo in coda**. Non fa distinzioni tra i tipi del processo, **la latenza aumenta**.
- Uno scheduler a bassa latenza tende a **scegliere i processi interattivi**. Gli altri processi rimangono fermi in coda, **il throughput diminuisce**.

**Algoritmi di Scheduling CPU**
Gli algoritmi proposti sono esclusivamente **algoritmi di scheduling dei processi**.

**First Come First Served (FCFS)**
Non considera la prelazione. Un processo continuerà ad eseguire fino ad I/O o fine.
1) Il processo viene accodato tramite logica **first in**.
2) Il processo viene assegnato al processore seguendo la logica **first out**. Si parte dal primo processo in coda, vengono presi in fila.

**Vantaggi**
- L'algoritmo è veramente semplice. Funzionamento base di una coda.
- L'algoritmo è eseguibile da processori senza clock hardware. Non implementa prelazione quindi un processo non potrà mai tornare in coda.

**Svantaggi**
- L'attesa di un processo non è la migliore. Dipende tutto dall'ordine dei processi inseriti in coda, quindi il tempo di attesa medio è quasi sempre maggiore rispetto ad altri algoritmi.
- È senza prelazione. Un processo può monopolizzare il processore e **starvare** processi importanti come **un terminale**. Pessimo nei sistemi **time sharing**.

**Shortest Job First (SJF)**
Non considera la prelazione.
1) Il processo viene accodato tramite logica **first in**
2) Il processo viene assegnato al processore a seconda del suo **CPU burst**. Si parte dal processo con CPU burst minore fino ad arrivare al processo con CPU burst maggiore. Se esistono processi con CPU burst uguale **si applica il criterio FCFS**.

**Vantaggi**
- Tra gli algoritmi senza prelazione è quello col tempo di attesa medio minore.

**Svantaggi**
- Richiede la conoscenza del CPU burst di ogni processo. Va **calcolata**, non abbiamo questo dato. Lo scheduler eseguito è un SJF-like con **stima del CPU burst**.
- Inusabile in scheduler con prelazione. I processi eseguono **poco alla volta** e il tempo stimato cambia rapidamente. L'informazione su cui SJF si affida **è sbagliata**.

**Stima CPU Burst**
La durata del prossimo CPU burst viene trovato tramite **media esponenziale** del CPU burst **passato** e **presente**. La formula è `tn+1 = a * tn + (1 - a) * told.`
- Il parametro `a ∈ [0, 1]` decide quanto **peso dare ai valori passati**.
  - `a = 0:` Consideri solo told, ovvero i tempi precedenti.
  - `a = 1:` Consideri solo l'ultimo tempo.
  - `a = 1/2:` Stesso peso per entrambi.

![[Media-Esponenziale.png]]

**Shortest Remaining Time First (SRTF)**
Considera la prelazione. Un processo può essere fermato dallo scheduler.
1) Il processo viene accodato tramite logica **first in**.
2) Il processo viene assegnato al processore a seconda del suo **CPU burst residuo**. Viene inserito un processo alla volta in coda, quando un processo con CPU burst residuo più piccolo viene accodato viene cambiato processo. **Variante di SJF time sharing**.

**Vantaggi**
- Diminuisce il tempo medio rispetto ad SJF grazie alla prelazione.

**Svantaggi**
- Il CPU burst è un parametro da calcolare, come per SJF. Lo scheduler esegue una variante SRTF-like con stima del CPU burst.

**Priority (PRIO)**
Può considerare come non considerare la prelazione.
1) Il processo viene accodato tramite logica **first in**.
2) Ad ogni processo viene assegnata una **priorità**.
3) Il processo viene assegnato al processore a seconda della sua priorità. **Minore è la priorità più è importante**. Se due processi hanno stessa priorità si usa criterio FCFS.

**Vantaggi**
- Priority da priorità ai processi **che ritiene importanti**. Fondamentale per i sistemi interattivi. Un processo **importante** è un processo che spesso interagisce con l'utente.

**Svantaggi**
- Priority è soggetto a **starvation dei processi**. Se un processo ha un tempo di attesa lungo ed usa un terminale, **viene impedita l'interazione tra l'utente e l'SO**.

**Combattere Starvation**
Per gli scheduler con priorità è essenziale. Si usa il concetto di **aging**.
- **Aging**: Aumento della priorità di altri processi man mano che il tempo di attesa sale.

**Scelta della Priorità**
Le priorità sono solitamente **numeri interi ordinati**. Possono essere definite in due modi:
- **Internamente**: Dall'SO tramite grandezze misurabili tipo memoria o CPU burst.
- **Esternamente**: Da fattori esterni all'SO, ad ogni processo si assegna una priorità.
  
**Round Robin (RR)**
Considera la prelazione.
1) Il processo viene accodato tramite logica **first in**.
2) A partire dal primo processo esso verrà eseguito **per un quanto di tempo**. Finito il tempo si passa al prossimo. È una **FIFO con prelazione**.

**Vantaggi**
L'algoritmo fornisce attese **molto basse**. L'attesa di un processo è limitata a `(n-1) * q.`
- `n` è il numero di processi, `q` il quanto di tempo.

**Svantaggi**
Il tempo di completamento di un processo CPU-bound è più lento rispetto ad FCFS.
- In quel caso il processo **viene eseguito interamente**, qui a pezzi.

**Quanti di Tempo**
- **Quanto Fisso**: Viene prestabilito e non cambia per tutta la durata. Semplice da gestire ma non è adatto per tutte le tipologie di processo.
- **Quanto Variabile**: Il kernel calcola **dinamicamente** il quanto di tempo massimo dell'esecuzione del processo a seconda del suo **tipo** e della **priorità di esecuzione**.

![[Quanto.png]]

**Impatto Quanto Completamento**
Il tempo di completamento tende a diminuire se il **quanto è più grande** dell'80% del tempo di CPU burst dei processi. Non dev'essere troppo più grande, diventerebbe **FCFS II.**

**Categorie di Processi**
- **Interattivi**: Devono andare in esecuzione il **più presto** possibile (**foreground**).
- **Non Interattivi Brevi**: Vanno in esecuzione **meno spesso** dei foreground (**background**).
- **Non Interattivi Lunghi**: Vanno in esecuzione solo quando non ci sono altri processi da eseguire (**batch**).

**Problematiche**
1) È necessario **classificare i processi** per sapere il **loro gruppo appartenente**.
2) È necessario sapere **schedulare per primi** i processi appartenenti alle **classi importanti**.

**Scheduler Multilivello**
È uno scheduler che utilizza più code. **Risolve i problemi menzionati**.
1) Ciascuna coda ha un proprio algoritmo di scheduling classico (RR, FCFS...)
2) Viene scelta la coda dalla quale prendere il processo. Per determinare ciò viene messa una **priorità** per ogni coda, quindi (**se esiste**) verrà preso dalla coda con priorità più alta.
3) Il processo viene scelto. Dopo l'esecuzione viene rimesso nella coda da cui proviene.

![[Scheduler-Multilivello.png]]

**Parametri di Progetto**
1) Numero di code.
2) Tipo di algoritmo da usare.
3) Criterio per scegliere in che coda mettere i processi.

**Esempio Reale**
Si suddivide la coda di **pronto** in cinque code diverse:
1) Processi di sistema (RR)
2) Processi interattivi (RR)
3) Processi interattivi di editing (RR)
4) Processi in background (FCFS)
5) Processi batch (FCFS)

![[Coda-Scheduler-Multilivello.png]]

**Vantaggi**
- I processi più importanti sono inseriti nelle code con più priorità. La **latenza** è molto bassa per questo tipo di processi.

**Svantaggi**
- I processi nelle code con priorità più bassa soffrono di **starvation**, non si ha garanzia sulla latenza di questi processi. Se il processo può variare da CPU-bound a I/O-bound allora lo scheduler multilivello **è inefficace e richiede aging**.

**Multilevel Feedback Queue**
È lo scheduler multilivello con retroazione. In questo caso un processo può rientrare in una coda diversa rispetto a quella assegnata originariamente.
- **Adattamento dinamico della priorità** di un processo a seconda del suo cambio di natura.
- **Aging** dei processi a bassa priorità.

**Retroazione**
La retroazione tende a raggruppare processi con CPU burst simile nella stessa coda.
- Processi con CPU burst **maggiori** si muovono alle code con priorità più bassa.
- Processi con CPU burst **minori** si muovono alle code con priorità più alta.

**Parametri di Progetto**
- Numero di code.
- Tipo di algoritmo da usare.
- Criterio per spostare un processo in una coda con priorità maggiore.
- Criterio per spostare un processo in una coda con priorità minore.
- Criterio per scegliere in che coda mettere i processi.

**Esempio Concreto**
Si usano tre code di scheduling con priorità `0, 1, 2.`
- La coda a priorità superiore ha sempre prelazione sulla coda a priorità inferiore.
- Un processo entra a priorità zero. Se il processo non finisce entro il primo quanto viene spostato nella coda inferiore. Se per **dieci** volte non finisce il processo viene spostato in una coda superiore.

![[Multilevel-Feedback-Queue.png]]

**Aspettative Scheduler**
- I processi con CPU burst brevi (I/O bound, usano terminali) vengono eseguiti molto velocemente.
- I processi con CPU burst un'po più lunghi vengono eseguiti velocemente.
- I processi batch vengono eseguiti solo se il sistema non è impegnato.
- I processi con **CPU burst variabile** si adattano alla loro coda ottimale.

da fare scheduling in sistemi smp