# Soluzioni agli esercizi

Di seguito si possono trovare le soluzioni a tutti gli esercizi proposti. Si ribadisce che non sono le uniche soluzioni possibili, ma semplicemente quelle che ha trovato il sottoscritto.

## 1. Trovare i tre file più pesanti

```
find -type f -print0 2>/dev/null | xargs -0 du | sort -n | tail -n 3
```

**Spiegazione**

La pipeline individua i file presenti nella directory corrente e nelle sue sottodirectory, ne calcola la dimensione e mostra le tre righe con i valori numerici più alti.

- `find -type f -print0`: cerca tutti gli elementi di tipo file (`-type f`) e ne stampa i percorsi separandoli con il carattere NUL (`-print0`). Questo permette di gestire correttamente anche i nomi contenenti spazi o caratteri speciali.

- `2>/dev/null`: reindirizza lo standard error (`stderr`) verso `/dev/null`, ignorando eventuali messaggi di errore prodotti da `find`, per esempio in caso di permessi insufficienti.

- `xargs -0 du`: riceve i percorsi separati da NUL (`-0`) e li passa al comando `du`, che calcola lo spazio occupato da ciascun file.

- `sort -n`: ordina le righe in base al valore numerico iniziale, cioè la dimensione riportata da `du`, in ordine crescente.

- `tail -n 3`: restituisce le ultime tre righe dell'output ordinato, corrispondenti ai tre valori numerici maggiori.

**Concetto chiave:** `-print0` e `xargs -0` consentono di passare i nomi dei file senza confondere gli spazi contenuti nei percorsi con i separatori tra argomenti.

## 2. Ottenere gli utenti distinti associati ai processi

```
ps aux | tail -n +2 | cut -d' ' -f1 | sort -u
```

**Spiegazione**

La pipeline parte dall'elenco dei processi attivi, estrae il primo campo delle righe e rimuove i duplicati.

- `ps aux`: visualizza l'elenco dei processi attivi, includendo quelli appartenenti a tutti gli utenti. La prima colonna contiene l'utente proprietario del processo.

- `tail -n +2`: stampa l'output a partire dalla seconda riga, eliminando così l'intestazione introdotta da `ps aux`.

- `cut -d' ' -f1`: divide ogni riga utilizzando lo spazio come delimitatore (`-d' '`) ed estrae il primo campo (`-f1`).

- `sort -u`: ordina i risultati e rimuove le righe duplicate (`-u`, *unique*), lasciando una sola occorrenza per ciascun valore.

**Concetto chiave:** la combinazione di `sort -u` permette di ottenere valori distinti senza dover implementare esplicitamente un controllo dei duplicati.

## 3. Contare le righe contenenti `bash` nei file nascosti

```
find -type f -name ".*" -print0 2>/dev/null | xargs -0 cat | grep "bash" | wc -l
```

**Spiegazione**

La pipeline cerca i file il cui nome inizia con un punto, ne legge il contenuto e conta le righe che contengono la stringa `bash`.

- `find -type f -name ".*" -print0`: individua i file regolari (`-type f`) i cui nomi corrispondono al pattern `.*`, ovvero quelli che iniziano con un punto. I percorsi vengono separati dal carattere NUL.

- `2>/dev/null`: nasconde eventuali messaggi di errore prodotti dalla ricerca.

- `xargs -0 cat`: passa i percorsi individuati a `cat`, che legge e concatena il contenuto dei file sullo standard output.

- `grep "bash"`: seleziona le righe che contengono la stringa `bash`.

- `wc -l`: conta le righe ricevute in ingresso e restituisce il numero totale.

**Concetto chiave:** `grep` filtra le righe in base a una condizione, mentre `wc -l` conta le righe che superano il filtro. Il risultato rappresenta quindi il numero di righe corrispondenti, non il numero totale di occorrenze della parola `bash`.

## 4. Verificare l'esistenza di una directory

```
[ -d /tmp ] && echo "Esiste" || echo "Errore"
```

**Spiegazione**

La soluzione utilizza una condizione della shell e gli operatori logici `&&` e `||` per stampare un messaggio in base all'esito del controllo.

- `[ -d /tmp ]`: verifica se `/tmp` esiste ed è una directory. L'opzione `-d` restituisce un esito positivo se la condizione è soddisfatta.

- `&& echo "Esiste"`: esegue `echo "Esiste"` soltanto se il controllo precedente ha avuto successo.

- `|| echo "Error` seguito dalla riga `e"`: esegue il secondo comando se il controllo precedente è fallito. La stringa tra virgolette comprende anche il ritorno a capo presente nel codice, perciò il messaggio viene stampato su due righe.

**Concetto chiave:** `&&` esegue il comando successivo in caso di successo, mentre `||` lo esegue in caso di fallimento. Questi operatori consentono di concatenare comandi sulla base del loro codice di uscita, senza ricorrere a un blocco `if`.

## 5. Contare le righe prodotte da `strace`, escludendo le ultime due

```
strace -e trace=memory ls 2>&1 | head -n -2 | wc -l
```

**Spiegazione**

La pipeline esegue `ls` sotto osservazione tramite `strace`, un programma che permette di tracciare le chiamate di sistema effettuate da un processo. Successivamente, conta le righe dell'output escludendo le ultime due.

- `strace -e trace=memory ls`: esegue `ls` tracciando le chiamate di sistema classificate come operazioni di memoria, secondo la selezione offerta da `strace`.

- `2>&1`: reindirizza lo standard error (`stderr`) sullo standard output (`stdout`). È necessario perché `strace` scrive normalmente le informazioni di tracciamento sullo standard error; in questo modo anche tali informazioni possono essere elaborate dalla pipeline.

- `head -n -2`: stampa tutte le righe dell'input tranne le ultime due. Questa forma di `head` è supportata da GNU Coreutils.

- `wc -l`: conta le righe rimaste e restituisce il totale.

**Concetto chiave:** `2>&1` permette di unificare i due flussi di output prima di passarli attraverso una pipe. In questo modo, anche i messaggi normalmente inviati allo standard error possono essere filtrati e contati dai comandi successivi.
