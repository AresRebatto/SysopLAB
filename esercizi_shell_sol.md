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

## 6. Trovare i file più grandi di 50 MB in `/var/log`

```
[ -d /var/log ] && 
find /var/log -print0 -size +50M 2>/dev/null | 
xargs -0 -I{} du -k {} 2>/dev/null | 
sort -nr
```

**Spiegazione**

La pipeline cerca in `/var/log` i file che superano i 50 MB, ne calcola la dimensione in kilobyte e li ordina dal più grande al più piccolo.

- `[ -d /var/log ]`: verifica che `/var/log` esista e sia una directory.

- `&&`: esegue il comando successivo solo se la verifica precedente ha successo.

- `find /var/log -print0 -size +50M`: cerca i percorsi sotto `/var/log` e stampa i nomi separati da un carattere NUL, così da gestire correttamente spazi e altri caratteri speciali nei nomi. `-size +50M` seleziona gli elementi la cui dimensione supera 50 MiB secondo le regole di `find`. Per limitare esplicitamente la ricerca ai file regolari, si può aggiungere `-type f`.

- `2>/dev/null`: scarta i messaggi di errore, per esempio quelli dovuti a permessi insufficienti.

- `xargs -0 -I{} du -k {}`: interpreta i separatori NUL e calcola la dimensione di ciascun percorso in kilobyte.

- `sort -nr`: ordina numericamente (`-n`) in ordine decrescente (`-r`).

Concetto chiave: `-print0` e `xargs -0` permettono di passare nomi di file contenenti spazi senza suddividerli erroneamente.

Variante più leggibile:

```
[ -d /var/log ] && 
find /var/log -type f -size +50M -print0 2>/dev/null |
xargs -0 -I{} du -h {} 2>/dev/null |
sort -hr
```

Qui `du -h` mostra dimensioni leggibili, come `60M` o `1.2G`, e `sort -hr` le ordina in base al valore numerico riconosciuto. La variante è più comoda da leggere, ma il formato delle dimensioni può variare in funzione delle unità utilizzate.

## 7. Individuare la chiamata di sistema più frequente durante `df -hT`

```
strace -f df -hT 2>&1 | 
cut -d'(' -f1 | 
uniq -c | 
sort -nr | 
head -n 1
```

**Spiegazione**

La pipeline esegue `df -hT` sotto osservazione tramite `strace` e cerca la riga relativa al nome di chiamata che compare più frequentemente.

- `strace -f df -hT`: traccia le chiamate di sistema effettuate da `df -hT`. L'opzione `-f` permette di seguire anche i processi figli.

- `2>&1`: unisce lo standard error allo standard output, consentendo di elaborare le righe di tracciamento.

- `cut -d'(' -f1`: mantiene la parte precedente alla prima parentesi aperta, che normalmente contiene il nome della chiamata di sistema.

- `uniq -c`: conta le righe consecutive identiche.

- `sort -nr`: ordina i conteggi dal più alto al più basso.

- `head -n 1`: mantiene soltanto la prima riga, cioè quella con il conteggio maggiore.

Attenzione: `uniq -c` conta soltanto le occorrenze consecutive. Se lo stesso nome compare in punti diversi dell'output, il conteggio può essere suddiviso in più gruppi. Per ottenere il conteggio globale corretto, bisogna prima ordinare le righe:

```
strace -f df -hT 2>&1 |
cut -d'(' -f1 |
sort |
uniq -c |
sort -nr |
head -n 1
```

Concetto chiave: per contare tutte le occorrenze di una stessa riga con `uniq -c`, è necessario raggrupparle prima con `sort`.

## 8. Contare le righe del file `/etc/passwd` che non contengono `nologin` o `false`

```
[ -f /etc/passwd ] && cat /etc/passwd | 
grep -Ev "nologin|false" | 
uniq -c |
wc -l
```

**Spiegazione**

La pipeline verifica l'esistenza di `/etc/passwd`, elimina le righe che contengono `nologin` oppure `false` e conta quante righe rimangono dopo l'eliminazione dei duplicati consecutivi.

- `[ -f /etc/passwd ]`: verifica che il percorso esista e sia un file regolare.

- `&&`: esegue la pipeline soltanto se la verifica ha successo.

- `cat /etc/passwd`: stampa il contenuto del file. In questo caso è superfluo, perché `grep` può leggere direttamente il file.

- `grep -Ev "nologin|false"`: `-E` abilita le espressioni regolari estese, `-v` seleziona le righe che non corrispondono. Il simbolo `|` indica un'alternativa: è sufficiente che compaia una delle due stringhe.

- `uniq -c`: raggruppa le righe identiche consecutive e ne conta le occorrenze.

- `wc -l`: conta le righe prodotte da `uniq -c`.

Attenzione: `wc -l` non conta direttamente gli utenti: conta i gruppi di righe rimasti dopo `uniq`. Se l'obiettivo è semplicemente contare le righe che superano il filtro, è sufficiente:

```
[ -f /etc/passwd ] && grep -Ev 'nologin|false' /etc/passwd | wc -l
```

Concetto chiave: `grep -v` esclude le righe corrispondenti al modello specificato, mentre `wc -l` conta le righe dell'output.

## 9. Trovare i PID dei processi associati a `root` e `sshd`

```
ps aux | grep "root" | grep "sshd" | 
tr -s ' ' | cut -d' ' -f2 | sort -nr
```

**Spiegazione**

La pipeline esamina l'elenco dei processi, filtra le righe che contengono entrambe le stringhe `root` e `sshd`, estrae il secondo campo e ordina i valori numerici in ordine decrescente.

- `ps aux`: mostra i processi in esecuzione, con informazioni come utente, PID e comando.

- `grep "root"`: mantiene le righe contenenti la stringa `root`.

- `grep "sshd"`: tra le righe rimaste, mantiene quelle contenenti `sshd`.

- `tr -s ' '`: comprime le sequenze di spazi consecutivi in un solo spazio.

- `cut -d' ' -f2`: divide le righe usando lo spazio come delimitatore e seleziona il secondo campo, che nell'output standard di `ps aux` corrisponde al PID.

- `sort -nr`: ordina i PID numericamente in ordine decrescente.

Attenzione: i due `grep` cercano le stringhe nell'intera riga, non necessariamente nella colonna dell'utente e in quella del comando. Inoltre, comprimere gli spazi non elimina eventuali spazi iniziali, che potrebbero alterare la selezione dei campi.

Concetto chiave: `tr -s` normalizza i separatori ripetuti, mentre `cut` estrae un campo individuato tramite delimitatore e posizione.

## 10. Contare le chiamate di sistema il cui nome corrisponde a una certa espressione regolare

```
strace df -h 2>&1 | grep -iE "^s.[aeiou]" | 
cut -d'(' -f1 | uniq -c | sort -nr
```

**Spiegazione**

La pipeline esegue `df -h` sotto osservazione tramite `strace`, seleziona alcune righe del tracciamento e conta le occorrenze consecutive dei nomi di chiamata rimasti.

- `strace df -h`: traccia le chiamate di sistema eseguite da `df -h`.

- `2>&1`: reindirizza lo standard error sullo standard output, permettendo alla pipeline di elaborare anche le righe di tracciamento.

- `grep -iE "^s.[aeiou]"`: seleziona le righe che corrispondono al modello:

  - `-i`: ignora la distinzione tra maiuscole e minuscole;

  - `-E`: abilita le espressioni regolari estese;

  - `^`: indica l'inizio della riga;

  - `s`: richiede una `s` come primo carattere;

  - `.`: indica un carattere qualsiasi;

  - `[aeiou]`: richiede una vocale come terzo carattere.

- `cut -d'(' -f1`: estrae la parte precedente alla prima parentesi aperta, normalmente il nome della chiamata di sistema.

- `uniq -c`: conta le occorrenze consecutive di ciascuna riga.

- `sort -nr`: ordina i conteggi dal maggiore al minore.

Attenzione: come nell'esercizio 7, `uniq -c` non aggrega le occorrenze separate da altre righe. Per ottenere i conteggi complessivi, aggiungi `sort` prima di `uniq -c`.

```
strace df -h 2>&1 | grep -iE "^s.[aeiou]" |
cut -d'(' -f1 | sort | uniq -c | sort -nr
```

Concetto chiave: una regex permette di filtrare i nomi in base alla loro struttura, mentre `sort | uniq -c` consente di contare globalmente le occorrenze identiche.

## 11. Trovare i file di `aresr` più grandi di 50 KB, con dimensione di sei cifre

```
find -user aresr -type f -size +50k | 
sed "s/ /-/g" | xargs du -k | 
sed "s/\t/-/g" | grep -E "^[0-9]{6}" | 
sort -nr
```

**Spiegazione**

La pipeline cerca i file appartenenti all'utente `aresr` che superano una certa dimensione, calcola il loro spazio occupato in kilobyte, seleziona quelli la cui dimensione riportata ha sei cifre e ordina i risultati numericamente in ordine decrescente.

- `find -user aresr -type f -size +50k`: cerca i file regolari appartenenti all'utente `aresr` e con dimensione superiore a 50 KiB.

- `sed "s/ /-/g"`: sostituisce ogni spazio nei percorsi con un trattino.

- `xargs du -k`: esegue `du -k` sui percorsi ricevuti, mostrando lo spazio occupato in kilobyte.

- `sed "s/\t/-/g"`: sostituisce i caratteri di tabulazione nell'output di `du` con trattini.

- `grep -E "^[0-9]{6}"`: seleziona le righe che iniziano con esattamente sei cifre consecutive. Il simbolo `^` indica l'inizio della riga e `{6}` richiede sei ripetizioni della cifra precedente.

- `sort -nr`: ordina le righe in base al valore numerico iniziale, dal maggiore al minore.

Attenzione: questa soluzione presenta un problema: sostituire gli spazi nei percorsi con trattini non modifica i nomi reali dei file. Di conseguenza, `xargs du` potrebbe cercare percorsi inesistenti. Inoltre, `du -k` misura lo spazio occupato su disco, non necessariamente la dimensione effettiva del file.

Una variante più semplice, senza `awk` né cicli, che seleziona i file in base alla dimensione effettiva in byte di sei cifre è:

```
find -user aresr -type f -printf '%s %p\n' |
grep -E '^[0-9]{6} ' |
sort -nr
```

- `-printf '%s %p\n'`: stampa la dimensione in byte (`%s`), seguita dal percorso (`%p`).

- `grep -E '^[0-9]{6} '`: seleziona le righe con una dimensione di sei cifre.

- `sort -nr`: ordina i risultati in base alla dimensione.

Questa variante presuppone che i nomi dei file non contengano caratteri di nuova riga.

Concetto chiave: per selezionare dimensioni con un numero preciso di cifre, un'espressione regolare come `^[0-9]{6}` è sufficiente; occorre però distinguere la dimensione effettiva del file dallo spazio occupato su disco.


