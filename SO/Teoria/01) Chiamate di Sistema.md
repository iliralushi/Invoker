**SO e Programmi Applicativi**
- **Sistema Operativo**: È composto dal **kernel** che è un software ponte tra HW ed apps e dal **sistema di base** che permette all'SO di avviarsi e mostrare un'**interfaccia testuale**.
- **Programmi Applicativi**: Tutto ciò che non è kernel o sistema di base.

![](SO.png)

**Struttura Completa**
Abbiamo una struttura **a guscio**.
- Kernel -> Sistema di base -> Programmi applicativi.

![](Kernel.png)
![](Sistema-Base.png)
![](Programmi-Applicativi.png)

**Interazione Aplicazione-Kernel**
Un'applicazione richiede un **servizio** al kernel. Il kernel **risponde** opportunamente alla richiesta e la manda all'applicazione.

![](App-Kernel.png)

**User Mode e Kernel Mode**
- **User Mode**: L'app in user mode **esegue calcoli** con **privilegi ridotti**. Per esempio, non ha il permesso di alterare la memoria di altre applicazioni o del kernel. La modalità protegge da usi **maliziosi/sbadati** dei permessi kernel mode.
- **Kernel Mode**: L'app entra in kernel mode quando **richiede un servizio**. Si hanno privilegi massimi, l'applicazione esegue il frammento di codice richiesto del kernel. Una volta che il servizio è finito l'applicazione **torna in user mode**.

**User Space e Kernel Space**
- **User Space**: Indirizzi di memoria utilizzabili da un'applicazione.
- **Kernel Space**: Indirizzi di memoria utilizzabili dal kernel.

**Livelli di Privilegio**
Nelle CPU Intel sono presenti **quattro** anelli di privilegio. In GNU/Linux si usano solitamente:
- **Livello 0**: Kernel mode.
- **Livello 3**: User mode.

**Chiamata di Sistema**
Meccanismo con cui l'applicazione **richiede** un servizio al kernel.
1) Salvataggio dei registri.
2) **Passaggio** da user mode a kernel mode.
3) Esecuzione funzione di servizio.
4) Salvataggio dei **valori di ritorno** in registri temporanei.
5) **Passaggio** da kernel mode a user mode.
6) Ripristino registri ed esecuzione del programma.

**Passaggio dei Parametri**
- Una **chiamata di sistema** accetta massimo 6 parametri. Per poterne inserire di più bisogna prendere come parametro un **puntatore a struttura dati**.
- Il **passaggio dei parametri** avviene tramite registri. Il registro `eax` contiene l'ID numerico della chiamata di sistema.

**Ingresso in Kernel Mode**
- **Pre-Pentium 2**: Si usava la **trap software 128**, `int $0x80`. `int` è interruzione ed il numero `$0x80, (128 in dec)` è il numero di interruzione in Linux che invoca syscalls.
- **Post-Pentium 2**: Istruzione assembly `sysenter.`

**Invocazione Funzione di Servizio**
Una volta entrati in kernel mode viene eseguita una **funzione di servizio del kernel**, `system_call()` che ha il compito di invocare la **vera** funzione di servizio.

**Funzione di Servizio**
Le **funzioni di lavoro** hanno il prefisso `sys_`
- Se si invoca la chiamata di sistema `getpid()` allora nel kernel esisterà una funzione `sys_getpid().`

**Ritorno in User Space**
Il valore di ritorno della **funzione di lavoro** è salvato nel registro **eax**. 
- Se è stato usato `sysenter` per entrare in kernel mode, per uscire si usa `sysexit.`

**Libreria C**
Libreria **wrapper** per facilitare l'uso di funzioni di servizio del kernel, **GNU C Library**.

![](Wrapper-C.png)

**Convenzioni di Chiamata**
Convenzioni standard del C.
- I parametri vengono spinti sullo **stack in ordine inverso**, segue una jump to subroutine (salto alla funzione) tramite istruzione `call.`