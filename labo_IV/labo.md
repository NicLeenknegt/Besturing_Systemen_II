# labo

## pipe

ontvangt een array van 2 filedscriptors.

Beide de 1ste is het **READ** end de 2de vormt het **WRITE** end.

```bash
fork();
pipe();
```

* Beide maken een pipe aan.
* Deze pipes staan los van elkaar.

```bash
pipe();
fork();
```

* Er word **1** pipe aangemaakt.
* Beide hebben toegang tot de pipe.
* De child kan naar de parent data toeschrijven.

!['alt text'](images/parent_child_pipe.jpg 'pipe')

### piping in CMD

```bash
grep | nl
```

Deze commando's creeëren een proces die met elkaar communiceren via een unnamed pipe.

```bash
mkfifo p
```

Maakt een **FIFO** aan, word ook queue of pipe genoemd.

## CODE

Dit is het doel:

!['alt text'](images/parent_2_child_piping.jpg 'pipe')

1ste stap Child process leest van parent process. 

!['alt text'](images/parent_2_child_piping_1_child.jpeg 'pipe')

```c
  1 #include <unistd.h>
  2 #include <fcntl.h>
  3 #include <sys/types.h>
  4 #include <sys/wait.h>
  5
  6 int main(int argc, char **argv) {
  7         int fd[2];
  8
  9         if (pipe(fd)<0){
 10                 perror(argv[0]);
 11                 return 1;
 12         }
 13         int pid;
 14         if ((pid=fork()) < 0) {
 15                 perror(argv[0]);
 16                 exit(1);
 17         }
 18
 19         if (pid==0){
 20                 //child process
 21
 22                 //sluit schrijf uiteinde
 23                 //Dit process moet alleen lezen.
 24                 close(fd[1]);
 25                 int number;
 26
 27                 // synchronisatie adhv write
 28                 //while(read(fd[0], &number, 4) !=4);
 29
 30                 //while niet nodig het programma wacht hier totdat de write is afgerond.
 31                 read(fd[0], &number, sizeof(int));
 32
 33                 printf("CHILD: I've read number %d", number);
 34
 35                 //child moet worden afgesloten.
 36                 return 0;
 37         }
 38         //PARENT
 39         // De read in de child wacht totdat de parent heeft geschreven
 40         sleep(15);
 41         int n=0x676;
 42         write(fd[1], &n, 4);
 43         waitpid(pid, NULL, 0);
 44         return 0;
 45s
 46 }
```

We naken nu meerdere childs aan.

!['alt text'](images/1_parent_2_childs_3_pipes.jpg 'pipe')

```c
  1 #include <unistd.h>
  2 #include <fcntl.h>
  3 #include <sys/types.h>
  4 #include <sys/wait.h>
  5
  6 int main(int argc, char **argv) {
  7         int fd_P_C1[2];
  8         int fd_C1_C2[2];
  9         int fd_P_C2[2];
 10
 11         if (pipe(fd_P_C1)<0){
 12                 perror(argv[0]);
 13                 return 1;
 14         }
 15
 16         if (pipe(fd_P_C2)<0){
 17                 perror(argv[0]);
 18                 return 1;
 19         }
 20
 21         if (pipe(fd_C1_C2)<0){
 22                 perror(argv[0]);
 23                 return 1;
 24         }
 25         // IMAGE 1 PARENT heeft nu 3 pipes.
 26
 27         int pid[2];
 28         if ((pid[0]=fork()) < 0) {
 29                 perror(argv[0]);
 30                 exit(1);
 31         }
 32
 33         if (pid[0]==0){
 34                 //child 1
 35                 return 0;
 36         }
 37
 38         if ((pid[1]=fork()) < 0) {
 39                 perror(argv[0]);
 40                 exit(1);
 41         }
 42
 43         if (pid[1]==0){
 44                 //child 2
 45                 return 0;
 46         }
 47        return 0;
 48 }
```

Nu sluiten we uiteinden.

!['alt text'](images/1_parent_2_child_cut_pipes.jpg 'pipe')

```c
  1 #include <unistd.h>
  2 #include <fcntl.h>
  3 #include <sys/types.h>
  4 #include <sys/wait.h>
  5
  6 int main(int argc, char **argv) {
  7         int fd_P_C1[2];
  8         int fd_C1_C2[2];
  9         int fd_P_C2[2];
 10
 11         if (pipe(fd_P_C1)<0){
 12                 perror(argv[0]);
 13                 return 1;
 14         }
 15
 16         if (pipe(fd_P_C2)<0){
 17                 perror(argv[0]);
 18                 return 1;
 19         }
 20
 21         if (pipe(fd_C1_C2)<0){
 22                 perror(argv[0]);
 23                 return 1;
 24         }
 25         // IMAGE 1 PARENT heeft nu 3 pipes.
 26
 27         int pid[2];
 28         if ((pid[0]=fork()) < 0) {
 29                 perror(argv[0]);
 30                 exit(1);
 31         }
 32
 33         if (pid[0]==0){
 34                 //child 1
 35
 36                 close(fd_P_C2[0]);
 37                 close(fd_P_C2[1]);
 38
 39                 return 0;
 40         }
 41
 42         if ((pid[1]=fork()) < 0) {
 43                 perror(argv[0]);
 44                 exit(1);
 45         }
 46
 47         if (pid[1]==0){
 48                 //child 2
 49
 50                 close(fd_P_C1[0]);
 51                 close(fd_P_C1[1]);
 52
 53                 return 0;
 54         }
 55         //PARENT
 56
 57         close(fd_C1_C2[0]);
 58         close(fd_C1_C2[1]);
 59         return 0;
 60
 61 }
```

Nu maken we de pipes 1-richting.

!['alt text'](images/unidirectional_pipes.jpg 'pipe')

```c
  1 #include <unistd.h>
  2 #include <fcntl.h>
  3 #include <sys/types.h>
  4 #include <sys/wait.h>
  5
  6 int main(int argc, char **argv) {
  7         int fd_P_C1[2];
  8         int fd_C1_C2[2];
  9         int fd_P_C2[2];
 10
 11         if (pipe(fd_P_C1)<0){
 12                 perror(argv[0]);
 13                 return 1;
 14         }
 15
 16         if (pipe(fd_P_C2)<0){
 17                 perror(argv[0]);
 18                 return 1;
 19         }
 20
 21         if (pipe(fd_C1_C2)<0){
 22                 perror(argv[0]);
 23                 return 1;
 24         }
 25         // IMAGE 1 PARENT heeft nu 3 pipes.
 26
 27         int pid[2];
 28         if ((pid[0]=fork()) < 0) {
 29                 perror(argv[0]);
 30                 exit(1);
 31         }
 32
 33         if (pid[0]==0){
 34                 //child 1
 35
 36                 close(fd_P_C2[0]);
 37                 close(fd_P_C2[1]);
 38
 39                 //Pipes write kant sluiten
 40                 close(fd_P_C1[1]);
 41                 close(fd_C1_C2[0]);
 42                 return 0;
 43         }
 44
 45         if ((pid[1]=fork()) < 0) {
 46                 perror(argv[0]);
 47                 exit(1);
 48         }
 49
 50         if (pid[1]==0){
 51                 //child 2
 52
 53                 //pipes permanent sluitem
 54                 close(fd_P_C1[0]);
 55                 close(fd_P_C1[1]);
 56
 57                 //Pipes write kant sluiten
 58                 close(fd_P_C2[1]);
 59                 close(fd_C1_C2[1]);
 60                 return 0;
 61         }
 62         //PARENT
 63
 64         //Pipes permanent sluiten.
 65         close(fd_C1_C2[0]);
 66         close(fd_C1_C2[1]);
 67
 68
 69         //Pipes write kant sluiten
 70         close(fd_P_C1[0]);
 71         close(fd_P_C2[0]);
 72         return 0;
 73 }
```

Nu voegen we de waardes toe die worden gepipet.

```c
  1 #include <unistd.h>
  2 #include <fcntl.h>
  3 #include <sys/types.h>
  4 #include <sys/wait.h>
  5 #include <stdio.h>
  6 #include <stdlib.h>
  7
  8 int main(int argc, char **argv) {
  9         int g;
 10         int fd_P_C1[2];
 11         int fd_C1_C2[2];
 12         int fd_P_C2[2];
 13
 14         if (pipe(fd_P_C1)<0){
 15                 perror(argv[0]);
 16                 return 1;
 17         }
 18
 19         if (pipe(fd_P_C2)<0){
 20                 perror(argv[0]);
 21                 return 1;
 22         }
 23
 24         if (pipe(fd_C1_C2)<0){
 25                 perror(argv[0]);
 26                 return 1;
 27         }
 28         // IMAGE 1 PARENT heeft nu 3 pipes.
 29
 30         int pid[2];
 31         if ((pid[0]=fork()) < 0) {
 32                 perror(argv[0]);
 33                 exit(1);
 34         }
 35
 36         if (pid[0]==0){
 37                 //child 1
 38
 39                 close(fd_P_C2[0]);
 40                 close(fd_P_C2[1]);
 41
 42                 //Pipes write kant sluiten
 43                 close(fd_P_C1[1]);
 44                 close(fd_C1_C2[0]);
 45
 46                 read(fd_P_C1[0], &g, sizeof(int));
 47                 printf("CHILD 1 read %d from PARENT\n", g);
 48                 printf("CHILD 1 wrote %d to CHILD 2\n", g);
 49                 write(fd_C1_C2[1], &g, sizeof(int));
 50
 51                 return 0;
 52         }
 53
 54         if ((pid[1]=fork()) < 0) {
 55                 perror(argv[0]);
 56                 exit(1);
 57         }
 58
 59         if (pid[1]==0){
 60                 //child 2
 61
 62                 //pipes permanent sluitem
 63                 close(fd_P_C1[0]);
 64                 close(fd_P_C1[1]);
 65
 66                 //Pipes write kant sluiten
 67                 close(fd_P_C2[1]);
 68                 close(fd_C1_C2[1]);
 69
 70
 71                 read(fd_P_C2[0], &g, sizeof(int));
 72                 printf("CHILD 1 read %d from PARENT\n", g);
 73
 74                 int g2;
 75                 read(fd_C1_C2[0], &g2, sizeof(int));
 76                 printf("CHILD 1 read %d from CHILD 2\n", g2);
 77
 78                 printf("The sum is %d\n", g+g2);
 79                 return 0;
 80         }
 81         //PARENT
 82
 83         //Pipes permanent sluiten.
 84         close(fd_C1_C2[0]);
 85         close(fd_C1_C2[1]);
 86
 87
 88         //Pipes write kant sluiten
 89         close(fd_P_C1[0]);
 90         close(fd_P_C2[0]);
 91         g=7;
 92         write(fd_P_C1[1], &g, sizeof(int));
 93         g=658;
 94         write(fd_P_C2[1], &g, sizeof(int));
 95         waitpid(pid[0], NULL, 0);
 96         waitpid(pid[1], NULL, 0);
 97         return 0;
 98 }
 ```

### Exercise 6

Pas de gegeven code aan zodat het ouderproces het grootste van de gegeneerde getallen bepaalt en vervolgens ieder kindproces op de hoogte brengt wie de winnaar is, ttz. welk proces het grootste getal heeft gegenereerd. De uitvoer met zes kindprocessen ziet er als volgt uit:

```bash
Process 1819 is the winner
Im the winner!
Process 1819 is the winner
Process 1819 is the winner
Process 1819 is the winner
Process 1819 is the winner
```

!['alt text'](images/ex_6.jpg 'pipe')

template voor N childprocesses. Hier begint het met 6.

```c
 1 #include <unistd.h>
  2 #include <fcntl.h>
  3 #include <sys/types.h>
  4 #include <sys/wait.h>
  5 #include <stdio.h>
  6 #include <stdlib.h>
  7
  8 # define N 6
  9
 10 int main(int argc, char **argv) {
 11         int pid[N];
 12         int i;
 13
 14         for(i = 0; i < N; i++){
 15                 if ((pid[i]=fork())<0) {
 16                         perror(argv[0]);
 17                         exit(1);
 18                 }
 19
 20                 if (pid[i]==0) {
 21                         // child i
 22
 23                         exit(0);
 24                 }
 25         }
 26
 27         //PARENT
 28
 29         for (i=0;i<N;i++) {
 30                 waitpid(pid[i], NULL, 0);
 31         }
 32
 33         return 0;
 34 }
```

Sequentiële uitvoer en de kindprocess sturen naar de parent, de parent kan nog niet terugsturen:

!['alt text'](images/ex_6_uni_directional.jpg 'pipe')

```c
  1 #include <unistd.h>
  2 #include <fcntl.h>
  3 #include <sys/types.h>
  4 #include <sys/wait.h>
  5 #include <stdio.h>
  6 #include <stdlib.h>
  7
  8 # define N 6
  9
 10 int main(int argc, char **argv) {
 11         int pid[N];
 12         int fd[N][2];
 13         int i;
 14
 15         for(i = 0; i < N; i++){
 16                 //PIPE VOOR FORK
 17                 if(pipe(fd[i])<0) {
 18                         perror(argv[0]);
 19                         exit(1);
 20                 }
 21
 22                 if ((pid[i]=fork())<0) {
 23                         perror(argv[0]);
 24                         exit(1);
 25                 }
 26
 27                 if (pid[i]==0) {
 28                         // child i
 29                         //RANDOM NUMBER
 30
 31                         //Hier niet bruikbaar, elk kindproces gaat dezelfde reeks hebben.
 32                         //srand(time(0));
 33
 34                         //Hierdoor gaat elk kindprocess een andere reeks hebben.
 35                         srand(getpid());
 36
 37                         close(fd[i][0]);
 38                         unsigned int g=rand();
 39                         printf("child %d generated %d\n", i , rand());
 40                         write(fd[i][1], &g, sizeof(unsigned int));
 41
 42                         exit(0);
 43                 }
 44                 close(fd[i][1]);
 45         }
 46
 47         //PARENT
 48
 49         for (i = 0; i<N;i++) {
 50                 unsigned int n;
 51                 read(fd[i][0], &n, sizeof(n));
 52                 printf("Read %u from child %d\n", n, i);
 53         }
 54         for (i = 0; i<N;i++) {
 55                 waitpid(pid[i], NULL, 0);
 56         }
 57
 58         for (i=0;i<N;i++) {
 59                 waitpid(pid[i], NULL, 0);
 60         }
 61
 62         return 0;
 63 }
```

Achterhaald nu het hoogste getal.

```c
  1 #include <unistd.h>
  2 #include <fcntl.h>
  3 #include <sys/types.h>
  4 #include <sys/wait.h>
  5 #include <stdio.h>
  6 #include <stdlib.h>
  7
  8 # define N 6
  9
 10 typedef struct info{
 11         int pid;
 12         int number;
 13 } info;
 14
 15
 16 int main(int argc, char **argv) {
 17         int pid[N];
 18         int fd[N][2];
 19         int i;
 20
 21    for(i = 0; i < N; i++){
 22                 //PIPE VOOR FORK
 23                 if(pipe(fd[i])<0) {
 24                         perror(argv[0]);
 25                         exit(1);
 26                 }
 27
 28                 if ((pid[i]=fork())<0) {
 29                         perror(argv[0]);
 30                         exit(1);
 31                 }
 32
 33                 if (pid[i]==0) {
 34                         // child i
 35                         //RANDOM NUMBER
 36
 37                         //Hier niet bruikbaar, elk kindproces gaat dezelfde reeks hebben.
 38                         //srand(time(0));
 39
 40                         //Hierdoor gaat elk kindprocess een andere reeks hebben.
 41                         srand(getpid());
 42
 43                         close(fd[i][0]);
 44                         unsigned int g=rand();
 45                         printf("child %d generated %d\n", i , rand());
 46                         write(fd[i][1], &g, sizeof(unsigned int));
 47
 48                         exit(0);
 49                 }
 50                 //PARENT
 51                 close(fd[i][1]);
 52         }
 53
 54         int index = 0;
 55         unsigned int n;
 56         read(fd[0][0], &n, sizeof(unsigned int));
 57
 58         for (i = 1; i<N;i++) {
 59                 unsigned int tmp;
 60                 read(fd[i][0], &tmp, sizeof(unsigned int));
 61                 if (n>tmp){
 62                         index=i;
 63                         n=tmp;
 64                 }
 65                 printf("Read %u from child %d\n", tmp, i);
 66         }
 67
 68         printf("%d created highest value: %d\n", index, n);
 69
 70
 71         for (i=0;i<N;i++) {
 72                 waitpid(pid[i], NULL, 0);
 73         }
 74
 75         return 0;
 76 }
```

Synchronisatie word hier geforceerd met reads en writes, De struct word hier niet gebruikt:

```c
  1 #include <unistd.h>
  2 #include <fcntl.h>
  3 #include <sys/types.h>
  4 #include <sys/wait.h>
  5 #include <stdio.h>
  6 #include <stdlib.h>
  7
  8 # define N 6
  9
 10 typedef struct info{
 11         int pid;
 12         int number;
 13 } info;
 14
 15
 16 int main(int argc, char **argv) {
 17         int pid[N];
 18         int fd[N][2];
 19         int fd2[N][2];
 20         int i;
 21
 22         for(i = 0; i < N; i++){
 23                 //PIPE VOOR FORK
 24                 if(pipe(fd[i])<0) {
 25                         perror(argv[0]);
 26                         exit(1);
 27                 }
 28
 29                 if (pipe(fd2[i])<0){
 30                         perror(argv[0]);
 31                         exit(1);
 32                 }
 33
 34                 if ((pid[i]=fork())<0) {
 35                         perror(argv[0]);
 36                         exit(1);
 37                 }
 38
 39                 if (pid[i]==0) {
 40                         // child i
 41                         //RANDOM NUMBER
 42
 43                         //Hier niet bruikbaar, elk kindproces gaat dezelfde reeks hebben.
 44                         //srand(time(0));
 45
 46                         //Hierdoor gaat elk kindprocess een andere reeks hebben.
 47                         srand(getpid());
 48
 49                         close(fd[i][0]);
 50                         close(fd[i][1]);
 51
 52                         unsigned int g=rand();
 53                         printf("child %d generated %d\n", i , rand());
 54                         write(fd[i][1], &g, sizeof(unsigned int));
 55
 56                         int pid_winner;
 57                         read(fd2[i][0],&pid_winner, sizeof(int));58
 59                         if (pid_winner == getpid()) {
 60                                 printf("I am the winner\n");
 61                         } else {
 62                                 printf("Process %d is the winner\n", pid_winner);
 63                         }
 64
 65                         exit(0);
 66                 }
 67                 //PARENT
 68                 close(fd[i][1]);
 69                 close(fd2[i][0]);
 70         }
 71
 72         int index = 0;
 73         unsigned int n;
 74         read(fd[0][0], &n, sizeof(unsigned int));
 75
 76         for (i = 1; i<N;i++) {
 77                 unsigned int tmp;
 78                 read(fd[i][0], &tmp, sizeof(unsigned int));
 79                 if (n>tmp){
 80                         index=i;
 81                         n=tmp;
 82                 }
 83                 printf("Read %u from child %d\n", tmp, i);
 84         }
 85
 86         printf("%d created highest value: %d\n", index, n);
 87
 88         for (i=0;i<N;i++){
 89                 write(fd2[i][1], &pid[index], sizeof(pid[index]));
 90
 91         }
 92
 93
 94         for (i=0;i<N;i++) {
 95                 waitpid(pid[i], NULL, 0);
 96         }
 97
 98         return 0;
 99 }
```

## threads

```bash
man pthread_create
```

Om te compilen.

```bash
gcc -pthread <file>.c
```

```c
  1 #include <pthread.h>
  2 #include <stdio.h>
  3
  4 #define N 4
  5
  6 void *worker(void *a) {
  7         int *n=(int*)a;
  8         int i;
  9         for (i=0;i<100;i++){
 10                 printf("%d\n", *n);
 11         }
 12         return ;
 13 }
 14
 15 int main(int arc, char** argv) {
 16         srand(time(0));
 17         pthread_t threads[N];
 18         int numbers[N]={1, 2, 3, 4};
 19         int i;
 20         for (i=0;i<N;i++){
 21                 pthread_create(&threads[i],NULL, worker, (void*)&numbers[i]);
 22         }
 23
 24         for (i=0;i<N;i++){
 25                 pthread_join(threads[i],NULL);
 26         }
 27         return 0;
 28 }
```