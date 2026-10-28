# L1

Prima di avviare un processo, il sistema operativo deve fare alcune cose come assegnargli il PID. La macchina fornisce delle astrazioni. Le principali:

- Syscall
- Memoria
- CPU
- Disco

Chiamata di sistema permette a un processo di richiedere un servizio al sistema operativo in cui è richiesto di interagire con l'hardware (es. scrivere su un file). L'OS prende controllo della CPU (processo va in blocked) e esegue un handler della chiamata di sistema (syscall handler).

L'interrupt è un segnale che viene notificato alla CPU. Può essere hardware e software (lanciato dall'asm). La procedura di gestione eseguita dalla CPU è disaccoppiata dall'esecuzione del processo: ha un proprio spazio di memoria.

## Linux Shell

L'editor che useremo è vim, un editor di tipo **modale**, in cui si hanno delle modalità diverse:

- Insert mode, per scrivere
- Command mode, per dare i comandi
  Premere I per entrare in Inser mode, andare Esc per andare in command mode. Le lettere principali sono:
- :w per salvare
- :q per uscire. Per forzare, e quindi non salvare, :q!

**La Shell** è un *interprete di comandi*. Le principali sono bash e zhs.

## Syscall

asm lo abbiamo già visto, ma abbiamo visto una RISC(Reduced Instruction Set Computer), mentre i386, quella che useremo noi, è un CISC (Complex Instruction Set Computer).

Quando faccio una chiamata di sistema, dobbiamo fargli capire che chiamata vogliamo effettuare. Quello che facciamo è caricare `EAX` e fargli la mov(carico il registro) del numero identificativo della syscall che vogliamo eseguire.

La syscall poi vuole degli argomenti, quindi carichi prima `EAX` e poi carichi i registri richiesti con gli argomenti richiesti. Dopo, lanci una `int` (Interrupt software).

- `EAX = 1` è la sys_exit
- `EAX = 2` è la sys_fork
- `EAX = 3` è la sys_read

Quando si scrive il programma in asm, ci sono due sezioni: `.data`, `.txt`.

La prima è per l'allocazione statica, quindi quando scrivi il codice sai già il valore di quelle variabili che rimangono costanti per tutta l'esecuzione.

`.txt` invece contiene il codice asm, che poi viene assemblato e diventa codice macchina. Questa sezione è RO e ne carica una sola copia in memoria.

```x86asm
.data
.text
mov aex, $3 ; questo e un commento (cominciano con punto e virgola)
```

`clear` per pulire terminale 

`ls -lah` (in formato di lista, tutto compreso file nascosti, human readable) per listare. Spiegare colonne

```
drwxr-sr-x  4 user user  160 Sep 30 23:30 .
drwxr-sr-x  4 root root   60 Sep 30 20:03 ..
-rw-r--r--  1 root root   49 Feb 27  2017 .gitconfig
drwx--S---  3 user user   60 Sep 30 20:03 .kde
-rw-r--r--  1 user user   19 Sep 30 20:03 .su-to-rootrc
-rw-------  1 user user 1.6K Sep 30 23:30 .viminfo
drwxr-sr-x 10 user user  200 Mar 13  2014 asm_examples
-rw-r--r--  1 user user  30K Sep 30 23:29 asm_examples.tar
-rw-r--r--  1 user user   31 Sep 30 23:27 main.c
```

I permessi sono relativi all'utente, poi al gruppo di appartenenza e infine gli altri. 

`cat nome_file` per vedere il contenuto del file

`cd nome_dir` change directory
`tar -xvf nome_archivio` per estrarre

## ASM

Prendiamo i sorgenti asm e li assembliamo con l'assemblatore. Noi usiamo **nasm**.

```shell
$ nasm -f elf helloworld.asm
```

Questo genera un file oggetto in formato binario elf di un file sorgente `helloworld.asm`. Se abbiamo più file oggetto separati e li vogliamo unire per fare un unico grande file dobbiamo linkare:

```shell
$ ld -m elf_i386 helloworld.o -o helloworld
```

Vediamo l'hello world:

```x86asm
SECTION .data
msg     db      'Hello World!', 0Ah    ; assign msg variable with your message string

SECTION .text
global  _start

_start:

    mov     edx, 13    ; number of bytes to write - one for each letter plus 0Ah (line feed character)
    mov     ecx, msg   ; move the memory address of our message string into ecx
    mov     ebx, 1     ; write to the STDOUT file
    mov     eax, 4     ; invoke SYS_WRITE (kernel opcode 4)
    int     80h
```

_start è un'etichetta che ci indica dove inizia il file sorgente. Se lo eseguiamo:

```
Hello World!
Segmentation fault
```

Questo perché il PC continua ad incrementare perché non gli abbiamo detto dove concludere e arriva oltre alla porzione di memoria accessibile dal processo. Quindi dobbiamo modificare il programma:

```
SECTION .data
msg     db      'Hello World!', 0Ah    ; assign msg variable with your message string

SECTION .text
global  _start

_start:

    mov     edx, 13    ; number of bytes to write - one for each letter plus 0Ah (line feed character)
    mov     ecx, msg   ; move the memory address of our message string into ecx
    mov     ebx, 1     ; write to the STDOUT file
    mov     eax, 4     ; invoke SYS_WRITE (kernel opcode 4)
    int     80h				 ; launch a syscall 

    mov     eax, 1		; invoke SYS_EXIT
    int     80h				
```

Quindi qui facciamo due chiamate di sistema: una per stampare e l'altra per la sys_exit. Questo è quello che si trova in E2.

Quando però, per esempio, vogliamo leggere da tastiera non conosciamo il numero di caratteri, quindi va calcolato.

```x86asm
SECTION .data
msg     db      'Hello, solab2 world!', 0Ah ; we can modify this now without having to update anywhere else in the program

SECTION .text
global  _start

_start:

    mov     ebx, msg       ; move the address of our message string into EBX
    mov     eax, ebx       ; move the address in EBX into EAX as well (Both now point to the same segment in memory)

nextchar:
    cmp     byte [eax], 0  ; compare the byte pointed to by EAX at this address against zero (Zero is an end of string delimiter)
    jz      finished       ; jump (if the zero flagged has been set) to the point in the code labeled 'finished'
    inc     eax            ; increment the address in EAX by one byte (if the zero flagged has NOT been set)
    jmp     nextchar       ; jump to the point in the code labeled 'nextchar'

finished:
    sub     eax, ebx       ; subtract the address in EBX from the address in EAX
                            ; remember both registers started pointing to the same address (see line 15)
                            ; but EAX has been incremented one byte for each character in the message string
                            ; when you subtract one memory address from another of the same type
                            ; the result is number of segments between them - in this case the number of bytes

    mov     edx, eax       ; EAX now equals the number of bytes in our string
    mov     ecx, msg       ; the rest of the code should be familiar now
    mov     ebx, 1
    mov     eax, 4
    int     80h

    mov     ebx, 0
    mov     eax, 1
    int     80h
```

Quello che questo codice fa è, sostanzialmente, leggere la stringa in memoria fino a quando non trova 0 e incrementa eax per ogni byte trovato. Quello che succede alla fine della stringa è che ebx contiene l'indirizzo di dove comincia la stringa, eax quello di dove termina: eax-ebx è quindi il numero di byte della stringa.

In MIPS per le chiamate a procedura si usava JAL, ovvero un'istruzione che salta alla procedura e salva nel registro $RA l'indirizzo di ritorno (quello dell'istruzione subito dopo la JAL).

Una volta che la procedura era terminata facevano `jr $ra` che è una jump return. Questa istruzione ci aiuta a evitare di dover scrivere a mano  l'indirizzo di ritorno e gestire tutto manualmente:

```MIPSasm
main:
    li      $a0, 5          # Load first argument into $a0
    li      $a1, 10         # Load second argument into $a1
    jal     add_numbers     # Call subroutine (saves return address in $ra)
    # Result is now in $v0

# --- Subroutine ---
add_numbers:
    add     $v0, $a0, $a1   # Add $a0 and $a1, put result in return register $v0
    jr      $ra             # Return to the instruction after jal
```

In i386 seguiremo una procedura:

1. Memorizzare indirizzo in cui si trova l'istruzione nel momendo della chiamata
2. Eseguire la subroutine
3. Ritornare all'istruzione dopo

Per fare questo useremo lo stack, ma automaticamente usano `call` e `ret`:

```x86asm
section .text
global _start

_start:
    mov eax, 5          ; Prepariamo il numero 5 in EAX
    mov ebx, 999        ; !!! IMPORTANTE: EBX contiene un dato prezioso del programma principale

    ; Chiamata alla funzione (salva l'indirizzo di ritorno sullo stack)
    call raddoppia      

    ; Al ritorno, EAX contiene il risultato (10).
    ; EBX contiene ancora 999! Non è stato alterato dalla funzione.
    
    ; Fine del programma
    mov eax, 1          
    int 0x80

; --- FUNZIONE ---
raddoppia:
    ; 1. SALVATAGGIO (Prologo)
    push ebx            ; Salva il vecchio valore di EBX (999) sullo stack

    ; 2. CORPO DELLA FUNZIONE
    mov ebx, eax        ; Copia il valore di EAX (5) dentro EBX
    add eax, ebx        ; EAX = EAX + EBX (5 + 5 = 10)

    ; 3. RIPRISTINO (Epilogo)
    pop ebx             ; Ripristina il valore originale di EBX (999) dallo stack
    
    ret                 ; Ritorna al chiamante
```

La distinzione tra procedura e etichetta è quindi che la prima è raggiunta con la `call`, mentre la label la raggiungi con una `jump`.

In architettura i386, inoltre, i registri sono una risorsa limitata e condivisa. Quando la tua funzione raddoppia decide di usare il registro EBX per fare i suoi calcoli, rischia di sovrascrivere e distruggere i dati che il programma principale aveva memorizzato in quel registro (nel nostro esempio, il numero 999).
Per evitare questo problema, l'assembly x86 segue delle regole precise chiamate Convenzioni di Chiamata (Calling Conventions). I registri si dividono in due categorie:

1. Callee-Saved Registers (Registri preservati dalla funzione): EBX, ESI, EDI, EBP. Se una funzione vuole usare uno di questi registri, deve salvarne il valore iniziale sullo stack (push) e rimetterlo a posto prima di uscire (pop).
2. Caller-Saved Registers (Registri volatili): EAX, ECX, EDX. La funzione è libera di modificarli senza chiedere il permesso. Se il programma principale ci tiene a quei dati, deve salvarli lui prima di fare la CALL.

Vediamo poi due comandi simili a cat utili:

```bash
$ head -n 5 file.asm
$ tail -n 5 file.asm
```

Che stampano rispettivamente le prime 5 righe e le ultime 5.

## File esterni

Ora vogliamo scrivere le procedure in un file sorgente a parte dal file dove si trova la procedura che fa da punto di ingresso. Per farlo, immaginiamo di avere un sorgente con dentre `functions.asm` e dentro `main.asm`, in testa, useremo un include:

```x86asm
%include        'functions.asm'
```

A questo punto dobbiamo assemblare e linkare i due file:

```bash
$ nasm -f elf functions.asm ; nasm -f elf main.asm ; ld -m elf_i386 main.o functions.o -o final_bin
```

### Esempio

functions.asm:

```x86asm
;------------------------------------------
; int slen(String message)
; String length calculation function
slen:
    push    ebx
    mov     ebx, eax

nextchar:
    cmp     byte [eax], 0
    jz      finished
    inc     eax
    jmp     nextchar

finished:
    sub     eax, ebx
    pop     ebx
    ret


;------------------------------------------
; void sprint(String message)
; String printing function
sprint:
    push    edx
    push    ecx
    push    ebx
    push    eax
    call    slen

    mov     edx, eax
    pop     eax

    mov     ecx, eax
    mov     ebx, 1
    mov     eax, 4
    int     80h

    pop     ebx
    pop     ecx
    pop     edx
    ret


;------------------------------------------
; void exit()
; Exit program and restore resources
quit:
    mov     ebx, 0
    mov     eax, 1
    int     80h
    ret
```

helloworld-inc.asm:

```x86asm
%include        'functions.asm'                            ; include our external file

SECTION .data
msg1    db      'Hello, solab2 world!', 0Ah                ; our first message string
msg2    db      'This is how we recycle in NASM.', 0Ah     ; our second message string

SECTION .text
global  _start

_start:

    mov     eax, msg1      ; move the address of our first message string into EAX
    call    sprint         ; call our string printing function

    mov     eax, msg2      ; move the address of our second message string into EAX
    call    sprint         ; call our string printing function

    call    quit           ; call our quit function
```

```shell
$ nasm -f elf functions.asm ; nasm -f elf helloworld-inc.asm ; ld -m elf_i386 helloworld-inc.o func
tions.o -o final_bin
```

Quello che stama è:

```
Hello, solab2 world!
This is how we recycle in NASM.
This is how we recycle in NASM.
```

Questo perché la prima volta, sprint stampa fino a quando non trova 0, ma nel segmento data nessuno delle due variabili ha uno 0. Quindi la prima volta parte dall'indirizzo di 'H' e continua fino a quando non finisce data, stampando entrambi i messaggi. La seconda volta parte da 'T' e stampa sempre fino alla fine di data. Il tipo `db` indica un insieme di caratteri (di bytes).

Per ovviare, dobbiamo mettere il terminatore di stringa 0 alla fine delle stringhe. Lo 0Ah è l'equivalente di una newline, quindi data diventa:

```x86asm
%include        'functions.asm'                            ; include our external file

SECTION .data
msg1    db      'Hello, solab2 world!', 0h, 0Ah                ; our first message string
msg2    db      'This is how we recycle in NASM.', 0h, 0Ah     ; our second message string
; ...
```

## gdb

E' un debugger che ci permette di osservare lo stato del programma in vari punti. Per farlo dobbiamo assemblare inserendo i simboli, ovvero le label, che ci aiutano a identificare l'entry point:

```shell
$ nasm -f elf -gstabs  functions.asm ; nasm -f elf -gstabs helloworld-inc.asm ; ld -m elf_i386 helloworld-inc.o func
tions.o -o final_bin
```

Quindi avviamo il debugger:

```shell
$ gdb final_bin
```

Quindi

```gdb
(gdb) b _start
```

Mette il break point alla procedura d'ingresso. Quindi:

```gdb
(gdb) run
```

E si ferma all'inizio. Quindi possiamo fare

```gdb
(gdb) x/i10 _start
```

Per stampare le prime 10 istruzioni della procedura _start.

## Lettura

Prendiamo il seguente sorgente:

```
%include        'functions.asm'

SECTION .data
msg1        db      'Please enter your name: ', 0h     ; message string asking user for input
msg2        db      'Hello, ', 0h                      ; message string to use after user has entered their name

SECTION .bss
sinput:     resb    255                                ; reserve a 255 byte space in memory for the users input string

SECTION .text
global  _start

_start:

    mov     eax, msg1
    call    sprint

    mov     edx, 255       ; number of bytes to read
    mov     ecx, sinput    ; reserved space to store our input (known as a buffer)
    mov     ebx, 0         ; write to the STDIN file
    mov     eax, 3         ; invoke SYS_READ (kernel opcode 3)
    int     80h

    mov     eax, msg2
    call    sprint

    mov     eax, sinput    ; move our buffer into eax (Note: input contains a linefeed)
    call    sprint         ; call our print function

    call    quit
```

In ebx mettiamo il file di lettura. I classici subito aperti sono stdin (0), stdout (1) e stderr (2); .bss contiene i dati dinamici.