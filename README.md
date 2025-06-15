Mikołaj Sarna 
### Zadania terminalowe (50% punktów)

Zapisz komendy dla poszczególnych operacji z wiersza poleceń. Dołącz do rozwiązania utworzone pliki.

1.cd ~
2.mkdir raporty
3.chmod 700 raporty
4.cd raporty
5.touch statystyki.csv
6.echo "miasto,populacja,rok" > statystyki.csv
7.
echo "miasto,populacja,rok" > statystyki.csv
echo "Warszawa,1M,2024" >> statystyki.csv
echo "Rzeszow,500k,2023" >> statystyki.csv
echo "Gdańsk,452k,2024" >> statystyki.csv
echo "Wrocław,629k,2004" >> statystyki.csv
8.cat statystyki.csv
9.touch .archiwum_statystyk.csv
10.ls -la
13.tail -n +2 statystyki.csv | sort | nl > .archiwum_statystyk2.csv
14.diff .archiwum_statystyk.csv .archiwum_statystyk2.csv
15.sed 's/,/ /g' .archiwum_statystyk.csv > .archiwum_statystyk.csv.tmp && mv .archiwum_statystyk.csv.tmp .archiwum_statystyk.csv
16.sed -n '/,[^,]*8[^,]*,/p' statystyki.csv
17.df -h | awk 'NR==1{print $6, $3, $4} NR>1{used+=$3; avail+=$4; print $6, $3, $4} END{print "SUM", used, avail}'


---

### Projekt – Make, IPC, sygnały, GIT (max. 50%)

1.
mkdir projekt_ipc
cd projekt_ipc
nano agent.h

#ifndef AGENT_H
#define AGENT_H

void agent(const char* name, int read_fd, int write_fd);

#endif
2.
nano agent.c
CC = gcc
CFLAGS = -Wall -std=c99 -g

coordinator.o: coordinator.c coordinator.h agent.h
	$(CC) $(CFLAGS) -c coordinator.c

agent.o: agent.c agent.h
	$(CC) $(CFLAGS) -c agent.c

main: main.c coordinator.o agent.o
	$(CC) $(CFLAGS) -o koordynator main.c coordinator.o agent.o

all: coordinator.o agent.o main

clean:
	rm -f *.o koordynator

4.
nano coordinator.h
#ifndef COORDINATOR_H
#define COORDINATOR_H

void coordinator();

#endif

nano coordinator.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>

void create_agent(const char* name) {
    int pipefd[2];
    if (pipe(pipefd) == -1) {
        perror("pipe");
        exit(1);
    }

    pid_t pid = fork();
    if (pid == 0) {
        close(pipefd[1]);
        dup2(pipefd[0], STDIN_FILENO);
        close(pipefd[0]);

        execl("./agent", "agent", name, NULL);
        perror("execl");
        exit(1);
    } else if (pid > 0) {
        close(pipefd[0]);
    } else {
        perror("fork");
        exit(1);
    }
}
nano agent.c
#include <string.h>
#include <unistd.h>
#include "coordinator.h"
#include <stdio.h>

int main(int argc, char* argv[]) {
    if (argc < 2) {
        fprintf(stderr, "Agent: brak imienia\n");
        return 1;
    }
    char* name = argv[1];

    printf("%s: Gotowy do misji\n", name);
    fflush(stdout);

    char buf[100];
    while (fgets(buf, sizeof(buf), stdin) != NULL) {
        if (strncmp(buf, "Identify!", 9) == 0) {
            printf("My codename is %s\n", name);
            fflush(stdout);
        } else if (strncmp(buf, "Status?", 7) == 0) {
            printf("%s: My PID is: %d\n", name, getpid());
            fflush(stdout);
        }
    }
    return 0;
}
nano main.c
#include <stdio.h>

void create_agent(const char* name); 

int main() {
    create_agent("Gamma");
    create_agent("Beta");
    create_agent("Alpha");
    create_agent("Delta");
    getchar(); 
    return 0;
}
nano coordinator.h
#ifndef COORDINATOR_H
#define COORDINATOR_H

void create_agent(const char* name);

#endif

gcc -o agent agent.c
gcc -c coordinator.c
gcc -o koordynator main.c coordinator.o
./koordynator
5.
cd projekt_ipc
