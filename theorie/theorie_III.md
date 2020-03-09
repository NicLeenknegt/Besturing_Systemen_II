# theorie III

## touch

leeg bestand aanmaken met gegeven naam.

Datum van bestanden wijzigen naar huidige datum.

```bash
touch -c {0..9} <dest-file>
```

* Wijzigt de datum van bestanden als ze bestaan. Zon der c optie worden niet bestaande bestanden aangemaakt.

```bash
touch -t [YYYYmmdduuTT] <dest-file>
```

* -t zet datum van bestand om naar gegeven datum.

```bash
touch -r <orgin-file> <dest-file>
```

* zet datum van dest om naar datum van origin.

### script

```bash
touch -r t2 t1 ;
x=0;
tput civis; # cursor onzichtbaar maken
until [[ t2 -nt t1 ]];
do sleep 0.5;
echo -n $'\r' ${ARROW[$((++x%8))]} ' '; # Maakt draaiende pijl
done;
tput civis; # cursor zichtbaar maken
```

* een script dat wacht totdat t2 recenter wordt dan t1.
* animatie stopt als t2 is gewijzigd.
* een belangrijke constructie.

---

## mktemp

* Maakt tijdelijke file voor scripts.

```bash
mktemp -p. XXX
```

* Maakt unieke tijdelijke file.

```bash
T=$(mktemp -p. XXX)
declare -p T # NAKIJKEN
rm -f $T
```

* stopt de inhoud van de file in T.

---

## comm

```bash
shuf -n15 lees | sort > t1;
shuf -n15 lees | sort > t2;
```

* 'lees' is een bestand vol met namen.

```bash
comm t1 t2
```

* Wordt gebruikt voor gesorteerde bestanden.
* Toont 3 kolommen:
    * 1: lijnen alleen in 1
    * 2: lijnen alleen in 2
    * 3: lijnen alleen in 3

```bash
comm -23 t1 t2
```

* toont alleen 2de en 3de kolomm.

```bash
comm -12 t1 t2
```

* toont alleen 1ste en 2de kolomm.

### script

```bash
shuf lees > t;
```

```bash
while read;
do echo $REPLY; # REPLY nakijken.
done < t;
```

* Leest inhoud uit.

```bash
while read;
do ((RANDOM%7)) && echo $REPLY; # REPLY nakijken.
done < t;
```

* sommige files worden niet getoond.

---

## diff

```bash
diff t1 t2;
```

```bash
diff -c t1 t2;
```

```bash
diff -u t1 t2;
```

* Toont verschillen tussen files.
* Verschillende formats.

```bash
diff -y t1 t2;
```

* Toont diff gelijkaardig aan kolomn structuur van comm

```bash
diff -yW30 t1 t2;
```

* Toont diff gelijkaardig aan kolomn structuur van comm. Met kolomnbreeddte 30.

```bash
diff -iyW30 t1 t2;
```

* Case insensitive

```bash
diff -iyW30 --supress-common-line t1 t2;
```

* zelf nakijken

```bash
diff -BiyW30 --supress-common-line t1 t2;
```

* zelf nakijken

### script

```bash
while read;
do ((RANDOM%7)) && {((RANDOM%7)) && echo $REPLY  || echo ${REPLY^^} }  ; # REPLY nakijken.
done < t;
```

---

## sum

* md5 files nakijken

```bash
mv5sum
```

```bash
sha224sum
```

```bash
sha256sum
```

```bash
sha384sum
```

```bash
sha512sum
```

* checksums om integriteit van files na te kijken.

```bash
sha512sum <origin-file> > t; # stopt sum in file
sha512sum -c t; # check checksum in file
```

---

## locate

* Zoekt bestanden op basis van patroon. het patroon matcht met de filenaam

```bash
locate "* *"
```

* Zoekt alle files met spaties.

```bash
locate -b "* *"
```

* Zoekt alle files met spaties.
* Zoekt alleen in eindfile en niet in tussenfolders

```bash
md5sum ${locate -b "* *"} > t
```

* geeft foutmeldingen.

```bash
locate -b "* *" | tr '\n' '\0' 
```

* tr vormt karakters om. Hier vervangt het de lijnscheidingstekens

```bash
locate -b "* *" | tr '\n' '\0' | hexdump  -C
```

* hexdump nakijken, het is iets met checksum.

```bash
locate -b "* *" | tr '\n' '\0' | xargs -0 md5sum
```

* xargs vormt de invoer om naar een opdrachtregel.
* xargs voedert invoer aan md5sum

```bash
locate -b "* *" | tr '\n' '\0' | xargs -0 md5sum > t
md5sum -c t
```

* BELANGRIJK EXAMEN
* sommige inputs zijn folders, geen files.

md5deep is niet nodig als ge locate gebruikt. locate maakt het al recursief.

---

## xargs

```bash
cat tel | echo
```

* werkt niet, echo heeft string nodig.
* pakt invoer als parameters

```bash
cat tel | printf "%-15s %-15s %-15s\n"
```

* werkt niet, printf heeft string nodig.
* pakt invoer als parameters

```bash
cat tel | xargs printf "%-15s %-15s %-15s\n"
```

* xargs construeert printf opdracht.

---

## paste

* t1 en t2 zijn files.

```bash
paste t1 t2
```

* past de input files uit.

```bash
paste -s t1 t2
```

* paste de inputfiles uit op 1 lijn.
* BELANGRIJK EXAMEN

```bash
cat <file> | paste -sd ';'
```

* vervangt de tabs in de lijn van paste door ';'

```bash
cat <file> | paste -sd $' \n'
```

* vervangt de tabs in de lijn van paste door een spatie en een newline.

```bash
shuf -zi 0-9 | paste | hexdump -C
```

* zelf nakijken

---

## shuf

```bash
shuf -zi 0-9
```

* -z voorkomt newlines. alles op 1 lijn

---

## cat

* Pipen naar cat is verboden tenzij dat het cat is met opties.

```bash
cat -v <file>
```

* toont alle unicode characters

```bash
cat -T <file>
```

* toont alle tabs

```bash
cat -e <file>
```

* toont dollar tekens op punt vna de newlines.

---

## join

* db en dw zijn files

```bash
join -t $'\t' dw db
```

```bash
join -t $'\t' -1 1 -2 1 -o 1.2,2.2,2.1 dw db
```

* toont verschillende delen van bestanden in 1 format. ZELF NAKIJKEN.

```bash
join -t $'\t' -1 1 -2 1 -o 1.2,2.2,2.1 -a 2 dw db
```

* [-a 2 ] Ik wil zowiezo 2de bestand zien, inner join achtig.

---

## split

```bash
split -b200KB words
```

* split de file in stukken van 200kb

```bash
T=$(mktemp -p. XXX)
split -b200KB words $T/
```

* split de file in stukken van 200kb
* plaats alles in tijdelijke map
* naaconventie van gesplitte stukken is aa tot xx

```bash
T=$(mktemp -p. XXX)
split -b200KB -d -a2 words $T/
```

* [-d] numerike prefix voor splitbestandsnamen
* [-a2] lengte van namen is 2

```bash
cat $T/* > t
```

* plakt alle gesplitte bestanden te samen.
* Werkt ook met executable.

```bash
sha384sum words t
```

* Met checksum kan men de inhoud van de files checken.

```bash
tail -l $T/00
head -l $T/01
```

* We zien dat de woorden halverwege zijn gesplit.

```bash
T=$(mktemp -p. XXX)
split -C -b200KB -d -a2 words $T/
```

* houd lijnen intact

```bash
T=$(mktemp -p. XXX)
split -l 10000 -d -a2 words $T/
```

* [ -l ] word gesplit per 10000 lijnen.

```bash
T=$(mktemp -p. XXX)
split -n l/10 -d -a2 words $T/
```

* [ -l/N ] Wordt opgedeeld in 10 gelijke stukken.

```bash
T=$(mktemp -p. XXX)
nl words | split -l 10000 -d -a2 - $T/
```

* de uitvoer van nl word gesplit
* [ split - ] accepteert de uitvoer van het vorige stuk in de pipeline.s

```bash
T=$(mktemp -p. XXX)
nl words | split -n l/10000 -d -a2 - $T/
```

* [ -n l/1000 ] gaat niet met [ - ]. omdat die niet op voorhand kan berekenen.

---
