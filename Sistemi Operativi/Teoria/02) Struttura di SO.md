**Stratificazione SO**
Un SO è progettato **per strati** e contengono le funzionalità dell'SO. L'insieme degli strati forma uno **stack software**. Lo strato viene implementato basandosi sulla funzionalità degli **strati precedenti**. 

![](Stack-Software.png)

**Contenuto Strato**
Uno strato contiene **l'implementazione** delle **funzioni e strutture dati** usate per fornire le funzionalità necessarie allo strato superiore. L'implementazione può essere:
- **Hardware**: Uso di chip preinstallati ROM/EEPROM.
- **Software**: Uso di librerie statiche/dinamiche.

**Generico Strato Funzionale**
1) Le funzionalità dello strato `m` fanno uso delle funzionalità dello strato `m-1.` Ogni strato ha una parte **pubblica** ed una **privata**. Qui lo strato `m+1` può sfruttare la funzione `fmj.`
2) Lo strato `m` definisce funzioni **private di supporto** che possono sfruttare le funzioni pubbliche dello strato `m-1` ma non sono visibili allo strato superiore.
3) I primi due step vengono riapplicati fino all'ultimo strato: le funzioni dello strato `m+1` vengono implementate usando le funzionalità **pubbliche** dello strato `m.` 


![](Strato-Funzionale.png)
![](Strato-Funzionale2.png)

**Vantaggi**
- **Sviluppo**: Il primo strato è **implementato a se**, considera solo l'hardware. Gli altri strati vengono creati **sulla base dello strato precedente**.
- **Debugging**: Abbiamo una **stack di strati**; è molto più facile trovare un problema. Se fino allo strato `m-1` funziona tutto, il problema risiederà nello strato `m.`

**Svantaggi**
Definire uno strato è complicato, difficile determinare **l'inizio** e **la fine** di esso.
- **ES**: Il sistema di gestione della memoria ha bisogno del driver del disco per implementare il meccanismo di **swapping**. Lo strato dei driver del disco **va collocato al di sotto** rispetto allo strato della memoria. In un SO moderno si hanno **moltissimi vincoli** di questo genere.

**Macro Kernel SO**
Tutti i servizi vengono eseguiti in **kernel mode**, le funzionalità essenziali del kernel vengono contenute in un immagine **eseguita al boot**. Le applicazioni eseguono in **user mode**.

![](Macro-Kernel.png)

**Vantaggi e Svantaggi**
- **Vantaggi**: L'esecuzione **dei servizi** è la più veloce possibile.
- **Svantaggi**: Kernel **fragile**, un crash blocca la CPU. Kernel di **dimensioni enormi**; le istruzioni potrebbero non entrare in **CPU cache**, riducendo di molto le prestazioni.

**Dimensione Codebase UNIX**
Con l'aumentare dei servizi è aumentata anche la **dimensione del kernel**.
- Per allegerirne il peso si possono usare **moduli caricabili** o SO basati su **micro kernel**.

**Moduli Caricabili**
Sono **file oggetto** contenenti funzionalità del kernel, (dis)attivabili a tempo di esecuzione.
- **Vantaggi**: Dimensione del kernel **diminuita**. Un crash non blocca il sistema.
- **Svantaggi**: Lieve perdita di **prestazioni** dovuta alla (dis)attivazione.

**Micro Kernel SO**
Il kernel esegue solo i **servizi essenziali**, il resto è eseguito tramite **server applicativi**. I server comunicano tramite un **sistema a messaggi**.
- **Kernel**: Mediatore per server. Gestisce lo **scheduler CPU**, l'allocatore di memoria...
- **Applicazioni**: Si occupano del **file system** e dei vari driver dei dispositivi.

**Vantaggi e Svantaggi**
- **Vantaggi**: Estendibile facilmente, vengono aggiunti server **senza toccare il kernel**. Il kernel è **ridotto notevolmente**. L'SO è **robusto rispetto ai crash**, se muore un server se ne crea una nuova istanza.
- **Svantaggi**: Meno performante di un **Macro Kernel SO**, necessari scambi tra kernel ed user e viceversa ad ogni **chiamata di sistema** e messaggio scambiato.

![](Micro-Kernel.png)

**Hybrid Kernel SO**
Variante del **Micro Kernel** con una differenza; i driver dei dispositivi vengono eseguiti in **kernel mode**. 

![](Hybrid-Kernel.png)
![](MVMVH.png)