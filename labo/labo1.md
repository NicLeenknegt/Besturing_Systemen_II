# Notas besturingssystemen

## Deel 1

#### 1.1

?

#### 1.2

##### mail
`-vv` zeer verbose error 

```bash
mail -vv -S smtp=smtp.hogent.be -r jari.duyvejonck@ugent.be
```

### vr4

???

### vr5

`wc --` tab tab --> geeft opsomming van options commando.

### vr6

```bash
# Volgende commando's geven ongeveer zelfde uitput
ls 
ll

# geeft word, line ... count
wc [file]
```

## compile in c

* `-E` Preprocessing: maakt een intermindiare file aan.
* `-S` compiler: in assembleer taal.
* `-c` assembler: geeft objectfile terug.
* eenmaal object klaar is wordt via de link editor (linker) omgezet naar een .out file

na het uitvoeren van het commando worden alle tussenliggende bestanden automatisch verwijderd.

```bash
            -E         -S            -c     linken
test.c --> test.i --> test.s --> test.o --> a.out
```

```bash

gcc -c [file]
```

`U` betekent undefined en wil zeggen dat de code niet bekend is.

```bash
[root@localhost ~]# nm test.o
         U __isoc99_scanf
00000000 T main
         U printf
```

## Oef2

#### 2.1

```bash
# voorwaarts zoeken
/patern
```

#### 2.2

```bash
# ignore case sensitive
man -i passwd
```

#### 2.3

Conventional  section  names  include NAME, SYNOPSIS, CONFIGURATION, DESCRIPTION, OPTIONS, EXIT STATUS, RETURN VALUE, ERRORS, ENVIRONMENT, FILES, VERSIONS, CONFORM‐
       ING TO, NOTES, BUGS, EXAMPLE, AUTHORS, and SEE ALSO.

#### 2.4

```bash
# zoek in sysopsis (systemoperations)
man 2 read
```

#### 2.5

`-h` geeft bestands grote in humanreadable formaat weer

```bash
ls -l -h
```

#### 2.6

```bash
alias
# alias cp='cp -i'
# alias egrep='egrep --color=auto'
# alias fgrep='fgrep --color=auto'
# alias grep='grep --color=auto'
# alias l.='ls -d .* --color=auto'
# alias ll='ls -l --color=auto'
# alias ls='ls --color=auto'
# alias mv='mv -i'
# alias rm='rm -i'
# alias which='alias | /usr/bin/which --tty-only --read-alias --show-dot --show-tilde'

# Hoe kleuren weg werken
# oplossing rechtstreeks naar locatie gaan waar commando opgeslagen is

whereis ls
# ls: /usr/bin/ls /usr/share/man/man1/ls.1.gz /usr/share/man/man1p/ls.1p.gz

/usr/bin/ls -l
```


## working of man

```bash
# zoekt in alle manpages met string printf in alle secties 3.
man 3 printf

# copy man in volgende commando
man -P cat less
```

##


## c project

* extern duid aan dat variable in een andere file (header file) staat. Maar moet wel nog geïnstantieerd worden.
* `r` duidt aan op readonly mode. (zie manpage gcc voor andere mogelijke opties.)
* see description `getline` via `man getline`.

```C
#include <stdio.h>
#include <stdlib.h>

extern int lijnnummer = 0;

void printfile(const char *naam) {
        int n;
        FILE *f = fopen(naam, "r");
        // geen array, weten niet wat de maximale grote van een lijn gaat zijn.
        char *buffer;
        // zet waarde 0 zie man getline
        n = 0;
        getline(&buffer, &n, "r");
}
```

opstellen project

```make
prinfile: printfile.o printline.o
        gcc -o printfile printfile.o printline.o
printfile.o: printfile.c
        gcc -c printfile.c
printline.o: printline.c
        gcc -c printline.c
clean:
        rm -f *.o
```

```bash
# volgende commando roept commando op geassocieerd met clean
make clean
```

## Deel 2

### File operatie

merk op `dynamically linked` heeft extra code nodig van buiten het programma nodig.

```bash
file hello.c
# hello.c: C source, ASCII text
file hello
# hello: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=51195af0197b7ce394c656cffa53e1c43359ca72, not stripped
```

### gcc --static

* Hij kopieerd alle code van nodige libraries mee waardoor file groter wordt. `hello2` kan nu wel worden uitgevoerd waar c lib niet aanwezig is.
* Merk op dat het nu `statically linked`
* `Strip` verwijderd alle symbool info dat nodig kan zijn om het executable te reverse engineren naar de oorsprongkelijke code.

```bash
gcc --static hello.c -o hello2
ll -h hello*
# -rwxr-xr-x 1 root root 7.2K Feb 10 14:46 hello
# -rwxr-xr-x 1 root root 728K Feb 10 14:50 hello2

file hello2
# hello2: ELF 32-bit LSB executable, Intel 80386, version 1 (GNU/Linux), statically linked, for GNU/Linux 2.6.32, BuildID[sha1]=019bad86b41b806f4af1507acda12ab187066c97, not stripped


strip hello

ll -h hello
-rwxr-xr-x 1 root root 5.5K Feb 10 14:54 hello
# -rwxr-xr-x 1 root root 5.5K Feb 10 14:54 hello
```

### debuggen

* `-g` always compile with this option if you want to debug
* `quit` exit gdb
* `gdb [file]` debug file
* `list [functie]` give source code (filter mogelijk)
* `br [lijn nummer]` plaast breakpoint op lijnnummer
* `br [condition]` pause only when breakpoint condition is true ex: `br [function] if [variable] = [value]`
* `r` start programm
* `c` continue program if stopped by breakpoint
* `gdb -tui` give debug terminal gui 'of some kind :p'
* `n` if stopped by breakpoint next step
* `info locals` gives information of al local variables
* `info registers` handy for looking at pointers
* `watchpoint [variable]` give information everytime state l changes

#### Hardware/software watchpoint

Vermijd software watchpoints omdat dit zeer traag kan gaan.
Enkel bij zeer complexe points zal gdb er voor kiezen voor softwaire matige watchpoints.

### Memory leak detection

```bash
valgrind ./program
```
