# La Shell

Oggi approfondiamo la shell. La filosofia Linux è quella di usare tanti piccoli programmi iperspecializzati, in contrasto con la visione di Windows.

Nella shell possiamo fare una **pipe**, ovvero l'accoppiamento di stdout di un blocco a monte e lo stdin del blocco a valle (semplificazione). I blocchi della pipe si dividono mediante `|`. Vediamo un esempio:

```sh
strace -f -e trace=memory gcc -v -o fork1 fork1.c 2>&1 |
grep 'mmap2' |
cut -d',' -f2 |
tr -d ' ' |
paste -sd+ |
sed 's/+/ + /g'
```

`strace` segue tutte le chiamate di sistema effettuate da un processo. Coi flag `-f` segue anche i processi figli generati, `-e trace=memory` ci permette di tracciare solo le chiamate relative alla memoria. Quindi chiamamo il compilatore `gcc` e compiliamo un file `fork1.c` e destiniamo lo `stderr` (2) sullo `stdout` (1).

È importante perché `strace` scrive normalmente le proprie tracce su `stderr`. Senza questa redirezione, la pipe potrebbe ricevere soltanto l'output di `gcc`, mentre le righe delle chiamate di sistema resterebbero fuori dalla pipeline.

`grep` serve invece per la ricerca di una specifica stringa in una sequenza (in questo caso gli viene passata da `strace`) e nel nostro caso cerca tutte le righe che contengono `mmap2`. Se per esempio `strace` produce:

```sh
brk(NULL) = 0x56555000
mmap2(NULL, 8192, PROT_READ, MAP_PRIVATE, 3, 0) = 0xf7fc0000
munmap(0xf7fc0000, 8192) = 0
mmap2(NULL, 4096, PROT_READ | PROT_WRITE, MAP_PRIVATE, 4, 0) = 0xf7fb0000
```

Quel `grep` ci ritorna:

```sh
mmap2(NULL, 8192, PROT_READ, MAP_PRIVATE, 3, 0) = 0xf7fc0000
mmap2(NULL, 4096, PROT_READ | PROT_WRITE, MAP_PRIVATE, 4, 0) = 0xf7fb0000
```

`cut` invece ci serve come se fosse una funzione split: separa le righe usando come separatore la virgola (`-d','`) e seleziona il secondo campo `-f2`, ovvero, nell'esempio sopra, seleziona 8192 e 4096.

`tr` invece serve a modificare i caratteri di una stringa. Con la flag `-d`, la modifica effettuata è la delete: gli chiediamo di eliminare gli spazi. Questo serve perché all'inizio dei numeri, dopo la virgola, c'è uno spazio che non vogliamo.

Infine `paste` ci serve a stampare in serie (`-s`) usando come divisore il `+`(`-d+`). Quindi partendo dall'output di prima di tr simile a una cosa del genere:

```sh
8192
4096
```

otteniamo:

```sh
8192+4096
```

Infine `sed` è un editor di flussi: `sed 's/+/ + /g'` dice di cercare tutte le ricorrenze `+` nella stringa (`s/+/`), quindi sostituirla con ` + ` a tutte le ricorrenze trovate (`/ + /g`). Con questo, il risultato di prima diventa:

```sh
8192 + 4096
```

<span style="color:#e64340">N.B.</span> Durante l'esame sarà vietato *l'uso dei cicli* e l'uso dell'istruzione *awk*.

## Primi comandi

Abbiamo già visto il comando `cat`. Esistono alcuni file nei sistemi Linux che mantengono le informazioni relative allo stato corrente della macchina. Per esempio, abbiamo `/proc/meminfo`:

```sh
cat /proc/meminfo
```

Se poi volessimo solo stampare, di tutto l'output, le informazioni relative alla malloc, possiamo usare `grep`:

```sh
cat /proc/meminfo | grep 'malloc'
```

Che, per esempio, a me produce:

```sh
VmallocTotal:   34359738367 kB
VmallocUsed:       33920 kB
VmallocChunk:          0 kB
```

Per creare un file possiamo usare `touch`, se invece vogliamo verificarne la creazione possiamo usare `test` o, in caso, anche la sua versione contratta:

```sh
test -f filename.txt
[ -f filename]
```

Questo non produce alcun risultato a schermo, ma comunica il risultato col codice d'uscita. Per visualizzarlo dobbiamo fare:

```sh
echo $?
```

`echo` ci permette di stampare qualcosa sullo `stdout`, mentre `$` ci permette di riferirci a delle variabili (come le variabili d'ambiente), mentre `?` è una particolare variabile della shell che contiene l'exit code dell'ultimo processo eseguito. Nel caso di `test`, se ritorna `0`, allora il file esiste, se ritorna `1` non esiste o è una directory. Se poi vogliamo stampare tutte le **variabili d'ambiente** possiamo usare

```sh
printenv
```

Che ritorna variabili come `HOME`, ovvero il percorso di dove si trova la home, `USER`, ovvero il nome dell'utente corrente, `SHELL`, ovvero dove si trova il percorso della shell corrente e anche `PATH`, dove si trovano i percorsi dei binari dei comandi che usiamo con degli alias come `strace`. 

Possiamo anche **creare delle variabili** locali:

```sh
my_var="La mia variabile"
echo $my_var
```

Possiamo avviare anche una nuova shell, cancellando le variabili locali, facendo una forma di reinizializzazione col comando:

```sh
bash
```

Possiamo poi**modificare** anche le **impostazioni della shell** usando `set`. Se per esempio volessimo reindirizzare lo `stdout`:

```sh
set > file.txt
set >> file.txt
```

Il primo azzera il file prima di iniziare a usarlo per reindirizzare il flusso, mentre il secondo fa append.

Possiamo poi trasformare una variabile locale in globale usando `export`.   

Esiste anche usare **l'espansione multipla**, che ci permette di generare più stringhe a partire da una. Per esempio:

```sh
echo file{1,2,3}.txt
```

Stampa:

```sh
file1.txt file2.txt file3.txt
```

Ancora, abbiamo `type`, che ci dà delle specifiche sui comandi. Per esemempio:

```sh
type ls
```

Ci ritorna:

```sh
ls is aliased to `ls --color=auto'
```

Possiamo fare anche confronti con `if`, come per esempio il confronto tra stringhe:

```sh
if ["al" = "bl"]
```

Possiamo anche usare operatori logici per fare condizioni composte. E' possibile fare cose usando la logica booleana. Per esempio immaginiamo di voler verificare la presenza di un file `fish` in `/bin`:

```sh
[ -f /bin/fish ] && echo "Il file esiste" || echo "Il file non esiste"
```

Funziona perché sfruttiamo i cortocircuiti logici. La shell valuta `&&` e `||` da sinistra a destra, sfruttando il cortocircuito logico: `&&` esegue il comando successivo solo se quello precedente termina con codice `0` (vero), mentre `||` lo esegue solo se il precedente termina con codice diverso da `0` (falso). Quindi, se `[ -f /bin/fish ]` ha successo, viene eseguito il primo `echo` e, dato che questo termina con codice `0`, il secondo `echo` viene saltato; se il test fallisce, `&&` salta il primo `echo` e `||` esegue il secondo.

Infine comando importantissimo è `ps` che serve a visualizzare un'istantanea dei processi in esecuzione nel sistema. Per esempio:

```sh
ps aux
```

Mostra una lista dettagliata di *tutti* i processi in esecuzione nel sistema, inclusi quelli degli altri utenti e quelli senza un terminale associato. Si può usare il `more` nella pipe per visualizzare scorrendo.

```sh
ps aux | more
```

In generale, si può costruire una tabella con i comandi visti nella lezione (slide oltre che appunti):

| **Comando** | **Descrizione / Funzione**                                                                                     |
| ----------- | -------------------------------------------------------------------------------------------------------------- |
| `ls`        | Elenca il contenuto delle directory.<br><br><br>                                                               |
| `echo`      | Mostra una riga di testo o stampa l'output sullo schermo.<br><br><br>                                          |
| `touch`     | Crea un file vuoto o modifica il timestamp di un file esistente.<br><br><br>                                   |
| `sleep`     | Ritarda l'esecuzione per un determinato intervallo di tempo.<br><br><br>                                       |
| `rm`        | Rimuove file o directory.<br><br><br>                                                                          |
| `cat`       | Concatena file e ne stampa il contenuto sullo standard output.<br><br><br>                                     |
| `man`       | Interfaccia per consultare i manuali di riferimento dei comandi.<br><br><br>                                   |
| `test`      | Verifica i tipi di file e confronta valori/stringhe (restituisce un valore di ritorno).<br><br><br>            |
| `sort`      | Ordina le righe di un file di testo.<br><br><br>                                                               |
| `date`      | Mostra o imposta la data e l'ora di sistema.<br><br><br>                                                       |
| `less`      | Filtro per la visualizzazione di testi pagina per pagina a schermo.<br><br><br>                                |
| `which`     | Localizza il percorso di un comando eseguibile.<br><br><br>                                                    |
| `ps`        | Report dei processi in esecuzione nel sistema.<br><br><br>                                                     |
| `tail`      | Mostra la parte finale (ultime righe) di un file.<br><br><br>                                                  |
| `wc`        | Conta il numero di righe, parole e byte/caratteri in un file.<br><br><br>                                      |
| `dc`        | Calcolatrice ad alta precisione a riga di comando.<br><br><br>                                                 |
| `grep`      | Cerca e stampa le righe che corrispondono a un determinato pattern.<br><br><br>                                |
| `cut`       | Rimuove o estrae sezioni/campi da ogni riga di un file.<br><br><br>                                            |
| `du`        | Stima e stampa l'occupazione di spazio su disco di file o directory.<br><br><br>                               |
| `cd`        | Cambia la directory di lavoro corrente.<br><br><br>                                                            |
| `cp`        | Copia file e directory.<br><br><br>                                                                            |
| `mv`        | Sposta o rinomina file e directory.<br><br><br>                                                                |
| `mkdir`     | Crea nuove directory.<br><br><br>                                                                              |
| `rmdir`     | Rimuove directory vuote.<br><br><br>                                                                           |
| `df`        | Mostra lo spazio su disco disponibile e utilizzato nei file system.<br><br><br>                                |
| `pwd`       | Stampa il percorso della directory di lavoro corrente.<br><br><br>                                             |
| `printenv`  | Visualizza le variabili d'ambiente globali.<br><br><br>                                                        |
| `set`       | Visualizza tutte le variabili della sessione (incluse le locali) o imposta le opzioni della shell.<br><br><br> |
| `export`    | Esporta una variabile locale rendendola variabile d'ambiente globale.<br><br><br>                              |
| `unset`     | Rimuove/elimina una variabile di ambiente o di shell.<br><br><br>                                              |
| `expr`      | Calcola ed esegue espressioni aritmetiche o di confronto.<br><br><br>                                          |
| `let`       | Valuta espressioni aritmetiche all'interno della shell.<br><br><br>                                            |
| `su`        | Cambia l'identità dell'utente corrente assumendo i privilegi di un altro utente (es. root).<br><br><br>        |
| `sudo`      | Esegue un comando con i privilegi da superutente (root).<br><br><br>                                           |
| `xargs`     | Costruisce ed esegue comandi prendendo gli argomenti dallo standard input.<br><br><br>                         |
| `sed`       | Editor di flusso per filtrare e trasformare il testo (es. sostituzioni).<br><br><br>                           |
| `head`      | Mostra le prime righe di un file o dell'output.<br><br><br>                                                    |
| `bash`      | Avvia una nuova sessione dell'interprete di comandi Bash / subshell.<br><br><br>                               |
| `exit`      | Chiude la sessione corrente della shell o termina uno script con uno specifico stato.                          |

## Esercizi

1. Devi ottenere il numero di righe dell'elenco dei file presenti in `/usr/bin` il cui nome contiene la stringa `bash`.

2. Scrivi un comando che mostri i primi 5 nomi di file di `/usr/bin` in ordine alfabetico inverso.

3. Usando una pipe, scrivi un comando che:

   1. Elenchi i processi in esecuzione con `ps`.

   2. Selezioni quelli la cui riga contiene `bash`.

   3. Estragga soltanto la prima colonna dell'output filtrato.

   4. Conti quante righe rimangono.

4. Devi scrivere un'unica riga di comandi Bash che esegua questa procedura:

   1. Crea una directory chiamata `/analisi` solo se non esiste già.

   2. Crea al suo interno tre file vuoti: `log1.txt`, `log2.txt` e `log3.txt`.

   3. Scrive nei file, rispettivamente, queste righe:

      - `log1.txt`: `ERROR`, `INFO`, `WARNING`

      - `log2.txt`: `INFO`, `INFO`, `ERROR`

      - `log3.txt`: `WARNING`, `WARNING`, `INFO`



## Soluzioni

1.

```sh
ls /usr/bin | grep "bash" | wc -l
```



2.

```sh
ls /usr/bin | sort -dr | head -n 5
```



3.

```sh
ps aux | grep "bash" | tr -s ' ' | cut -d' ' -f1 | wc -l
```



4.

```sh
[ ! -d /analisi ] && mkdir /analisi;
    touch /analisi/log{1,2,3}.txt; 
    echo "ERROR, INFO, WARNING" > /analisi/log1.txt; 
    echo "INFO, INFO, ERROR" > /analisi/log2.txt; 
    echo "WARNING, WARNING, INFO" > /analisi/log3.txt
```

Per questo esercizio si deve o essere `root`, o creare una cartella altrove .