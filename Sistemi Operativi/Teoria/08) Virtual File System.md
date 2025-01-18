### Virtual File System

**Virtual File System**
È una componente del kernel. In Linux è implementato dalla directory `$LINUX/fs.`

**Utilità - VFS**
Fornisce una visione omogenea ed indipendente dai dispositivi del contenuto dei files e delle directory. È ottenuto da periferiche locali, periferiche di rete e kernel.

**Utilità - Visione Omogenea**
- **Esempio**: Si hanno tre dischi `A, B, C` e avviene la scrittura in un applicazione a cui sono collegati dei file in `B.` Se si cambiano di posto `B ed A` l'applicazione non ha più accesso ai file, quindi la scrittura non può avvenire.

**Schema**

![[VFS.png]]

**Compiti VFS**
Decompone il percorso di un file con lo scopo di trovare i dispositivi in cui si trova. Gestisce descrittori di file che puntano alle possibili operazioni nel file system.

**Struct Inode**
Nei sistemi UNIX il file control block è rappresentato da una struttura dati inserita su un blocco chiamata inode. In EXT4 è definito dalla struttura dati struct `ext4_inode` nel file
- È definito in `$LINUX/fs/ext4/ext4.h.`

**Inode VFS**
L'inode può variare negli altri file system. Il VFS definisce un inode comune a tutti nella struttura `struct inode.`
- Viene creata al primo uso di un file, il contenuto viene messo su disco e l'inode viene mantenuto in memoria centrale.

**Struct File**
- Rappresenta un file aperto. È definito in `$LINUX/include/fs.h.`
- Contiene informazioni sul puntatore dell'inode contenuto nell'VFS, posizione, modalità di apertura, percorso e puntatori alle operazioni possibili sul file.

**Struct Dentry**
- Rappresenta un elemento del percorso del file. È definito in `$LINUX/include/dcache.h.`
- **Esempio**: Il file `/bin/vi` è composto da una directory ed un file. La prima volta che il kernel accede al percorso, costruisce le due dentry e, vengono inserite in un albero in memoria centrale (dentry cache).

**Dentry Cache**
- **Path Name Lookup**: L'operazione di analisi del percorso di un file. Solitamente è una operazione costosa, con la dentry cache si riduce alla visita di un albero binario in memoria centrale.
- Implementato in `$LINUX/fs/namei.c.` La funzione è `vfs_path_lookup().`

**Struct File_Operations**
È un array di puntatori a funzione che rappresenta le possibili operazioni su un file.
- **Operazioni**: Lettura, scrittura, apertura, chiusura, flush...
- È definito in `$LINUX/include/fs.h.`

**Struct Dentry_Operations**
È un array di puntatori a funzione che rappresenta le possibili operazioni su un elemento del percorso. È definito in `$LINUX/include/dcache.h.`
- **Operazioni**: Calcolo hash sul nome, confronto tra stringhe, eliminazione della cache...
- Impacchettata a 64 byte in modo da occupare una linea di cache. Accesso istantaneo.

### Albero Directory

**Albero Directory**

![[Albero-VFS.png]]

**Descrizione Directory**
- **/bin**: Comandi necessari per il sistema di base testuale.
- **/sbin**: Comandi di amministrazione per utenti, periferiche e l'eseguibile init.
- **/boot**: 
### Virtual File System - Usi