# labo

## Shell input and Output

```bash
tr 'a-z' 'A-Z' 0 < [filename]

wc d* # count aantal files met beginnende letter d
wc w* 1> t1 2> t2 # 2 is error

wc w* > t1 2 > t1 # beide uitvoer in zelfde file plaatsen, maar werkt niet om dat lijnen overschreven worden

#oplossing
wc w* 1> t1 2>&1 # 1 wordt omgeleid naar bestand, 2 wordt met '&' toegevoegd aan 1

# het maakt niet uit of omleiding voor of na het commando plaats vind
1> t1 wc w* # kan dus ook

1> t1 # is de efficienste manier om een bestand (t1 nu) te legen.
```

`{ wc w* ; ls ;} > t1 2> t2` 
* geeft uitvoer van beide commandos samen uit in de t-bestanden
* laatste `;` is noodzakelijk anders syntax fouten

`f() { wc w* ; ls ; }`
* creert een functie die beide commandos samenvoegt om later te gebruiken
* `f` bij uitvoer van f voert vorige 2 commandos uit
* uitvoer functie kan ook worden omgeleid.

`shuf -n 10 -ri 0-9 | tee t1 t2`
* uitvoer opsplitsen in 3 kanalen
    * t1 kanaal
    * t2 kanaal
    * standaard uitvoer

`shuf -n 10 -ri 0-9 | tee t1 t2 | sort | uniq -d`
* het standaard kanaal wordt gefilterd en delete alle niet unique waarden
* andere kanalen worden niet gesorteerd of gefilterd

`shuf -n 10 -ri 0-9 | tee -a t1 t2 | sort | uniq -d`
* `-a` zorgt voor dat alles geappend word in de files

`man -f kill`
* geeft alle secties van `kill` weer

`info [command]`
* open docummentatie van command in emacs
* info documentatie is uitgebreider dan manpages
* bestaat uit verschillende pagina's en kan navigeerd worden via links in de info pages

`man read`
* geeft alle documentatie van bash
* te veel documentatie en is fout
* `help read` is een beknoptere optie

`help "d*"`
* filterd opties met d
* `"` is noodzakelijk, anders worden alle files met beginnende letter d terug gegeven in het commando

## linux directory hierarchy

`tree -d [path]`
* geeft boomstructuur van de file structuur vanaf het gegeven path

`tree -d [path] | less`
* `less` maakt het makkelijker om alles te navigeren

`tree -dfi [path]`
* geeft het volledig path en niet de naam

`tree -d -L 2 [path]`
* `-L [number]` enkel 2 niveaus diep

`tree -dfi -L 2 [path] | tail -n +2 | head -n -2`
* `tail -n +2` geef alles behalve de eerste 2 lijnen `-n`
* `head -n -2` geef alles behalve de laatste 3 lijnen `-n`

`tree [path]`
* `[dir] -> [path]` pijltjes wijst op een gelinkte file (harde en symbolische links)

`pushd` & `popd`
* houd een stack bij van welke directory ge zijd

`ln -s [paht]`
* creert symbolische link

`ls -tdl *`
* `-t` sort op tijd

`stat [file]`
* geeft meta van file

`unalias cp`