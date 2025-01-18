**Virtualizzazione**
Un **processo** che comporta all'usare risorse hw/sw di un sistema con lo scopo di creare un **sistema finto** su cui è possibile installare periferiche ed un SO.
- Tecnica utilizzata per provare SO senza doverli installare fisicamente.

![](Virtualizzazione.png)

**Astrazione Software**
Sono tutte le funzioni, strutture dati e meccanismi che creano un oggetto che non esiste **fisicamente**. Un'astrazione software viene creata **sfruttando le risorse hw/sw** dell'host.

**Astrazione di CPU**
La CPU finta è **mappata direttamente** quindi utilizzerà le risorse della CPU vera. In pratica quindi sfrutterà la CPU vera per un **periodo di tempo**.

**Astrazione di Memoria**
La memoria finta è **mappata direttamente** alla memoria vera. In pratica quindi sfrutterà la memoria vera per una certa **frazione di spazio**, quindi ne usa solo una parte **ben definita**.

**Astrazione di Disco**
II disco finto è **mappato indirettamente** quindi il disco vero per accederne ai suoi contenuti dovrà usare un **software mediatore**. Il disco finto viene **mappato sul file system** dell'host e viene rappresentato da un **file** salvato su disco vero (**blocco disco finto -> blocco file vero**).

**Configurazione Virtualbox**
1) Usare il bottone new per creare una nuova VM.
2) Inserire il nome della macchina, scegliere la famiglia ed il tipo dell'SO. Usare `Linux 2.6/3.x/4.x (64 bit)` per installare un SO non in elenco.
3) Inserire il quantitativo di memoria principale da usare. Bisogna stare attenti a non allocare troppa memoria al guest, le prestazioni dell'host potrebbero risentirne.
4) Creare un disco fisso virtuale.