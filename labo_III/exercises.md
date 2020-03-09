# labo

## multi threading

template voor multi-threading. Dit template gaat ervan uit dat de parallell uit te voeren code allemaal even lang duren. Hier wordt er geen 8 child processen aangemaakt maar 3.

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <stdlib.h>
#include <fcntl.h>
#include <unistd.h>

#define N 17

int main(int argc, char **argv) {
        int i;
        int pid[N];
        for (i = 0; i < N; i++) {
                if ((pid[i] = fork())<0){
                        perror(argv[0]);
                        return 1;
                } else if (pid[i] == 0) {
                        // parallell uit te voeren code.
                }

        }
        for (i = 0; i < 3; i++){
                waitpid(pid[i], NULL, 0);
        }
        return 0;
}
```

## ex 3.1

* argv[0] is de benaming die word gebruikt voor het proces, dus het is belangrijk dat het een duidelijk naam is.

variant met execve:

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <stdlib.h>
#include <fcntl.h>
#include <unistd.h>

#define N 17

int main(int argc, char **argv) {
        int i;
        int pid[N];
        for (i = 0; i < N; i++) {
                if ((pid[i] = fork())<0){
                        perror(argv[0]);
                        return 1;
                } else if (pid[i] == 0) {
                        //execlp("writestring", "hello");
                        char * args[3] = {"writestring", "hello", 0};
                        char * env[1] = {NULL};
                        if (execve("/root/Documents/Best_II/labo_III/writestring",args, env ) < 0) {
                                perror(argv[0]);
                                return 1;
                        }
                        /*
 * Execve schrijft de code van het kindproces uit dus de code eronder wordt hier niet
 * meer uitgevoerd. Dus er kan na de execve geen gebruik meer worden gemaakt van malloc
 */
                }

        }
        for (i = 0; i < 3; i++){
                waitpid(pid[i], NULL, 0);
        }
        printf("Im the parent PID=%d", getpid());
        waitpid(-1, NULL, 0);
        return 0;
}
```

## ex 4

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <sys/stat.h>
#include <stdlib.h>
#include <fcntl.h>

int main(int argc, char **argv) {
        if (argc != 0 )
                return 1;

        int fd;

        if((fd = open(argv[1], O_RDONLY))<0) {
                perror(argv[1]);
                return 1;
        }

        struct stat sb;

        if (fstat(fd, &sb)<0) {
                perror(argv[0]);
                return 1;
        }

        if (!S_ISREG(sb.st_mode)){
                fprintf(stderr, "%s %s\n", argv[1], "is not a regular file");
                exit(1);
        }
        orig = sb.st_mtime;
        while(1) {
                fstat(fd, &sb);
                if (orig!=sb.st_mtime) {
                        printf("FILE CHANGED\n");
                        orig= sb.st_mtime;
                }

        }

        close(fd);
        return 0;
}
```
