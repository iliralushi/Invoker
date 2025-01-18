**SO Moderno**
- **Multiprogrammato**: Più applicazioni in esecuzione contemporaneamente.
- **Multiutente**: Più utenti che possono usare il dispositivo contemporaneamente.
- **Time Sharing**: I programmi vengono eseguiti **sequenzialmente** in piccole parti invece che tutti contemporaneamente.

**Eseguibile vs Processo**
- **Programma Eseguibile**: Programma salvato su **storage device**. Contiene il **codice macchina**, alcune aree di memoria e delle funzione utili per il **debugging**. Non può andare in **esecuzione da solo**.
- **Processo**: Rappresentazione del kernel in termini di **strutture dati** e **funzioni di gestione** di un **programma eseguibile in esecuzione**.

**Processo**
- **Strutture Dati**: Rappresentazione di uno **stato interno** e puntatori a **risorse in uso** come aree di memoria o altri programmi.
- **Funzioni di Gestione**: Creazione, terminazione, comunicazione e sincronizzazione dei processi. Esecuzione di un programma eseguibile.

**PID**
Il PID è un **numero univoco** assegnato dal kernel per ogni singolo processo. È rappresentato dal tipo di dato `pid_t` che è un **intero non negativo**.
- Viene assegnato seguendo la formula `next_pid = (prev_pid + 1) % max_pid.`

**Automa a Stati - Stato Interno**
- **In Esecuzione**: La CPU sta eseguendo codice.
- **Bloccato**: Attesa di un evento come il ricevimento di dati da una periferica.
- **Pronto**: Si è verificato un evento. Il processo è pronto a ri/eseguire la sua esecuzione.
- **Terminato**: Non c'è più codice da eseguire. Vengono liberate le risorse occupate e viene distrutto il processo

![](Automa-Stati.png)

**Linux: Esecuzione**
Nei sistemi GNU/Linux gli stati **pronto** e **in esecuzione** vengono accorpati in `TASK_RUNNING.`

**Linux: Comunicazione**
Un processo può comunicare con altri tramite la ricezione e l'invio di **segnali**. Se il processo sta ricevendo dati da **una periferica** e viene chiuso (`segnale CTRL+C`) il kernel può rimanere in una **condizione instabile**.
- L'attesa è gestita da due stati. `TASK_INTERRUPTIBLE` e `TASK_UNINTERRUPTIBLE`.

**Linux: Sincronizzazione**
Un processo può crearne un altro ed aspettare che finisca. Una volta terminato, esso salva **il suo stato di uscita** in una struttura dati, in modo che il processo creante lo possa leggere. Gli stati di uscita in Linux sono due:
- `EXIT_ZOMBIE:` Il processo creato **è morto**, le sue risorse **non sono state deallocate**. Il processo creante è pronto a leggere lo stato di uscita.
- `EXIT_DEAD:` Lo stato di uscita è stato letto dal processo creante, vengono deallocate le risorse utilizzate.

**Linux: Debugging**
Il debugging di un processo in Linux è gestito in due stati:
- `TASK_STOPPED:` Il processo è stato stoppato (`segnale CTRL+Z`).
- `TASK_TRACED:` Il processo sta venendo tracciato da un debugger.

![](Automa-Stati-Linux.png)

**PCB**
Quando viene creato un processo il kernel del SO crea una **struttura dati** detta **Process Control Block (PCB)** che contiene:
- Puntatori alle risorse prenotate.
- Informazioni di stato del processo.

![](PCB.png)

**Linux: PCB**
Il PCB in Linux è definito nella struttura dati `struct task_struct.`

**Creazione di Processi**
Un processo in esecuzione può creare altri processi. Questo meccanismo è una **clonazione** di un processo, detta **forking**. Al termine del forking il processo **figlio** è identico al processo **padre**. Entrambi i processi riprendono **dall'istruzione successiva alla clonazione**.
- **Processo Clonante**: Padre.
- **Processo Clonato**: Figlio.

**Chiamata di Sistema - fork()**
In UNIX per clonare un processo si usa la chiamata di sistema `fork().` In Linux è definita dalla funzione di servizio `do_fork()` definita nel file `$LINUX/kernel/fork.c`
- `do_fork()` identifica le risorse da clonare, la clonazione avviene tramite la funzione `copy_process().`

![](Fork.png)

**Dopo la Clonazione**
Al termine del forking entrambi i processi riprendono l'esecuzione dall'ultima istruzione. Riusciamo a distinguere padre dal figlio grazie al valore di ritorno di `fork().`
- -1 con **errore**, 0 se invocata su **processo figlio**, PID del figlio se invocata sul **padre**.

**Organizzazione Processi UNIX**
SO UNIX hanno l'organizzazione dei processi **ad albero**.
1) Un processo iniziale `init` viene creato manualmente dal kernel. Ha sempre 1 come PID.
2) `init` fa partire servizi del kernel tramite `fork()` ed `exec().` Uno dei servizi è `getty` che è il **gestore del login su console.**
3) `getty` una volta loggati fa partire la shell `bash` che esegue `ls.`

![](Albero-Processi.png)

**Esecuzione Programmi**
`fork()` si occupa solo di clonare processi. I programmi vengono caricati tramite la funzione `execve().`

**Come Usare Forking**
Solitamente si usa per creare comandi **composti in pipeline**.
- `find . -name \*.c | wc -l`

![](Pipeline-Comando.png)

**Fork Bomb - Bad Forking**
Una **fork bomb** è un processo che riproduce processi figli **ricorsivamente** all'infinito. È un processo **speciale** e viene usato per i **Denial of Service (DoS)**.
- Ogni processo creato **consuma risorse** e rallenta la macchina. Il kernel **impone** un tetto di processi **massimi eseguibili** in un momento e una volta raggiunto il numero non parte più nessuna applicazione.

![](Forkbomb.png)

**Terminazione Processi**
La chiamata di sistema `exit()` termina un processo.
1) Mette il processo in stato `EXIT_ZOMBIE.`
2) Attende la lettura del suo stato di uscita dal processo padre.
3) Svuota gli stream aperti e gestisce i segnali.
4) Mette il processo in stato `EXIT_DEAD.`
5) Libera le risorse prenotate.
6) Elimina il PCB.
7) Schedula l'esecuzione di un altro processo.

**Processi Orfani e Child Reaping**
Se un processo genitore **termina** i suoi figli diventano **orfani**.
- **Reparenting**: Il processo orfano diventa **figlio del processo** `init.`

**Process Group**
Ogni comando usato nella shell crea un **gruppo di processi** identificato dal **PGID** che è un **identificatore di gruppo**. È pari al PID del **primo** processo nella pipeline. In questo modo è possibile inviare segnali a **tutti** i processi del gruppo (**process group leader**).
- `Rappresentazione per ls -al | grep .bash | less -Mr.`

![](Organizzazione-Gruppi.png)

**Session**
Una **sessione** è un **insieme** di gruppi di processi che condividono lo stesso terminale che può essere **terminale di login (tty)** o **pseudo-terminale**. Una sessione è individuata da ID sessione detto **SID**. L'ID di sessione è il PID del processo, **session leader**, il processo che crea la sessione tramite la chiamata di sistema `setsid().`

**Gruppo in Foreground e Background**
Ad ogni istante, **solo un gruppo di processi** può leggere dal terminale. Prendono il nome di **foreground process group**. Gli altri gruppi di processi non hanno questa proprietà, hanno il nome di **background process group**.
- **Background**: `ls -lR / > out.txt 2>err.txt &`
- **Foreground**: `ls -lR / > out.txt 2>err.txt`

``` BASH
echo $$
	400 # PID di BASH

# Pipeline eseguita in background. Abbiamo due processi legati in pipe. Il
# comando non può leggere input da terminale.

find / 2> /dev/null | wc -l & 
	[1] 659 # PID dell'ULTIMO processo. Il process group leader è PID - 1.

# Pipeline eseguita in foreground. Abbiamo due proecssi legati in pipe. Il
# comando può leggere input da terminale.
sort | uniq -c
```

![](PID-PGID-SID.png)