# La Shell - parte II

Di solito, non si scrive tutta la pipe, ma si fa blocco per blocco e si vede l'output per poi metterli tutti insieme. Inoltre noi non lavoreremo mai come root, quindi uno specifico blocco potrebbe generare errori legati ai permessi o altri tipi di errori.

Nelle pipe bisogna quindi decidere come gestire gli errori. Per verificarli, possiamo fare redirezione dello `stderr`.

Una delle istruzioni che ci tornano molto utili è la `find` con formato:

```sh
find percorso predicato
```

Il percorso ci permette di specificare il percorso dove trovare i file, mentre il predicato ci dice cosa cercare.

Possiamo poi fare anche un'elaborazione senza pipe, ma non protegge dall'esplosione della shell:

```sh
comando $(find percorso predicato)
```

`$(...)` permette di aprire una **subshell**. Possiamo poi fare in modo che la fine sostituisca la newline con NUL (`\0`) con `-print0`:

```sh
find -print0
```

Questo può servire quando si fanno le pipe con comandi che hanno bisogno del separatore NUL per riconoscere diverse stringhe e il newline li mettono tutti insieme.

Possiamo poi specificare anche il tipo di file che vogiamo trovare con `-type`. Se vogiamo solo i file puri e non anche le directory possiamo fare:

```sh
find -type file -print0
```

Immaginiamo ora di voler trovare il file più grande:

```sh
find -type file -print0 | xargs -0 du | sort -n | tail -n 1
```

`-0` ci dice che vogliamo come separatore il terminatore di stringhe e `du` ci dà la dimensione in byte. Quindi `sort` ci permette di ordinare numerico. Poi prendiamo l'ultimo elemento.

Poi immaginiamo di voler mostrare la dimensione in KB:

```sh
sudo find -print0 | xargs -0 du | sort -n | tail -n 1 | 
cut -f2 | xargs -I{} du -sh {}
```

`cut` ci permette di prendere il secondo campo, ovvero il file. `-I` dice sostanzialmente una cosa: nel mio comando userò `{}` come **segnaposto** per indicare dove mettere l'input. Se per esempio l'output di `cut` (input di `xargs`)  è `"./docker ubuntu"`, `-I{}` dice che d'ora in poi, quando vedo `{}` mi riferisco all'input `"./docker ubuntu"`. In questo modo l'input viene aggiunto in coda, cosa che non è strettamente necessario visto che è quello che già fa `xargs`. Può essere utile però quando l'ordine è importante come quando si usa `cp`.

Alternativamente si può usare anche:

```sh
find /home -user user -exec du -k {} \;
```

Che è il vecchio modo di farlo. `-exec` era l'opzione di `find` per permette di dare un altro comando. `-user` permette di specificare di quale utente bisogna trovare i file. Questo mostra tutti i file con le dimensioni. `\;` termina la definizione del comando da eseguire per `-exec` e ci permette di eseguire lo stesso comando per ogni risultato di `find`.

Altro comando utilissimo, che però sarà vietato all'esame, è **`awk`** che serve per leggere testo strutturato, selezionare campi, filtrare righe ed eseguire calcoli sui dati. È uno strumento potentissimo e complesso, oltre ad essere sostanzialmente un **linguaggio di programmazione**.

```sh
find /home -user user -exec du -k {} \; | 
awk '{s=s+1} END {print "Total used: ", s}'
```

`awk` qui ci permette di fare la somma mentre si acquisiscono tutti i dati. Una volta finito `END` ce lo segnala e si stampa la somma. Ogni tanto possiamo anche incorrere in errori e possiamo reindirizzare a `/dev/null`:

```sh
comando 2> /dev/null
```

Esercizio: trovare il numero totale di tutti i file nella directory corrente. Soluzione:

```sh
ls -RAl | grep -E "^d|-" | wc -l
```

Questo ci permette di listare tutte le directory ricorsivamente, mostrando *quasi* tutto (`-A` ci evita infatti che venga mostrata per ogni directory il `.` della directory stessa e `..` della directory sovrastante) e in formato list. Poi facciamo la `grep` abilitando le regex con `-E`. Quello che diciamo con quell'espressione è che deve iniziare o con la lettera d o col - (o è un file o una directory). Si potrebbe estendere anche agli altri tipi di file (c, b, ecc.). A dire che deve cominciare con una o (|) con l'altra c'è `^`, che in gergo viene chiamata **ancora**.

C'è poi il comando `tar`, che ci permette di lavorare con gli archivi compressi.
