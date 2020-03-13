# theorie 3

## xargs

Xargs is essentieel voor linux, het veranderd standard input in command line arguments.
Commandos zoals echo of cp kunnen alleen command line arguments accepteren.
Xargs kan standard input en command line arguments accepteren.
Xargs zet deze dan om naar command line arguments.
Dus voor commandos zoals echo of cp kan Xargs handig zijn in pipelining.
Awk heeft gelijkaardige functionaliteit.

### [standard input](http://www.linfo.org/standard_input.html)

Standard input of stdin zijn streams van data, er zijn 3 soorten stream.
1 input stream, standard input, en 2 output streams, standard output en standard error.
Deze streams zijn plain text, het uit de keyboard komen of uit een file.

```bash
wc file
```

De file word omgezet naar standard input dat wc dan kan lezen.

```bash
wc
```

Als men gewoon wc ingeeft als commando dan kan de gebruiker zelf via het toetsenbord standard input intypen.
Als men op control + d duwt eindigt de input en word de normale output van wc gegeven.

#### [input redirection](https://www.tutorialspoint.com/unix/unix-io-redirections.htm)

Dit is de **<** operand.
Dit zet files om naar standard input.

```bash
wc -l file
```

output:

>2 file

door de -l optie word de filenaam naast de count geplaatst

```bash
wc -l < file
```

output:

>2

de filenaam word niet getoond omdat de file word omgezet naar standard input.
wc kan dus de naam niet achterhalen.

#### [output redirection](https://www.tutorialspoint.com/unix/unix-io-redirections.htm)

Dit is de **>** operand.
Dit zet standard input om naar een bestand.

| [standard input] | > | [file] |
|---|---|---|

### commandos

```bash
locate -b "*README*"
```

zoekt alle bestanden die locate bevat.

```bash
md5sum $(locate -b "*README*")
```

doet md5sum in een keer op honderden bestanden.
Files met spaties lukt niet?
Files met kapkes gaat ook niet.

```bash
locate -b "* *" | nl
```

Zoeken naar files met spaties.


* nl: Voegt nummering toe aan output

```bash
md5sum $(locate -b "* *" | nl)
```

geeft foutmeldingen vanwege files met spaties

Output:

>md5sum: Text: No such file or directory\
>md5sum: 2/Pristine: No such file or directory\
>md5sum: Packages/Color: No such file or directory\
>md5sum: Scheme: No such file or directory

```bash
locate -b "* *" | tr '\n' '\0' | xargs -0 md5sum
```

Dit lukt wel maar geeft foutmeldingen bij directories.

```bash
wc words
```

output:

>17428 178428 1998896

Het punt is dat het veel accenten bevat

```bash
xargs < words
```

output:

>xargs: unmatched single quote; by default quotes are special to xargs unless you use the -0 option

geeft een foutmelding. xargs accepteert geen woorden met \' in.

```bash
tr $'\'' _ < words | xargs | wc
```

output:

>16 178428 1998896

Heef thet opgedeeld in 16 lijnen?

Voor xargs maakt het niet uit hoe groot de opdrachtregels is.

Voor vieze karakters of te veel tekst gebruikt men best xargs.

---

## find

```bash
locate -b "* *"
```

```bash
rm <file>
```

```bash
locate -b "* *"
```

De \<file> is er nog.
Het systeem neemt snapshots.
locate kijkt door de snapshot.

```bash
find
```

zoekt door het huidge filesysteem in tegensteling tot locate.
Kijk naar slides voor opties.

```bash
find / -name "* *" -type f
```

zoekt alle bestanden in de root.
Patroon altijd tussen kapjes.
Als het patroon niet tussen kapjes staat gaat de shell het interpreteren.

* / dit verwijst naar de root.
* type 
    * f: verwijst naar directories

Duurt langer dan locate want het gebruikt geen snapshot.
Alls we files verwijderen zien we dat ze niet meer zichtbaar zijn.

```bash
find / -name "* *" -type f -printf "%s\t%p"
```

* printf: Niets te maken met het commando
    * %s: toont fileszie
    * %p: toontpad.
    * %t: toont timestamp

```bash
touch -t 201501010000 t1
touch -t 201601010000 t2
```

| 2016 | 01 | 01 | 00 | 00 |
| --- | --- | --- | --- | --- |
| Year | Month | Day | Hour | Minute |

Om files tussen 2 tijden te vinden maakt men eerst 2 referentie files.

```bash
find / -name "* *" -newer t1 ! -newer t2 -type f -printf "%s\t%p"
```

* newer: nieuwer dan
* ! omgekeerde predicaat


```bash
find / -name "* *" -newer t1 ! -newer t2 -type f -size 2M -printf "%s\t%p"
```

* size: grootte

File moet exact 2 mega zijn.

```bash
find / -name "* *"  -size +2M -printf "%s\t%p"
```

* size +2M : zoekt naar files groter dan 2 mega.

```bash
find / -name "* *" -maxdepth 3 -size +2M -printf "%s\t%p"
```

Slechte volgorde van argumenten

```bash
find / -maxdepth 3 -name "* *" -size +2M -printf "%s\t%p"
```

Max diepte van 3 folders.

### We willen gevonden files in een temp folder steken

```bash
find / -name "* *"  -size -2M -type f | tr "\n" "\0" | xargs -0 cp $T
```

* xargs cp: Hier kopieert hij alles naar de laatste file in de lijst. We moeten alle files in 1 aparte temp folder steken.

```bash
find / -name "* *"  -size -2M -type f | tr "\n" "\0" | xargs -0 -i -p cp {} $T
```

* xargs -i: Laat ons toe een pad te specifieren.
    * Behandeld maar 1 argument per keer.
* xargs -p: Geeft een waarschuwing voor uitvoer.

```bash
find / -name "* *"  -size -2M -type f | tr "\n" "\0" | xargs -0 -i -t cp {} $T
```

* xargs -t: Hier zegt hij wat hij heeft gedaan id uitvoer.

```bash
find / -name "* *"  -size -2M -type f -exec cp {} $T \; 2> /dev/null
```

* -exec: Gaat iets doen met elke file die het vindt. 
    * \\; altijd nodig om exec te eindigen.

```bash
find / -name "* *"  -size -2M -type f -exec cp {} $T \; 2> /dev/null
```

```bash
find / -name "* *"  -size -2M -type f -exec cp {} $T \; | wc
```

```bash
find / -name "* *"  -size -2M -type f -ok cp {} $T \;
```

* -ok: Vraagt toestemming bij elke lijn.

```bash
find / -name "* *"  -size -2M -type f -exec md5sum {} $T \;
```

* Berekent md5sum op elk bestand.

```bash
find / -name "* *"  -size -2M -type f -exec md5sum {} $T \; | wc
```

* veel lijnen. kan dit beter?

```bash
find / -name "* *"  -size -2M -type f -exec wc {} \;
```

Telt de newlines, words en bytes van elke bestand dat matcht.

equivalent:
> wc \<file1>\
> wc \<file2>\
> wc \<file3>\
> wc \<file4>\
> wc \<file5>\
> wc \<file6>\
> wc ...

```bash
find / -name "* *"  -size -2M -type f -exec wc {} \+
```

* \\+ Ipv veel lijnen voegt het 1 lijn uit met daarop alle files.


equivalent:
> wc \<file1> \<file2> \<file3> \<file4> \<file5> \<file6> ...

Veel sneller dan het vorige commando.

EXAMEN

```bash
cat -n <file>
```

lijn nummers op blanco lijnen

```bash
cat -b <file>
```

geen lijn nummers op blanco lijnen. BELANGRIJK

```bash
cat -s <file>
```

squeeze

```bash
tr -d '\n' < file
```

verwijderd enig voorkomen van '\n'.
De tekst plakt nu wel aan in elkaar in een doorlopende tekst.
Dit willen we niet.

```bash
tr -s '\n' < file
```

Pakt SET1, in dit geval '\n' en vervangt herhaalde sequenties van dit karakter door 1 verschijning van dit karakter. 

```bash
tr -v '^$' file
```

```bash
tac <file
```

Lijst achterstevoren

```bash
rev <file
```

Lijnen op originele volgorde, karakters omgekeerd

## Lengte van string

```bash
wc -l <file
```

Newline count.

```bash
wc -w <file
```

Word count

```bash
wc -c <file
```

byte count

```bash
x="<line>"
declare $x
echo ${#x}
```

NOG NIET DE JUISTE SYNTAX

```bash
x="<line>"
declare $x
echo $x | wc -c
```

```bash
x="<line>"
declare $x
wc -c <<< "$x"
```

```bash
x="<line>"
declare $x
y=$(wc -c <<< "$x")
echo $y
```

```bash
y=$(wc -l <<< <file>)
declare y
declare -p y
declare .. 
echo ${#x}
```

## EXAMEN GEEF MIJ GROOTSTE LIJN VAN BESTAND

```bash
wc -L <file>
```

geeft lengte van langste lijn van bestand.

```bash
strings -n $(wc -L  < <file>) <file>
```

Geeft langste lijn van bestand.
Strings -n zoekt naar lijnen groter of gelijk aan een gegeven lengte in een gegeven file.
wc -L geeft lengte van langste lijn in file.
Dus een combinatie van de 2 geeft de langste lijn in een file.

```bash
declare f=<file>
strings -n $(wc -L  < "$f") "$f"
```

Nu moeten we de filename niet dubbel schrijven

---

## cut

```bash
nl -ba -nrz <file>
```

output:

>000001	weerwr'\
>000002\
>000003\
>000004	qqwrqwr'\
>000005\
>000006	qrr'
>000007\
>000008	qrqwr'\
>000009\
>000010\
>000011	reegrewqjdlwjekthewlthelthhkweklth\
>000012\
>000013\
>000014\
>000015	qwrqwr' ewgewgewggwe\
>000016	qwrqrw'\
>000017\
>000018	wejlqeqwehwqelkhkl\
>000019	iqrqwr' bwepk;ew\

index bevat string met 6 karakters

```bash
nl -ba -nrz <file> | cut -c5-
```

Toont alles van 5de kolom tot einde.
Dus de eerste 4 characters in de index worden verwijderd.

```bash
ps -aef | cut -c10-21,48-
```

Toont alles van kolom 10 tot 21 en alles na 48

### in csv bestand

Werkt misschien in andere bestanden.

```bash
cut -d\: -f1,4 <file>
```

toont Veld 1 en 4 van file.

EXAMEN

```bash
cut -d\: -f2,3 --complement <file>
```

Geeft alles behalve kolom die men wilt.

### niet in csv bestand

```bash
rev <file> | cut -d' ' -f1,2  <file> | rev <file>
```

NAKIJKEN, EXAMEN

## tail & head

```bash
head -n 8 <file> | tail -n 2
```

```bash
head -n 8 <file> | tail -n +7
```

```bash
head -n -2 <file>
```

```bash
ls -l /dev/urandom
```

Elke keer als men er uit leest krijgt men een random byte.

```bash
tial -c 80 /dev/urandom
```

Lukt niet, men geraakt niet aan het eind omdat het constant nieuwe rand bytes maakt.

```bash
head -c 80 /dev/urandom
```

Lukt wel.

```bash
head -c 80 /dev/urandom | od -w4
```

octodump

```bash
head -c 80 /dev/urandom | od -w4 -An
```

```bash
head -c 80 /dev/urandom | od -w4 -tu1 -An
```

* -w4: 4 bytes per output lijn
* -t: output format
    * u1: unsigned decimal met grootte van 1 byte
* -A: adres radix
    * n: None

Lijkt op componenenten van IP-adressen.

```bash
head -c 80 /dev/urandom | od -w4 -tu1 -An | tr -s ' ' '.'
```

```bash
head -c 80 /dev/urandom | od -w4 -tu1 -An | tr -s ' ' '.' | cut -c2-
```

lijkt op ipv4-adressen.

```bash
head -c 320 /dev/urandom | od -w16 -tx2 -An | tr -s ' ' ':' | cut -c2-
```

Lijkt op ipv6 adressen.

* -w16: 16 bytes per output lijn
* -t: output format
    * x2: hexadecimaal met grootte van 2 bytes.

```bash
sort -t$'\t' -k2,2m -k1,1r dw
```

* -k: is sorteersleutel definitie. 

```bash
cut -d$'\t' -f 2 dw
```

```bash
cut -d$'\t' -f 2 dw | sort | uniq
cut -d$'\t' -f 2 dw | sort | uniq -u
cut -d$'\t' -f 2 dw | sort | uniq -c
```

* uniq:
    * -u: print alleen unieke lijnen
    * -c: prefix van number of occurences

---

## printf

```bash
printf
```

gebruikt men het best voor formattering van output