**Comportamento Processo**
Un processo in esecuzione si alterna a due fasi:
1) **CPU Burst**: Elaborazione del kernel o dell'user.
2) **Wait**: Attesa di I/O o evento.

![[Comportamento-Processo.png]]

**Vincolazione**
- **Resource Bound**: Un processo è resource bounded quando la sua performance è correlata alla **quantità di risorse disponibili**.
- **Event Bound**: Un processo è event bounded quando la sua performance è correlata dalla **frequenza di eventi**.

**Esempi Programmi**
- **CPU**: `factor` fattorizza un numero intero. È vincolato **dalla CPU**.
- **Disco**: `dd` trasferisce dati a blocchi e periferiche. È vincolato **dal disco**.
- **Rete**: `wget` scarica documenti dal web. È vincolato **dalla velocità di rete**.

**CPU + I/O Bound**
- **Processo CPU-Bound**: Produce **poche** sequenze di CPU Burst lunghe e fa **poche** richieste di I/O.
- **Processo I/O-Bound**: Processo che produce **molte** sequenze di CPU Burst corte e fa **molte** richieste di I/O.

![[CPU-IO-Bound.png]]

**Processi Importanti**
1) **Sistema Desktop**: Interattività con l'utente **elevata**. Abbiamo molte richieste di I/O e dispositivi come disco e rete. Si prediligono i **processi I/O-bound**.
2) **Sistema Batch**: Interattività con l'utente **molto limitata**. Abbiamo la necessità di svolgere quante più operazioni di calcolo possibili. Si prediligono i **processi CPU-bound**.
   
**Grado di Multiprogrammazione**
È composto dalla **somma** di due fattori:
1) Numero di processi in esecuzione. Dipende dalla frequenza con la quale **l'utente fa partire processi**.
2) Numero di processi **pronti** per l'esecuzione. Esso dipende dalla frequenza con la quale i processi **escono**.

**Gestore dei Processi**
Il suo obiettivo è quello di gestire il **grado di multiprogrammazione** senza degradare le prestazioni **della macchina** e **dei processi**. Inoltre, deve dare la precedenza ai processi adatti per il tipo di SO considerato:
- Desktop -> Processi **I/O-Bound**.
- Server    -> Processi **CPU-Bound**.

**Scheduler e Dispatcher**
- **Scheduler**: Sceglie il prossimo processo da eseguire, facendo attenzione a scegliere un processo **pertinente al tipo di SO utilizzato** al momento.
- **Dispatcher**: Sostituisce il **PCB del processo attuale** col **PCB del processo schedulato** in maniera efficiente.

**Scenario**
1) Un processo in kernel mode richiede un'operazione di lettura (I/O) causando **un'attesa**. La lettura è bloccante e finchè il dispositivo non risponde **il processo è fermo**.
2) Per non far rimanere la CPU ferma la funzione di sistema **invoca lo scheduler**. Quindi il processo viene sostituito con uno nuovo.
3) Lo scheduler sceglie il processo secondo **vari criteri**: ha priorità, viene scelto a caso o circolarmente, non esegue da tanto tempo...
4) Lo scheduler invoca **il dispatcher** che si occupa di sostituire il PCB del processo attuale con quello del processo schedulato. Il nuovo processo può riassumere l'esecuzione.

![[Scheduler.png]]

**Quando Schedulare**
1) Il processo è bloccato e attende la fine di un evento.
2) Il processo termina l'esecuzione.

**Prelazione**
Interruzione di un processo che **occupa troppe risorse** in favore ad un processo che sta aspettando l'esecuzione. Un processo CPU-bound può causare **starvation** di risorse.

**Scheduling con Prelazione**
1) Il processo è bloccato ed attende la fine di un evento.
2) Il processo termina l'esecuzione.
3) È stato creato un nuovo processo.
4) Un processo viene interrotto da un'interruzione.
5) Un processo cambia lo stato bloccato a pronto.

**Scheduling in Linux**
Usa la prelazione. È implementato nella funzione `schedule().`
- Viene definita nel file `$LINUX/kernel/sched/core.c.`

1) Sceglie la `task_struct` di un nuovo processo.
2) Salva lo stato del processo attuale nella sua `task_struct.`
3) Ripristina lo stato contenuto nel PCB del processo schedulato nei registri.
4) Salta alla prossima istruzione da eseguire.

**schedule() Invocazioni**
- Blocco dovuto ad I/O.
- Blocco dovuto a sincronizzazione tra processi.
- Al termine di una chiamata di sistema.
- Al termine di un **gestore delle interruzioni**.

**Flag di Rischedulazione**
Lo scheduler rischedula un processo solo se forzato da **eventi** come una **priorità maggiore** del processo schedulato. In caso contrario viene favorito il processo attuale.
- Il kernel usa il flag `TIF_NEED_RESCHED` della `task_struct` del processo attuale per capire se c'è necessità di rischedulare.

**Schedulazione Condizionata**
Il flag viene controllato tramite la funzione del kernel `need_resched()` definita nel file `$LINUX/include/linux/sched.h.`
- Se la funzione ritorna 1 (**TRUE**) allora il kernel invoca `schedule().`

**Funzione Rientrante**
Una funzione del kernel può **essere rinvocata** senza che l'invocazione precedente abbia finito. È una **conseguenza inevitabile** (multiprocessori).

**Rientranza**

![[Rientranza.png]]

1) Seppur `sys_read()` abbia invocato `schedule()` e si salta nel processo B, la `schedule()` non è ancora finita.
2) Nel momento in cui parte il **gestore delle interruzioni** verrà invocata `schedule()` mentre l'istanza prima **è ancora in corso**. Viene rischedulato il processo A.
3) Al termine della `schedule()` il controllo torna alle funzioni invocate da `sys_read().`

**Vantaggi**
Il kernel è più efficiente in architetture SMP.

**Svantaggi**
Le funzioni devono essere progettate in modo da **non aver conflitti** se vengono eseguite in contemporanea. Se non rispettano certi vincoli vengono dette **funzioni non rientranti** e non devono essere eseguite su più CPU.
- Un esempio sono le variabili globali. Se una funzione la modifica e l'altra accede nello stesso momento si ha un risultato imprevidibile. Limite alla scalabilità del kernel.

**Idle Task**
Processo speciale **eseguito dal kernel** quando non ci sono processi disponibili.
- Mette a riposo la CPU, aspetta una interruzione. `PID = 0.`

**Cambio di Contesto**
È il nome dell'operazione di scambio tra processi. In questa operazione viene salvato il contesto del processo attuale e ripristinato il contesto del processo schedulato.
- **Contesto**: Contenuto del PCB.

**Salto a Nuovo Processo**
Si sfrutta **l'Instruction Pointer**, un campo contenuto nel contesto. Se questo registro viene ripristinato per ultimo la CPU salta **direttamente** alla prossima istruzione utile della traccia.
- **Problema**: Un processo appena creato con `fork()` dev'essere inizializzato prima di eseguire. Otteniamo undefined behaviour usando l'instruction pointer senza fare niente.

**Salto a Nuova Traccia**
1) Si spinge sullo stack l'indirizzo della prossima istruzione da eseguire come `ret_from_fork()` o la prossima istruzione della traccia.
2) Si **salta** ad una funzione in cui vengono fatti gli ultimi preparativi. Non viene usata la `CALL` in modo da non toccare lo stack e non cambiare l'ordine di esecuzione.
3) Una volta terminata la funzione farà esegue una `RET.` In questo modo l'instruction pointer viene caricato con l'ultimo valore presente sullo stack.
  
**switch_to()**
È definita nel file `$LINUX/arch/x86/include/asm/switch_to.h` ed implementa il **salto alla nuova traccia**.
- La funzione inizializzatrice è la `ret_from_fork().`