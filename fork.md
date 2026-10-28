# `fork` in assembly x86-32 (i386)

In Linux su x86-32, `fork` è una **chiamata di sistema** che crea un nuovo processo, detto *figlio*, duplicando il contesto del processo chiamante.

La chiamata restituisce valori diversi nei due processi:

- **Processo padre:** riceve il PID del figlio (un valore positivo).
- **Processo figlio:** riceve `0`.
- **Errore:** restituisce un valore negativo, che rappresenta il codice d'errore nelle convenzioni delle syscall Linux.

## Esempio NASM

```nasm
section .text
global _start

_start:
    mov eax, 2      ; numero della syscall fork su Linux i386
    int 0x80        ; entra nel kernel

    cmp eax, 0
    je figlio       ; eax = 0: siamo nel processo figlio

padre:
    ; Qui continua il processo padre.
    mov ebx, 0
    mov eax, 1      ; syscall exit
    int 0x80

figlio:
    ; Qui continua il processo figlio.
    mov ebx, 0
    mov eax, 1      ; syscall exit
    int 0x80
```

## Come funziona

1. Su Linux i386, il numero della syscall viene messo in `EAX`; per `fork` è `2`.
2. `int 0x80` trasferisce il controllo al kernel, che crea il processo figlio.
3. Quando la syscall termina, **entrambi i processi riprendono dall'istruzione successiva a `int 0x80`**.
4. Il valore restituito in `EAX` permette di distinguere i due percorsi: `0` nel figlio e PID del figlio nel padre.
5. Ciascun processo esegue il proprio ramo e termina con `exit` (syscall `1`).

**Da ricordare:** `fork` non crea due flussi di esecuzione dentro lo stesso processo: crea un secondo processo. Padre e figlio proseguono da un punto di esecuzione equivalente, ma hanno contesti di processo distinti. L'esempio è specifico per Linux a 32 bit e usa la convenzione storica `int 0x80`.
