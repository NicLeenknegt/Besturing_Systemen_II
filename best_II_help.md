# best_II_Labo_III

## fork()

start kind process. copy vh parent process en actieve kind processen.
Maakt dus een exacte kopie van het huidige process.

![alt text](images/fork_code.png "fork")

* 1ste fork maakt kopie van het parent process
* 2de fork maakt kopie van het parent process en het child process
* 3de fork maakt kopie van het parent process en de 2 child processen.

## CMD: ps 

### -ef

Toont process ids en toont wie kind is van wie.

### -aux
Toont de status.

Status:
* S is sleeping.
* R is running.
* T is stopped: zelf aangevraagd om te pauzeren.

## CMD: htop

grafische weergave van processen. Ge kunt ook daarin bepalen welke CPU welk process moet doen.

### Tree (f5 indrukken)

Toont op een grafische manier de relatie tussen parent en child processes.

### Nice (f8 indrukken)

Hoe hoger de Nice waarde hoe minder prioriteit. Met f8 stijgt de Niceness maar daalt de prioriteit.

### Kill

stuurt kill signaal. Er zijn meerdere sig9 en sig15 zijn de meest frequente.
