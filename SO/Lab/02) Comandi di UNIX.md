**Comandi**
I comandi in UNIX sono suddivisi in **interni** ed **esterni**.
- **Shell Builtin**: Compreso nella **shell**. Quando viene eseguito la shell esegue la funzione associata al comando senza caricare altro.
- **External Command**: Compreso in un **eseguibile esterno** alla shell (**BASH**). Si trovano in una delle directory in `$PATH.` Alla sua esecuzione la shell esegue una nuova applicazione.

**Help**
`help` è un comando **builtin** che mostra documentazione sui comandi shell builtin.
- **Documentazione**: Descrizione breve/estesa, funzionalità, argomenti, valori di uscita...

**Man**
`man` è un comando **esterno** che mostra **documentazione** sui comandi esterni.
- Può prendere come parametro un numero che rappresenta il numero di sezione.
- I comandi possono avere più pagine di manuale come `printf.` Senza parametri aggiuntivi `man` scandisce le sezioni in ordine e ritorna la prima voce.

**Capitoli del Manuale**
Le pagine di `man` sono suddivise in nove sezioni, `man man` le mostra tutte.

**Man Page Builtins**
**Debian** mette a disposizione una pagina di manuale contenente tutti i comandi **builtin di BASH** chiamata `man bash-builtins.`

**Apropos**
`apropos` è un comando che cerca comandi. Cerca i nomi esatti delle voci di manuale nelle sue pagine a partire da **parole chiavi** fornite dall'utente.
- `apropos -s 1,2 list directory` ricerca tutte le pagine del manuale nelle sezioni 1 e 2 che contengono le parole list o directory.
- `apropos -r '.*create.*process':` ricerca tutte le pagine del manuale riguardante il lancio di nuove applicazioni.

**Ripetizione di Comandi**
- Per ripetere l'ultimo comando immesso si usa `!!.`
- Per ripetere l'ultimo comando con permessi di amministratore si usa `sudo !!.`
- Per ripetere l'ultimo comando che inizia per e si usa `!e.`

**Ripetizione di Argomenti**
Per ripetere l'ultimo argomento del comando precedente si usa `Alt-.`
- Se si ripete più volte si pescano gli argomenti precedenti.

**History dei Comandi**
Possiamo ripescare un comando utilizzando la history dei comandi tramite `CTRL+R.`
- Supporta la ricerca, se usiamo `CTRL+R ls` vediamo tutti i comandi che contengono `"ls".`

**Sostituire Comando Precedente**
Si può effettuare una sostituzione nel comando precedente.
- `^stringa_da_rimuovere^stringa_da_sostituire`
- `ls testo.txt` seguito da `^ls^cat` esegue `cat testo.txt.`