# Esercizi

Di seguito si ripotano solo i testi degli esercizi. Per la soluzione (una proposta di soluzione, non per forza l'unica) è possibile andare in [esercizi_shell_sol.md](esercizi_shell_sol.md).

1) 

Trovare i 3 file più grandi all'interno della tua cartella personale, gestendo correttamente eventuali spazi nei nomi dei file.

2) 

Analizzare i processi in esecuzione per estrarre una lista alfabetica e senza duplicati di tutti gli utenti che hanno almeno un processo attivo.

3) 

Contare quante righe contengono la parola "bash" all'interno di tutti i file nascosti (che iniziano col punto) della directory corrente.

4) 

Verificare se la cartella temporanea `/tmp` esiste ed è effettivamente una directory. Se lo è, stampare "Esiste", altrimenti stampare "Errore".

5) 

Eseguire il comando `ls` tracciando solo le chiamate di sistema relative alla memoria, scartando il normale output del comando e visualizzando solo quante chiamate sono state effettuate.

6) 

Costruire una catena logica che verifichi l'esistenza della cartella di sistema `/var/log`. Se è effettivamente una directory, cerca al suo interno tutti i file regolari più grandi di 50 Megabyte, estraendone lo spazio occupato in blocchi da 1 KB per poi ordinarli dal più pesante al più leggero

7) 

Analizzare il comportamento a basso livello del comando `df -hT`. Devi tracciare il comando includendo i processi figli, redirigere l'output per poterlo filtrare, isolare il nome esatto della chiamata di sistema (scartando gli argomenti racchiusi tra parentesi) e individuare qual è la singola syscall ripetuta più frequentemente in assoluto.

8) 

Eseguire il parsing del file `/etc/passwd` dopo averne verificato l'esistenza. Devi scartare (invertendo la ricerca) tutte le utenze di sistema a cui è assegnata una shell invalida o disattivata (percorsi che terminano con `nologin` o `false`), estrarre il percorso delle shell rimanenti, rimuovere i duplicati e contare il totale di queste shell attive uniche.

9) 

Ottenere lo status dei processi del sistema per isolare l'esatto identificativo numerico (PID) di tutti i task gestiti dall'utente `root` che riguardano il demone `sshd`. L'output finale deve contenere unicamente la lista dei PID ordinati numericamente dal più alto al più basso.

10) 

Utilizzare strace (linux syscall tracer) per listare tutte le chiamate di sistema effettuate durante l'esecuzione del comando df -h , il cui nome inizi con la lettera s ed il cui nome abbia, in terza posizione, una vocale. Ordinarle per frequenza di chiamata ed estrarre la chiamata di sistema effettuata piu' frequentemente. Stampare in output 'n syscallname' dove n e' il numero di occorrenze della chiamata a syscallname. Risolvete l'esercizio utilizzando una pipeline.  (es dell'esame).

11)

Cerca tutti i file regolari posseduti dall'utente `root`, con una dimensione superiore a 50 Kilobyte. Utilizza un passaggio sicuro per i nomi con spazi e calcola il loro peso in blocchi da 1 KB. Delinea la struttura tabellare dell'output sostituendo le eventuali tabulazioni iniziali con un singolo spazio. Successivamente, comprimi tutti gli spazi multipli in un singolo carattere separatore trattino `-`. Dal testo formattato, estrai unicamente i file la cui dimensione (ora isolata prima del trattino) è composta da **esattamente 6 cifre**, invertendo poi l'ordine per avere il più grande in cima.
