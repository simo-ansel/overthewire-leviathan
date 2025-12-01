# OverTheWire **Leviathan** — Walkthrough & Appunti

[![Stato](https://img.shields.io/badge/Completato-Livelli_0%E2%86%927-brightgreen)](#)
[![Sistema](https://img.shields.io/badge/OS-Ubuntu_22.04_LTS-lightgrey)](#)
[![Ultimo aggiornamento](https://img.shields.io/badge/Aggiornamento-2025--12--01-blue)](#)

> Percorso riproducibile attraverso **OverTheWire — Leviathan**.  
> Questo README documenta l’approccio, i comandi e le tecniche usate per risolvere i livelli.  
> Tutte le **password sono oscurate** in conformità con la policy ufficiale di OverTheWire.

---

## 🧭 Introduzione
Questa guida raccoglie le soluzioni per i livelli di **Leviathan (0 → 7)**, con:
- comandi e workflow utilizzati;
- spiegazione dei passaggi logici;
- esempi di snippet testabili su Linux (**Ubuntu 22.04 LTS**).

---

## 🧰 Tools — principali comandi e servizi

### 🌐 Rete
`ssh`

### 📁 Filesystem & gestione file
`ls` `cd` `pwd` `mktemp` `touch` `ln` `chmod` `cat` 

### 🧾 Testo & stream processing
`grep` `strings` `tr` `echo` `printf`

### 🔡 Analisi binari & tracing
`ltrace` `gdb`

### 🧪 Utility di debugging / conversione
`printf`

---

# 🧩 Percorso dei Livelli
- Ogni livello introduce un nuovo concetto di sicurezza o un comando Linux utile alla progressione.  
- Le password sono **oscurate** per rispetto della policy di OverTheWire.  
- Tutti gli esempi sono **riproducibili su Ubuntu 22.04 LTS**.

<details>
<summary><b>Indice rapido livelli 0 → 7</b></summary>

- [🔹 Livello 0 → 1](#-livello-0--1)  
- [🔹 Livello 1 → 2](#-livello-1--2)  
- [🔹 Livello 2 → 3](#-livello-2--3)  
- [🔹 Livello 3 → 4](#-livello-3--4)  
- [🔹 Livello 4 → 5](#-livello-4--5)  
- [🔹 Livello 5 → 6](#-livello-5--6)  
- [🔹 Livello 6 → 7](#-livello-6--7)  

</details>

---

## ✅ Formato dei contenuti
Ogni livello seguente è strutturato con:

1. **🎯 Obiettivo** — descrizione breve del livello;
2. **💻 Comandi principali** — snippet eseguibili;
3. **🧠 Spiegazione** — analisi del funzionamento;  
4. **🪄 Takeaway** — concetto o tecnica da ricordare.

---

## 🔹 Livello 0 → 1

### 🎯 Obiettivo
Accedere come `leviathan0` e trovare la password per il livello successivo.

### 💻 Comandi principali

```bash
leviathan0@leviathan:~$ ls -a
.  ..  .backup  .bash_logout  .bashrc  .profile

leviathan0@leviathan:~$ cd .backup/

leviathan0@leviathan:~/.backup$ cat bookmarks.html | grep leviathan
<DT><A HREF="http://leviathan.labs.overthewire.org/passwordus.html | This will be fixed later, the password for leviathan1 is [REDACTED]" ADD_DATE="1155384634" LAST_CHARSET="ISO-8859-1" ID="rdf:#$2wIU71">password to leviathan1</A>

# → password per il livello 1
# [REDACTED]
```

### 🧠 Spiegazione
La password è inclusa in un file di backup dei bookmarks. `grep` filtra la riga che contiene "leviathan" e rivela la password.

### 🪄 Takeaway
- Controllare directory nascoste per file di configurazione o backup;
- `grep` è estremamente pratico per scavare informazioni in file di testo.

---

## 🔹 Livello 1 → 2

### 🎯 Obiettivo
Analizzare il binario `check` per scoprire la password per il livello successivo.

### 💻 Comandi principali
```bash
leviathan1@leviathan:~$ ls
check

leviathan1@leviathan:~$ strings ./check
td8 
/lib/ld-linux.so.2
_IO_stdin_used
puts
__stack_chk_fail
system
getchar
__libc_start_main
printf
setreuid
strcmp
geteuid
libc.so.6
GLIBC_2.4
GLIBC_2.34
GLIBC_2.0
__gmon_start__
secr
love
password: 
/bin/sh
Wrong password, Good Bye ...
;*2$"0
GCC: (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0
...
(check output truncated for brevity)
```

```bash
leviathan1@leviathan:~$ ltrace ./check
__libc_start_main(0x80490ed, 1, 0xffffd384, 0 <unfinished ...>
printf("password: ")                                                                 = 10
getchar(0, 0, 0x786573, 0x646f67password: testpassword
)                                                    = 116
getchar(0, 116, 0x786573, 0x646f67)                                                  = 101
getchar(0, 0x6574, 0x786573, 0x646f67)                                               = 115
strcmp("tes", "sex")                                                                 = 1
puts("Wrong password, Good Bye ..."Wrong password, Good Bye ...
)                                                 = 29
+++ exited (status 0) +++
```

```bash
leviathan1@leviathan:~$ ./check
password: sex
$ whoami
leviathan2
$ cat /etc/leviathan_pass/leviathan2
# → password per il livello 2
# [REDACTED]
```

### 🧠 Spiegazione
- `strings` mostra stringhe leggibili nel binario;
- `ltrace` mostra le chiamate runtime: si vede che il programma legge caratteri (`getchar`) e effettua un `strcmp`;
- Eseguendo il binario e inserendo "sex" si ottiene accesso alla shell del livello successivo e si legge la password.

### 🪄 Takeaway
- Combinare analisi statica (`strings`) e dinamica (`ltrace`) è efficace su binari semplici.

---

## 🔹 Livello 2 → 3

### 🎯 Obiettivo
Usare `printfile` per leggere la password del livello successivo, aggirando problemi con spazi nei nomi dei file.

### 💻 Comandi principali
```bash
leviathan2@leviathan:~$ mktemp -d
/tmp/tmp.l2kik6XJX6

leviathan2@leviathan:~$ touch /tmp/tmp.l2kik6XJX6/"test file.txt"

leviathan2@leviathan:~$ ls -la /tmp/tmp.l2kik6XJX6
total 436
drwx------    2 leviathan2 leviathan2   4096 Dec  1 10:22 .
drwxrwx-wt 1441 root       root       438272 Dec  1 10:23 ..
-rw-rw-r--    1 leviathan2 leviathan2      0 Dec  1 10:22 test file.txt

leviathan2@leviathan:~$ ltrace ./printfile /tmp/tmp.l2kik6XJX6/"test file.txt"
__libc_start_main(0x80490ed, 2, 0xffffd354, 0 <unfinished ...>
access("/tmp/tmp.l2kik6XJX6/test file.tx"..., 4)                                     = 0
snprintf("/bin/cat /tmp/tmp.l2kik6XJX6/tes"..., 511, "/bin/cat %s", "/tmp/tmp.l2kik6XJX6/test file.tx"...) = 42
geteuid()                                                                            = 12002
geteuid()                                                                            = 12002
setreuid(12002, 12002)                                                               = 0
system("/bin/cat /tmp/tmp.l2kik6XJX6/tes".../bin/cat: /tmp/tmp.l2kik6XJX6/test: No such file or directory
/bin/cat: file.txt: No such file or directory
 <no return ...>
--- SIGCHLD (Child exited) ---
<... system resumed> )                                                               = 256
+++ exited (status 0) +++

leviathan2@leviathan:~$ ln -s /etc/leviathan_pass/leviathan3 /tmp/tmp.l2kik6XJX6/test

leviathan2@leviathan:~$ ls -la /tmp/tmp.l2kik6XJX6
total 436
drwx------    2 leviathan2 leviathan2   4096 Dec  1 10:24 .
drwxrwxrwt 1442 root       root       438272 Dec  1 10:24 ..
lrwxrwxrwx    1 leviathan2 leviathan2     30 Dec  1 10:24 test -> /etc/leviathan_pass/leviathan3
-rw-rw-r--    1 leviathan2 leviathan2      0 Dec  1 10:22 test file.txt

leviathan2@leviathan:~$ chmod 777 /tmp/tmp.l2kik6XJX6

leviathan2@leviathan:~$ ./printfile /tmp/tmp.l2kik6XJX6/"test file.txt"
# → password per il livello 3
# [REDACTED]
/bin/cat: file.txt: No such file or directory
```

### 🧠 Spiegazione
- `printfile` costruisce una stringa per `system("/bin/cat ...")`, ma non gestisce nomi con spazi: il comando diventa `/bin/cat /tmp/.../test file.txt` e `cat` interpreta "test" e "file.txt" come due argomenti;
- Creando un symlink chiamato `test` che punta al file con la password, e impostando permessi adeguati, si riesce a far leggere al programma il contenuto desiderato.

### 🪄 Takeaway
- Creare symlink e gestire i permessi è una tecnica utile per aggirare limitazioni nella gestione dei nomi di file.

---

## 🔹 Livello 3 → 4

### 🎯 Obiettivo
Individuare la password richiesta da `level3` per ottenere la shell del livello successivo.

### 💻 Comandi principali
```bash
leviathan3@leviathan:~$ ls
level3

leviathan3@leviathan:~$ ./level3
Enter the password> test
bzzzzzzzzap. WRONG

leviathan3@leviathan:~$ ltrace ./level3
__libc_start_main(0x80490ed, 1, 0xffffd374, 0 <unfinished ...>
strcmp("h0no33", "kakaka")                                                           = -1
printf("Enter the password> ")                                                       = 20
fgets(Enter the password> test 
"test\n", 256, 0xf7fae5c0)                                                     = 0xffffd14c
strcmp("test\n", "snlprintf\n")                                                      = 1
puts("bzzzzzzzzap. WRONG"bzzzzzzzzap. WRONG
)                                                           = 19
+++ exited (status 0) +++

leviathan3@leviathan:~$ ./level3
Enter the password> snlprintf  
[You've got shell]!
$ whoami
leviathan4
$ cat /etc/leviathan_pass/leviathan4
# → password per il livello 4
# [REDACTED]
```

### 🧠 Spiegazione
- `ltrace` mostra il confronto fatto tramite `strcmp` e che il programma legge con `fgets`;
- Inserendo la stringa corretta (notare il newline gestito da `fgets`), il binario apre una shell di livello superiore.

### 🪄 Takeaway
- `ltrace` è utile per vedere esattamente quali stringhe vengono confrontate e come il programma legge l'input.

---

## 🔹 Livello 4 → 5

### 🎯 Obiettivo
Decodificare la stringa binaria trovata nella directory `.trash/bin` per ottenere la password per il livello successivo.

### 💻 Comandi principali
```bash
leviathan4@leviathan:~$ ls -a
.  ..  .bash_logout  .bashrc  .profile  .trash

leviathan4@leviathan:~$ cd .trash/

leviathan4@leviathan:~/.trash$ ls
bin

leviathan4@leviathan:~/.trash$ ./bin
00110000 01100100 01111001 01111000 01010100 00110111 01000110 00110100 01010001 01000100 00001010

leviathan4@leviathan:~/.trash$ ltrace ./bin
__libc_start_main(0x80490ad, 1, 0xffffd364, 0 <unfinished ...>
fopen("/etc/leviathan_pass/leviathan5", "r")                                         = 0
+++ exited (status 255) +++

leviathan4@leviathan:~/.trash$ echo "00110000 01100100 01111001 01111000 01010100 00110111 01000110 00110100 01010001 01000100 00001010" \
| tr ' ' '\n' \
| while read b; do printf "\\$(printf '%03o' "$((2#$b))")"; done; \
printf "\n"
# → password per il livello 5
# [REDACTED]
```

### 🧠 Spiegazione
- Il binario stampa la password in formato binario (stringhe di bit);
- Con una pipeline che trasforma i blocchi binari in caratteri ASCII si ottiene la password leggibile.

### 🪄 Takeaway
- Saper convertire da binario a testo in shell è una skill pratica e veloce.

---

## 🔹 Livello 5 → 6

### 🎯 Obiettivo
Ottenere la password per il livello successivo tramite symlink.

### 💻 Comandi principali
```bash
leviathan5@leviathan:~$ ls
leviathan5

leviathan5@leviathan:~$ ln -s /etc/leviathan_pass/leviathan6 /tmp/file.log

leviathan5@leviathan:~$ ./leviathan5
# → password per il livello 6
# [REDACTED]
```

### 🧠 Spiegazione
- Creando un link simbolico verso il file contenente la password e lanciando il binario, il programma legge il file di destinazione e stampa la password.

### 🪄 Takeaway
- Symlink possono essere sfruttati per far leggere a programmi con percorsi fissi file arbitrari.

---

## 🔹 Livello 6 → 7

### 🎯 Obiettivo
Reverse-engineering del binario `leviathan6` con `gdb` per trovare il codice a 4 cifre che permette l'elevazione a `leviathan7`.

### 💻 Comandi principali
```bash
leviathan6@leviathan:~$ ls
leviathan6

leviathan6@leviathan:~$ ./leviathan6
usage: ./leviathan6 <4 digit code>

leviathan6@leviathan:~$ ./leviathan6 1234
Wrong

# Avvio gdb per analisi
leviathan6@leviathan:~$ gdb --args leviathan6 0000
... GDB header omitted ...
(gdb) disassemble main
Dump of assembler code for function main:
   0x080491c6 <+0>:	lea    0x4(%esp),%ecx
   0x080491ca <+4>:	and    $0xfffffff0,%esp
   0x080491cd <+7>:	push   -0x4(%ecx)
   0x080491d0 <+10>:	push   %ebp
   0x080491d1 <+11>:	mov    %esp,%ebp
   0x080491d3 <+13>:	push   %ebx
   0x080491d4 <+14>:	push   %ecx
   0x080491d5 <+15>:	sub    $0x10,%esp
   0x080491d8 <+18>:	mov    %ecx,%eax
   0x080491da <+20>:	movl   $0x1bd3,-0xc(%ebp)
   0x080491e1 <+27>:	cmpl   $0x2,(%eax)
   0x080491e4 <+30>:	je     0x8049206 <main+64>
   0x080491e6 <+32>:	mov    0x4(%eax),%eax
   0x080491e9 <+35>:	mov    (%eax),%eax
   0x080491eb <+37>:	sub    $0x8,%esp
   0x080491ee <+40>:	push   %eax
   0x080491ef <+41>:	push   $0x804a008
   0x080491f4 <+46>:	call   0x8049040 <printf@plt>
   0x080491f9 <+51>:	add    $0x10,%esp
   0x080491fc <+54>:	sub    $0xc,%esp
   0x080491ff <+57>:	push   $0xffffffff
   0x08049201 <+59>:	call   0x8049080 <exit@plt>
   0x08049206 <+64>:	mov    0x4(%eax),%eax
   0x08049209 <+67>:	add    $0x4,%eax
   0x0804920c <+70>:	mov    (%eax),%eax
   0x0804920e <+72>:	sub    $0xc,%esp
   0x08049211 <+75>:	push   %eax
   0x08049212 <+76>:	call   0x80490a0 <atoi@plt>
   0x08049217 <+81>:	add    $0x10,%esp
   0x0804921a <+84>:	cmp    %eax,-0xc(%ebp)
   0x0804921d <+87>:	jne    0x804924a <main+132>
   0x0804921f <+89>:	call   0x8049050 <geteuid@plt>
   0x08049224 <+94>:	mov    %eax,%ebx
   0x08049226 <+96>:	call   0x8049050 <geteuid@plt>
   0x0804922b <+101>:	sub    $0x8,%esp
   0x0804922e <+104>:	push   %ebx
   0x0804922f <+105>:	push   %eax
   0x08049230 <+106>:	call   0x8049090 <setreuid@plt>
   0x08049235 <+111>:	add    $0x10,%esp
   0x08049238 <+114>:	sub    $0xc,%esp
   ...
End of assembler dump.

(gdb) break *0x0804921a
Breakpoint 1 at 0x804921a

(gdb) run
Starting program: /home/leviathan6/leviathan6 0000
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

Breakpoint 1, 0x0804921a in main ()

(gdb) info registers
eax            0x0                 0
ecx            0xffffd4e2          -11038
edx            0x0                 0
ebx            0xf7fade34          -134554060
esp            0xffffd250          0xffffd250
ebp            0xffffd268          0xffffd268
esi            0xffffd340          -11456
edi            0xf7ffcb60          -134231200
eip            0x804921a           0x804921a <main+84>
eflags         0x286               [ PF SF IF ]
...

(gdb) print $ebp-0xc
$1 = (void *) 0xffffd25c

(gdb) x 0xffffd4cc
0xffffd4cc:	0x61687461

(gdb) print/d 0x00001bd3
$2 = 7123

(gdb) exit
A debugging session is active.

	Inferior 1 [process 38] will be killed.

Quit anyway? (y or n) y

# Esecuzione finale con codice corretto
leviathan6@leviathan:~$ ./leviathan6 7123
$ whoami
leviathan7
$ cat /etc/leviathan_pass/leviathan7
# → password per il livello 7
# [REDACTED]
```

### 🧠 Spiegazione
- Disassemblando `main` si nota che il programma popola `-0xc(%ebp)` con `0x1bd3`;
- Il programma chiama `atoi` sull'argomento fornito e poi lo confronta (`cmp %eax, -0xc(%ebp)`);
- Convertendo `0x1bd3` in decimale si ottiene `7123` — il codice giusto da passare a `leviathan6`;
- Inserendo `7123` si ottiene la shell dell'utente `leviathan7` e si legge la password successiva.

### 🪄 Takeaway
- GDB è lo strumento principe per estrarre valori hard-coded da binari.
- Sapere convertire esadecimali in decimali è una operazione pratica nel reverse engineering.

---

## 🏁 Epilogo

Completare **Leviathan 0 → 7** rafforza competenze in:
- analisi binaria e reverse engineering (strings, ltrace, gdb);
- gestione di filesystem temporanei e symlink per aggirare limiti;
- tecniche pratiche per leggere file protetti tramite programmi SUID-like.

---

## 🙏 Crediti

**OverTheWire — Leviathan** è un progetto degli autori OverTheWire. Tutti i diritti dei contenuti originali appartengono ai rispettivi proprietari.

